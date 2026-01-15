Testing Postgres Domains | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

With Atlas, you can test and validate PostgreSQL domain types and constraints alongside every other schema element. his helps you catch validation issues and data logic bugs before they reach production and ensures that your custom types behave as expected across environments.

In this guide, you’ll learn how to use Atlas's [`schema test`](/testing/schema) command to test [Postgres Domain Types](https://www.postgresql.org/docs/current/domains.html).

## Database Domains[​](#database-domains "Direct link to Database Domains")


A `domain` is a user-defined data type that is based on an existing data type but with optional constraints and default values.

[Atlas Pro Feature](/features#pro)

Domains are currently available only to [Atlas Pro users](/features#pro). To use this feature, run:
```codeBlockLines_AdAo
atlas login
```

## Project Setup[​](#project-setup "Direct link to Project Setup")


### Schema File[​](#schema-file "Direct link to Schema File")


For this example, let's assume we have the following schema, including a domain:

schema.hcl
```codeBlockLines_AdAo
schema "public" {}table "users" {  schema = schema.public  column "id" {    type = int  }  column "name" {    type = text  }  column "zip" {    type = domain.us_postal_code  }  primary_key {    columns = [column.id]  }}domain "us_postal_code" {  schema = schema.public  type   = text  null   = true  check "us_postal_code_check" {    expr = "((VALUE ~ '^\\d{5}$'::text) OR (VALUE ~ '^\\d{5}-\\d{4}$'::text))"  }}
```
The schema has a simple `users` table, with `id`, `name`, and `zip` as columns. The domain, `us_postal_code`, extends the type `text`, ensuring that the data entered conforms to the standard formats used for U.S. postal codes, such as "12345" or "12345-6789".

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


Let's start off with a simple test that will check two use-cases:

1.  A valid US Postal Code.
2.  And invalid US Postal Code.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "postal" {  parallel = true  exec {    sql = "select '12345'::us_postal_code"  }  catch {    sql = "select 'hello'::us_postal_code"  }}
```
This test uses the [`catch`](/testing/schema#catch-command) command which expects the SQL statement to fail. In this case, we expect "hello" to fail as a valid ZIP code.

info

To learn more about how to write tests, view the [testing docs](/testing/schema).

To run this test, run:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should be similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: domains (3ms)PASS
```

### Table Driven Test[​](#table-driven-test "Direct link to Table Driven Test")


Another alternative is to write a [table driven test](/testing/schema#table-driven-tests). This test uses the `for_each` meta-argument, which accepts a map or a set of values and is used to generate a test case for each item in the set or map.

Following similar logic to the example above, we will create _two_ table driven tests to check each case:

1.  A valid US Postal Code.
2.  And invalid US Postal Code.

schema.test.hcl
```codeBlockLines_AdAo
test "schema" "us_postal_code_check_valid" {  parallel = true  for_each = [    {input = "12345", expected = "valid"},    {input = "12345-6789", expected = "valid"},  ]  log {    message = "Testing postal code: ${each.value.input} -> Expected: ${each.value.expected}"  }  exec {    sql = "SELECT '${each.value.input}'::us_postal_code"  }}test "schema" "us_postal_code_check_invalid" {  parallel = true  for_each = [    {input = "hello", expected = "invalid"},    {input = "123", expected = "invalid"},  ]  log {    message = "Testing postal code: ${each.value.input} -> Expected: ${each.value.expected}"  }  catch {    sql = "SELECT '${each.value.input}'::us_postal_code"  }}
```
In the first test, we check postal codes that are supposed to be valid. In the second test, we use the [`catch`](/testing/schema#catch-command) command, expecting the input to be incorrect.

To run the tests, run:
```codeBlockLines_AdAo
atlas schema test --env dev
```
The output should be similar to:

Test Output
```codeBlockLines_AdAo
-- PASS: us_postal_code_check_valid/1 (4ms)    schema.test.hcl:7: Testing postal code: 12345 -> Expected: valid-- PASS: us_postal_code_check_valid/2 (10ms)    schema.test.hcl:7: Testing postal code: 12345-6789 -> Expected: valid-- PASS: us_postal_code_check_invalid/2 (12ms)    schema.test.hcl:25: Testing postal code: 123 -> Expected: invalid-- PASS: us_postal_code_check_invalid/1 (12ms)    schema.test.hcl:25: Testing postal code: hello -> Expected: invalidPASS
```
*   [Database Domains](#database-domains)
*   [Project Setup](#project-setup)
    *   [Schema File](#schema-file)
    *   [Config File](#config-file)
*   [Writing Tests](#writing-tests)
    *   [Simple Test](#simple-test)
    *   [Table Driven Test](#table-driven-test)