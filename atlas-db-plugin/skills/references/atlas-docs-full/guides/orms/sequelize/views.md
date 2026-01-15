Adding Database Views to Sequelize Schemas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Views are powerful feature in relational databases that allow you to create virtual tables based on the result of a SQL query. They provide a way to simplify complex queries by encapsulating them within a reusable structure, making the database schema easier to understand and maintain.

For instance, a view can consolidate data from multiple related tables into a single, unified dataset, enabling streamlined access to frequently used information. This is particularly useful for reporting or creating read-only abstractions over raw data.

This guide demonstrates how to integrate views with your Sequelize models and set up schema migrations to handle both views and Sequelize models as a unified migration unit using Atlas.

[Atlas Pro Feature](/features#pro)

Atlas support for [Views](/atlas-schema/hcl#view) used in this guide is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Getting started with Atlas and Sequelize[​](#getting-started-with-atlas-and-sequelize "Direct link to Getting started with Atlas and Sequelize")


Before we continue, ensure you have installed the [Atlas Sequelize Provider](https://github.com/ariga/atlas-provider-sequelize) on your Sequelize project.

To set up, follow along the [getting started guide](/guides/orms/sequelize) for Sequelize and Atlas.

## Composite Schema[​](#composite-schema "Direct link to Composite Schema")


Sequelize models are mostly used for defining tables and interacting with the database. Views as well as many other database native objects do not have representation in Sequelize models.

In order to extend our PostgreSQL schema to include both our Sequelize models and views related to them, we configure Atlas to read the state of the schema from a [Composite Schema](/atlas-schema/projects#data-source-composite_schema) data source. Follow the steps below to configure this for your project:

1\. Let's define a simple model: `users`:

*   user.js

models/user.js
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {    const User = sequelize.define('user', {        name: {            type: DataTypes.STRING,            allowNull: false        },        is_active: {            type: DataTypes.BOOLEAN,            defaultValue: true        }    });    return User;};
```
2\. Next step, let's create a view `active_users` that filters active users from the `users` table:

schema.sql
```codeBlockLines_AdAo
-- Create a view "active_users"CREATE VIEW "active_users" AS SELECT * FROM "users" WHERE "is_active" = true;
```
3\. In your [`atlas.hcl`](/guides/orms/sequelize#standalone-mode) config file, add a `composite_schema` that includes both your Sequelize models and your views defined in `schema.sql`:

atlas.hcl
```codeBlockLines_AdAo
data "composite_schema" "app" {  // First, load the schema  schema "public" {    url = data.external_schema.sequelize.url  }  // Next, load the view  schema "public" {    url = "file://schema.sql"  }}env "local" {  src = data.composite_schema.app.url  dev = "docker://postgres/16/dev?search_path=public"}
```

## Usage[​](#usage "Direct link to Usage")


After setting up our composite schema, we can get its representation using the `atlas schema inspect` command, generate schema migrations for it, apply them to a database, and more. Below are a few commands to get you started with Atlas:

#### Inspect the Schema[​](#inspect-the-schema "Direct link to Inspect the Schema")


The `atlas schema inspect` command is commonly used to inspect databases. However, we can also use it to inspect our `composite_schema` and print the SQL representation of it:
```codeBlockLines_AdAo
atlas schema inspect \  --env local \  --url env://src \  --format '{{ sql . }}'
```
The command above prints:
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE "users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "is_active" boolean NULL DEFAULT true, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));-- Create "active_users" viewCREATE VIEW "active_users" ("id", "name", "is_active", "createdAt", "updatedAt") AS SELECT users.id,    users.name,    users.is_active,    users."createdAt",    users."updatedAt"   FROM users  WHERE (users.is_active = true);
```

#### Generate Migrations for the Schema[​](#generate-migrations-for-the-schema "Direct link to Generate Migrations for the Schema")


To generate a migration for the schema, run the following command:
```codeBlockLines_AdAo
atlas migrate diff \  --env local
```
Note that a new migration file is created with the following contents:

migrations/20241204125925.sql
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE "users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "is_active" boolean NULL DEFAULT true, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));-- Create "active_users" viewCREATE VIEW "active_users" ("id", "name", "is_active", "createdAt", "updatedAt") AS SELECT users.id,    users.name,    users.is_active,    users."createdAt",    users."updatedAt"   FROM users  WHERE (users.is_active = true);
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

## Querying the View in Sequelize[​](#querying-the-view-in-sequelize "Direct link to Querying the View in Sequelize")


Create a new file `activeUser.js` with the fields you want to query from the view. Place this file outside your `models` directory so atlas will not create table for it (we already defined the view in the schema).

views/activeUser.js
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {    return sequelize.define('active_user', {        name: {            type: DataTypes.STRING,            allowNull: false        }    });};
```
To query the `active_users` view, use the following code snippet:
```codeBlockLines_AdAo
const activeUsers = require("./sequelize/views/activeUser");return activeUsers.findAll(whereCondition);
```
*   [Getting started with Atlas and Sequelize](#getting-started-with-atlas-and-sequelize)
*   [Composite Schema](#composite-schema)
*   [Usage](#usage)
*   [Querying the View in Sequelize](#querying-the-view-in-sequelize)