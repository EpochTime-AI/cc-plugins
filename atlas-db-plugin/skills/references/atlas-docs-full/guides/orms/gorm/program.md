Automatic Schema Migrations for GORM (Program Mode) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

This document describes how to set up the GORM Atlas Provider to load your GORM schema into Atlas in **Go Program Mode**. Go Program Mode is for more advanced scenarios where you need more control specifying which structs to consider as models.

Using this mode, you can load your GORM schema into Atlas by writing a Go program that imports your GORM models and uses the provider as a library to generate the schema.

If all of your GORM models are in a single package, and either embed `gorm.Model` or contain `gorm` struct tags, consider using the [Standalone Mode](/guides/orms/gorm/standalone) instead.

### Installation[​](#installation "Direct link to Installation")


1.  Install Atlas from macOS or Linux by running:
```codeBlockLines_AdAo
curl -sSf https://atlasgo.sh | sh
```
See [atlasgo.io](https://atlasgo.io/getting-started#installation) for more installation options.

2.  Install the provider by running:
```codeBlockLines_AdAo
go get -u ariga.io/atlas-provider-gorm
```

### Setup[​](#setup "Direct link to Setup")


In Go Program Mode, you can use the provider as a library in your Go program to load your GORM schema into Atlas.

1.  Create a new program named `loader/main.go` with the following contents:
```codeBlockLines_AdAo
package mainimport (    "fmt"    "io"    "os"    "ariga.io/atlas-provider-gorm/gormschema"    "github.com/<yourorg>/<yourrepo>/path/to/models")func main() {    stmts, err := gormschema.New("mysql").Load(&models.User{})    if err != nil {        fmt.Fprintf(os.Stderr, "failed to load gorm schema: %v\n", err)        os.Exit(1)    }    io.WriteString(os.Stdout, stmts)}
```
info

Be sure to replace `github.com/<yourorg>/<yourrepo>/path/to/models` with the import path to your GORM models. In addition, replace the model types (e.g `models.User`) with the types of your GORM models.

2.  In your project directory, create a new file named `atlas.hcl` with the following contents:
```codeBlockLines_AdAo
data "external_schema" "gorm" {  program = [    "go",    "run",    "-mod=mod",    "./loader",  ]}env "gorm" {  src = data.external_schema.gorm.url  dev = "docker://mysql/8/dev"  migration {    dir = "file://migrations"  }  format {    migrate {      diff = "{{ sql . \"  \" }}"    }  }}
```

### Verify Setup[​](#verify-setup "Direct link to Verify Setup")


Next, let's verify Atlas is able to read our desired schema, by running the [`schema inspect`](/inspect) command, to inspect our desired schema (GORM models).
```codeBlockLines_AdAo
atlas schema inspect --env gorm --url "env://src"
```
Notice that this command uses `env://src` as the target URL for inspection, meaning "the schema represented by the `src` attribute of the `local` environment block."

Given we have a simple GORM model `user` :

user.go
```codeBlockLines_AdAo
type User struct {    gorm.Model    Name    string    Age     int}
```
We should get the following output after running the `inspect` command above:
```codeBlockLines_AdAo
table "users" {  schema = schema.dev  column "id" {    null           = false    type           = bigint    unsigned       = true    auto_increment = true  }  column "created_at" {    null = true    type = datetime(3)  }  column "updated_at" {    null = true    type = datetime(3)  }  column "deleted_at" {    null = true    type = datetime(3)  }  column "name" {    null = true    type = longtext  }  column "age" {    null = true    type = bigint  }  primary_key {    columns = [column.id]  }  index "idx_users_deleted_at" {    columns = [column.deleted_at]  }}schema "dev" {  charset = "utf8mb4"  collate = "utf8mb4_0900_ai_ci"}
```

## Usage[​](#usage "Direct link to Usage")


Now that your project is set up, choose between the two workflows offered by Atlas for generating and planning migrations:

*   **Versioned Migrations**: Set up a migration directory for your project, creating a version-controlled source of truth of your database schema.

*   **Declarative Migrations**: Set up a Terraform-like workflow where each migration is calculated as the diff between your desired state and the current state of the database.

### Getting Started with the Versioned Workflow[​](#getting-started-with-the-versioned-workflow "Direct link to Getting Started with the Versioned Workflow")


Using the `atlas migrate diff` command, you can automatically generate SQL migration files based on changes made to your GORM models that you can then integrate with GORM's migration system.

Suppose you have the following GORM models across different packages in your project:

models/user.go
```codeBlockLines_AdAo
package modelsimport (    "github.com/yourorg/yourrepo/blog"    "gorm.io/gorm")type User struct {	gorm.Model	Name  string      `gorm:"size:255;not null"`	Email string      `gorm:"size:255;uniqueIndex;not null"`	Bio   string      `gorm:"type:text"`	Posts []blog.Post `gorm:"foreignKey:UserID"`}
```
blog/post.go
```codeBlockLines_AdAo
package blogimport (    "gorm.io/gorm")type Post struct {    gorm.Model    Title     string         `gorm:"not null"`    Content   string         `gorm:"type:text"`    UserID    uint           `gorm:"not null"`}
```
Update your `loader/main.go` to include all your models:

loader/main.go
```codeBlockLines_AdAo
package mainimport (    "fmt"    "io"    "os"    "ariga.io/atlas-provider-gorm/gormschema"    "github.com/yourorg/yourrepo/models"    "github.com/yourorg/yourrepo/blog")func main() {    stmts, err := gormschema.New("mysql").Load(        &models.User{},        &blog.Post{},    )    if err != nil {        fmt.Fprintf(os.Stderr, "failed to load gorm schema: %v\n", err)        os.Exit(1)    }    io.WriteString(os.Stdout, stmts)}
```
Using the Go Program mode configuration file for the provider, generate a migration file by running:
```codeBlockLines_AdAo
atlas migrate diff --env gorm
```
This will generate a migration file in the `migrations` directory, similar to this:
```codeBlockLines_AdAo
migrations├── 20250819061933.sql└── atlas.sum1 directory, 2 files
```
Examining the contents of `20250819061933.sql`:

20250819061933.sql
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE `users` (  `id` bigint unsigned NOT NULL AUTO_INCREMENT,  `created_at` datetime(3) NULL,  `updated_at` datetime(3) NULL,  `deleted_at` datetime(3) NULL,  `name` varchar(255) NOT NULL,  `email` varchar(255) NOT NULL,  PRIMARY KEY (`id`),  INDEX `idx_users_deleted_at` (`deleted_at`),  UNIQUE INDEX `idx_users_email` (`email`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-- Create "posts" tableCREATE TABLE `posts` (  `id` bigint unsigned NOT NULL AUTO_INCREMENT,  `created_at` datetime(3) NULL,  `updated_at` datetime(3) NULL,  `deleted_at` datetime(3) NULL,  `title` longtext NOT NULL,  `content` text NULL,  `user_id` bigint unsigned NOT NULL,  PRIMARY KEY (`id`),  INDEX `fk_users_posts` (`user_id`),  INDEX `idx_posts_deleted_at` (`deleted_at`),  CONSTRAINT `fk_users_posts` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON UPDATE NO ACTION ON DELETE NO ACTION) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```
Atlas automatically generated a migration file that will create the `users` and `posts` tables in your database.

Next, let's alter the `User` model to add a new field:

models/user.go
```codeBlockLines_AdAo
type User struct {    ID        uint           `gorm:"primaryKey"`    CreatedAt time.Time    UpdatedAt time.Time    DeletedAt gorm.DeletedAt `gorm:"index"`    Name      string         `gorm:"not null"`    Email     string         `gorm:"uniqueIndex;not null"`    Bio       string         `gorm:"type:text"`    Posts     []Post         `gorm:"foreignKey:UserID"`}
```
Create a new migration file by running the same command:
```codeBlockLines_AdAo
atlas migrate diff --env gorm
```
The new file in the `migrations` directory contains the following SQL:

20250819062017.sql
```codeBlockLines_AdAo
-- Modify "users" tableALTER TABLE `users` ADD COLUMN `bio` text NULL;
```

#### Next Steps[​](#next-steps "Direct link to Next Steps")


Follow our [Versioned Migrations](/versioned/intro) docs for applying the generated migration files to your database and learning more about using this workflow.

### Getting Started with the Declarative Workflow[​](#getting-started-with-the-declarative-workflow "Direct link to Getting Started with the Declarative Workflow")


Using the `atlas schema apply` command, Atlas will plan and apply the changes directly to your target database based on the current state of your GORM schema. Atlas will prompt you to confirm the migration plan before applying it to the database.

To apply your schema changes declaratively, run:
```codeBlockLines_AdAo
atlas schema apply --env gorm -u "mysql://root:password@localhost:3306/mydb"
```
Where the `-u` flag accepts the [URL](/concepts/url) to the target database.

#### Advanced Usage: Custom GORM Configuration[​](#advanced-usage-custom-gorm-configuration "Direct link to Advanced Usage: Custom GORM Configuration")


To supply custom `gorm.Config{}` object to the provider use the Go Program Mode with the `WithConfig` option. For example, to disable foreign keys:
```codeBlockLines_AdAo
loader := New("sqlite", WithConfig(    &gorm.Config{        DisableForeignKeyConstraintWhenMigrating: true,    },))
```

#### Next Steps[​](#next-steps-1 "Direct link to Next Steps")


Follow our [Declarative Migrations](/declarative/apply) docs to learn more about using this workflow.

## Going Further[​](#going-further "Direct link to Going Further")


Once you have Atlas integrated with your GORM project, consider exploring these additional features:

*   **[Set up CI/CD](/guides/modern-database-ci-cd)**: Automate your schema migrations using GitHub Actions, GitLab CI, or other CI platforms.

*   **[Enforce Migration Safety](/versioned/lint)**: Use Atlas's migration policies to catch potentially dangerous migration operations before they reach production.

*   **[Drift Detection & Schema Monitoring](/monitoring)**: Monitor your production databases for schema drift and unauthorized changes.

*   [Installation](#installation)
*   [Setup](#setup)
*   [Verify Setup](#verify-setup)
*   [Usage](#usage)
    *   [Getting Started with the Versioned Workflow](#getting-started-with-the-versioned-workflow)
    *   [Getting Started with the Declarative Workflow](#getting-started-with-the-declarative-workflow)
*   [Going Further](#going-further)