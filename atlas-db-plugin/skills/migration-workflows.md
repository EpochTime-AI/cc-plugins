---
name: migration-workflows
description: Master Atlas migration commands, planning, applying, and linting migrations for both declarative and versioned workflows
---

# Atlas Migration Workflows

Learn how to plan, apply, lint, and manage database migrations using Atlas commands.

## Declarative Migration Workflow

### 1. Plan Migrations

Compare current database state with desired state and generate migration plan:

```bash
# Basic plan - shows SQL without applying
atlas migrate diff --env local

# Plan with specific schema
atlas migrate diff migration_name --env local

# Dry-run: plan changes but don't apply
atlas migrate plan --env local --dry-run
```

### 2. Review Generated SQL

```bash
# View generated migration file
cat migrations/20240115120000_create_users.sql

# Expected output:
-- alter table "users" to add column "email"
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL;
CREATE UNIQUE INDEX idx_email ON users(email);
```

### 3. Apply Migrations

```bash
# Apply all pending migrations
atlas migrate apply --env local

# Apply with verbose output
atlas migrate apply --env local --verbose

# Dry-run before applying
atlas migrate apply --env local --dry-run

# Apply to specific database
atlas migrate apply --url "mysql://user:pass@localhost/mydb"
```

### 4. Verify State

```bash
# Check migration status
atlas migrate status --env local

# Output shows applied and pending migrations
# Applied: 3 migrations
# Pending: 0 migrations
```

## Versioned Migration Workflow

### 1. Create New Migration

```bash
# Create empty migration with timestamp
atlas migrate new create_users --env local

# Output:
# Created migration: migrations/20240115_120000_create_users.sql

# Edit the migration file and add SQL
cat > migrations/20240115_120000_create_users.sql << 'EOF'
-- Create users table
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
EOF
```

### 2. Plan Migrations from Schema

```bash
# Generate migration from schema definition
atlas migrate plan "add_posts_table" --env local

# Atlas compares schema.hcl with database state
# Creates migration file automatically
```

### 3. Lint Migrations

Validate migrations for errors and best practices:

```bash
# Lint all migrations
atlas migrate lint --env local

# Lint specific migration
atlas migrate lint --env local --latest 2

# Check for:
# - Syntax errors
# - Unsafe operations
# - Data loss risks
# - Best practice violations
```

Example lint output:
```
Migration 001_create_users.sql:
  ⚠ WARNING: Dropping column "temp" without checking if data exists
  ✓ All other checks passed
```

### 4. Apply Migrations

```bash
# Apply all pending migrations
atlas migrate apply --env local

# Apply latest 2 migrations
atlas migrate apply --env local --latest 2

# Apply with transaction wrapping
atlas migrate apply --env local --allow-dirty
```

### 5. Rollback Migrations

```bash
# Create down migration (SQL to revert changes)
atlas migrate down --env local

# Revert to specific migration version
atlas migrate down --env local --to 20240101120000
```

## Project Configuration (atlas.hcl)

### Declarative Setup

```hcl
env "local" {
  # Database connection
  url = "postgres://user:password@localhost/mydb"

  # Development database for planning
  dev = "postgres://user:password@localhost/mydb_dev"

  # Schema definition
  migration {
    dir = "file://migrations"
    format = sql
  }

  # Desired schema source
  schema {
    src = "file://schema.hcl"
  }

  # Output formatting
  format {
    migrate {
      apply = "{{ sql . }}"
      plan = "{{ sql . }}"
    }
  }
}
```

### Versioned Setup

```hcl
env "prod" {
  url = "postgres://user:password@prod-db.com/mydb"

  migration {
    dir = "file://migrations"
    format = sql
    auto_approve = false  # Require manual approval
  }

  format {
    migrate {
      apply = "-- Migration applied\n{{ sql . }}"
    }
  }
}
```

## Common Commands Reference

| Command | Purpose |
|---------|---------|
| `atlas migrate diff` | Plan declarative migrations |
| `atlas migrate apply` | Execute migrations |
| `atlas migrate status` | Show migration status |
| `atlas migrate lint` | Validate migrations |
| `atlas migrate new` | Create versioned migration |
| `atlas migrate down` | Generate rollback migration |
| `atlas schema inspect` | Inspect current schema |
| `atlas schema diff` | Compare two schemas |

## Multi-Environment Setup

```hcl
// atlas.hcl

env "dev" {
  url = "postgres://localhost/dev_db"
  migration { dir = "file://migrations" }
}

env "staging" {
  url = "postgres://staging.example.com/staging_db"
  migration { dir = "file://migrations" }
}

env "prod" {
  url = "postgres://prod.example.com/prod_db"
  migration {
    dir = "file://migrations"
    auto_approve = false
  }
}
```

Running migrations per environment:
```bash
atlas migrate apply --env dev
atlas migrate apply --env staging
atlas migrate apply --env prod
```

## Migration Validation

### Before applying migrations, Atlas validates:

1. **Syntax** - SQL is valid for the target database
2. **Safety** - Detects data loss (dropping columns, tables)
3. **Uniqueness** - Migration names don't conflict
4. **Ordering** - Migrations are applied in correct order
5. **Idempotency** - Migrations can run multiple times safely

### Address validation warnings:

```bash
# Apply despite warnings
atlas migrate apply --allow-dirty --env local

# Review specific migration
atlas migrate lint --env local --latest 1
```

## Rollback Strategies

### Approach 1: Down Migrations (Versioned)

```sql
-- 001_create_users.sql
CREATE TABLE users (id INT PRIMARY KEY);

-- 001_create_users.down.sql
DROP TABLE users;
```

Apply down migration:
```bash
atlas migrate down --env local
```

### Approach 2: State Comparison (Declarative)

Modify schema.hcl to previous state:
```hcl
// schema.hcl - Remove table definition
// Atlas generates a DROP TABLE migration
```

Then plan and apply:
```bash
atlas migrate diff rollback --env local
atlas migrate apply --env local
```

## Handling Errors

### "Database version mismatch"
```bash
# Check migration status
atlas migrate status --env local

# Reset to specific version (careful!)
atlas migrate set --env local 20240101120000
```

### "Column does not exist"
```bash
# Inspect actual database schema
atlas schema inspect --env local

# Compare with expected schema
atlas schema diff --env local
```

## Best Practices

1. **Always plan before applying** - Use dry-run first
2. **Test in dev environment** - Run migrations locally first
3. **Lint migrations** - Check for issues before applying
4. **Version control migrations** - Commit migration files to git
5. **Review migration SQL** - Understand what changes are being made
6. **Backup before production** - Always backup production databases
7. **Use transactions** - Atlas wraps migrations in transactions by default
8. **Document complex changes** - Add comments explaining why changes are needed

## Resources

- **Atlas CLI Docs**: https://atlasgo.io/cli-reference
- **Migration Commands**: https://atlasgo.io/docs/commands
- **Best Practices**: https://atlasgo.io/guides/migration-best-practices

## Local References

For complete migration workflow documentation, see:
- `references/atlas-docs-full/docs.md` - Migration overview (declarative & versioned)
- `references/atlas-docs-full/guides/modern-database-ci-cd.md` - Modern CI/CD patterns
- `references/atlas-docs-full/guides/migration-dirs/template-directory.md` - Migration directory templates
- `references/README.md` - Full documentation index
