# Atlas Documentation References

This directory contains the complete original Atlas documentation crawled from https://atlasgo.io.

## Structure

```
references/
├── README.md                    # This file - index of all documentation
├── original-documentation.md    # Documentation sources and mapping
└── atlas-docs-full/             # 103+ pages of crawled documentation (1.4MB)
    ├── docs.md
    ├── getting-started.md
    ├── guides.md
    └── guides/
        ├── ai-tools/
        ├── ci-platforms/
        ├── database-per-tenant/
        ├── deploying/
        ├── migration-dirs/
        ├── migration-tools/
        ├── mysql/
        ├── orms/
        ├── terraform/
        └── testing/
```

## Quick Index

### Core Concepts
- `atlas-docs-full/docs.md` - Main Atlas documentation
- `atlas-docs-full/getting-started.md` - Installation and quick start

### Schema Definition
- `atlas-docs-full/docs.md` - Schema-as-code overview (HCL, SQL, ORM)

### Migration Workflows
- Declarative migrations documentation throughout guides
- Versioned migrations documentation throughout guides
- CLI reference in main docs

### CI/CD Integration
- `atlas-docs-full/guides/ci-platforms/github-declarative.md`
- `atlas-docs-full/guides/ci-platforms/github-versioned.md`
- `atlas-docs-full/guides/ci-platforms/gitlab-declarative.md`
- `atlas-docs-full/guides/ci-platforms/gitlab-versioned.md`
- `atlas-docs-full/guides/ci-platforms/bitbucket-*.md`
- `atlas-docs-full/guides/ci-platforms/circleci-*.md`
- `atlas-docs-full/guides/ci-platforms/azure-devops-*.md`

### Deployment Guides
- `atlas-docs-full/guides/deploying/intro.md`
- `atlas-docs-full/guides/deploying/aws-ecs-fargate.md`
- `atlas-docs-full/guides/deploying/fly-io.md`
- `atlas-docs-full/guides/deploying/helm.md`
- `atlas-docs-full/guides/deploying/k8s-*.md`
- `atlas-docs-full/guides/deploying/cloud-sql-via-github-actions.md`
- `atlas-docs-full/guides/deploying/secrets.md`

### ORM Integration
- `atlas-docs-full/guides/orms/gorm.md` + `guides/orms/gorm/*`
- `atlas-docs-full/guides/orms/prisma.md`
- `atlas-docs-full/guides/orms/sequelize.md` + `guides/orms/sequelize/*`
- `atlas-docs-full/guides/orms/sqlalchemy.md`
- `atlas-docs-full/guides/orms/typeorm.md` + `guides/orms/typeorm/*`
- `atlas-docs-full/guides/orms/doctrine.md`
- `atlas-docs-full/guides/orms/django.md`
- `atlas-docs-full/guides/orms/drizzle.md`
- `atlas-docs-full/guides/orms/beego.md`

### Testing
- `atlas-docs-full/guides/testing/docker-compose.md`
- `atlas-docs-full/guides/testing/github-actions.md`
- `atlas-docs-full/guides/testing/data-migrations.md`
- `atlas-docs-full/guides/testing/views.md`
- `atlas-docs-full/guides/testing/functions.md`
- `atlas-docs-full/guides/testing/procedures.md`
- `atlas-docs-full/guides/testing/triggers.md`
- `atlas-docs-full/guides/testing/domains.md`

### Migration Tools
- `atlas-docs-full/guides/migration-tools/golang-migrate.md`
- `atlas-docs-full/guides/migration-tools/goose-import.md`
- `atlas-docs-full/guides/atlas-vs-flyway.md`
- `atlas-docs-full/guides/atlas-vs-liquibase.md`
- `atlas-docs-full/guides/atlas-vs-ssdt.md`
- `atlas-docs-full/guides/migrate-flyway-to-atlas.md`

### Advanced Topics
- `atlas-docs-full/guides/database-per-tenant/` - Multi-tenant patterns
- `atlas-docs-full/guides/terraform/` - Terraform integration
- `atlas-docs-full/guides/migration-dirs/template-directory.md`
- `atlas-docs-full/guides/reviewed-approved-migrations.md`
- `atlas-docs-full/guides/environment-promotion.md`
- `atlas-docs-full/guides/go-templates.md`

## Usage

These files serve as complete reference material for the Atlas skills. When skills provide condensed practical guidance, these files contain:

- Complete syntax references
- All configuration options
- Edge cases and advanced features
- Detailed troubleshooting steps
- Database-specific implementation details

## File Count

Total documentation files: 103+ markdown files covering 1.4MB of content

## Source

All documentation crawled from https://atlasgo.io using the inform tool on 2026-01-15.

Atlas v1.0 documentation included.
