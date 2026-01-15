CI/CD for Databases on GitLab (Versioned Migrations) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

GitLab is a popular, open-source alternative to GitHub. In addition to a self-hosted version, GitLab also offers a hosted version at [gitlab.com](https://gitlab.com). Similar to GitHub, GitLab offers users storage for Git repositories, issue tracking, and CI/CD pipelines.

In this guide we will demonstrate how to use [GitLab CI](https://docs.gitlab.com/ee/ci/) and Atlas to setup CI pipelines for your database schema changes using the **versioned migrations** workflow.

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

## Versioned Migrations Workflow[​](#versioned-migrations-workflow "Direct link to Versioned Migrations Workflow")


In the [versioned workflow](/versioned/intro), changes to the schema are represented by a _migration directory_ in your codebase. Each file in this directory represents a transition to a new version of the schema.

Based on our blueprint for [Modern CI/CD for Databases](/guides/modern-database-ci-cd), our pipeline will:

1.  [Lint](/versioned/lint) new migration files whenever a merge request (MR) is opened.
2.  [Push](/versioned/intro#pushing-migrations-to-the-schema-registry) the migration directory to the [Schema Registry](/cloud/features/registry) when changes are merged to the mainline branch.
3.  [Apply](/versioned/apply) new migrations to our database.

### Creating a simple migration directory[​](#creating-a-simple-migration-directory "Direct link to Creating a simple migration directory")


If you don't have a migration directory yet, create one by running the following commands:
```codeBlockLines_AdAo
atlas migrate new --edit
```
and paste the following in the editor:
```codeBlockLines_AdAo
-- create table "users"CREATE TABLE users(	id int NOT NULL,	name varchar(100) NULL,	PRIMARY KEY(id));-- create table "blog_posts"CREATE TABLE blog_posts(	id int NOT NULL,	title varchar(100) NULL,	body text NULL,	author_id int NULL,	PRIMARY KEY(id),	CONSTRAINT author_fk FOREIGN KEY(author_id) REFERENCES users(id));
```

### Pushing a migration directory to Atlas Cloud[​](#pushing-a-migration-directory-to-atlas-cloud "Direct link to Pushing a migration directory to Atlas Cloud")


Run the following command from the parent directory of your migration directory to create a "migration directory" repo in your Atlas Cloud organization (replace "app" with the name you want to give to your new repository):

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "docker://postgres/15/dev?search_path=public"
```
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "docker://mysql/8/dev"
```
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "docker://mariadb/latest/dev"
```
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "sqlite://dev?mode=memory"
```
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "docker://sqlserver/2022-latest"
```
```codeBlockLines_AdAo
$ atlas migrate push app \  --dev-url "docker://clickhouse/23.11"
```
info

If the migration directory contains multiple schemas, adjust the [dev-url](/concepts/dev-database) accordingly.

Atlas will print a URL leading to your migrations on Atlas Cloud. You can visit this URL to view your migrations.

### Setting up GitLab CI[​](#setting-up-gitlab-ci "Direct link to Setting up GitLab CI")


Create a `.gitlab-ci.yml` file with the following pipelines, based on the type of your database. Remember to replace "app" with the real name of your repository.

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse

.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - postgres:latestvariables:  POSTGRES_DB: dev  POSTGRES_USER: user  POSTGRES_PASSWORD: passstages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: "postgres://user:pass@postgres/dev?sslmode=disable"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      revisions-schema: public      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - mysql:latestvariables:  MYSQL_ROOT_PASSWORD: pass  MYSQL_DATABASE: devstages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: "mysql://root:pass@mysql/dev"  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: "mysql://root:pass@mysql/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - mariadb:latestvariables:  MYSQL_ROOT_PASSWORD: pass  MYSQL_DATABASE: devstages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: "maria://root:pass@mariadb/dev"  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: "maria://root:pass@mariadb/dev"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:lateststages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: "sqlite://db?mode=memory"  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: "sqlite://db?mode=memory"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - name: mcr.microsoft.com/mssql/server:2022-latest    alias: sqlservervariables:    ACCEPT_EULA: Y    MSSQL_PID: Developer    MSSQL_SA_PASSWORD: P@ssw0rd0995stages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: sqlserver://sa:P@ssw0rd0995@sqlserver:1433?database=master      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
.gitlab.yml
```codeBlockLines_AdAo
image: ubuntu:latestservices:  - name: clickhouse/clickhouse-server:23.10    alias: clickhousevariables:  CLICKHOUSE_DB: dev  CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1  CLICKHOUSE_PASSWORD: pass  CLICKHOUSE_USER: rootstages:  - lint  - push  - applyinclude:  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-lint@~latest    inputs:      stage: lint      dir: "file://migrations"      atlas-cloud-token: $ATLAS_CLOUD_TOKEN      dir-name: "app"      dev-url: clickhouse://root:pass@localhost:9000/dev  - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-push@~latest    inputs:      stage: push      dir: "file://migrations"      dir-name: "app"      dev-url: clickhouse://root:pass@localhost:9000/dev      atlas-cloud-token: $ATLAS_CLOUD_TOKEN        - component: $CI_SERVER_FQDN/arigaio/atlas/migrate-apply@~latest    inputs:      stage: apply      dir: "file://migrations"      url: $DB_URL      atlas-cloud-token: $ATLAS_CLOUD_TOKEN
```
Let's break down what this file is doing:

1.  The `migrate-lint` component will run on every new merge request. If new migrations are detected, Atlas will lint them and post the report as a merge request comment like this:

![Screenshot of a GitLab merge request comment from Atlas, showing a successful linting report for a new database migration.](/u/gitlab/mr-lint.png)

2.  After the merge request is merged into to main branch, the `migrate-push` component will push the new state of the schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud.
3.  Then, the `migrate-apply` component will deploy the new migrations to your database.

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's take our pipeline for a spin:

1.  Locally, create a new branch and add a new migration with `atlas migrate new --edit`. Paste the following in th editor:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE `test` (`c1` INT)
```
2.  Commit and push the changes.
3.  In Gitlab, open a merge request.
4.  View the lint report generated by Atlas. Follow the links to see the changes visually on Atlas Cloud.
5.  Merge the MR.
6.  When the pipeline is finished running, check your database to see if the changes were applied.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use GitLab CI/CD with Atlas to set up a modern CI/CD pipeline for versioned database migrations. Here's what we accomplished:

*   **Automated migration linting** on every pull request to catch issues early
*   **Centralized migration management** by pushing to Atlas Cloud's Schema Registry
*   **Automated deployments** to your target database when changes are merged

For more information on the versioned workflow, see the [Versioned Migrations documentation](/versioned/intro).

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
*   [Versioned Migrations Workflow](#versioned-migrations-workflow)
    *   [Creating a simple migration directory](#creating-a-simple-migration-directory)
    *   [Pushing a migration directory to Atlas Cloud](#pushing-a-migration-directory-to-atlas-cloud)
    *   [Setting up GitLab CI](#setting-up-gitlab-ci)
    *   [Testing our pipeline](#testing-our-pipeline)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)