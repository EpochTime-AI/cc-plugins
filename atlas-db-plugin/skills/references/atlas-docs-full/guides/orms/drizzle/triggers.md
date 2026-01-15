Drizzle Triggers: A Guide to Managing Database Triggers | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

While Drizzle ORM does not have a built-in API for database triggers, you can manage database triggers alongside your Drizzle schemas to execute custom code in response to specific events, such as `INSERT`, `UPDATE`, or `DELETE` operations. Triggers are commonly used for creating audit logs, enforcing complex data validation rules, or automatically updating related data. By using triggers, you can build more robust and reliable applications without adding extra complexity to your application code. This guide explains how to manage database triggers in your Drizzle ORM projects using Atlas.

[Atlas Pro Feature](/features#pro)

Atlas support for [Triggers](/atlas-schema/hcl#trigger) used in this guide is available exclusively to Pro users. To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## How to Manage Database Triggers in Drizzle Projects with Atlas[​](#how-to-manage-database-triggers-in-drizzle-projects-with-atlas "Direct link to How to Manage Database Triggers in Drizzle Projects with Atlas")


To include both your Drizzle models and custom triggers, we'll configure Atlas to combine both schemas using the [`composite_schema`](/atlas-schema/projects#data-source-composite_schema) data source.

Follow these steps in the [Drizzle Quick Start](/guides/orms/drizzle/getting-started) guide to set up your Drizzle project. After setting up your Drizzle project, You structure your project as follows:

*   atlas.hcl
*   drizzle.config.ts
*   schema.ts
```codeBlockLines_AdAo
data "external_schema" "drizzle" {    program = [       "npx",      "drizzle-kit",      "export",    ]}env "local" {  dev = "docker://postgres/16/dev?search_path=public"  schema {    src = data.external_schema.drizzle.url  }  migration {    dir = "file://atlas/migrations"  }}
```
```codeBlockLines_AdAo
import { defineConfig } from 'drizzle-kit';export default defineConfig({  out: './drizzle',  schema: './schema.ts',  dialect: 'postgresql'});
```
```codeBlockLines_AdAo
import { pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';export const User = pgTable('User', {  id: serial('id').primaryKey(),  name: text('name').notNull(),  email: text('email').notNull().unique(),});
```
1\. **Modify Your Drizzle Models**

Start with your existing Drizzle models or create new ones. For this example, we'll use `User` and `UserAuditLog`:

src/schema.ts
```codeBlockLines_AdAo
import { integer, pgTable, serial, text, timestamp } from 'drizzle-orm/pg-core';export const User = pgTable('User', {  id: serial('id').primaryKey(),  name: text('name').notNull(),  email: text('email').notNull().unique(),});export const UserAuditLog = pgTable('UserAuditLog', {  id: serial('id').primaryKey(),  operationType: text('operationType').notNull(),  operationTime: timestamp('operationTime').notNull().defaultNow(),  oldValue: text('oldValue').nullable(),  newValue: text('newValue').nullable(),});
```
2\. **Create SQL File with Trigger Definitions**

Create a file named `triggers.sql` containing the trigger function and triggers.

*   PostgreSQL
*   MySQL

triggers.sql
```codeBlockLines_AdAo
-- Function to audit changes in the User table.CREATE OR REPLACE FUNCTION audit_user_changes()RETURNS TRIGGER AS $$BEGIN    IF (TG_OP = 'INSERT') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, newValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(NEW));        RETURN NEW;    ELSIF (TG_OP = 'UPDATE') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue, newValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD), row_to_json(NEW));        RETURN NEW;    ELSIF (TG_OP = 'DELETE') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD));        RETURN OLD;    END IF;    RETURN NULL;END;$$ LANGUAGE plpgsql;-- Triggers for INSERT, UPDATE, and DELETE on the User table.CREATE TRIGGER user_insert_audit AFTER INSERT ON "User" FOR EACH ROW EXECUTE FUNCTION audit_user_changes();CREATE TRIGGER user_update_audit AFTER UPDATE ON "User" FOR EACH ROW EXECUTE FUNCTION audit_user_changes();CREATE TRIGGER user_delete_audit AFTER DELETE ON "User" FOR EACH ROW EXECUTE FUNCTION audit_user_changes();
```
triggers.sql
```codeBlockLines_AdAo
CREATE TRIGGER user_insert_audit AFTERINSERT	ON `User` FOR EACH ROW BEGININSERT INTO	`UserAuditLog` (operationType, operationTime, newValue)VALUES	('INSERT', CURRENT_TIMESTAMP, JSON_OBJECT('id', NEW.id, 'email', NEW.email));END;CREATE TRIGGER user_update_audit AFTERUPDATE ON `User` FOR EACH ROW BEGININSERT INTO	`UserAuditLog` (operationType, operationTime, oldValue, newValue)VALUES	('UPDATE', CURRENT_TIMESTAMP, JSON_OBJECT('id', OLD.id, 'email', OLD.email), JSON_OBJECT('id', NEW.id, 'email', NEW.email));END;CREATE TRIGGER user_delete_audit AFTER DELETE ON `User` FOR EACH ROW BEGININSERT INTO	`UserAuditLog` (operationType, operationTime, oldValue)VALUES	('DELETE', CURRENT_TIMESTAMP, JSON_OBJECT('id', OLD.id, 'email', OLD.email));END;
```
3\. **Use `composite_schema` to Combine Drizzle and Trigger Definitions**

Modify your `atlas.hcl` file to include the `composite_schema` data source to combine the Drizzle schema with the triggers:

atlas.hcl
```codeBlockLines_AdAo
data "external_schema" "drizzle" {    program = [       "npx",      "drizzle-kit",      "export",    ]}data "composite_schema" "drizzle-extended" {    schema "public" {        url = data.external_schema.drizzle.url    }    schema "public" {        url = "file://triggers.sql"    }}env "local" {    dev = "docker://postgres/16/dev?search_path=public"    schema {        src = data.composite_schema.drizzle-extended.url    }    migration {        dir = "file://atlas/migrations"    }}
```
In this file, we first define an `external_schema` data source that loads the Drizzle schema from the schema definition file and then use the `composite_schema` data source to combine the Drizzle schema with the triggers defined in `triggers.sql`:

## Usage[​](#usage "Direct link to Usage")


#### Verify our setup[​](#verify-our-setup "Direct link to Verify our setup")


To verify our setup is correct, we can inspect the schema using the `atlas schema inspect` command to see how Atlas combines the Drizzle schema with the triggers:
```codeBlockLines_AdAo
atlas schema inspect --env local --url env://schema.src --format "{{ sql . }}"
```
Output:
```codeBlockLines_AdAo
-- Create "User" tableCREATE TABLE "User" ("id" serial NOT NULL, "name" text NOT NULL, "email" text NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "User_email_unique" UNIQUE ("email"));-- Create "audit_user_changes" functionCREATE FUNCTION "audit_user_changes" () RETURNS trigger LANGUAGE plpgsql AS $$BEGIN    IF (TG_OP = 'INSERT') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, newValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(NEW));        RETURN NEW;    ELSIF (TG_OP = 'UPDATE') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue, newValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD), row_to_json(NEW));        RETURN NEW;    ELSIF (TG_OP = 'DELETE') THEN        INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue)        VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD));        RETURN OLD;    END IF;    RETURN NULL;END;$$;-- Create trigger "user_delete_audit"CREATE TRIGGER "user_delete_audit" AFTER DELETE ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();-- Create trigger "user_insert_audit"CREATE TRIGGER "user_insert_audit" AFTER INSERT ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();-- Create trigger "user_update_audit"CREATE TRIGGER "user_update_audit" AFTER UPDATE ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();-- Create "UserAuditLog" tableCREATE TABLE "UserAuditLog" ("id" serial NOT NULL, "operationType" text NOT NULL, "operationTime" timestamp NOT NULL DEFAULT now(), "oldValue" text NULL, "newValue" text NULL, PRIMARY KEY ("id"));
```

#### Generate Migration Files[​](#generate-migration-files "Direct link to Generate Migration Files")


Next, let's see how we can generate new migration files using the `atlas migrate diff` command:
```codeBlockLines_AdAo
atlas migrate diff --env local
```
Output:
```codeBlockLines_AdAo
.├── atlas│   └── migrations│       ├── 20241204122818.sql│       └── atlas.sum├── atlas.hcl├── drizzle.config.ts├── schema.ts└── triggers.sql
```

#### Apply Migrations to the Database[​](#apply-migrations-to-the-database "Direct link to Apply Migrations to the Database")


First, create a PostgreSQL development database with Docker:
```codeBlockLines_AdAo
docker run --name postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres:latest
```
Then, apply the migrations to the database using the `atlas migrate apply` command:
```codeBlockLines_AdAo
atlas migrate apply --env local --url "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable"
```
Output:

Output
```codeBlockLines_AdAo
Migrating to version 20250103091149 (1 migrations in total):  -- migrating version 20250103091149    -> CREATE TABLE "User" ("id" serial NOT NULL, "name" text NOT NULL, "email" text NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "User_email_unique" UNIQUE ("email"));    -> CREATE FUNCTION "audit_user_changes" () RETURNS trigger LANGUAGE plpgsql AS $$       BEGIN           IF (TG_OP = 'INSERT') THEN               INSERT INTO "UserAuditLog" (operationType, operationTime, newValue)               VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(NEW));               RETURN NEW;           ELSIF (TG_OP = 'UPDATE') THEN               INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue, newValue)               VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD), row_to_json(NEW));               RETURN NEW;           ELSIF (TG_OP = 'DELETE') THEN               INSERT INTO "UserAuditLog" (operationType, operationTime, oldValue)               VALUES (TG_OP, CURRENT_TIMESTAMP, row_to_json(OLD));               RETURN OLD;           END IF;           RETURN NULL;       END;       $$;    -> CREATE TRIGGER "user_delete_audit" AFTER DELETE ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();    -> CREATE TRIGGER "user_insert_audit" AFTER INSERT ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();    -> CREATE TRIGGER "user_update_audit" AFTER UPDATE ON "User" FOR EACH ROW EXECUTE FUNCTION "audit_user_changes"();    -> CREATE TABLE "UserAuditLog" ("id" serial NOT NULL, "operationType" text NOT NULL, "operationTime" timestamp NOT NULL DEFAULT now(), "oldValue" text NULL, "newValue" text NULL, PRIMARY KEY ("id"));  -- ok (10.9565ms)  -------------------------  -- 63.790958ms  -- 1 migration  -- 6 sql statements
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


By combining Drizzle ORM with Atlas, you can effectively manage both your application's data models and version-controlled database triggers. This approach lets you leverage drizzle's ORM capabilities while maintaining advanced database features like triggers through Atlas's schema management.

## Frequently Asked Questions (FAQ)[​](#frequently-asked-questions-faq "Direct link to Frequently Asked Questions (FAQ)")


### What is a Drizzle trigger?[​](#what-is-a-drizzle-trigger "Direct link to What is a Drizzle trigger?")


A Drizzle trigger is a database trigger that is managed as part of your Drizzle ORM schema. It allows you to define and version-control automated actions (e.g., logging, validation) that run in response to specific database events like `INSERT`, `UPDATE`, or `DELETE`.

### How to create triggers in Drizzle ORM?[​](#how-to-create-triggers-in-drizzle-orm "Direct link to How to create triggers in Drizzle ORM?")


While Drizzle ORM does not have a dedicated API for creating triggers, you can manage them by defining raw SQL statements and integrating them into your schema management workflow with a tool like Atlas. This guide demonstrates how to use Atlas's `composite_schema` feature to combine your Drizzle schema with SQL files containing trigger definitions, enabling you to version and manage them alongside your tables.

*   [How to Manage Database Triggers in Drizzle Projects with Atlas](#how-to-manage-database-triggers-in-drizzle-projects-with-atlas)
*   [Usage](#usage)
*   [Conclusion](#conclusion)
*   [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
    *   [What is a Drizzle trigger?](#what-is-a-drizzle-trigger)
    *   [How to create triggers in Drizzle ORM?](#how-to-create-triggers-in-drizzle-orm)