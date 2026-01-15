Working with template directories | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas supports working with dynamic template-based directories, where their content is computed based on the data variables injected at runtime. These directories adopt the [Go-templates](https://pkg.go.dev/text/template#hdr-Actions) format, the very same format used by popular CLIs such as `kubectl`, `docker` or `helm`.

To create a template directory, you first need to create an Atlas configuration file (`atlas.hcl`) and define the `template_dir` data source there:

atlas.hcl
```codeBlockLines_AdAo
data "template_dir" "migrations" {  path = "migrations"  vars = {}}env "dev" {  migration {    dir = data.template_dir.migrations.url  }}
```
The `path` defines a path to a local directory, and `vars` defines a map of variables that will be used to interpolate the templates in the directory.

### Basic Example[​](#basic-example "Direct link to Basic Example")


We start our guide with a simple MySQL-based example where migration files are manually written and the auto-increment initial value is configuration based. Let's run `atlas migrate new` with the `--edit` flag and paste the following statement:
```codeBlockLines_AdAo
-- Create "users" table.CREATE TABLE `users` (  `id` bigint NOT NULL AUTO_INCREMENT,  `role` enum('user', 'admin') NOT NULL,  `data` json,  PRIMARY KEY (`id`)) AUTO_INCREMENT={{ .users_initial_id }};
```
After creating our first migration file, the `users_initial_id` variable should be defined in `atlas.hcl`. Otherwise, Atlas will fail to interpolate the template.

atlas.hcl
```codeBlockLines_AdAo
data "template_dir" "migrations" {  path = "migrations"  vars = {    users_initial_id = 1000  }}env "dev" {  dev = "docker://mysql/8/dev"  migration {    dir = data.template_dir.migrations.url  }}
```
In order to test our migration directory, we can run `atlas migrate apply` on a temporary MySQL container that Atlas will spin up and tear down automatically for us:
```codeBlockLines_AdAo
atlas migrate apply \  --env dev \  --url docker://mysql/8/dev
```
Example output

Output
```codeBlockLines_AdAo
Migrating to version 20230719093802 (1 migrations in total):  -- migrating version 20230719093802    -> CREATE TABLE `users` (  `id` bigint NOT NULL AUTO_INCREMENT,  `role` enum('user', 'admin') NOT NULL,  `data` json,  PRIMARY KEY (`id`)) AUTO_INCREMENT=1000;  -- ok (30.953207ms)  -------------------------  -- 74.773738ms  -- 1 migrations   -- 1 sql statements
```

### Inject Data Variables From Command Line[​](#inject-data-variables-from-command-line "Direct link to Inject Data Variables From Command Line")


Variables are not always static, and there are times when we need to inject them from the command line. The Atlas configuration file supports this injection using the `--var` flag. Let's modify our `atlas.hcl` file such that the value of the `users_initial_id` variable isn't statically defined and must be provided by the user executing the CLI:

atlas.hcl
```codeBlockLines_AdAo
variable "users_initial_id" {  type = number}data "template_dir" "migrations" {  path = "migrations"  vars = {    users_initial_id = var.users_initial_id  }}env "dev" {  dev = "docker://mysql/8/dev"  migration {    dir = data.template_dir.migrations.url  }}
```
Trying to execute `atlas migrate apply` without providing the `users_initial_id` variable, will result in an error:
```codeBlockLines_AdAo
Error: missing value for required variable "users_initial_id"
```
Let's run it the right way and provide the variable from the command line:
```codeBlockLines_AdAo
atlas migrate apply \  --env dev \  --url docker://mysql/8/dev \  --var users_initial_id=1000
```
Example output

Output
```codeBlockLines_AdAo
Migrating to version 20230719093802 (1 migrations in total):  -- migrating version 20230719093802    -> CREATE TABLE `users` (  `id` bigint NOT NULL AUTO_INCREMENT,  `role` enum('user', 'admin') NOT NULL,  `data` json,  PRIMARY KEY (`id`)) AUTO_INCREMENT=1000;  -- ok (30.953207ms)  -------------------------  -- 74.773738ms  -- 1 migrations   -- 1 sql statements
```

### Read Data Variables From File[​](#read-data-variables-from-file "Direct link to Read Data Variables From File")


Let's add a bit more complexity to our example by inserting seed data to the `users` table. But, to keep our configuration file tidy, we'll keep the seed data in a different file (`seed_data.json`) and read it from there.

First, we'll create a new migration file by running `atlas migrate new seed_users --edit` and paste the following statement:
```codeBlockLines_AdAo
{{ range $line := .seed_users }}  INSERT INTO `users` (`role`, `data`) VALUES ('user', '{{ $line }}');{{ end }}
```
The file above expects a data variable named `seed_users` of type `[]string`. It then loops over this variable and `INSERT`s a record into the `users` table for each JSON line.

For the sake of this example, let's define an example `seed_users.json` file and update the `atlas.hcl` file to inject the data variable from its content:

seed\_users.json
```codeBlockLines_AdAo
{"name": "Ariel"}{"name": "Rotem"}
```
atlas.hcl
```codeBlockLines_AdAo
variable "users_initial_id" {  type = number}locals {  # The path is relative to the `atlas.hcl` file.  seed_users = split("\n", file("seed_users.json"))}data "template_dir" "migrations" {  path = "migrations"  vars = {    seed_users       = local.seed_users    users_initial_id = var.users_initial_id  }}env "dev" {  dev = "docker://mysql/8/dev"  migration {    dir = data.template_dir.migrations.url  }}
```
To check that our data interpolation works as expected, let's run `atlas migrate apply` on a temporary MySQL container that Atlas will spin up and tear down automatically for us:
```codeBlockLines_AdAo
atlas migrate apply \                         --env dev \  --url docker://mysql/8/dev \  --var users_initial_id=1000
```
Example output

Output
```codeBlockLines_AdAo
Migrating to version 20230719102332 (2 migrations in total):  -- migrating version 20230719093802    -> CREATE TABLE `users` (  `id` bigint NOT NULL AUTO_INCREMENT,  `role` enum('user', 'admin') NOT NULL,  `data` json,  PRIMARY KEY (`id`)) AUTO_INCREMENT=1000;  -- ok (38.380244ms)  -- migrating version 20230719102332    -> INSERT INTO `users` (`role`, `data`) VALUES ('user', '{"name": "Ariel"}');    -> INSERT INTO `users` (`role`, `data`) VALUES ('user', '{"name": "Rotem"}');  -- ok (13.313962ms)  -------------------------  -- 95.387439ms  -- 2 migrations   -- 3 sql statements
```

#### Working with JSON[​](#working-with-json "Direct link to Working with JSON")


Most modern SQL databases support JSON casting and extraction, allowing developers to work with JSON data types directly. Atlas extends this capability by providing a convenient way to work with JSON data in migration directories using the `jsondecode` function, which casts a JSON string into a native object. This is particularly useful when inserting structured JSON data into a table or extracting fields from a JSON array.

For example, if we want to import entire rows from a JSON array, we can use `jsondecode` to convert the content into an iterable list of objects. First, we modify our `seed_users.json` file to include more fields and ensure it is a valid JSON array:

seed\_users.json
```codeBlockLines_AdAo
[  {"name": "Hucci", "age": 30},  {"name": "Lorenzo", "age": 25},  {"name": "Michele", "age": 28}]
```
Next, we update our `atlas.hcl` file to read this JSON array and assign it to a local variable:

atlas.hcl
```codeBlockLines_AdAo
locals {  seed_users = file("seed_users.json")}data "template_dir" "migrations" {  path = "migrations"  vars = {    seed_users = local.seed_users  }}env "dev" {  dev = "docker://mysql/8/dev"  migration {    dir = data.template_dir.migrations.url  }}
```
Finally, we modify the SQL template in our migration file to decode the JSON and insert each user's data:
```codeBlockLines_AdAo
{{ range $user := jsondecode .seed_users }}  INSERT INTO `users` (`role`, `name`, `age`) VALUES ('user', '{{ $user.name }}', '{{ $user.age }}');{{ end }}
```
Note: Ensure that the `users` table schema defines the `name` and `age` columns before running this migration.

### Running `migrate diff` on template directories[​](#running-migrate-diff-on-template-directories "Direct link to running-migrate-diff-on-template-directories")


When running the `atlas migrate diff` command on a template directory, we want to ensure that the data variables defined in our `atlas.hcl` are shared between the desired state (e.g., HCL or SQL schema) and the current state of the migration directory, to get an accurate SQL script that moves our database from its previous state to the new one.

Let's demonstrate this using an HCL schema that describes our desired schema and expects one variable: `users_initial_id`.

schema.hcl
```codeBlockLines_AdAo
variable "users_initial_id" {  type = number}table "users" {  schema = schema.public  column "id" {    type = bigint  }  column "role" {    type = enum("user", "admin")  }  column "data" {    type = json    null = true  }  primary_key {    columns = [column.id]  }  auto_increment = var.users_initial_id}schema "public"{}
```
Then, we update our `atlas.hcl` configuration to inject the data variable to this schema file and then use it as our desired state:
```codeBlockLines_AdAo
variable "users_initial_id" {  type = number}locals {  seed_users = split("\n", file("seed_users.json"))}data "template_dir" "migrations" {  path = "migrations"  vars = {    seed_users       = local.seed_users    users_initial_id = var.users_initial_id  }}data "hcl_schema" "app" {  path = "schema.hcl"  vars = {    users_initial_id = var.users_initial_id  }}env "dev" {  src = data.hcl_schema.app.url  dev = "docker://mysql/8/dev"  migration {    dir = data.template_dir.migrations.url  }}
```
To test that our data interpolation works as expected, let's run `atlas migrate diff` and ensure the HCL schema and the migration directory are in sync:
```codeBlockLines_AdAo
atlas migrate diff \  --env dev \  --var users_initial_id=1000
```
```codeBlockLines_AdAo
The migration directory is synced with the desired state, no changes to be made
```
Then, let's change our `data` column to be `NOT NULL` by updating the `schema.hcl` file and run `atlas migrate diff`:

schema.hcl
```codeBlockLines_AdAo
  column "data" {    type = json-   null = true+   null = false  }
```
```codeBlockLines_AdAo
atlas migrate diff modify_user_data \  --env dev \  --var users_initial_id=1000
```
After checking our migration directory, we can see that Atlas has generated a new migration file that modifies the `data` column to be `NOT NULL`, while leaving the template files untouched:

migrations/20230720074923\_modify\_user\_data.sql
```codeBlockLines_AdAo
-- Modify "users" tableALTER TABLE `users` MODIFY COLUMN `data` json NOT NULL;
```

### Conclusion[​](#conclusion "Direct link to Conclusion")


In this example, we've seen how to use the `template_dir` data source to create a migration directory whose content is dynamically computed at runtime, based on the data variables defined in the `atlas.hcl` file. We've also seen how the data variables can be injected from various sources, such as JSON files or CLI flags. Lastly, we've showed how data variables can be shared between template directories and HCL schemas to ensure commands like `atlas migrate diff` can be utilized to generate migration plan automatically for us.

Have questions? Feedback? Find our team [on our Discord server](https://discord.gg/zZ6sWVg6NT).

*   [Basic Example](#basic-example)
*   [Inject Data Variables From Command Line](#inject-data-variables-from-command-line)
*   [Read Data Variables From File](#read-data-variables-from-file)
*   [Running `migrate diff` on template directories](#running-migrate-diff-on-template-directories)
*   [Conclusion](#conclusion)