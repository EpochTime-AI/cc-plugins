CI/CD for Databases on GitLab (Declarative) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

GitLab is a popular, open-source alternative to GitHub. In addition to a self-hosted version, GitLab also offers a hosted version at [gitlab.com](https://gitlab.com). Similar to GitHub, GitLab offers users storage for Git repositories, issue tracking, and CI/CD pipelines.

In this guide we will demonstrate how to use [GitLab CI](https://docs.gitlab.com/ee/ci/) and Atlas to setup CI pipelines for your database schema changes using the **declarative migrations** workflow.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


### Installing Atlas[​](#installing-atlas "Direct link to Installing Atlas")


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

Installation instructions can be found [here](/docs#installation).

After installing Atlas locally, you will need to log in to your organization. You can do this by running the following command:
```codeBlockLines_AdAo
atlas login
```

### Creating a bot token[​](#creating-a-bot-token "Direct link to Creating a bot token")


In order to report the results of your CI runs to Atlas Cloud, you will need to create a bot token for Atlas Cloud to use.

Follow [these instructions](/cloud/bots) to create a token and copy it.

Next, in your Gitlab project go to **Settings** -> **CI/CD** -> **Variables** and create a new variable called `ATLAS_CLOUD_TOKEN`. Paste your token in the value field.

info

Make sure the variables are exported (the "Protect variable" checkbox is unchecked), so that they are available to all branches.

### Creating a variable for your database URL[​](#creating-a-variable-for-your-database-url "Direct link to Creating a variable for your database URL")


To avoid having plain-text database URLs which may contain sensitive information in your configuration files, create another variable named `DB_URL` and populate it with the URL (connection string) of your database.

To learn more about formatting URLs for different databases, see the [URL documentation](/concepts/url).

### Creating a Gitlab access token (optional)[​](#creating-a-gitlab-access-token-optional "Direct link to Creating a Gitlab access token (optional)")


Atlas will need permissions to comment lint reports on merge requests. To enable it, in your Gitlab project go to **Settings** -> **Access Tokens**. Create a new token. The role field should be set to "Reporter" or higher, and the "API" checkbox should be checked.

Copy the token, and then go to **Settings** -> **CI/CD** -> **Variables** and create a new variable called `GITLAB_TOKEN`. Paste the token in the value field.

## Declarative Migrations Workflow[​](#declarative-migrations-workflow "Direct link to Declarative Migrations Workflow")


In the [declarative workflow](/declarative/apply), developers provide the desired state of the database, as code. Atlas can read database schemas from various formats such as [plain SQL](/atlas-schema/sql), [Atlas HCL](/atlas-schema/hcl), [ORM models](/atlas-schema/external), and even another live database. Atlas then connects to the target database and calculates the diff between the current state and the desired state. It then generates a migration plan to bring the database to the desired state.

In this guide, we will use the SQL schema format.

### Our goal[​](#our-goal "Direct link to Our goal")


When a merge request containing changes to the schema, we want Atlas to:

*   Compare the current state (your database) with the new desired state.
*   Create a migration plan show it to the user for approval.
*   Mark the plan as approved when the merge request is approved and merged.
*   During deployment, use the approved plan to apply the changes to the database.

### Creating a simple SQL schema[​](#creating-a-simple-sql-schema "Direct link to Creating a simple SQL schema")


Create a file named `schema.sql` and fill it with the following content:
```codeBlockLines_AdAo
-- create table "users"CREATE TABLE users(	id int NOT NULL,	name varchar(100) NULL,	PRIMARY KEY(id));-- create table "blog_posts"CREATE TABLE blog_posts(	id int NOT NULL,	title varchar(100) NULL,	body text NULL,	author_id int NULL,	PRIMARY KEY(id),	CONSTRAINT author_fk FOREIGN KEY(author_id) REFERENCES users(id));
```
Then, create a configuration file for Atlas named `atlas.hcl` as follows:

atlas.hcl
```codeBlockLines_AdAo
env "gitlab" {    url = getenv("DB_URL")    schema {        src = "file://schema.sql"        repo {            name = "app"        }    }}
```

### Pushing the schema to Atlas Cloud[​](#pushing-the-schema-to-atlas-cloud "Direct link to Pushing the schema to Atlas Cloud")


To push our initial schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud, run the following command:

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://postgres/15/dev?search_path=public" \  --env gitlab
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://mysql/8/dev" \  --env gitlab
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://mariadb/latest/dev" \  --env gitlab
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "sqlite://dev?mode=memory" \  --env gitlab
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://sqlserver/2022-latest" \  --env gitlab
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://clickhouse/23.11" \  --env gitlab
```

### Setting up GitLab CI[​](#setting-up-gitlab-ci "Direct link to Setting up GitLab CI")


Create a `.gitlab-ci.yml` file with the following pipelines, based on the type of your database.

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse

.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - postgres:latestvariables:  POSTGRES_DB: dev  POSTGRES_USER: user  POSTGRES_PASSWORD: passstages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - mysql:latestvariables:  MYSQL_ROOT_PASSWORD: pass  MYSQL_DATABASE: devstages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: "mysql://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: "mysql://root:pass@mysql/dev"      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: "mysql://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: "mysql://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - mariadb:latestvariables:  MYSQL_ROOT_PASSWORD: pass  MYSQL_DATABASE: devstages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: "mariadb://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: "mariadb://root:pass@mysql/dev"      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: "mariadb://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: "mariadb://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:lateststages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: "sqlite://db?mode=memory"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: "sqlite://db?mode=memory"      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: "sqlite://db?mode=memory"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: "sqlite://db?mode=memory"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - name: mcr.microsoft.com/mssql/server:2022-latest    alias: sqlservervariables:    ACCEPT_EULA: Y    MSSQL_PID: Developer    MSSQL_SA_PASSWORD: P@ssw0rd0995stages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - name: clickhouse/clickhouse-server:23.10    alias: clickhousevariables:  CLICKHOUSE_DB: dev  CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1  CLICKHOUSE_PASSWORD: pass  CLICKHOUSE_USER: rootstages:  - plan  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan@~latest    inputs:      stage: plan      env: gitlab      dev-url: clickhouse://root:pass@localhost:9000/dev      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-push@~latest    inputs:      stage: push      env: gitlab      dev-url: clickhouse://root:pass@localhost:9000/dev      latest: true      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-plan-approve@~latest    inputs:      stage: push      env: gitlab      dev-url: clickhouse://root:pass@localhost:9000/dev      atlas-cloud-token: $ATLAS_CLOUD_TOKEN  - component: $CI_SERVER_FQDN/arigaio/atlas/schema-apply@~latest    inputs:      stage: apply      env: gitlab      dev-url: clickhouse://root:pass@localhost:9000/dev      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
1.  When a new merge request is opened, the `schema-plan` component will check if the desired state of the schema was changed. If it was, Atlas will generate a migration plan, lint it and post the report as a merge request comment.

2.  When the merge request is merged, two things happen: First, the updated schema is pushed to the schema registry by the `schema-push` component. Second, the plan that was created in the merege request will be approved.

3.  The `schema-apply` component will then be used to apply the new state of the schema to the database, using the plan that was just approved.

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's see our CI/CD pipeline in action!

#### Step 1: make a schema change[​](#step-1-make-a-schema-change "Direct link to Step 1: make a schema change")


Let's add the "address" column to the users table:

schema.sql
```codeBlockLines_AdAo
-- create table "users"CREATE TABLE users(	id int NOT NULL,	name varchar(100) NULL,	address varchar(100) NULL,	PRIMARY KEY(id));-- create table "blog_posts"CREATE TABLE blog_posts(	id int NOT NULL,	title varchar(100) NULL,	body text NULL,	author_id int NULL,	PRIMARY KEY(id),	CONSTRAINT author_fk FOREIGN KEY(author_id) REFERENCES users(id));
```
Now, let's commit the change to a new branch, push it to GitLab and open a merge request. The `schema-plan` component will use Atlas to create a migration plan from the current state of the database to the new desired state:

![Screenshot of a GitLab merge request comment from Atlas, showing a successful linting report for a new database migration.](/u/gitlab/plan-lint.png)

There are two things to note:

*   The comment also includes instructions to edit the plan. This is usefull when the plan has lint issues (for example, dropping a column will raise a "desctructive changes" error).
*   The plan is created in a "pending" state, which means Atlas can't use it yet against the real database.

### Merging the changes[​](#merging-the-changes "Direct link to Merging the changes")


Let's hit the merge button to merge the changes with the main branch. A new pipeline will be fired, with 3 jobs: The `schema-plan-approve` job will approve the plan that was generated earlier, the `schema-push` job will sync the new desired state in the schema registry, And then the `schema-apply` job will deploy the changes to our database.

![Screenshot of a successful GitLab CI pipeline job, showing the 'schema-apply' step has completed.](/u/gitlab/schema-apply.png)

The last thing to do is to inspect our database to make sure the changes were applied correctly:

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "docker://postgres/15/dev?search_path=public"Schemas are synced, no changes to be made.
```
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "docker://mysql/8/dev"Schemas are synced, no changes to be made.
```
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "docker://mariadb/latest/dev"Schemas are synced, no changes to be made.
```
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "sqlite://dev?mode=memory"Schemas are synced, no changes to be made.
```
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "docker://sqlserver/2022-latest"Schemas are synced, no changes to be made.
```
```codeBlockLines_AdAo
$ atlas schema diff \  --from $DB_URL \  --to "file://schema.sql" \  --dev-url "docker://clickhouse/23.11"Schemas are synced, no changes to be made.
```

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use GitLab CI/CD with Atlas to set up a modern CI/CD pipeline for declarative schema migrations. Here's what we accomplished:

*   **Automated migration linting** on every pull request to catch issues early
*   **Centralized migration management** by pushing to Atlas Cloud's Schema Registry
*   **Automated deployments** to your target database when changes are merged

For more information on the declarative workflow, see the [Declarative Migrations documentation](/declarative/apply).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [migration testing](/testing/migrate) to validate your migrations
*   Read about [declarative migrations](/declarative/apply) as an alternative workflow
*   Check out the [GitLab CI/CD reference](/integrations/gitlab-ci-components) for all available actions

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token](#creating-a-bot-token)
    *   [Creating a variable for your database URL](#creating-a-variable-for-your-database-url)
    *   [Creating a Gitlab access token (optional)](#creating-a-gitlab-access-token-optional)
*   [Declarative Migrations Workflow](#declarative-migrations-workflow)
    *   [Our goal](#our-goal)
    *   [Creating a simple SQL schema](#creating-a-simple-sql-schema)
    *   [Pushing the schema to Atlas Cloud](#pushing-the-schema-to-atlas-cloud)
    *   [Setting up GitLab CI](#setting-up-gitlab-ci)
    *   [Testing our pipeline](#testing-our-pipeline)
    *   [Merging the changes](#merging-the-changes)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)