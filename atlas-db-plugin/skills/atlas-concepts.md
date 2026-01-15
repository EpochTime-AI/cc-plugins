---
name: atlas-concepts
description: Learn Atlas database schema migration fundamentals, comparing declarative vs versioned workflows and understanding core concepts
---

# Atlas Database Migration Concepts

Atlas is a language-independent tool for managing and migrating database schemas using modern DevOps principles. It provides two distinct workflows for handling database migrations.

## Workflows Overview

### Declarative Migrations

The **declarative** approach works like Terraform for databases:

1. **Define desired state** - Write your target schema in HCL, SQL, or via ORM (Prisma, GORM, etc.)
2. **Compare states** - Atlas inspects the current database and compares it to your desired state
3. **Plan migrations** - Automatically generates the SQL migrations needed
4. **Apply changes** - Executes the migration plan to reach desired state

**Best for**: Teams wanting infrastructure-as-code database management, minimal migration files.

### Versioned Migrations

The **versioned** approach uses explicit migration files like traditional tools:

1. **Write migrations** - Create numbered migration files (001_create_users.sql, 002_add_index.sql)
2. **Plan changes** - Atlas automatically plans migrations from your schema definition
3. **Lint migrations** - Validate migrations for errors and best practices
4. **Apply sequentially** - Execute migrations in order with rollback capability

**Best for**: Teams needing explicit control, audit trails, and compatibility with existing migration tools.

## Core Concepts

### Schema-as-Code

Define your database structure in one of three ways:

- **HCL**: Atlas-native language with rich features like templates and validations
  ```hcl
  table "users" {
    schema = schema.public
    column "id" {
      type = int
      auto_increment = true
    }
    primary_key {
      columns = [column.id]
    }
  }
  ```

- **SQL**: Standard DDL statements
  ```sql
  CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL
  );
  ```

- **ORM**: Generate from your ORM models (Prisma, GORM, Django, etc.)
  - Atlas inspects your ORM and derives schema automatically
  - Supports migrations directly from ORM definitions

### Migration Planning

Atlas generates migration plans by:

1. **Reading current state** - Connects to database and inspects current schema
2. **Comparing states** - Identifies differences between current and desired schemas
3. **Creating plan** - Generates SQL with minimal, safe operations
4. **Previewing** - Shows SQL before applying (useful in CI/CD)
5. **Executing safely** - Applies migrations with transactional safety

### Atlas Project Configuration

The `atlas.hcl` file centralizes your migration setup:

```hcl
env "dev" {
  url = "mysql://user:pass@localhost/mydb"
  dev = "mysql://user:pass@localhost/dev"

  migration {
    dir = "file://migrations"
  }

  format {
    migrate {
      apply = format_sql
    }
  }
}
```

## Quick Comparison

| Feature | Declarative | Versioned |
|---------|------------|-----------|
| Schema files | Single desired-state | Multiple versioned |
| Auto-planning | Yes | Yes |
| Rollback | Via state comparison | Via explicit down migrations |
| Audit trail | State history | Explicit migration files |
| Learning curve | Lower | Moderate |
| Team control | Less granular | More explicit |

## Installation

```bash
# macOS with Homebrew
brew install ariga/tap/atlas

# Linux/Windows via curl
curl -sSf https://atlasgo.sh | sh

# Docker
docker pull arigaio/atlas
```

## Common Use Cases

### Scenario 1: Team adopting DevOps practices
Use **declarative** to version control your schema and automatically synchronize databases across environments.

### Scenario 2: Existing migration-based workflow
Use **versioned** to maintain explicit migration history while gaining Atlas's planning and linting benefits.

### Scenario 3: ORM-based development
Use **schema from ORM** to keep Atlas in sync with your GORM/Prisma models automatically.

### Scenario 4: Multi-database setup
Use **environment configuration** to manage different databases (dev, staging, prod) with single migration definitions.

## Key Resources

- **Official Docs**: https://atlasgo.io/docs
- **Installation Guide**: https://atlasgo.io/getting-started
- **Schema Reference**: https://atlasgo.io/atlas-schema
- **CLI Reference**: https://atlasgo.io/cli-reference

## Local References

For complete documentation, see:
- `references/atlas-docs-full/docs.md` - Full main documentation
- `references/atlas-docs-full/getting-started.md` - Complete installation guide
- `references/README.md` - Index of all 103+ documentation files
