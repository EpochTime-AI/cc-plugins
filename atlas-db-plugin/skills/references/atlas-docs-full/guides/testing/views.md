Testing Database Views | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas lets you test and validate your database views as part of your schema-as-code workflow. With Atlas, every change to a view can be versioned, tested, and automatically validated, reducing the risk of regressions and broken queries going unnoticed. This helps ensure your analytics, privacy boundaries, and business logic represented in views are always correct and reviewable at every stage of development.

In this guide we will learn how to use Atlas's [`schema test`](/testing/schema) command to test database views.

## Database Views[​](#database-views "Direct link to Database Views")


A `view` is a virtual table in the database, defined by a statement that queries rows from one or more existing tables or views.

[Atlas Pro Feature](/features#pro)

Views are currently available only to [Atlas Pro users](/features#pro). To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Project Setup[​](#project-setup "Direct link to Project Setup")


### Schema File[​](#schema-file "Direct link to Schema File")


For this example, let's assume we have the following schema, including a view:

schema.hcl
```codeBlockLines_AdAo
schema "public" {}table "users" {  schema = schema.public  column "id" {    type = int  }  column "name" {    type = text  }  column "ssn" {    type = varchar(9)  }}view "clean_users" {  schema = schema.public  column "id" {    type = int  }  column "name" {    type = text  }  as         = <<-SQL  SELECT u.id, u.name    FROM ${table.users.name} AS u  SQL  depends_on = [table.users]  comment    = "A view to select active users without sensitive data"}
```
The schema has a simple `users` table, with `id`, `name`, and `ssn` as columns. The view, `clean_users`, selects the `id` and `name` columns from `users`, ensuring we aren't selecting any sensitive data (SSNs).

### Config File[​](#config-file "Direct link to Config File")


Before we begin testing, create a [config file](/atlas-schema/projects#project-files) named `atlas.hcl`.

In this file we will create an environment, specify the source of our schema, and a URL for our [dev database](/concepts/dev-database).

We will also create a file named `schema.test.hcl` to write our tests, and add it to the `atlas.hcl` file in the test block.

atlas.hcl
```codeBlockLines_AdAo
env "dev" {  src = "file://schema.hcl"  dev = "docker://postgres/15/dev?search_path=public"  # Test configuration for local development.  test {    schema {      src = ["schema.test.hcl"]    }  }}
```

## Writing Tests[​](#writing-tests "Direct link to Writing Tests")


### Simple Test[​](#simple-test "Direct link to Simple Test")


Let's start off with a simple test that will do the following:

1.  Seed data into the `users` table.
2.  Run a query on the `clean_users` view.
3.  Ensure the data is retrieved as expected.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "view" {  # Seeding to test view.  exec {    sql = "INSERT INTO users (id, name) VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Charlie');"  }  log {    message = "Seeded the user's table"  }  # Expected exec to pass.  exec {    sql = <<SQL    SELECT id, name    FROM clean_users;    SQL  }  log {    message = "Tested the view"  }  # Validates data.  exec {    sql = "SELECT id, name FROM clean_users;"    format = table    output = <<TAB id |  name----+--------- 1  | Alice 2  | Bob 3  | CharlieTAB  }  log {    message = "Table is as expected"  }}
```
info

To learn more about how to write tests, view the [testing docs](/testing/schema).

To run this test, run:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should be similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: view (1ms)    schema.test.hcl:6: Seeded the user's table    schema.test.hcl:17: Tested the view    schema.test.hcl:33: Table is as expectedPASS
```

### Table Driven Test[​](#table-driven-test "Direct link to Table Driven Test")


Another alternative is to write a [table driven test](/testing/schema#table-driven-tests). This test uses the `for_each` meta-argument, which accepts a map or a set of values and is used to generate a test case for each item in the set or map.

Following similar logic to the example above, our table driven test will:

1.  Seed data into the `users` table.
2.  Run a query on the `clean_users` view.
3.  Ensure the data is retrieved as expected.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "view" {  for_each = [    {id = 1, name = "Alice"},    {id = 2, name = "Bob"},    {id = 3, name = "Charlie"}  ]  # Seed the `users` table.  exec {    sql = "INSERT INTO users (id, name) VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Charlie');"  }  # Query the `clean_users` view.  exec {    sql = "SELECT id, name FROM clean_users WHERE id IN (1, 2, 3);"  }  # Check each ID returns the right user.  log {    message = "Testing ${each.value.id} -> ${each.value.name}"  }}
```
To run this test, run:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should be similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: view/1 (3ms)    schema.test.hcl:62: Testing 1 -> Alice-- PASS: view/2 (633µs)    schema.test.hcl:62: Testing 2 -> Bob-- PASS: view/3 (643µs)    schema.test.hcl:62: Testing 3 -> CharliePASS
```
*   [Database Views](#database-views)
*   [Project Setup](#project-setup)
    *   [Schema File](#schema-file)
    *   [Config File](#config-file)
*   [Writing Tests](#writing-tests)
    *   [Simple Test](#simple-test)
    *   [Table Driven Test](#table-driven-test)