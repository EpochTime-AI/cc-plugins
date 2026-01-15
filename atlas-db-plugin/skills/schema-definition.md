---
name: schema-definition
description: Write database schemas using Atlas HCL, SQL, or ORM formats with practical examples for tables, indexes, constraints
---

# Atlas Schema Definition Guide

Define your database schema using HCL, SQL, or ORM declarations. This skill covers syntax, best practices, and common patterns.

## HCL Schema Definition

HCL is Atlas's native schema language with powerful features like templates and validation.

### Basic Table Structure

```hcl
schema "public" {
  comment = "Standard public schema"
}

table "users" {
  schema = schema.public
  comment = "User accounts"

  column "id" {
    type = bigint
    auto_increment = true
    comment = "Primary key"
  }

  column "email" {
    type = varchar(255)
    null = false
    comment = "User email address"
  }

  column "created_at" {
    type = timestamp
    default = sql("CURRENT_TIMESTAMP")
  }

  primary_key {
    columns = [column.id]
  }

  index "idx_email" {
    columns = [column.email]
    unique = true
  }
}
```

### Key Column Types

```hcl
column "name" {
  type = varchar(100)        # String with max length
}

column "age" {
  type = int                 # Integer
}

column "balance" {
  type = decimal(10, 2)      # Decimal with precision
}

column "active" {
  type = bool
  default = true             # Boolean with default
}

column "metadata" {
  type = json                # JSON type
}

column "created" {
  type = timestamp
  default = sql("CURRENT_TIMESTAMP")
}

column "tags" {
  type = text
  null = true                # Nullable column
}
```

### Relationships & Constraints

```hcl
table "posts" {
  schema = schema.public

  column "id" {
    type = bigint
    auto_increment = true
  }

  column "user_id" {
    type = bigint
  }

  column "title" {
    type = varchar(255)
    null = false
  }

  column "content" {
    type = text
  }

  primary_key {
    columns = [column.id]
  }

  # Foreign key constraint
  foreign_key "fk_user" {
    columns = [column.user_id]
    ref_columns = [table.users.column.id]
    on_delete = CASCADE
    on_update = CASCADE
  }

  # Unique constraint
  index "idx_unique_title_user" {
    columns = [column.title, column.user_id]
    unique = true
  }

  # Check constraint
  check "content_not_empty" {
    expr = "content IS NOT NULL OR title IS NOT NULL"
  }
}
```

### Indexes

```hcl
# Single column index
index "idx_status" {
  columns = [column.status]
}

# Composite index
index "idx_user_created" {
  columns = [column.user_id, column.created_at]
}

# Unique index
index "idx_unique_email" {
  columns = [column.email]
  unique = true
}

# Partial index (database-specific)
index "idx_active_users" {
  columns = [column.email]
  where = "active = true"
}
```

## SQL Schema Definition

Use standard DDL when you prefer SQL or need specific database features:

```sql
-- Create schema
CREATE SCHEMA IF NOT EXISTS public;

-- Create users table
CREATE TABLE public.users (
  id BIGSERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create posts table with foreign key
CREATE TABLE public.posts (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES public.users(id) ON DELETE CASCADE,
  UNIQUE(user_id, title)
);

-- Create indexes
CREATE INDEX idx_posts_user ON public.posts(user_id);
CREATE INDEX idx_posts_created ON public.posts(created_at);
```

## ORM Schema Integration

### Prisma Schema

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id    Int     @id @default(autoincrement())
  title String
  user  User    @relation(fields: [userId], references: [id])
  userId Int
}
```

Atlas can derive the database schema from Prisma and use it for migrations.

### GORM Models

```go
type User struct {
  ID    uint
  Email string `gorm:"uniqueIndex"`
  Name  string
  Posts []Post
}

type Post struct {
  ID     uint
  Title  string
  UserID uint
  User   User
}
```

## Migration Project Structure

```
my-project/
├── atlas.hcl              # Project configuration
├── schema.hcl             # Desired schema definition
└── migrations/
    ├── 20240101120000_create_users.sql
    ├── 20240102120000_create_posts.sql
    └── 20240103120000_add_index.sql
```

## Configuration File Pattern

```hcl
// atlas.hcl - Central configuration

env "local" {
  url = "mysql://root:password@localhost:3306/mydb"
  dev = "mysql://root:password@localhost:3306/mydb_dev"

  migration {
    dir = "file://migrations"
    format = sql
  }

  schema {
    src = "file://schema.hcl"
  }

  format {
    migrate {
      apply = "-- Applying schema changes\n\n{{ sql . }}\n"
    }
  }
}
```

## Data Type Reference

### PostgreSQL
- `INT`, `BIGINT`, `SMALLINT`
- `VARCHAR(n)`, `TEXT`
- `NUMERIC(p,s)`, `DECIMAL(p,s)`
- `BOOLEAN`
- `TIMESTAMP`, `DATE`, `TIME`
- `JSON`, `JSONB`
- `UUID`
- `ARRAY`

### MySQL
- `INT`, `BIGINT`, `SMALLINT`
- `VARCHAR(n)`, `TEXT`
- `DECIMAL(p,s)`, `FLOAT`
- `BOOLEAN` (alias for TINYINT)
- `TIMESTAMP`, `DATETIME`, `DATE`, `TIME`
- `JSON`
- `ENUM`

### SQLite
- `INTEGER`
- `TEXT`
- `REAL`
- `BLOB`

## Best Practices

1. **Always use primary keys** - Every table should have a primary key
2. **Foreign keys for relationships** - Use explicit foreign keys, not just IDs
3. **Meaningful indexes** - Index on commonly filtered/sorted columns
4. **Timestamps for audit** - Add created_at and updated_at timestamps
5. **Not null constraints** - Define null=false for required fields
6. **Comments** - Document tables and columns for clarity
7. **Naming conventions** - Use snake_case for tables, columns, indexes

## Common Patterns

### User + Posts Pattern
```hcl
table "users" {
  column "id" { type = bigint; auto_increment = true }
  column "email" { type = varchar(255); null = false }
  primary_key { columns = [column.id] }
  index "idx_email" { columns = [column.email]; unique = true }
}

table "posts" {
  column "id" { type = bigint; auto_increment = true }
  column "user_id" { type = bigint; null = false }
  column "title" { type = varchar(255) }
  primary_key { columns = [column.id] }
  foreign_key "fk_user" {
    columns = [column.user_id]
    ref_columns = [table.users.column.id]
    on_delete = CASCADE
  }
}
```

## Resources

- **Atlas Schema Docs**: https://atlasgo.io/atlas-schema
- **HCL Reference**: https://atlasgo.io/atlas-schema/hcl
- **SQL Reference**: https://atlasgo.io/atlas-schema/sql
- **ORM Support**: https://atlasgo.io/atlas-schema/external

## Local References

For complete schema documentation, see:
- `references/atlas-docs-full/docs.md` - Schema-as-code overview
- `references/atlas-docs-full/guides/orms/` - Complete ORM integration guides
- `references/README.md` - Full documentation index
