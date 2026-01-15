Using Row-Level Security in Sequelize | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Row-level security (RLS) in PostgreSQL enables tables to implement policies that limit access or modification of rows according to the user's role, enhancing the basic SQL-standard privileges provided by `GRANT`.

Once activated, all access to the table has to adhere to these policies. If no policies are defined on the table, it defaults to a deny-all rule, meaning no rows can be seen or mutated. These policies can be tailored to specific commands, roles, or both, allowing for detailed management of who can access or change data.

This guide explains how to attach RLS Policies to your Sequelize models and configure the schema migration to manage both the RLS and the Sequelize models as a single migration unit using Atlas.

[Atlas Pro Feature](/features#pro)

Atlas support for [Row-Level Security Policies](/atlas-schema/hcl#row-level-security-policy) used in this guide is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Getting started with Atlas and Sequelize[​](#getting-started-with-atlas-and-sequelize "Direct link to Getting started with Atlas and Sequelize")


Before we continue, ensure you have installed the [Atlas Sequelize Provider](https://github.com/ariga/atlas-provider-sequelize) on your Sequelize project.

To set up, follow along the [getting started guide](/guides/orms/sequelize) for Sequelize and Atlas.

## Composite Schema[​](#composite-schema "Direct link to Composite Schema")


The Sequelize models are mostly used for defining tables and interacting with the database. Table policies or any other database native objects do not have representation in Sequelize models.

In order to extend our PostgreSQL schema to include both our Sequelize models and their policies, we configure Atlas to read the state of the schema from a [Composite Schema](/atlas-schema/projects#data-source-composite_schema) data source. Follow the steps below to configure this for your project:

1\. Let's define a simple schema with two models (tables): `users` and `tenants`:

*   user.js
*   tenant.js

user.js
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {    const User = sequelize.define('User', {        name: {            type: DataTypes.STRING,            allowNull: false        }    });    User.associate = function(models) {        User.belongsTo(models.Tenant, {            foreignKey: 'tenantId',            as: 'tenant'        });    };    return User;};
```
tenant.js
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {    const Tenant = sequelize.define('Tenant', {        name: {            type: DataTypes.STRING,            allowNull: false        }    });    Tenant.associate = function(models) {        Tenant.hasMany(models.User, {            foreignKey: 'tenantId',            as: 'Users'        });    };    return Tenant;};
```
2\. Now, suppose we want to limit access to the `users` table based on the `tenantId` field. We can achieve this by defining a Row-Level Security (RLS) policy on the `users` table. Below is the SQL code that defines the RLS policy:

schema.sql
```codeBlockLines_AdAo
--- Enable row-level security on the users table.ALTER TABLE "Users" ENABLE ROW LEVEL SECURITY;-- Create a policy that restricts access to rows in the users table based on the current tenant.CREATE POLICY tenant_isolation ON "Users"    USING ("tenantId" = current_setting('app.current_tenant')::integer);
```
3\. In your [`atlas.hcl`](/guides/orms/sequelize#standalone-mode) config file, add a `composite_schema` that includes both your custom security policies in `schema.sql` and your Sequelize models:

atlas.hcl
```codeBlockLines_AdAo
data "composite_schema" "app" {  // First, load the schema  schema "public" {    url = data.external_schema.sequelize.url  }  // Next, load the RLS policies  schema "public" {    url = "file://schema.sql"  }}env "local" {  src = data.composite_schema.app.url  dev = "docker://postgres/15/dev?search_path=public"}
```

## Usage[​](#usage "Direct link to Usage")


After setting up our composite schema, we can get its representation using the `atlas schema inspect` command, generate schema migrations for it, apply them to a database, and more. Below are a few commands to get you started with Atlas:

#### Inspect the Schema[​](#inspect-the-schema "Direct link to Inspect the Schema")


The `atlas schema inspect` command is commonly used to inspect databases. However, we can also use it to inspect our `composite_schema` and print the SQL representation of it:
```codeBlockLines_AdAo
atlas schema inspect \  --env local \  --url env://src \  --format '{{ sql . }}'
```
The command above prints the following SQL. Note, the `tenant_isolation` policy is defined in the schema after the `users` table:
```codeBlockLines_AdAo
-- Create "Tenants" tableCREATE TABLE "Tenants" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));-- Create "Users" tableCREATE TABLE "Users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "tenantId" integer NOT NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "Users_tenantId_fkey" FOREIGN KEY ("tenantId") REFERENCES "Tenants" ("id") ON UPDATE CASCADE ON DELETE CASCADE);-- Enable row-level security for "Users" tableALTER TABLE "Users" ENABLE ROW LEVEL SECURITY;-- Create policy "tenant_isolation"CREATE POLICY "tenant_isolation" ON "Users" AS PERMISSIVE FOR ALL TO PUBLIC USING ("tenantId" = (current_setting('app.current_tenant'::text))::integer);
```

#### Generate Migrations For the Schema[​](#generate-migrations-for-the-schema "Direct link to Generate Migrations For the Schema")


To generate a migration for the schema, run the following command:
```codeBlockLines_AdAo
atlas migrate diff \  --env local
```
Note that a new migration file is created with the following content:

migrations/20240712090543.sql
```codeBlockLines_AdAo
-- Create "Tenants" tableCREATE TABLE "Tenants" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"));-- Create "Users" tableCREATE TABLE "Users" ("id" serial NOT NULL, "name" character varying(255) NOT NULL, "tenantId" integer NOT NULL, "createdAt" timestamptz NOT NULL, "updatedAt" timestamptz NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "Users_tenantId_fkey" FOREIGN KEY ("tenantId") REFERENCES "Tenants" ("id") ON UPDATE CASCADE ON DELETE CASCADE);-- Enable row-level security for "Users" tableALTER TABLE "Users" ENABLE ROW LEVEL SECURITY;-- Create policy "tenant_isolation"CREATE POLICY "tenant_isolation" ON "Users" AS PERMISSIVE FOR ALL TO PUBLIC USING ("tenantId" = (current_setting('app.current_tenant'::text))::integer);
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

### Code Example[​](#code-example "Direct link to Code Example")


Let's add some data to the `Tenants` and `Users` tables and test the RLS policy. Below is an example of how to add data by creating a `seed.js` file:

seed.js
```codeBlockLines_AdAo
const { Sequelize, DataTypes } = require('sequelize');const sequelize = new Sequelize('postgres://postgres:pass@localhost:5432/database?search_path=public&sslmode=disable');// Import modelsconst Tenant = require('./path/to/models/tenant')(sequelize, DataTypes);const User = require('./path/to/models/user')(sequelize, DataTypes);(async () => {    try {        await sequelize.authenticate();        console.log('Connection established successfully.');        // Insert tenants        await Tenant.bulkCreate([          { id: 1, name: 'Tenant A', createdAt: new Date(), updatedAt: new Date() },          { id: 2, name: 'Tenant B', createdAt: new Date(), updatedAt: new Date() },        ]);        // Insert users        await User.bulkCreate([          { id: 1, name: 'John Doe', createdAt: new Date(), updatedAt: new Date(), tenantId: 1 },          { id: 2, name: 'Jane Smith', createdAt: new Date(), updatedAt: new Date(), tenantId: 2 },        ]);        console.log('Data inserted successfully.');    } catch (error) {        console.error('Error inserting tenants:', error);    } finally {        await sequelize.close();    }})();
```
And run the seed script:
```codeBlockLines_AdAo
node seed.js
```
After running the seed script, the `Users` table should have the following data:
```codeBlockLines_AdAo
+----+------------+----------+----------------------------+----------------------------+| id | name       | tenantId | createdAt                  | updatedAt                  ||----+------------+----------+----------------------------+----------------------------|| 1  | John Doe   | 1        | 2024-12-09 15:12:11.394+00 | 2024-12-09 15:12:11.394+00 || 2  | Jane Smith | 2        | 2024-12-09 15:12:11.394+00 | 2024-12-09 15:12:11.394+00 |+----+------------+----------+----------------------------+----------------------------+
```
Now, lets test the RLS policy by creating a test script `testRLS.js`:

testRLS.js
```codeBlockLines_AdAo
const { Sequelize, DataTypes } = require('sequelize');const sequelize = new Sequelize('postgres://postgres:pass@localhost:5432/database?search_path=public&sslmode=disable');const User = require('./my-models/user')(sequelize, DataTypes);(async () => {    try {        await sequelize.authenticate();        console.log('Database connected.');        // Set the tenant for the current session        const tenantIdToTest = 1; // Replace with the tenant ID you want to test        await sequelize.query(`SET app.current_tenant = '${tenantIdToTest}';`);        // Fetch users for the current tenant        const users = await User.findAll();        console.log(`Users for tenant ${tenantIdToTest}:`, users.map(user => user.toJSON()));        // Test for a different tenant        const otherTenantId = 2;        await sequelize.query(`SET app.current_tenant = '${otherTenantId}';`);        const otherTenantUsers = await User.findAll();        console.log(`Users for tenant ${otherTenantId}:`, otherTenantUsers.map(user => user.toJSON()));    } catch (error) {        console.error('Error testing RLS policy:', error);    } finally {        await sequelize.close();    }})();// Output:// Database connected.// Users for tenant 1: [ { id: 1, name: 'John Doe', tenantId: 1 } ]// Users for tenant 2: [ { id: 2, name: 'Jane Smith', tenantId: 2 } ]
```
And run the test script:
```codeBlockLines_AdAo
node testRLS.js
```
The output should show the users that belong to the tenant you set in the `tenantIdToTest` variable.

*   [Getting started with Atlas and Sequelize](#getting-started-with-atlas-and-sequelize)
*   [Composite Schema](#composite-schema)
*   [Usage](#usage)
    *   [Code Example](#code-example)