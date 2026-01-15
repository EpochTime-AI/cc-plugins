Testing Database Functions | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas allows you to bring your database functions under version control and review. By writing tests for your functions, you can ensure that your business logic behaves as expected and that future changes don't accidentally break important assumptions in your application. In this guide, you'll learn how to use Atlas's `schema test` command to validate your function logic as part of your schema testing workflows.

## Database Functions[​](#database-functions "Direct link to Database Functions")


Functions are predefined operations stored in a database that can be invoked to perform calculations, manipulate data, or execute tasks.

[Atlas Pro Feature](/features#pro)

Functions are currently available only to [Atlas Pro users](/features#pro). To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Project Setup[​](#project-setup "Direct link to Project Setup")


### Schema File[​](#schema-file "Direct link to Schema File")


For this example, let's assume we have the following schema, including a function:

schema.hcl
```codeBlockLines_AdAo
schema "public" {}table "users" {  schema = schema.public  column "id" {    type = int  }  column "name" {    type = text  }  primary_key {    columns = [      column.id    ]  }}table "transactions" {  schema = schema.public  column "id" {    type = int  }  column "user_id" {    type = int  }  column "amount" {    type = decimal  }  column "is_income" {    type = boolean    as {      expr = "positive(amount)"    }  }  primary_key {    columns = [column.id]  }  foreign_key "user_fk" {    columns = [column.user_id]    ref_columns = [table.users.column.id]    on_delete = CASCADE    on_update = NO_ACTION  }}function "positive" {  schema = schema.public  lang   = SQL  arg "v" {    type = decimal  }  return = boolean  as     = "SELECT v > 0"}
```
In the schema above we have a simple `users` table and a `transactions` table. In `transactions`, the `is_income` column checks if the `amount` is positive by calling the `positive` function.

Note that `is_income` is a [generated column](/atlas-schema/hcl#generated-columns), meaning its value is computed using other columns or deterministic expressions (in this case, via a function).

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


Let's start off with a simple test that will check that the function correctly recognizes positive numbers.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "positive_func" {  parallel = true  assert {    sql = "SELECT positive(1)"  }  log {    message = "First assertion passed"  }  assert {    sql = <<SQLSELECT NOT positive(0);SELECT NOT positive(-1);SQL  } log {    message = "Second assertion passed"  }}
```
The test first checks if positive(1) returns TRUE, and then verifies in the second assertion that positive(0) and positive(-1) return false.

Run the test by running:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should look similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: positive_func (2ms)    schema.test.hcl:74: First assertion passed    schema.test.hcl:83: Second assertion passedPASS
```

### Table Driven Test[​](#table-driven-test "Direct link to Table Driven Test")


Another alternative is to write a [table driven test](/testing/schema#table-driven-tests). This test uses the `for_each` meta-argument, which accepts a map or a set of values and is used to generate a test case for each item in the set or map.

Following similar logic to the test above, we will check the function for the integers: 0, 1, and -1.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "positive_func" {  parallel = true  for_each = [    {input: 1, expected: "t"},    {input: 0, expected: "f"},    {input: -1, expected: "f"},  ]  exec {    sql = "SELECT positive(${each.value.input})"    output = each.value.expected  }}
```
Run the test by running:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should look similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: positive_func/2 (5ms)-- PASS: positive_func/3 (10ms)-- PASS: positive_func/1 (10ms)PASS
```
*   [Database Functions](#database-functions)
*   [Project Setup](#project-setup)
    *   [Schema File](#schema-file)
    *   [Config File](#config-file)
*   [Writing Tests](#writing-tests)
    *   [Simple Test](#simple-test)
    *   [Table Driven Test](#table-driven-test)