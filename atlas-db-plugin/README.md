# Atlas DB Plugin

A comprehensive Claude Code plugin for **Atlas** - the language-independent database schema migration tool.

## Overview

This plugin provides in-depth guides and best practices for using Atlas to manage database schemas using modern DevOps principles. Learn both declarative (Terraform-like) and versioned migration workflows.

## Features

### 5 Comprehensive Skills

1. **Atlas Concepts** - Understand Atlas fundamentals, comparing declarative vs versioned workflows, and core architecture

2. **Schema Definition** - Master HCL, SQL, and ORM schema definitions with practical examples

3. **Migration Workflows** - Learn Atlas migration commands, planning, applying, linting, and rollback strategies

4. **CI/CD Integration** - Integrate Atlas into GitHub Actions, GitLab CI, Docker, and other CI/CD platforms

5. **ORM Integration** - Connect Atlas with GORM, Prisma, Sequelize, SQLAlchemy, TypeORM, Doctrine, and Django

## Quick Start

### Install the Plugin

```bash
# Add the Epochtime AI marketplace (if not already added)
claude plugin marketplace add EpochTime-AI/cc-plugins

# Install atlas-db-plugin
claude plugin install atlas-db-plugin
```

### Ask Questions

Ask Claude about Atlas topics:

```
"Explain the difference between declarative and versioned migrations in Atlas"
"How do I integrate Atlas with GitHub Actions?"
"Show me how to define a schema in HCL"
"How do I set up Atlas with GORM?"
"What's the best practice for handling migrations in production?"
```

## Skills Overview

### 1. Atlas Concepts
- Declarative workflow (Terraform-like)
- Versioned migrations workflow
- Core concepts (schema-as-code, migration planning, projects)
- Comparison table
- Installation methods
- Common use cases

**Triggered by**: Questions about Atlas fundamentals, workflow differences, architecture

### 2. Schema Definition
- HCL schema syntax with examples
- SQL DDL approach
- ORM schema integration (Prisma, GORM)
- Data type reference (PostgreSQL, MySQL, SQLite)
- Relationships and constraints
- Index patterns
- Best practices and naming conventions

**Triggered by**: Questions about writing schemas, table definitions, column types, relationships

### 3. Migration Workflows
- Declarative migration process (plan, review, apply, verify)
- Versioned migration process (create, plan, lint, apply)
- Project configuration patterns
- Environment setup
- Multi-environment management
- Rollback strategies
- Error handling
- Best practices

**Triggered by**: Questions about running migrations, migration commands, troubleshooting

### 4. CI/CD Integration
- GitHub Actions workflows (setup, planning, applying, linting)
- GitLab CI pipelines
- Docker integration
- Multi-environment deployment
- Approval and safety patterns
- Dry-run validation
- Troubleshooting CI issues
- Best practices

**Triggered by**: Questions about CI/CD, GitHub Actions, deployment, automation

### 5. ORM Integration
- GORM integration and best practices
- Prisma schema management
- Sequelize model definitions
- SQLAlchemy models
- TypeORM entities
- Django models
- Doctrine (PHP) entities
- Common ORM workflow
- Multi-model examples

**Triggered by**: Questions about ORM integration, generating migrations from models, framework-specific setup

## Installation & Setup

### Prerequisites

- Atlas CLI installed: https://atlasgo.io/getting-started
- Basic understanding of databases and SQL
- (Optional) Cloud account for Atlas Cloud features

### Installation Methods

```bash
# macOS with Homebrew
brew install ariga/tap/atlas

# Linux/Windows via curl
curl -sSf https://atlasgo.sh | sh

# Docker
docker pull arigaio/atlas

# GitHub Actions
- uses: ariga/setup-atlas@v0
```

## Common Usage Patterns

### Learning Atlas
1. Start with "Atlas Concepts" skill to understand workflows
2. Ask about "schema definition" for your database language
3. Learn "migration workflows" for your chosen approach

### Implementing Migrations
1. Use "Schema Definition" to write your first schema
2. Follow "Migration Workflows" commands step-by-step
3. Reference "CI/CD Integration" when setting up automation

### Team Integration
1. Set up CI/CD with "CI/CD Integration" skill
2. Choose your ORM from "ORM Integration"
3. Establish patterns for your team in "Atlas Concepts"

## Related Resources

- **Official Atlas Docs**: https://atlasgo.io/docs
- **Atlas GitHub**: https://github.com/ariga/atlas
- **CLI Reference**: https://atlasgo.io/cli-reference
- **Schema Guides**: https://atlasgo.io/atlas-schema

## Database Support

Atlas supports migrations for:
- PostgreSQL
- MySQL
- MariaDB
- SQLite
- SQL Server
- Oracle
- ClickHouse
- Snowflake
- Databricks

## ORM Support

Atlas integrates with:
- GORM (Go)
- Prisma (JavaScript/TypeScript)
- Sequelize (JavaScript/TypeScript)
- SQLAlchemy (Python)
- TypeORM (TypeScript)
- Doctrine (PHP)
- Django (Python)
- And many more

## Troubleshooting

### Plugin not showing skills
```bash
# Reinstall the plugin
claude plugin remove atlas-db-plugin
claude plugin install atlas-db-plugin
```

### Can't find information about specific topic
- Check "Atlas Concepts" for fundamentals
- Use "ORM Integration" if using an ORM
- Reference "CI/CD Integration" for automation
- Ask Claude the specific question

## Documentation Sources

All content in this plugin is derived from official Atlas documentation. Complete original documentation is available in:

```
skills/references/
├── atlas-docs-full/          # 103+ pages of original Atlas docs (1.4MB)
├── original-documentation.md # Documentation source mapping
└── atlas-concepts-docs.md    # Core concepts reference
```

### Covered Documentation Sections

- Main docs and CLI reference
- Schema definition (HCL, SQL, ORM)
- Migration workflows (declarative and versioned)
- CI/CD integration (GitHub Actions, GitLab CI, Docker, K8s)
- ORM integration (GORM, Prisma, Sequelize, SQLAlchemy, TypeORM, etc.)
- Deployment guides and best practices
- Database-specific guides

The original documentation can be consulted for:
- Complete CLI command reference
- Advanced configuration options
- Database-specific features
- Edge cases and troubleshooting

## Contributing

Suggestions for improving this plugin? File an issue on GitHub:
https://github.com/EpochTime-AI/cc-plugins/issues

## License

MIT License - See LICENSE file in the repository

---

Atlas is a project of Ariga (https://ariga.io)
Plugin maintained by Epochtime AI

---

**Made by Epochtime AI** | Atlas is a project of Ariga
