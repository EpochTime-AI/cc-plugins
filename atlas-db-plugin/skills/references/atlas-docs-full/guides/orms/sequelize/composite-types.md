Using Composite Types in Sequelize | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In PostgreSQL, a [composite type](https://www.postgresql.org/docs/current/rowtypes.html) is structured like a row or record, consisting of field names and their corresponding data types. Setting a Sequelize model as a composite type enables you to store complex and structured data in a single column.

This guide explains how to define a model type as a composite type in your Sequelize models and configure the schema migration to manage both the composite types and the Sequelize models as a single migration unit using Atlas.

[Atlas Pro Feature](/features#pro)

Atlas support for [Composite Types](/atlas-schema/hcl#composite-type) is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Getting started with Atlas and Sequelize[​](#getting-started-with-atlas-and-sequelize "Direct link to Getting started with Atlas and Sequelize")


Before we continue to composite types, ensure you have installed the [Atlas Sequelize Provider](https://github.com/ariga/atlas-provider-sequelize) on your Sequelize project.

To set up, follow along the [getting started guide](/guides/orms/sequelize) for Sequelize and Atlas.

## Composite Schema[​](#composite-schema "Direct link to Composite Schema")


The Sequelize models are mostly used for defining tables and interacting with the database. Composite types, or any other database objects do not have representation in Sequelize models - a composite type can be defined once, and may be used multiple times in different columns and models.

In order to extend our PostgreSQL schema to include both custom composite types and our Sequelize types, we configure Atlas to read the state of the schema from a [Composite Schema](/atlas-schema/projects#data-source-composite_schema) data source. Follow the steps below to configure this for your project:

1\. Create a `schema.sql` that defines the necessary composite type. In the same way, you can configure the composite type in [Atlas Schema HCL language](/atlas-schema/hcl-types#composite-type):

*   Using SQL
*   Using HCL

schema.sql
```codeBlockLines_AdAo
CREATE TYPE address AS (  street text,  city   text);
```
schema.hcl
```codeBlockLines_AdAo
schema "public" {}composite "address" {  schema = schema.public  field "street" {    type = text  }  field "city" {    type = text  }}
```
2\. In your Sequelize models, define a column that uses the composite type only in PostgreSQL dialect:

user.js
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {    const address = {        type: 'address',        fields: [            'street',            'city',        ],    };    const User = sequelize.define('User', {        name: {            type: DataTypes.STRING,            allowNull: false        },        address: address    });    return User;};
```
3\. In your [`atlas.hcl`](/guides/orms/sequelize#standalone-mode) config file, add a `composite_schema` that includes both your custom types defined in `schema.sql` and your Sequelize models:

atlas.hcl
```codeBlockLines_AdAo
data "composite_schema" "app" {  # Load custom types first.  schema "public" {    url = "file://schema.sql"  }  schema "public" {    url = data.external_schema.sequelize.url  }}env "local" {  src = data.composite_schema.app.url  dev = "docker://postgres/15/dev?search_path=public"}
```

## Usage[​](#usage "Direct link to Usage")


After setting up our schema, we can get its representation using the `atlas schema inspect` command, generate migrations for it, apply them to a database, and more. Below are a few commands to get you started with Atlas:

#### Inspect the Schema[​](#inspect-the-schema "Direct link to Inspect the Schema")


The `atlas schema inspect` command is commonly used to inspect databases. However, we can also use it to inspect our `composite_schema` and print the SQL representation of it:
```codeBlockLines_AdAo
atlas schema inspect \  --env local \  --url env://src \  --format '{{ sql . }}'
```
The command above prints the following SQL. Note, the `address` composite type is defined in the schema before its usage in the `address` field:
```codeBlockLines_AdAo
-- Create composite type "address"CREATE TYPE "address" AS ("street" text, "city" text);-- Create "Users" tableCREATE TABLE "Users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "address" "address" NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));
```

#### Generate Migrations For the Schema[​](#generate-migrations-for-the-schema "Direct link to Generate Migrations For the Schema")


To generate a migration for the schema, run the following command:
```codeBlockLines_AdAo
atlas migrate diff \  --env local
```
Note that a new migration file is created with the following content:

migrations/20240712090543.sql
```codeBlockLines_AdAo
-- Create composite type "address"CREATE TYPE "address" AS ("street" text, "city" text);-- Create "Users" tableCREATE TABLE "Users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "address" "address" NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));
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
*   [Getting started with Atlas and Sequelize](#getting-started-with-atlas-and-sequelize)
*   [Composite Schema](#composite-schema)
*   [Usage](#usage)