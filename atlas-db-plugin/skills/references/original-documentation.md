# Atlas Documentation Source Mapping

This file documents the relationship between plugin skills and original Atlas documentation files.

## Documentation Source

All content in atlas-db-plugin is derived from official Atlas documentation:
- **Source Website**: https://atlasgo.io
- **Crawl Date**: 2026-01-15
- **Version**: Atlas v1.0
- **Total Files**: 103+ markdown files (1.4MB)

## Skill-to-Documentation Mapping

### 1. atlas-concepts.md
**Purpose**: Core concepts, workflows overview, installation

**Primary Sources**:
- `atlas-docs-full/docs.md` - Main documentation hub
- `atlas-docs-full/getting-started.md` - Installation and quick start

**Covered Topics**:
- Declarative vs Versioned workflows
- Schema-as-code approaches (HCL, SQL, ORM)
- Atlas installation methods
- Core concepts overview

### 2. schema-definition.md
**Purpose**: Writing database schemas in HCL, SQL, and ORM formats

**Primary Sources**:
- `atlas-docs-full/docs.md` - Schema-as-code section
- ORM documentation throughout `atlas-docs-full/guides/orms/`

**Covered Topics**:
- HCL schema syntax and examples
- SQL DDL schema definition
- ORM integration overview
- Data types, relationships, constraints
- Index patterns and best practices

### 3. migration-workflows.md
**Purpose**: Planning, applying, linting migrations

**Primary Sources**:
- `atlas-docs-full/docs.md` - Migration sections
- `atlas-docs-full/guides/modern-database-ci-cd.md` - Modern CI/CD patterns
- `atlas-docs-full/guides/migration-dirs/template-directory.md` - Migration templates

**Related Files**:
- `atlas-docs-full/guides/reviewed-approved-migrations.md`
- `atlas-docs-full/guides/environment-promotion.md`
- `atlas-docs-full/guides/go-templates.md`

**Covered Topics**:
- Declarative migration workflow
- Versioned migration workflow
- Atlas CLI commands (diff, apply, lint, status)
- Multi-environment setup
- Rollback strategies

### 4. cicd-integration.md
**Purpose**: Integrating Atlas into CI/CD pipelines

**Primary Sources**:
- `atlas-docs-full/guides/ci-platforms/github-declarative.md`
- `atlas-docs-full/guides/ci-platforms/github-versioned.md`
- `atlas-docs-full/guides/ci-platforms/gitlab-declarative.md`
- `atlas-docs-full/guides/ci-platforms/gitlab-versioned.md`
- `atlas-docs-full/guides/ci-platforms/bitbucket-declarative.md`
- `atlas-docs-full/guides/ci-platforms/bitbucket-versioned.md`
- `atlas-docs-full/guides/ci-platforms/circleci-declarative.md`
- `atlas-docs-full/guides/ci-platforms/circleci-versioned.md`
- `atlas-docs-full/guides/ci-platforms/azure-devops-github.md`
- `atlas-docs-full/guides/ci-platforms/azure-devops-repos.md`

**Deployment Guides**:
- `atlas-docs-full/guides/deploying/intro.md`
- `atlas-docs-full/guides/deploying/aws-ecs-fargate.md`
- `atlas-docs-full/guides/deploying/fly-io.md`
- `atlas-docs-full/guides/deploying/helm.md`
- `atlas-docs-full/guides/deploying/k8s-argo.md`
- `atlas-docs-full/guides/deploying/k8s-argo-declarative.md`
- `atlas-docs-full/guides/deploying/k8s-flux.md`
- `atlas-docs-full/guides/deploying/k8s-init-container.md`
- `atlas-docs-full/guides/deploying/k8s-cloud-versioned.md`
- `atlas-docs-full/guides/deploying/k8s-operator-certs.md`
- `atlas-docs-full/guides/deploying/cloud-sql-via-github-actions.md`
- `atlas-docs-full/guides/deploying/connect-to-db-from-github-actions.md`
- `atlas-docs-full/guides/deploying/secrets.md`
- `atlas-docs-full/guides/deploying/secrets_secret-store_*.md` (AWS, GCP, HashiVault)
- `atlas-docs-full/guides/deploying/image.md`
- `atlas-docs-full/guides/deploying/remote-directories.md`

**Covered Topics**:
- GitHub Actions workflows
- GitLab CI pipelines
- Docker integration
- Kubernetes deployment patterns
- Multi-environment deployment
- Approval and safety patterns

### 5. orm-integration.md
**Purpose**: Integrating Atlas with popular ORMs

**Primary Sources**:

**GORM (Go)**:
- `atlas-docs-full/guides/orms/gorm.md`
- `atlas-docs-full/guides/orms/gorm/getting-started.md`
- `atlas-docs-full/guides/orms/gorm/standalone.md`
- `atlas-docs-full/guides/orms/gorm/program.md`
- `atlas-docs-full/guides/orms/gorm/generate-models.md`
- `atlas-docs-full/guides/orms/gorm/composite-types.md`
- `atlas-docs-full/guides/orms/gorm/domain-types.md`
- `atlas-docs-full/guides/orms/gorm/enum-types.md`
- `atlas-docs-full/guides/orms/gorm/extensions.md`
- `atlas-docs-full/guides/orms/gorm/row-level-security.md`
- `atlas-docs-full/guides/orms/gorm/triggers.md`
- `atlas-docs-full/guides/orms/gorm/views.md`
- `atlas-docs-full/guides/orms/gorm/visualize.md`

**Prisma (JavaScript/TypeScript)**:
- `atlas-docs-full/guides/orms/prisma.md`

**Sequelize (JavaScript/TypeScript)**:
- `atlas-docs-full/guides/orms/sequelize.md`
- `atlas-docs-full/guides/orms/sequelize/getting-started.md`
- `atlas-docs-full/guides/orms/sequelize/standalone.md`
- `atlas-docs-full/guides/orms/sequelize/script.md`
- `atlas-docs-full/guides/orms/sequelize/generate-models.md`
- `atlas-docs-full/guides/orms/sequelize/composite-types.md`
- `atlas-docs-full/guides/orms/sequelize/domain-types.md`
- `atlas-docs-full/guides/orms/sequelize/row-level-security.md`
- `atlas-docs-full/guides/orms/sequelize/triggers.md`
- `atlas-docs-full/guides/orms/sequelize/views.md`
- `atlas-docs-full/guides/orms/sequelize/visualize.md`
- `atlas-docs-full/guides/orms/sequelize/declarative-migrations-with-actions.md`

**TypeORM (TypeScript)**:
- `atlas-docs-full/guides/orms/typeorm.md`
- `atlas-docs-full/guides/orms/typeorm/generate-entities.md`

**SQLAlchemy (Python)**:
- `atlas-docs-full/guides/orms/sqlalchemy.md`

**Django (Python)**:
- `atlas-docs-full/guides/orms/django.md`

**Doctrine (PHP)**:
- `atlas-docs-full/guides/orms/doctrine.md`

**Others**:
- `atlas-docs-full/guides/orms/drizzle.md`
- `atlas-docs-full/guides/orms/drizzle/triggers.md`
- `atlas-docs-full/guides/orms/beego.md`

**Covered Topics**:
- Model definitions for each ORM
- Atlas configuration for ORM integration
- Migration generation from ORM models
- Best practices per ORM
- Common workflow patterns

## Additional Documentation

### Testing
- `atlas-docs-full/guides/testing/docker-compose.md`
- `atlas-docs-full/guides/testing/github-actions.md`
- `atlas-docs-full/guides/testing/data-migrations.md`
- `atlas-docs-full/guides/testing/views.md`
- `atlas-docs-full/guides/testing/functions.md`
- `atlas-docs-full/guides/testing/procedures.md`
- `atlas-docs-full/guides/testing/triggers.md`
- `atlas-docs-full/guides/testing/domains.md`

### Migration Tool Comparisons
- `atlas-docs-full/guides/atlas-vs-flyway.md`
- `atlas-docs-full/guides/atlas-vs-liquibase.md`
- `atlas-docs-full/guides/atlas-vs-ssdt.md`
- `atlas-docs-full/guides/migrate-flyway-to-atlas.md`
- `atlas-docs-full/guides/flyway-snapshot-alternative.md`
- `atlas-docs-full/guides/flyway-undo-alternative.md`
- `atlas-docs-full/guides/liquibase-diff-alternative.md`
- `atlas-docs-full/guides/liquibase-rollback-alternative.md`

### Migration Tool Integration
- `atlas-docs-full/guides/migration-tools/golang-migrate.md`
- `atlas-docs-full/guides/migration-tools/goose-import.md`

### Multi-Tenant Patterns
- `atlas-docs-full/guides/database-per-tenant/intro.md`
- `atlas-docs-full/guides/database-per-tenant/target-groups.md`
- `atlas-docs-full/guides/database-per-tenant/deploying.md`
- `atlas-docs-full/guides/database-per-tenant/rollout.md`
- `atlas-docs-full/guides/database-per-tenant/control-plane.md`

### Terraform Integration
- `atlas-docs-full/guides/terraform/named-databases.md`
- `atlas-docs-full/guides/terraform/opentaco.md`
- `atlas-docs-full/guides/mysql/terraform.md`

### AI Tools
- `atlas-docs-full/guides/ai-tools.md`
- `atlas-docs-full/guides/ai-tools/agent-skills.md`
- `atlas-docs-full/guides/ai-tools/claude-code-instructions.md`
- `atlas-docs-full/guides/ai-tools/cursor-rules.md`
- `atlas-docs-full/guides/ai-tools/github-copilot-instructions.md`

### Miscellaneous
- `atlas-docs-full/guides/atlas-in-docker.md`
- `atlas-docs-full/guides.md` - Guides overview

## Documentation Structure

```
references/
├── README.md                          # Quick index
├── original-documentation.md          # This file - source mapping
└── atlas-docs-full/                   # Complete Atlas documentation
    ├── docs.md                        # Main documentation
    ├── getting-started.md             # Quick start
    ├── guides.md                      # Guides overview
    └── guides/
        ├── ai-tools/                  # AI tool integrations
        ├── ci-platforms/              # CI/CD platform guides
        ├── database-per-tenant/       # Multi-tenant patterns
        ├── deploying/                 # Deployment strategies
        ├── migration-dirs/            # Migration templates
        ├── migration-tools/           # Tool integrations
        ├── mysql/                     # MySQL-specific
        ├── orms/                      # ORM integrations
        ├── terraform/                 # Terraform patterns
        └── testing/                   # Testing guides
```

## Usage Guidelines

When skills provide condensed guidance, consult these original files for:
- Complete syntax references
- All configuration options
- Edge cases and advanced features
- Database-specific implementations
- Detailed troubleshooting
- Migration from other tools

## Official Resources

- **Website**: https://atlasgo.io
- **GitHub**: https://github.com/ariga/atlas
- **Cloud Platform**: https://cloud.atlasgo.io
- **Documentation**: https://atlasgo.io/docs
- **CLI Reference**: https://atlasgo.io/cli-reference
