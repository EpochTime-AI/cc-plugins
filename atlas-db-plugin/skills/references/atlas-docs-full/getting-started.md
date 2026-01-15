Quick Introduction | Atlas Docs

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas is a language-independent tool for managing and migrating database schemas using modern DevOps principles. It offers two workflows:

*   **Declarative**: Similar to Terraform, Atlas compares the current state of the database to the desired state, as defined in an [HCL](/atlas-schema/hcl), [SQL](/atlas-schema/sql), or [ORM](/atlas-schema/external) schema. Based on this comparison, it generates and executes a migration plan to transition the database to its desired state.

*   **Versioned**: Unlike other tools, Atlas automatically plans schema migrations for you. Users can describe their desired database schema in [HCL](/atlas-schema/hcl), [SQL](/atlas-schema/sql), or their chosen [ORM](/atlas-schema/external), and by utilizing Atlas, they can plan, lint, and apply the necessary migrations to the database.

### Installation[​](#installation "Direct link to Installation")


*   macOS + Linux
*   Homebrew
*   Docker
*   Windows
*   CI
*   Manual Installation

To download and install the latest release of the Atlas CLI, simply run the following in your terminal:
```codeBlockLines_AdAo
curl -sSf https://atlasgo.sh | sh
```
Get the latest release with [Homebrew](https://brew.sh/):
```codeBlockLines_AdAo
brew install ariga/tap/atlas
```
To pull the Atlas image and run it as a Docker container:
```codeBlockLines_AdAo
docker pull arigaio/atlasdocker run --rm arigaio/atlas --help
```
If the container needs access to the host network or a local directory, use the `--net=host` flag and mount the desired directory:
```codeBlockLines_AdAo
docker run --rm --net=host \  -v $(pwd)/migrations:/migrations \  arigaio/atlas migrate apply  --url "mysql://root:pass@:3306/test"
```
Download the [latest release](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe) and move the atlas binary to a file location on your system PATH.

**GitHub Actions**

Use the [setup-atlas](https://github.com/marketplace/actions/setup-atlas) action to install Atlas in your GitHub Actions workflow:
```codeBlockLines_AdAo
- uses: ariga/setup-atlas@v0  with:    cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}
```
**Other CI Platforms**

For other CI/CD platforms, use the installation script. See the [CI/CD integrations](/integrations#cicd-platforms) for more details.

If you want to manually install the Atlas CLI, pick one of the below builds suitable for your system.

### MacOS


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-darwin-amd64-latest) ([md5](https://release.ariga.io/atlas/atlas-darwin-amd64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-darwin-amd64-latest.sha256))

![Download icon](/icons-docs/download.svg)

[ARM 64](https://release.ariga.io/atlas/atlas-darwin-arm64-latest) ([md5](https://release.ariga.io/atlas/atlas-darwin-arm64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-darwin-arm64-latest.sha256))

### Linux


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-linux-amd64-latest) ([md5](https://release.ariga.io/atlas/atlas-linux-amd64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-linux-amd64-latest.sha256))

![Download icon](/icons-docs/download.svg)

[ARM 64](https://release.ariga.io/atlas/atlas-linux-arm64-latest) ([md5](https://release.ariga.io/atlas/atlas-linux-arm64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-linux-arm64-latest.sha256))

### Windows


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe) ([md5](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe.md5)/[sha256](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe.sha256))

The default binaries distributed in official releases are released under the [Atlas EULA](https://ariga.io/legal/atlas/eula). If you would like obtain a copy of Atlas Community Edition (under an Apache 2 license) follow the instructions [here](/community-edition).

### Start a local database container[​](#start-a-local-database-container "Direct link to Start a local database container")


For the purpose of this guide, we will start a local Docker container running MySQL.
```codeBlockLines_AdAo
docker run --rm -d --name atlas-demo -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=example mysql
```
For this example, we will start with a schema that represents a `users` table, in which each user has an ID and a name:
```codeBlockLines_AdAo
CREATE table users (  id int PRIMARY KEY,  name varchar(100));
```
To create the table above on our local database, we can run the following command:
```codeBlockLines_AdAo
docker exec atlas-demo mysql -ppass -e 'CREATE table example.users(id int PRIMARY KEY, name varchar(100))'
```

### Inspecting our database[​](#inspecting-our-database "Direct link to Inspecting our database")


The `atlas schema inspect` command supports reading the database description provided by a URL and outputting it in three different formats: [Atlas DDL](/atlas-schema/hcl) (default), SQL, and JSON. In this guide, we will demonstrate the flow using both the Atlas DDL and SQL formats, as the JSON format is often used for processing the output using `jq`.

*   Atlas DDL (HCL)
*   SQL

To inspect our locally-running MySQL instance, use the `-u` flag and write the output to a file named `schema.hcl`:
```codeBlockLines_AdAo
atlas schema inspect -u "mysql://root:pass@localhost:3306/example" > schema.hcl
```
Open the `schema.hcl` file to view the Atlas schema that describes our database.

schema.hcl
```codeBlockLines_AdAo
table "users" {  schema = schema.example  column "id" {    null = false    type = int  }  column "name" {    null = true    type = varchar(100)  }  primary_key {    columns = [column.id]  }}
```
This block represents a [table](/atlas-schema/hcl#table) resource with `id`, and `name` columns. The `schema` field references the `example` schema that is defined elsewhere in this document. In addition, the `primary_key` sub-block defines the `id` column as the primary key for the table. Atlas strives to mimic the syntax of the database that the user is working against. In this case, the type for the `id` column is `int`, and `varchar(100)` for the `name` column.

To inspect our locally-running MySQL instance, use the `-u` flag and write the output to a file named `schema.sql`:
```codeBlockLines_AdAo
atlas schema inspect -u "mysql://root:pass@localhost:3306/example" --format '{{ sql . }}' > schema.sql
```
Open the `schema.sql` file to view the inspected SQL schema that describes our database.

schema.sql
```codeBlockLines_AdAo
-- create "users" tableCREATE TABLE `users` (  `id` int NOT NULL,  `name` varchar(100) NULL,  PRIMARY KEY (`id`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```
Now, consider we want to add a `blog_posts` table and have our schema represent a simplified blogging system.

Loading ERD...

Let's add the following to our inspected schema, and use Atlas to plan and apply the changes to our database.

*   Atlas DDL (HCL)
*   SQL

Edit the `schema.hcl` file and add the following `table` block:

schema.hcl
```codeBlockLines_AdAo
table "blog_posts" {  schema = schema.example  column "id" {    null = false    type = int  }  column "title" {    null = true    type = varchar(100)  }  column "body" {    null = true    type = text  }  column "author_id" {    null = true    type = int  }  primary_key {    columns = [column.id]  }  foreign_key "author_fk" {    columns     = [column.author_id]    ref_columns = [table.users.column.id]  }}
```
In addition to the elements we saw in the `users` table, here we can find a [foreign key](/atlas-schema/hcl#foreign-key) block, declaring that the `author_id` column references the `id` column on the `users` table.

Edit the `schema.sql` file and add the following `CREATE TABLE` statement:

schema.sql
```codeBlockLines_AdAo
-- create "blog_posts" tableCREATE TABLE `blog_posts` (  `id` int NOT NULL,  `title` varchar(100) NULL,  `body` text NULL,  `author_id` int NULL,  PRIMARY KEY (`id`),  CONSTRAINT `author_fk` FOREIGN KEY (`author_id`) REFERENCES `example`.`users` (`id`));
```
Now, let's apply these changes by running a migration. In Atlas, migrations can be applied in two types of workflows: _declarative_ and _versioned_.

### Declarative Migrations[​](#declarative-migrations "Direct link to Declarative Migrations")


The declarative approach requires the user to define the _desired_ end schema, and Atlas provides a safe way to alter the database to get there. Let's see this in action.

Continuing the example, in order to apply the changes to our database we will run the `apply` command:

*   Atlas DDL (HCL)
*   SQL
```codeBlockLines_AdAo
atlas schema apply \  -u "mysql://root:pass@localhost:3306/example" \  --to file://schema.hcl
```
```codeBlockLines_AdAo
atlas schema apply \  -u "mysql://root:pass@localhost:3306/example" \  --to file://schema.sql \  --dev-url "docker://mysql/8/dev"
```
Atlas presents the plan it created by displaying the SQL statements. For example, for a MySQL database we will see the following:
```codeBlockLines_AdAo
Planning migration statements (2 in total):  -- create "users" table:    -> CREATE TABLE `users` (         `id` int NOT NULL,         `name` varchar(100) NULL,         PRIMARY KEY (`id`)       ) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;  -- create "blog_posts" table:    -> CREATE TABLE `blog_posts` (         `id` int NOT NULL,         `title` varchar(100) NULL,         `body` text NULL,         `author_id` int NULL,         PRIMARY KEY (`id`),         INDEX `author_fk` (`author_id`),         CONSTRAINT `author_fk` FOREIGN KEY (`author_id`) REFERENCES `users` (`id`) ON UPDATE NO ACTION ON DELETE NO ACTION       ) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-------------------------------------------? Approve or abort the plan:  ▸ Approve and apply    Abort
```
Apply the changes, and that's it! You have successfully run a declarative migration.

To ensure that the changes have been made to the schema, you can run the `inspect` command again. This time, we use the `--web`/`-w` flag to open the Atlas Web UI and view the schema.
```codeBlockLines_AdAo
atlas schema inspect \  -u "mysql://root:pass@localhost:3306/example" \  --web
```
note

If you are using an old version of Atlas, you may need to replace the `--web` flag with `--visualize`.

### Versioned Migrations[​](#versioned-migrations "Direct link to Versioned Migrations")


Alternatively, the versioned migration workflow, sometimes called "change-based migrations", allows each change to the database schema to be checked-in to source control and reviewed during code-review. Users can still benefit from Atlas intelligently planning migrations for them, however they are not automatically applied.

To start, we will calculate the difference between the _desired_ and _current_ state of the database by running the `atlas migrate diff` command.

To run this command, we need to provide the necessary parameters:

*   `--dir` the URL to the migration directory, by default it is `file://migrations`.
*   `--to` the URL of the desired state. A state can be specified using a database URL, HCL or SQL schema, or another migration directory.
*   `--dev-url` a URL to a [Dev Database](/concepts/dev-database) that will be used to compute the diff.

*   Atlas DDL (HCL)
*   SQL
```codeBlockLines_AdAo
atlas migrate diff create_blog_posts \  --dir "file://migrations" \  --to "file://schema.hcl" \  --dev-url "docker://mysql/8/dev"
```
```codeBlockLines_AdAo
atlas migrate diff create_blog_posts \  --dir "file://migrations" \  --to "file://schema.sql" \  --dev-url "docker://mysql/8/dev"
```
Run `ls migrations`, and you will notice that Atlas has created two files:

*   20220811074144\_create\_blog\_posts.sql
*   atlas.sum
```codeBlockLines_AdAo
-- create "blog_posts" tableCREATE TABLE `blog_posts` (  `id` int NOT NULL,  `title` varchar(100) NULL,  `body` text NULL,  `author_id` int NULL,  PRIMARY KEY (`id`),  INDEX `author_id` (`author_id`),  CONSTRAINT `author_fk` FOREIGN KEY (`author_id`) REFERENCES `users` (`id`))
```
In addition to the migration directory, Atlas maintains a file name `atlas.sum` which is used to ensure the integrity of the migration directory and force developers to deal with situations where migration order or contents was modified after the fact.
```codeBlockLines_AdAo
h1:t1fEP1rSsGf1gYrYCjsGyEyuM0cnhATlq93B7h8uXxY=20220811074144_create_blog_posts.sql h1:liZcCBbAn/HyBTqBAEVar9fJNKPTb2Eq+rEKZeCFC9M=
```
Now that we have our migration files ready, you can use the `migrate apply` command to apply the changes to the database. To learn more about this process, check out the [Versioned Migrations Quickstart Guide](/versioned/intro)

### Next Step: Setup CI/CD[​](#next-step-setup-cicd "Direct link to Next Step: Setup CI/CD")


After getting familiar with the basics of Atlas, the next step is to integrate it into your development workflow. [Sign up](https://auth.atlasgo.cloud/signup) for a free trial to unlock Pro features, then set up your CI/CD pipeline by running the following command, and proceeding with the steps below:

Open a free trial account
```codeBlockLines_AdAo
atlas login
```
*   1) Create a Repo
*   2) Setup CI/CD
*   3) PR Example

![Create a Repo](/assets/images/create-repo-5765b109ec65eede46e0c1e833d1d404.png)

![Setup CI/CD](/assets/images/setup-ci-cd-921caaa527fa6aa1db113df58df529f9.png)

![Setup CI/CD](/assets/images/pr-example-35502be8b4b46f5d214af8c39f59ca70.png)

### Conclusion[​](#conclusion "Direct link to Conclusion")


In this short tutorial we learned how to use Atlas to inspect databases, as well as use declarative and versioned migrations. Read more about the use-cases for the two approaches [here](/concepts/declarative-vs-versioned) to help you decide which workflow works best for you.

Need help getting started?

We have a super friendly [#getting-started](https://discord.gg/rChPrC6x) channel on our community chat on Discord.

For web-based, free, and fun (GIFs included) support:

[Join our Discord server](https://discord.gg/zZ6sWVg6NT)

*   [Installation](#installation)
*   [Start a local database container](#start-a-local-database-container)
*   [Inspecting our database](#inspecting-our-database)
*   [Declarative Migrations](#declarative-migrations)
*   [Versioned Migrations](#versioned-migrations)
*   [Next Step: Setup CI/CD](#next-step-setup-cicd)
*   [Conclusion](#conclusion)