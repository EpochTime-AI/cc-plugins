---
name: orm-integration
description: This skill should be used when the user asks to "integrate Atlas with GORM", "use Atlas with Prisma", "Sequelize migrations with Atlas", "SQLAlchemy Atlas integration", "TypeORM and Atlas", "Django with Atlas", "Doctrine migrations", "ORM-driven migrations", "generate migrations from ORM models", or needs guidance on integrating Atlas with popular ORMs for automatic schema management
---

# Atlas ORM Integration

Use Atlas with your favorite ORM (GORM, Prisma, Sequelize, SQLAlchemy, TypeORM, Doctrine) to generate migrations from ORM models.

## GORM Integration

### Model Definition

```go
package models

import (
  "database/sql"
  "gorm.io/gorm"
)

type User struct {
  ID    uint   `gorm:"primaryKey"`
  Email string `gorm:"uniqueIndex"`
  Name  string
  Posts []Post `gorm:"foreignKey:UserID"`
}

type Post struct {
  ID     uint
  Title  string `gorm:"index"`
  UserID uint
  User   User `gorm:"constraint:OnDelete:CASCADE"`
}
```

### Atlas Configuration

```hcl
// atlas.hcl
env "local" {
  url = "mysql://user:password@localhost/mydb"

  // Load schema from GORM models
  migration {
    dir = "file://migrations"
  }

  schema {
    // Point to your GORM models package
    src = "file://models"
  }
}
```

### Generate & Apply Migrations

```bash
# Inspect current GORM schema
atlas schema inspect --env local

# Plan migrations from GORM models
atlas migrate diff gorm_migration --env local

# Apply migrations
atlas migrate apply --env local
```

### GORM Best Practices

```go
type User struct {
  ID        uint          `gorm:"primaryKey"`
  Email     string        `gorm:"uniqueIndex;not null"`
  Name      string
  CreatedAt time.Time     `gorm:"autoCreateTime"`
  UpdatedAt time.Time     `gorm:"autoUpdateTime"`

  // Define relationships
  Posts     []Post        `gorm:"foreignKey:UserID"`
}

type Post struct {
  ID        uint
  Title     string        `gorm:"index"`
  Content   string        `gorm:"type:text"`
  UserID    uint          `gorm:"not null"`
  User      User          `gorm:"constraint:OnDelete:CASCADE"`
  CreatedAt time.Time
}
```

## Prisma Integration

### Schema Definition

```prisma
// schema.prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]  @relation("UserPosts")
}

model Post {
  id     Int    @id @default(autoincrement())
  title  String
  user   User   @relation("UserPosts", fields: [userId], references: [id], onDelete: Cascade)
  userId Int
}
```

### Atlas Configuration

```hcl
// atlas.hcl
env "local" {
  url = getenv("DATABASE_URL")

  migration {
    dir = "file://prisma/migrations"
  }

  // Generate from Prisma schema
  schema {
    src = "file://prisma/schema.prisma"
  }
}
```

### Prisma Workflow

```bash
# Plan from Prisma schema
atlas migrate diff prisma_init --env local

# Apply migrations
atlas migrate apply --env local

# Check migration status
atlas migrate status --env local
```

## Sequelize Integration

### Model Definition

```javascript
// models/User.js
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define('User', {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true
    },
    email: {
      type: DataTypes.STRING,
      unique: true,
      allowNull: false
    },
    name: DataTypes.STRING
  });

  User.associate = (models) => {
    User.hasMany(models.Post, { foreignKey: 'userId' });
  };

  return User;
};

// models/Post.js
module.exports = (sequelize, DataTypes) => {
  const Post = sequelize.define('Post', {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true
    },
    title: DataTypes.STRING,
    userId: DataTypes.INTEGER
  });

  Post.associate = (models) => {
    Post.belongsTo(models.User, { foreignKey: 'userId' });
  };

  return Post;
};
```

### Configuration

```hcl
// atlas.hcl
env "local" {
  url = "mysql://root:password@localhost/mydb"

  migration {
    dir = "file://migrations"
  }

  schema {
    src = "file://models" // Point to models directory
  }
}
```

## SQLAlchemy Integration

### Model Definition

```python
# models.py
from sqlalchemy import Column, Integer, String, ForeignKey, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    name = Column(String(255))
    posts = relationship("Post", back_populates="user", cascade="all, delete")

class Post(Base):
    __tablename__ = 'posts'

    id = Column(Integer, primary_key=True)
    title = Column(String(255))
    user_id = Column(Integer, ForeignKey('users.id'), nullable=False)
    user = relationship("User", back_populates="posts")
```

### Atlas Configuration

```hcl
// atlas.hcl
env "local" {
  url = "mysql+pymysql://root:password@localhost/mydb"

  migration {
    dir = "file://migrations"
  }

  schema {
    src = "file://models"
  }
}
```

## TypeORM Integration

### Entity Definition

```typescript
// entities/User.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Post } from "./Post";

@Entity("users")
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column({ nullable: true })
  name: string;

  @OneToMany(() => Post, post => post.user)
  posts: Post[];
}

// entities/Post.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity("posts")
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  userId: number;

  @ManyToOne(() => User, user => user.posts, { onDelete: "CASCADE" })
  user: User;
}
```

### Configuration

```hcl
// atlas.hcl
env "local" {
  url = "mysql://root:password@localhost/mydb"

  migration {
    dir = "file://migrations"
  }

  schema {
    src = "file://dist/entities"  // Compiled JavaScript
  }
}
```

## Django Integration

### Model Definition

```python
# models.py
from django.db import models

class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=255, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'users'

class Post(models.Model):
    title = models.CharField(max_length=255)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'posts'
```

### Atlas Configuration

```hcl
// atlas.hcl
env "local" {
  url = "mysql://root:password@localhost/mydb"

  migration {
    dir = "file://migrations"
  }

  schema {
    src = "file://path/to/django/app"
  }
}
```

## Doctrine (PHP) Integration

### Entity Definition

```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ORM\Table(name: 'users')]
class User {
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private int $id;

    #[ORM\Column(type: 'string', unique: true)]
    private string $email;

    #[ORM\OneToMany(targetEntity: Post::class, mappedBy: 'user')]
    private Collection $posts;
}

#[ORM\Entity]
#[ORM\Table(name: 'posts')]
class Post {
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private int $id;

    #[ORM\Column(type: 'string')]
    private string $title;

    #[ORM\ManyToOne(targetEntity: User::class, inversedBy: 'posts')]
    #[ORM\JoinColumn(name: 'user_id')]
    private User $user;
}
```

## Common ORM Workflow

### Step 1: Define Models in Your ORM
Update your ORM models with new fields, relationships, etc.

### Step 2: Configure Atlas
Point Atlas to your ORM models directory.

### Step 3: Plan Migrations
```bash
atlas migrate diff migration_name --env local
```

### Step 4: Review Generated SQL
```bash
cat migrations/20240115_120000_migration_name.sql
```

### Step 5: Apply Migrations
```bash
atlas migrate apply --env local
```

## ORM-Specific Tips

### GORM
- Use `gorm:"index"` for indexes
- Use `gorm:"uniqueIndex"` for unique indexes
- Use `gorm:"constraint:OnDelete:CASCADE"` for foreign keys
- Use `gorm:"type:json"` for JSON columns

### Prisma
- Use `@unique` for unique constraints
- Use `@db.Text` for text fields
- Use `onDelete: Cascade` for foreign key actions
- Use `@default(autoincrement())` for auto-increment

### Sequelize
- Use `autoIncrement: true` for auto-increment
- Use `unique: true` for unique constraints
- Use `allowNull: false` for not null
- Use `references: { model: 'table', key: 'id' }` for foreign keys

### SQLAlchemy
- Use `unique=True` for unique constraints
- Use `nullable=False` for not null
- Use `ForeignKey()` for foreign keys
- Use `cascade="all, delete"` for cascade delete

## Multi-Model Example

```yaml
# atlas.hcl with Prisma + custom SQL
env "local" {
  url = "mysql://root:password@localhost/mydb"

  migration {
    dir = "file://migrations"
  }

  # Load from Prisma + custom schema.sql
  schema {
    src = "file://prisma/schema.prisma"
    src = "file://schema/custom.sql"
  }
}
```

## Best Practices

1. **Keep ORM models as source of truth** - Update models first, then apply migrations
2. **Review generated SQL** - Always check migrations before applying
3. **Test in development** - Run migrations locally before production
4. **Version control migrations** - Commit all migration files to git
5. **Document schema changes** - Add comments explaining migrations
6. **Use constraints** - Leverage ORM features for database constraints
7. **Monitor performance** - Index frequently queried columns

## Resources

- **GORM Guide**: https://atlasgo.io/guides/orms/gorm
- **Prisma Guide**: https://atlasgo.io/guides/orms/prisma
- **Sequelize Guide**: https://atlasgo.io/guides/orms/sequelize
- **SQLAlchemy Guide**: https://atlasgo.io/guides/orms/sqlalchemy
- **TypeORM Guide**: https://atlasgo.io/guides/orms/typeorm

## Local References

For complete ORM integration documentation, see:
- `references/atlas-docs-full/guides/orms/gorm.md` + `guides/orms/gorm/*` - Complete GORM guides
- `references/atlas-docs-full/guides/orms/prisma.md` - Prisma integration
- `references/atlas-docs-full/guides/orms/sequelize.md` + `guides/orms/sequelize/*` - Complete Sequelize guides
- `references/atlas-docs-full/guides/orms/sqlalchemy.md` - SQLAlchemy integration
- `references/atlas-docs-full/guides/orms/typeorm.md` + `guides/orms/typeorm/*` - Complete TypeORM guides
- `references/atlas-docs-full/guides/orms/doctrine.md` - Doctrine (PHP) integration
- `references/atlas-docs-full/guides/orms/django.md` - Django integration
- `references/atlas-docs-full/guides/orms/` - All ORM integration guides
- `references/README.md` - Full documentation index
