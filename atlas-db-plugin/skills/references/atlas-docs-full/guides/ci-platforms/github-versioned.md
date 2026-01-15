CI/CD for Databases on GitHub Actions (Versioned Migrations) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

GitHub Actions is a popular CI/CD platform integrated with GitHub repositories. It allows users to automate workflows for building, testing, and deploying applications.

In this guide, we will demonstrate how to use [GitHub Actions](https://docs.github.com/en/actions) and Atlas to set up CI pipelines for your database schema changes using the **versioned migrations** workflow.

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

After installing Atlas locally, log in to your organization by running the following command:
```codeBlockLines_AdAo
atlas login
```

### Creating a bot token[​](#creating-a-bot-token "Direct link to Creating a bot token")


To report CI run results to Atlas Cloud, create an Atlas Cloud bot token by following [these instructions](/cloud/bots) and copy it.

Next, in your GitHub repository, go to **Settings** -> **Secrets and variables** -> **Actions** and create a new secret named `ATLAS_CLOUD_TOKEN`. Paste your token in the value field.

### Creating a secret for your database URL[​](#creating-a-secret-for-your-database-url "Direct link to Creating a secret for your database URL")


To connect Atlas to your target database, create a URL (connection string) by following our [URL documentation](/concepts/url).

Create another secret named `DB_URL` and populate it with the URL of your database to avoid having sensitive information in your configuration files.

### Creating a GitHub personal access token (optional)[​](#creating-a-github-personal-access-token-optional "Direct link to Creating a GitHub personal access token (optional)")


Atlas will need permissions to comment lint reports on pull requests. To enable this, create a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with the `repo` scope. Then, add it as a secret named `GITHUB_TOKEN`.

## Versioned Migrations Workflow[​](#versioned-migrations-workflow "Direct link to Versioned Migrations Workflow")


In the [versioned workflow](/versioned/intro), changes to the schema are represented by a _migration directory_ in your codebase. Each file in this directory represents a transition to a new version of the schema.

Based on our blueprint for [Modern CI/CD for Databases](/guides/modern-database-ci-cd), our pipeline will:

1.  [Lint](/versioned/lint) new migration files whenever a pull request is opened.
2.  [Push](/versioned/intro#pushing-migrations-to-the-schema-registry) the migration directory to the [Schema Registry](/cloud/features/registry) when changes are merged to the main branch.
3.  [Apply](/versioned/apply) new migrations to our database.

Auto-generating migrations

Instead of generating migrations locally, you can use the [`migrate/diff`](/integrations/github-actions#arigaatlas-actionmigratediff) action as part of your CI pipeline.

### Creating a simple migration directory[​](#creating-a-simple-migration-directory "Direct link to Creating a simple migration directory")


If you don't have a migration directory yet, create one by running the following command:
```codeBlockLines_AdAo
atlas migrate new --edit
```
and paste the following in the editor:
```codeBlockLines_AdAo
-- create table "users"CREATE TABLE users(	id int NOT NULL,	name varchar(100) NULL,	PRIMARY KEY(id));-- create table "blog_posts"CREATE TABLE blog_posts(	id int NOT NULL,	title varchar(100) NULL,	body text NULL,	author_id int NULL,	PRIMARY KEY(id),	CONSTRAINT author_fk FOREIGN KEY(author_id) REFERENCES users(id));
```

### Pushing a migration directory to Atlas Cloud[​](#pushing-a-migration-directory-to-atlas-cloud "Direct link to Pushing a migration directory to Atlas Cloud")


Run the following command from the parent directory of your migration directory to create a "Migration Directory" repository in your Atlas Cloud organization (replace "app" with the name you want to give to your new repository):

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

### Setting up GitHub Actions[​](#setting-up-github-actions "Direct link to Setting up GitHub Actions")


Create a `.github/workflows/ci-atlas.yaml` file with the following content, based on the type of your database. Remember to replace "app" with the real name of your repository.

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse

.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    services:      # Spin up a postgres:15 container to be used as the dev-database for analysis.      postgres:        image: postgres:15        env:          POSTGRES_DB: dev          POSTGRES_PASSWORD: pass        ports:          - 5432:5432        options: >-          --health-cmd pg_isready          --health-interval 10s          --health-start-period 10s          --health-timeout 5s          --health-retries 5    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    services:      # Spin up a mysql:8 container to be used as the dev-database for analysis.      mysql:        image: mysql:8        env:          MYSQL_DATABASE: dev          MYSQL_ROOT_PASSWORD: pass        ports:          - 3306:3306        options: >-          --health-cmd "mysqladmin ping -ppass"          --health-interval 10s          --health-start-period 10s          --health-timeout 5s          --health-retries 10    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'mysql://root:pass@localhost:3306/dev'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'mysql://root:pass@localhost:3306/dev'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    services:      # Spin up a mariadb:11 container to be used as the dev-database for analysis.      mariadb:        image: mariadb:11        env:          MYSQL_DATABASE: dev          MYSQL_ROOT_PASSWORD: pass        ports:          - 3306:3306        options: >-          --health-cmd "healthcheck.sh --su-mysql --connect --innodb_initialized"          --health-interval 10s          --health-start-period 10s          --health-timeout 5s          --health-retries 10    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'maria://root:pass@localhost:3306/dev'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'maria://root:pass@localhost:3306/dev'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'sqlite://dev?mode=memory'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'sqlite://dev?mode=memory'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    services:      # Spin up a mcr.microsoft.com/mssql/server:2022-latest container to be used as the dev-database for analysis.      sqlserver:        image: mcr.microsoft.com/mssql/server:2022-latest        env:          ACCEPT_EULA: Y          MSSQL_PID: Developer          MSSQL_SA_PASSWORD: P@ssw0rd0995        ports:          - 1433:1433        options: >-          --health-cmd "/opt/mssql-tools/bin/sqlcmd -U sa -P P@ssw0rd0995 -Q \"SELECT 1\""          --health-interval 10s          --health-timeout 5s          --health-retries 5    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches:      - master    paths:      - .github/workflows/ci-atlas.yaml      - 'migrations/*'  pull_request:    paths:      - 'migrations/*'# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  atlas:    services:      # Spin up a clickhouse:23.10 container to be used as the dev-database for analysis.      clickhouse:        image: clickhouse/clickhouse-server:23.10        env:          CLICKHOUSE_DB: test          CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1          CLICKHOUSE_PASSWORD: pass          CLICKHOUSE_USER: root        ports:          - 9000:9000        options: >-          --health-cmd "clickhouse-client --host localhost --query 'SELECT 1'"          --health-interval 10s          --health-timeout 5s          --health-retries 5    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: '${{ secrets.ATLAS_CLOUD_TOKEN }}'      - uses: ariga/atlas-action/migrate/lint@v1        with:          dir: 'file://migrations'          dev-url: 'clickhouse://root:pass@localhost:9000/test'          dir-name: 'app'        env:          GITHUB_TOKEN: '${{ github.token }}'      - uses: ariga/atlas-action/migrate/push@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          dev-url: 'clickhouse://root:pass@localhost:9000/test'          dir-name: 'app'      - uses: ariga/atlas-action/migrate/apply@v1        if: github.ref == 'refs/heads/master'        with:          dir: 'file://migrations'          url: '${{ secrets.DB_URL }}'
```
Let's break down what this file is doing:

1.  The `migrate-lint` step will run on every pull request. If new migrations are detected, Atlas will lint them and post the report as a comment on the pull request. This helps you catch issues early in the development process.

![GitHub Actions pull request comment showing Atlas migration lint results with error details](/u/cloud/ci/gh-action-comment-err.png)

2.  The `migrate-push` step will run only when the pull request is merged into the main branch. It will push the new migration directory to the [Schema Registry](/cloud/features/registry) on Atlas Cloud, allowing you to manage your migrations in a centralized location.

3.  The `migrate-apply` step will then deploy the new migrations to your database.

### Testing our action[​](#testing-our-action "Direct link to Testing our action")


Let's take our new workflow for a spin. We will create a new migration, push it to the repository, and see how the GitHub Action runs.

1.  Locally, create a new branch and add a new migration with `atlas migrate new --edit`. Paste the following in the editor:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE `test` (`c1` INT)
```
2.  Commit and push the changes.
3.  In GitHub, open a new pull request for the branch you just pushed. This will trigger the `migrate-lint` step in the workflow.
4.  View the lint report generated by Atlas. Follow the links to see the changes visually on Atlas Cloud.
5.  Merge the pull request into the main branch. This will trigger the `migrate-push` and `migrate-apply` steps in the workflow.
6.  When the pipeline is finished running, check your database to see if the changes were applied.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use GitHub Actions with Atlas to set up a modern CI/CD pipeline for versioned database migrations. Here's what we accomplished:

*   **Automated migration linting** on every pull request to catch issues early
*   **Centralized migration management** by pushing to Atlas Cloud's Schema Registry
*   **Automated deployments** to your target database when changes are merged

For more information on the versioned workflow, see the [Versioned Migrations documentation](/versioned/intro).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [migration testing](/testing/migrate) to validate your migrations
*   Read about [declarative migrations](/declarative/apply) as an alternative workflow
*   Check out the [Github Actions reference](/integrations/github-actions) for all available actions

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token](#creating-a-bot-token)
    *   [Creating a secret for your database URL](#creating-a-secret-for-your-database-url)
    *   [Creating a GitHub personal access token (optional)](#creating-a-github-personal-access-token-optional)
*   [Versioned Migrations Workflow](#versioned-migrations-workflow)
    *   [Creating a simple migration directory](#creating-a-simple-migration-directory)
    *   [Pushing a migration directory to Atlas Cloud](#pushing-a-migration-directory-to-atlas-cloud)
    *   [Setting up GitHub Actions](#setting-up-github-actions)
    *   [Testing our action](#testing-our-action)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)