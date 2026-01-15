Testing Data Migrations | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Most commonly, migrations deal with schema changes, such as adding or removing columns, creating tables, or altering constraints. However, as your application evolves, you may need to add or refactor data within the database, which is where data migrations come in. For instance, you may need to seed data in a table, backfill data for existing records in new columns, or somehow transform existing data to accommodate changes in your application.

Data migrations can be especially tricky to get right, and mistakes can be problematic and irreversible. For this reason testing data migrations is crucial. Testing data migrations typically involves the following steps:

1.  Setting up an empty database.
2.  Applying migrations up to the one before the test.
3.  Seeding test data.
4.  Running the migration under test.
5.  Making assertions to verify the results.

This process can be cumbersome to set up and error-prone as it often involves writing an ad-hoc program to automate the steps mentioned above or manually testing the migration.

Atlas's [`migrate test`](/testing/migrate) command simplifies this by allowing you to define test cases in a concise syntax and acts as a harness to run these tests during local development and in CI.

In this guide we will learn how to use the `migrate test` command to test migration files.

[Atlas Pro Feature](/features#pro)

Testing is currently available only to [Atlas Pro users](/features#pro). To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Project Setup[​](#project-setup "Direct link to Project Setup")


Before we begin writing tests, we will start by setting up a project with a config file, schema file, and a migration directory containing a few migration files.

### Config File[​](#config-file "Direct link to Config File")


Create a [config file](/atlas-schema/projects#project-files) named `atlas.hcl`.

In this file we will define an environment, specify the source of our schema, and a URL for our [dev database](/concepts/dev-database).

We will also create a file named `migrate.test.hcl` to write our tests, and add it to the `atlas.hcl` file in the test block.

atlas.hcl
```codeBlockLines_AdAo
env "dev" {  src = "file://schema.hcl"  dev = "docker://postgres/15/dev"  # Test configuration for local development.  test {    migrate {      src = ["migrate.test.hcl"]    }  }}
```

### Schema File[​](#schema-file "Direct link to Schema File")


Next, we will create a schema containing two tables:

*   **`users`**: stores user-specific data, such as `id` and `email`.
*   **`posts`**: holds the post content, when it was created, and links each post to its author with a `user_id`.

*   schema.hcl
*   schema.sql

schema.hcl
```codeBlockLines_AdAo
schema "public" {}table "users" {  schema = schema.public  column "id" {    type    = int    null    = false  }  column "email" {    type    = varchar(255)    null    = false  }  primary_key {    columns = [column.id]  }}table "posts" {  schema = schema.public  column "id" {    type    = int    null    = false  }  column "title" {    type    = varchar(100)    null    = false  }  column "created_at" {    null = false    type = timestamp  }  column "user_id" {    type    = int    null    = false  }  primary_key {    columns = [column.id]  }  foreign_key "authors_fk" {    columns = [column.user_id]    ref_columns = [table.users.column.id]  }}
```
```codeBlockLines_AdAo
-- Add new schema named "public"CREATE SCHEMA IF NOT EXISTS "public";-- Create "users" tableCREATE TABLE "public"."users" ("id" integer NOT NULL, "email" character varying(255) NOT NULL, PRIMARY KEY ("id"));-- Create "posts" tableCREATE TABLE "public"."posts" ("id" integer NOT NULL, "title" character varying(100) NOT NULL, "created_at" TIMESTAMP NOT NULL, "user_id" integer NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "authors_fk" FOREIGN KEY ("user_id") REFERENCES "public"."users" ("id") ON UPDATE NO ACTION ON DELETE NO ACTION);
```

### Generating a Migration[​](#generating-a-migration "Direct link to Generating a Migration")


To generate our initial migration, we will run the following command:
```codeBlockLines_AdAo
atlas migrate diff --env dev
```
We should see a new `migrations` directory created with the following files:

*   20240807192632.sql
*   atlas.sum

20240807192632.sql
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE "users" ("id" integer NOT NULL, "email" character varying(255) NOT NULL, PRIMARY KEY ("id"));-- Create "posts" tableCREATE TABLE "posts" ("id" integer NOT NULL, "title" character varying(100) NOT NULL, "created_at" timestamp NOT NULL, "user_id" integer NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "authors_fk" FOREIGN KEY ("user_id") REFERENCES "users" ("id") ON UPDATE NO ACTION ON DELETE NO ACTION);
```
atlas.sum
```codeBlockLines_AdAo
h1:fzAvfmbDGG5M4KAjGXqAylAE3PawD5+6GqkVqvizMys=20240807192632.sql h1:oOoSEXM3VyBkPGeWdyoYfAVF1dFn79c3yGhsy3Ys8PI=
```

### Expanding Business Logic[​](#expanding-business-logic "Direct link to Expanding Business Logic")


Great! Now that we have a basic schema and migration directory, let's add some business logic before we begin testing.

We will add a column `latest_post_ts` to the `users` table, which will hold the timestamp of the user's most recent post. To automatically populate this column we will create a trigger `update_latest_post_trigger`.

*   schema.hcl
*   schema.sql

schema.hcl
```codeBlockLines_AdAo
schema "public" {}table "users" {  schema = schema.public  column "id" {    null = false    type = integer  }  column "email" {    null = false    type = character_varying(255)  }  column "latest_post_ts" {    null = true    type = timestamp  }  primary_key {    columns = [column.id]  }}table "posts" {  schema = schema.public  column "id" {    null = false    type = integer  }  column "title" {    null = false    type = character_varying(100)  }  column "created_at" {    null = false    type = timestamp  }  column "user_id" {    null = false    type = integer  }  primary_key {    columns = [column.id]  }  foreign_key "authors_fk" {    columns     = [column.user_id]    ref_columns = [table.users.column.id]  }}function "update_latest_post" {  schema = schema.public  lang   = PLpgSQL  return = trigger  as     = <<-SQL    BEGIN    UPDATE "public"."users"    SET "latest_post_ts" = (      SELECT MAX("created_at")      FROM "public"."posts"      WHERE "user_id" = NEW."user_id"    )    WHERE "id" = NEW."user_id";    RETURN NEW;    END;  SQL}trigger "update_latest_post_trigger" {  on = table.posts  after {    insert = true  }  for = ROW  execute {    function = function.update_latest_post  }}
```
schema.sql
```codeBlockLines_AdAo
-- Add new schema named "public"CREATE SCHEMA IF NOT EXISTS "public";-- Create "users" tableCREATE TABLE "public"."users" ("id" integer NOT NULL, "email" character varying(255) NOT NULL, "latest_post_ts" TIMESTAMP, PRIMARY KEY ("id"));-- Create "posts" tableCREATE TABLE "public"."posts" ("id" integer NOT NULL, "title" character varying(100) NOT NULL, "created_at" TIMESTAMP NOT NULL, "user_id" integer NOT NULL, PRIMARY KEY ("id"), CONSTRAINT "authors_fk" FOREIGN KEY ("user_id") REFERENCES "public"."users" ("id") ON UPDATE NO ACTION ON DELETE NO ACTION);-- Create the trigger functionCREATE OR REPLACE FUNCTION update_latest_post()RETURNS TRIGGER AS $$BEGINUPDATE "public"."users"SET "latest_post_ts" = (    SELECT MAX("created_at")    FROM "public"."posts"    WHERE "user_id" = NEW."user_id")WHERE "id" = NEW."user_id";RETURN NEW;END;$$ LANGUAGE plpgsql;-- Create the triggerCREATE TRIGGER update_latest_post_triggerAFTER INSERT ON "public"."posts"FOR EACH ROWEXECUTE FUNCTION update_latest_post();
```
Run the `migrate diff` command once more to generate another migration:
```codeBlockLines_AdAo
atlas migrate diff --env dev
```
The following migration should be created:

20240807192934.sql
```codeBlockLines_AdAo
-- Create "update_latest_post" functionCREATE FUNCTION "update_latest_post" () RETURNS trigger LANGUAGE plpgsql AS $$BEGINUPDATE "public"."users"SET "latest_post_ts" = (  SELECT MAX("created_at")  FROM "public"."posts"  WHERE "user_id" = NEW."user_id")WHERE "id" = NEW."user_id";RETURN NEW;END;$$;-- Create trigger "update_latest_post_trigger"CREATE TRIGGER "update_latest_post_trigger" AFTER INSERT ON "posts" FOR EACH ROW EXECUTE FUNCTION "update_latest_post"();-- Modify "users" tableALTER TABLE "users" ADD COLUMN "latest_post_ts" timestamp NULL;
```

## Back-filling Data[​](#back-filling-data "Direct link to Back-filling Data")


The trigger we created will automatically update the `latest_post_ts` column in the `users` table whenever a new post is added. However, we need to back-fill the existing data in the `posts` table to ensure that the `latest_post_ts` column is accurate.

To do this, let's edit the `20240730073842.sql` migration file to include a query that will update the `latest_post_ts` column for each user. Add the following SQL statement to the migration file:

20240730073842.sql
```codeBlockLines_AdAo
-- .. Redacted for brevity-- Back-fill the latest post timestamp for each userUPDATE "users"SET "latest_post_ts" = (  SELECT MAX("created_at")  FROM "posts"  WHERE "user_id" = "users"."id");
```
After changing the contents of a migration file we need to recalculate the `atlas.sum` file to ensure directory integrity. Run the following command:
```codeBlockLines_AdAo
atlas migrate hash --env dev
```

## Testing Migrations[​](#testing-migrations "Direct link to Testing Migrations")


Rewriting data on a production database can be risky (and scary) if not done correctly. Let's now write a test to ensure that our migration will run smoothly and leave our application in a consistent state.

To do so, our test will have the following logic:

1.  Migrate to the first version, before creating the trigger, modifying the `users` table and back-filling the data.
2.  Seed the `users` and `posts` tables with data.
3.  Run our new migration file, which will create the trigger and back-fill the `latest_post_ts` column.
4.  Verify that the `latest_post_ts` column is correctly populated.

migrate.test.hcl
```codeBlockLines_AdAo
test "migrate" "check_latest_post" {  migrate {    to = "20240807192632"  }  exec {    sql = <<-SQL      INSERT INTO users (id, email) VALUES (1, 'user1@example.com'), (2, 'user2@example.com');      INSERT INTO posts (id, title, created_at, user_id) VALUES (1, 'My First Post', '2024-01-23 00:51:54', 1), (2, 'Another Interesting Post', '2024-02-24 02:14:09', 2);    SQL  }  migrate {    to = "20240807192934"  }  exec {    sql = "select * from users"    format = table    output = <<TAB id |       email       |   latest_post_ts----+-------------------+---------------------1  | user1@example.com | 2024-01-23 00:51:542  | user2@example.com | 2024-02-24 02:14:09TAB  }  log {    message = "Data migrated successfully"  }}
```
Let's break down the test:

1.  We define a migration test named `check_latest_post`.
2.  We migrate to the first version of the schema, which does not include the trigger and the `latest_post_ts` column.
3.  We insert data into the `users` and `posts` tables. At this point, the `latest_post_ts` column does not exist.
4.  We migrate to the second version of the schema, which includes the trigger and the `latest_post_ts` column as well as our back-fill.
5.  We assert that the `latest_post_ts` column is correctly populated with the latest post timestamp for each user.

To run this test, we will run the [`migrate test`](/testing/migrate#migrate-command) command:
```codeBlockLines_AdAo
atlas migrate test --env dev
```
The output should be similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: check_latest_post (50ms)    migrate.test.hcl:24: Data migrated successfullyPASS
```
Great! Our test passed, and we can now confidently deploy our migration to production.

## Integrating with CI[​](#integrating-with-ci "Direct link to Integrating with CI")


Writing and running tests locally is a great start, but it's equally important to run these tests in a CI/CD pipeline. Atlas provides out-of-the-box integrations for running `migrate test` in popular CI/CD platforms such as [GitHub Actions](/integrations/github-actions#arigaatlas-actionmigratetest) and [CircleCI](/integrations/circleci-orbs#atlas-orbmigrate_test). However, you can easily integrate migration testing into any CI/CD platform by running the `atlas migrate test` command.

## Wrapping Up[​](#wrapping-up "Direct link to Wrapping Up")


In this guide we learned how to use the [`atlas migrate test`](/testing/migrate#migrate-command) command to test our migration files before deployment.

*   [Project Setup](#project-setup)
    *   [Config File](#config-file)
    *   [Schema File](#schema-file)
    *   [Generating a Migration](#generating-a-migration)
    *   [Expanding Business Logic](#expanding-business-logic)
*   [Back-filling Data](#back-filling-data)
*   [Testing Migrations](#testing-migrations)
*   [Integrating with CI](#integrating-with-ci)
*   [Wrapping Up](#wrapping-up)