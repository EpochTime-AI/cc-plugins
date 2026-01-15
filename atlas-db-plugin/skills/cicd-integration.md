---
name: cicd-integration
description: Integrate Atlas into GitHub Actions, GitLab CI, and other CI/CD platforms for automated database migrations
---

# Atlas CI/CD Integration

Automate database migrations in your CI/CD pipelines using Atlas with GitHub Actions, GitLab CI, and other platforms.

## GitHub Actions Integration

### Setup Atlas Action

```yaml
name: Database Migration

on:
  push:
    branches: [main]
    paths:
      - 'schema/**'
      - 'migrations/**'
      - 'atlas.hcl'

jobs:
  migrate:
    runs-on: ubuntu-latest

    steps:
      # Setup Atlas
      - uses: ariga/setup-atlas@v0
        with:
          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}

      # Checkout code
      - uses: actions/checkout@v4

      # Run migrations
      - name: Apply migrations
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          atlas migrate apply \
            --url "$DATABASE_URL" \
            --dir file://migrations
```

### Plan & Review Workflow

```yaml
name: Schema Migration Plan

on:
  pull_request:
    paths:
      - 'schema/**'
      - 'atlas.hcl'

jobs:
  plan:
    runs-on: ubuntu-latest

    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4

      - name: Plan migrations
        env:
          DATABASE_URL: ${{ secrets.STAGING_DB_URL }}
        run: |
          atlas migrate diff plan_${{ github.run_id }} \
            --env staging

      - name: Comment PR with migration plan
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('plan.sql', 'utf8');

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '## Migration Plan\n```sql\n' + plan + '\n```'
            });
```

### Declarative Migration Pipeline

```yaml
name: Declarative Migration

on:
  push:
    branches: [main]

env:
  ATLAS_CLOUD_TOKEN: ${{ secrets.ATLAS_CLOUD_TOKEN }}

jobs:
  migrate-dev:
    runs-on: ubuntu-latest
    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4

      - name: Migrate dev database
        run: |
          atlas migrate apply \
            --env dev \
            --url "mysql://root:password@localhost/dev_db"

  migrate-prod:
    runs-on: ubuntu-latest
    needs: migrate-dev
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4

      - name: Migrate production database
        env:
          PROD_DB_URL: ${{ secrets.PROD_DATABASE_URL }}
        run: |
          atlas migrate apply \
            --url "$PROD_DB_URL" \
            --dir file://migrations
```

### Linting Migrations in CI

```yaml
name: Migration Lint

on:
  pull_request:
    paths:
      - 'migrations/**'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4

      - name: Lint migrations
        run: |
          atlas migrate lint \
            --env local \
            --latest 10

      - name: Report results
        if: failure()
        run: |
          echo "Migration lint failed. Review the logs above."
          exit 1
```

## GitLab CI Integration

### Basic Pipeline

```yaml
stages:
  - plan
  - apply

variables:
  ATLAS_VERSION: latest

before_script:
  - curl -sSf https://atlasgo.sh | sh

plan:migration:
  stage: plan
  only:
    - merge_requests
  script:
    - atlas migrate diff my_migration --env staging
  artifacts:
    paths:
      - migrations/
    expire_in: 1 hour

apply:migration:
  stage: apply
  only:
    - main
  script:
    - atlas migrate apply --url $PROD_DATABASE_URL --dir file://migrations
  environment:
    name: production
```

### Multi-Environment Pipeline

```yaml
stages:
  - lint
  - plan
  - apply-dev
  - apply-prod

variables:
  ATLAS_URL_DEV: $CI_ENVIRONMENT_SLUG-db

lint:migrations:
  stage: lint
  script:
    - atlas migrate lint --env local --latest 5

plan:staging:
  stage: plan
  environment:
    name: staging
    action: prepare
  script:
    - atlas migrate diff staging_$CI_COMMIT_SHA --env staging
  artifacts:
    paths:
      - migrations/

apply:dev:
  stage: apply-dev
  environment:
    name: development
  script:
    - atlas migrate apply --env dev

apply:prod:
  stage: apply-prod
  when: manual
  environment:
    name: production
  script:
    - atlas migrate apply --env prod
```

## Docker Integration

```dockerfile
FROM arigaio/atlas:latest

WORKDIR /workspace

COPY atlas.hcl .
COPY migrations/ ./migrations/
COPY schema.hcl .

# Run migrations
CMD ["migrate", "apply", "--url", "mysql://user:pass@db/mydb"]
```

Use in docker-compose:

```yaml
version: '3'

services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: mydb

  migrate:
    image: atlas-migrations:latest
    depends_on:
      - db
    environment:
      DATABASE_URL: "mysql://root:password@db/mydb"
```

## Environment Variables

### Secure credential passing in CI/CD:

```yaml
# GitHub Actions
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  ATLAS_CLOUD_TOKEN: ${{ secrets.ATLAS_CLOUD_TOKEN }}

# GitLab CI
variables:
  DATABASE_URL: $PROD_DB_URL
  ATLAS_TOKEN: $CI_JOB_TOKEN
```

### atlas.hcl with environment variables:

```hcl
env "prod" {
  url = getenv("DATABASE_URL")

  migration {
    dir = "file://migrations"
    auto_approve = false
  }
}
```

## Approval & Safety Patterns

### Manual approval before production

```yaml
apply:prod:
  stage: deploy
  when: manual
  environment:
    name: production
  script:
    - atlas migrate apply --env prod
```

### Review migration before applying

```yaml
review:migration:
  stage: plan
  script:
    - atlas migrate diff preview --env staging
    - cat migrations/preview.sql
  artifacts:
    paths:
      - migrations/preview.sql
```

### Dry-run validation

```yaml
validate:migration:
  stage: validate
  script:
    - atlas migrate apply --env staging --dry-run
```

## Common Patterns

### Continuous Deployment

```yaml
# On every push to main, apply migrations
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4
      - run: |
          atlas migrate apply \
            --url ${{ secrets.PROD_DB_URL }}
```

### Feature Branch Testing

```yaml
# On pull request, plan & show migrations
on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: ariga/setup-atlas@v0
      - uses: actions/checkout@v4
      - run: |
          atlas migrate diff \
            --url "mysql://user:pass@localhost/test_db" \
            --dir file://migrations
```

### Multi-Database Deployment

```yaml
deploy:
  strategy:
    matrix:
      database: [mysql, postgres, sqlite]
  steps:
    - run: |
        atlas migrate apply \
          --env ${{ matrix.database }}
```

## Troubleshooting CI/CD Issues

### "Database connection refused"
- Verify `DATABASE_URL` secret is set
- Check database is accessible from CI runners
- Use `--dry-run` to test without connecting

### "Migration already applied"
```bash
# Check migration status
atlas migrate status --url $DATABASE_URL

# Set to specific version if needed
atlas migrate set --url $DATABASE_URL 20240115120000
```

### "Timeout on large database"
```yaml
# Increase timeout for slow databases
script:
  - atlas migrate apply --url $DATABASE_URL --timeout 10m
```

## Best Practices

1. **Test migrations locally first** - Run in development before CI
2. **Use dry-run in CI** - Validate before applying to production
3. **Require manual approval for production** - Use `when: manual`
4. **Version control all migrations** - Commit migration files
5. **Secure credentials** - Use CI/CD secrets, never hardcode
6. **Monitor migrations** - Log and alert on migration failures
7. **Maintain rollback strategy** - Keep down migrations up to date
8. **Separate dev/staging/prod** - Different configurations per environment

## Resources

- **GitHub Actions Setup**: https://github.com/marketplace/actions/setup-atlas
- **GitLab CI Guide**: https://atlasgo.io/guides/ci-platforms/gitlab-declarative
- **GitHub Actions Guide**: https://atlasgo.io/guides/ci-platforms/github-declarative

## Local References

For complete CI/CD documentation, see:
- `references/atlas-docs-full/guides/ci-platforms/github-declarative.md` - GitHub Actions declarative
- `references/atlas-docs-full/guides/ci-platforms/github-versioned.md` - GitHub Actions versioned
- `references/atlas-docs-full/guides/ci-platforms/gitlab-declarative.md` - GitLab CI declarative
- `references/atlas-docs-full/guides/ci-platforms/gitlab-versioned.md` - GitLab CI versioned
- `references/atlas-docs-full/guides/ci-platforms/` - All CI platform guides (Azure, Bitbucket, CircleCI)
- `references/atlas-docs-full/guides/deploying/` - Complete deployment guides
- `references/README.md` - Full documentation index
