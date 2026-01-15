Using Database Views in GORM | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Database views are virtual tables based on the result of a query. Views are helpful when you want to simplify complex queries, strengthen security by only selecting necessary data, or encapsulate the details of your table structures.

From a querying perspective, views and tables are identical, which means GORM can natively query any views that exist on the database. However, defining and managing views has previously had only [partial support](https://github.com/go-gorm/gorm/issues/4966) in GORM.

The Atlas GORM Provider provides an API that allows you to define database views in the form of GORM models, and with the help of [Atlas](https://atlasgo.io/getting-started), migration files can be automatically generated for them.

[Atlas Pro Feature](/features#pro)

Atlas support for [Views](/atlas-schema/hcl#view) used in this guide is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Getting Started with Atlas and GORM[​](#getting-started-with-atlas-and-gorm "Direct link to Getting Started with Atlas and GORM")


To get set up, follow our [_Getting Started_ guide](/guides/orms/gorm/getting-started#prerequisites) for GORM and Atlas, and ensure you have the [Atlas GORM Provider](https://github.com/ariga/atlas-provider-gorm) installed on your GORM project.

## Defining Views in GORM[​](#defining-views-in-gorm "Direct link to Defining Views in GORM")


To define a Go struct as a database `VIEW`, implement the [`ViewDefiner`](https://pkg.go.dev/ariga.io/atlas-provider-gorm/gormschema#ViewDefiner) interface. The `ViewDef()` method takes the `dialect` argument to determine the SQL dialect for generating the view.

The `gormschema` package provides two styles for defining a view's base query:

### BuildStmt Approach[​](#buildstmt-approach "Direct link to BuildStmt Approach")


The `BuildStmt` function allows you to define a query using the GORM API. This is useful when you need to leverage GORM's query building capabilities and maintain consistency with your existing GORM code.

Let's start with a simple example. Suppose we have a `User` model and we want to create a view that shows only working-aged users:

models.go
```codeBlockLines_AdAo
package modelsimport (    "gorm.io/gorm"    "ariga.io/atlas-provider-gorm/gormschema")type User struct {    gorm.Model    Name string    Age  int}// WorkingAgedUsers is mapped to the VIEW definition below.type WorkingAgedUsers struct {    Name string    Age  int}func (WorkingAgedUsers) ViewDef(dialect string) []gormschema.ViewOption {    return []gormschema.ViewOption{        gormschema.BuildStmt(func(db *gorm.DB) *gorm.DB {            return db.Model(&User{}).Where("age BETWEEN 18 AND 65").Select("name, age")        }),    }}
```

### CreateStmt Approach[​](#createstmt-approach "Direct link to CreateStmt Approach")


The `CreateStmt` function allows you to define a query using raw SQL. This is useful when you need to use SQL features that GORM does not support or when you want more direct control over the generated SQL.
```codeBlockLines_AdAo
func (WorkingAgedUsers) ViewDef(dialect string) []gormschema.ViewOption {    var stmt string    switch dialect {    case "mysql":        stmt = "CREATE VIEW working_aged_users AS SELECT name, age FROM users WHERE age BETWEEN 18 AND 65"    case "postgres":        stmt = "CREATE VIEW working_aged_users AS SELECT name, age FROM users WHERE age BETWEEN 18 AND 65"    case "sqlserver":        stmt = "CREATE VIEW [working_aged_users] ([name], [age]) AS SELECT [name], [age] FROM [users] WHERE [age] >= 18 AND [age] <= 65 WITH CHECK OPTION"    }    return []gormschema.ViewOption{        gormschema.CreateStmt(stmt),    }}
```

## Using Views with Atlas[​](#using-views-with-atlas "Direct link to Using Views with Atlas")


The View feature works in both **Standalone** and **Go Program** mode.

### Standalone Mode[​](#standalone-mode "Direct link to Standalone Mode")


For **Standalone** mode, if you place the view definition in the same package as your models, the provider will automatically detect and create migration files for them.

Your `atlas.hcl` configuration remains the same:

atlas.hcl
```codeBlockLines_AdAo
data "external_schema" "gorm" {  program = [    "go",    "run",    "-mod=mod",    "ariga.io/atlas-provider-gorm",    "load",    "--path", "./path/to/models",    "--dialect", "mysql" // | postgres | sqlite | sqlserver  ]}env "gorm" {  src = data.external_schema.gorm.url  dev = "docker://mysql/8/dev"  migration {    dir = "file://migrations"  }  format {    migrate {      diff = "{{ sql . \"  \" }}"    }  }}
```

### Go Program Mode[​](#go-program-mode "Direct link to Go Program Mode")


For **Go Program** mode, you can load the `VIEW` definition in the same way as a model:

loader/main.go
```codeBlockLines_AdAo
package mainimport (    "fmt"    "io"    "os"    "ariga.io/atlas-provider-gorm/gormschema"    "github.com/yourorg/yourrepo/models")func main() {    stmts, err := gormschema.New("mysql").Load(        &models.User{},             // Table-based model.        &models.WorkingAgedUsers{}, // View-based model.    )    if err != nil {        fmt.Fprintf(os.Stderr, "failed to load gorm schema: %v\n", err)        os.Exit(1)    }    io.WriteString(os.Stdout, stmts)}
```
In your project directory, create a new file named `atlas.hcl` with the following contents:

atlas.hcl
```codeBlockLines_AdAo
data "external_schema" "gorm" {  program = [    "go",    "run",    "-mod=mod",    "./loader",  ]}env "gorm" {  src = data.external_schema.gorm.url  dev = "docker://mysql/8/dev"  migration {    dir = "file://migrations"  }  format {    migrate {      diff = "{{ sql . \"  \" }}"    }  }}
```

## Generating Migrations[​](#generating-migrations "Direct link to Generating Migrations")


With the atlas.hcl configuration file set up, you can now run the following command to generate migration files for your views:
```codeBlockLines_AdAo
atlas migrate diff --env gorm
```
This will generate a migration file that includes both your tables and views. For example:

migrations/20240101120000.sql
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE `users` (  `id` bigint unsigned NOT NULL AUTO_INCREMENT,  `created_at` datetime(3) NULL,  `updated_at` datetime(3) NULL,  `deleted_at` datetime(3) NULL,  `name` longtext NULL,  `age` bigint NULL,  PRIMARY KEY (`id`),  INDEX `idx_users_deleted_at` (`deleted_at`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-- Create "working_aged_users" viewCREATE VIEW `working_aged_users` (  `name`,  `age`) AS select `users`.`name` AS `name`,`users`.`age` AS `age` from `users` where (`users`.`age` between 18 and 65);
```
Apply the migration file to your database to create the view by running:
```codeBlockLines_AdAo
atlas migrate apply --env gorm
```

## Using Views in Your Application[​](#using-views-in-your-application "Direct link to Using Views in Your Application")


Once the view is created, you can query it just like any other GORM model:
```codeBlockLines_AdAo
// Query the viewvar workingUsers []WorkingAgedUsersdb.Find(&workingUsers)// Use WHERE clausesvar experienced []WorkingAgedUsersdb.Where("age > ?", 30).Find(&experienced)// Count recordsvar count int64db.Model(&WorkingAgedUsers{}).Count(&count)
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


Database views provide a powerful way to encapsulate complex queries and create reusable data access patterns. By defining views as GORM models and managing them through Atlas migrations, you can maintain a clean, version-controlled database schema that evolves with your application.

For more information and advanced usage of views, refer to the [Atlas GORM Provider documentation](https://github.com/ariga/atlas-provider-gorm?tab=readme-ov-file#views).

## Need Help?[​](#need-help "Direct link to Need Help?")


You can reach us by clicking the Intercom bubble on our site or by [scheduling a demo](https://calendly.com/d/cq3j-ktn-2rc) with our team.

For questions or feedback, join the conversation on our [Discord server](https://discord.gg/zZ6sWVg6NT).

*   [Getting Started with Atlas and GORM](#getting-started-with-atlas-and-gorm)
*   [Defining Views in GORM](#defining-views-in-gorm)
    *   [BuildStmt Approach](#buildstmt-approach)
    *   [CreateStmt Approach](#createstmt-approach)
*   [Using Views with Atlas](#using-views-with-atlas)
    *   [Standalone Mode](#standalone-mode)
    *   [Go Program Mode](#go-program-mode)
*   [Generating Migrations](#generating-migrations)
*   [Using Views in Your Application](#using-views-in-your-application)
*   [Conclusion](#conclusion)
*   [Need Help?](#need-help)