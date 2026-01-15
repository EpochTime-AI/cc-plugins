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

### LSP Support (Language Server Protocol)

This plugin includes integrated support for the **Atlas Language Server** (`atlas-ls`), providing real-time code intelligence for Atlas HCL schema files:

- **Syntax highlighting** - Rich highlighting for Atlas HCL syntax
- **Auto-completion** - Intelligent suggestions for Atlas schema elements
- **Error detection** - Real-time validation of Atlas configuration files
- **Go-to-definition** - Navigate to schema definitions
- **Hover information** - Quick documentation lookup

**Supported file patterns:**
- `*.hcl` - General HCL files
- `*.atlas.hcl` - Atlas-specific configuration files
- `atlas.hcl` - Project configuration

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

- **Atlas CLI** installed: https://atlasgo.io/getting-started
- Basic understanding of databases and SQL
- (Optional) Cloud account for Atlas Cloud features
- **Atlas Language Server** (`atlas-ls`) for LSP features (optional)

### Installation Methods

#### Install Atlas CLI

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

#### Install Atlas Language Server (Optional - for LSP features)

The Atlas Language Server provides IDE-like code intelligence for Atlas HCL files.

**Automatic installation (VSCode users):**

The [Atlas HCL VSCode Extension](https://marketplace.visualstudio.com/items?itemName=Ariga.atlas-hcl) automatically downloads and configures `atlas-ls`.

**Manual installation:**

The `atlas-ls` binary is distributed from Ariga's release server:

```bash
# Download for your platform
# Linux (x86_64)
curl -L https://release.ariga.io/atlas-ls/atlas-ls-linux-amd64-latest -o atlas-ls
chmod +x atlas-ls
sudo mv atlas-ls /usr/local/bin/

# macOS (Intel)
curl -L https://release.ariga.io/atlas-ls/atlas-ls-darwin-amd64-latest -o atlas-ls
chmod +x atlas-ls
sudo mv atlas-ls /usr/local/bin/

# macOS (Apple Silicon)
curl -L https://release.ariga.io/atlas-ls/atlas-ls-darwin-arm64-latest -o atlas-ls
chmod +x atlas-ls
sudo mv atlas-ls /usr/local/bin/
```

**Verify installation:**

```bash
# Check if atlas-ls is in your PATH
which atlas-ls

# Test the language server
atlas-ls --help
```

**Note:** If you don't install `atlas-ls`, the plugin will still work perfectly for Atlas documentation and workflows. LSP features are optional enhancements for schema file editing.

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

### LSP not working

**Symptom:** No code intelligence for `.hcl` files

**Solution:**

1. **Check if `atlas-ls` is installed:**
   ```bash
   which atlas-ls
   # If not found, install it using the instructions above
   ```

2. **Verify the binary is executable:**
   ```bash
   chmod +x $(which atlas-ls)
   ```

3. **Check Claude Code LSP errors:**
   - Run `/plugin` in Claude Code
   - Switch to "Errors" tab
   - Look for `atlas-ls` related errors

4. **Common error: "Executable not found in $PATH"**
   - Make sure `atlas-ls` is in your system PATH
   - Restart Claude Code after installing `atlas-ls`

5. **Test the language server manually:**
   ```bash
   # Start the language server
   atlas-ls serve

   # Should start without errors
   # Press Ctrl+C to stop
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
