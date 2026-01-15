Using Postgres Enum Types in GORM | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[Enum types](https://www.postgresql.org/docs/current/datatype-enum.html) are data structures that consist of a predefined, ordered set of values.

This guide explains how to define a schema field that uses a native PostgreSQL enum type and configure the schema migration to manage both Postgres enums and the GORM model as a single migration unit using Atlas.

[Atlas Pro Feature](/features#pro)

Atlas support for [Composite Schema](/atlas-schema/projects#data-source-composite_schema) used in this guide is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Getting started with Atlas and GORM[​](#getting-started-with-atlas-and-gorm "Direct link to Getting started with Atlas and GORM")


Before we continue to enum types, ensure you have installed the [Atlas GORM Provider](https://github.com/ariga/atlas-provider-gorm) on your GORM project.

To set up, follow along the [getting started guide](/guides/orms/gorm) for GORM and Atlas.

## Composite Schema[​](#composite-schema "Direct link to Composite Schema")


The GORM package is mostly used for defining tables (our Go types) and interacting with the database. Enum types, or any other database objects do not have representation in GORM models - an enum type can be defined once, and may be used multiple times in different fields and models.

In order to extend our PostgreSQL schema to include both custom enum types and our GORM types, we configure Atlas to read the state of the schema from a [Composite Schema](/atlas-schema/projects#data-source-composite_schema) data source. Follow the steps below to configure this for your project:

1\. Create a `schema.sql` that defines the necessary enum type. In the same way, you can configure the enum type in [Atlas Schema HCL language](/atlas-schema/hcl-types#enum):

*   Using SQL
*   Using HCL

schema.sql
```codeBlockLines_AdAo
CREATE TYPE status AS ENUM ('active', 'inactive', 'pending');
```
schema.hcl
```codeBlockLines_AdAo
schema "public" {}enum "status" {  schema = schema.public  values = ["active", "inactive", "pending"]}
```
2\. In your GORM model, define an enum field that uses the underlying Postgres `ENUM` type:

models.go
```codeBlockLines_AdAo
type User struct {    gorm.Model    ID   uint    Status Status `gorm:"type:status"`}type Status stringconst (    Active   Status = "active"    Inactive Status = "inactive"    Pending  Status = "pending")func (p *Status) Scan(value interface{}) error {    *p = Status(value.([]byte))    return nil}func (p Status) Value() (driver.Value, error) {    return string(p), nil}
```
3\. In your [`atlas.hcl`](/guides/orms/gorm#standalone-mode) config file, add a `composite_schema` that includes both your custom types defined in `schema.sql` and your GORM model:

atlas.hcl
```codeBlockLines_AdAo
data "composite_schema" "app" {  # Load enum types first.  schema "public" {    url = "file://schema.hcl"  }  # Then, load the GORM models.  schema "public" {    url = data.external_schema.gorm.url  }}env "local" {  src = data.composite_schema.app.url  dev = "docker://postgres/15/dev?search_path=public"}
```

## Usage[​](#usage "Direct link to Usage")


After setting up our composite schema, we can get its representation using the `atlas schema inspect` command, generate schema migrations for it, apply them to a database, and more. Below are a few commands to get you started with Atlas:

#### Inspect the Schema[​](#inspect-the-schema "Direct link to Inspect the Schema")


The `atlas schema inspect` command is commonly used to inspect databases. However, we can also use it to inspect our `composite_schema` and print the SQL representation of it:
```codeBlockLines_AdAo
atlas schema inspect \  --env local \  --url env://src \  --format '{{ sql . }}'
```
The command above prints the following SQL. Note, the `status` enum type is defined in the schema before its usage in the `users.status` column:
```codeBlockLines_AdAo
-- Create enum type "status"CREATE TYPE "status" AS ENUM ('active', 'inactive', 'pending');-- Create "users" tableCREATE TABLE "users" ("id" bigserial NOT NULL, "created_at" timestamptz NULL, "updated_at" timestamptz NULL, "deleted_at" timestamptz NULL, "status" "status" NULL, PRIMARY KEY ("id"));-- Create index "idx_users_deleted_at" to table: "users"CREATE INDEX "idx_users_deleted_at" ON "users" ("deleted_at");
```

#### Generate Migrations For the Schema[​](#generate-migrations-for-the-schema "Direct link to Generate Migrations For the Schema")


To generate a migration for the schema, run the following command:
```codeBlockLines_AdAo
atlas migrate diff \  --env local
```
Note that a new migration file is created with the following content:

migrations/20240712090543.sql
```codeBlockLines_AdAo
-- Create enum type "status"CREATE TYPE "status" AS ENUM ('active', 'inactive', 'pending');-- Create "users" tableCREATE TABLE "users" ("id" bigserial NOT NULL, "created_at" timestamptz NULL, "updated_at" timestamptz NULL, "deleted_at" timestamptz NULL, "status" "status" NULL, PRIMARY KEY ("id"));-- Create index "idx_users_deleted_at" to table: "users"CREATE INDEX "idx_users_deleted_at" ON "users" ("deleted_at");
```

#### Apply the Migrations[​](#apply-the-migrations "Direct link to Apply the Migrations")


To apply the migration generated above to a database, run the following command:
```codeBlockLines_AdAo
atlas migrate apply \  --env local \  --url "postgres://postgres:pass@localhost:5432/database?search_path=public&sslmode=disable"
```
Apply the Schema Directly on the Database

Sometimes, there is a need to apply the schema directly to the database without generating a migration file. For example, when experimenting with schema changes, spinning up a database for testing, etc. In such cases, you can use the command below to apply the schema directly to the database:
```codeBlockLines_AdAo
atlas schema apply \  --env local \  --url "postgres://postgres:pass@localhost:5432/database?search_path=public&sslmode=disable"
```
Or, using the [Atlas Go SDK](https://github.com/ariga/atlas/tree/master/atlasexec):
```codeBlockLines_AdAo
ac, err := atlasexec.NewClient(".", "atlas")if err != nil {    log.Fatalf("failed to initialize client: %w", err)}// Automatically update the database with the desired schema.// Another option, is to use 'migrate apply' or 'schema apply' manually.if _, err := ac.SchemaApply(ctx, &atlasexec.SchemaApplyParams{    Env: "local",    URL: "postgres://postgres:pass@localhost:5432/database?search_path=public&sslmode=disable",}); err != nil {    log.Fatalf("failed to apply schema changes: %w", err)}
```
*   [Getting started with Atlas and GORM](#getting-started-with-atlas-and-gorm)
*   [Composite Schema](#composite-schema)
*   [Usage](#usage)