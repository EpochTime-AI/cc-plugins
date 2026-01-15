CI/CD for Databases on GitHub Actions (Declarative) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

GitHub Actions is a popular CI/CD platform integrated with GitHub repositories. It allows users to automate workflows for building, testing, and deploying applications.

In this guide, we will demonstrate how to use [GitHub Actions](https://docs.github.com/en/actions) and Atlas to set up CI pipelines for your database schema changes using the **declarative migrations** workflow.

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

## Declarative Migrations Workflow[​](#declarative-migrations-workflow "Direct link to Declarative Migrations Workflow")


In the [declarative workflow](/declarative/apply), developers provide the desired state of the database as code. Atlas can read database schemas from various formats such as [plain SQL](/atlas-schema/sql), [Atlas HCL](/atlas-schema/hcl), [ORM models](/atlas-schema/external), and even another live database. Atlas then connects to the target database and calculates the diff between the current state and the desired state. It then generates a migration plan to bring the database to the desired state.

In this guide, we will use the SQL schema format.

### Our goal[​](#our-goal "Direct link to Our goal")


When a pull request contains changes to the schema, we want Atlas to:

*   Compare the current state (your database) with the new desired state
*   Create a migration plan to show the user for approval
*   Mark the plan as "approved" when the pull request is approved and merged
*   Use the approved plan to apply the changes to the database during deployment

### Creating a simple SQL schema[​](#creating-a-simple-sql-schema "Direct link to Creating a simple SQL schema")


Create a file named `schema.sql` and paste the following:
```codeBlockLines_AdAo
-- create table "users"CREATE TABLE users(	id int NOT NULL,	name varchar(100) NULL,	PRIMARY KEY(id));-- create table "blog_posts"CREATE TABLE blog_posts(	id int NOT NULL,	title varchar(100) NULL,	body text NULL,	author_id int NULL,	PRIMARY KEY(id),	CONSTRAINT author_fk FOREIGN KEY(author_id) REFERENCES users(id));
```
Then, create a configuration file for Atlas named `atlas.hcl` as follows:

atlas.hcl
```codeBlockLines_AdAo
env "github" {    url = getenv("DB_URL")    schema {        src = "file://schema.sql"        repo {            name = "app"        }    }}
```

### Pushing the schema to Atlas Cloud[​](#pushing-the-schema-to-atlas-cloud "Direct link to Pushing the schema to Atlas Cloud")


Run the following command from the parent directory to create a "Schema" repository in your Atlas Cloud organization (replace "app" with the name you want to give to your new repository):

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://postgres/15/dev?search_path=public" \  --env github
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://mysql/8/dev" \  --env github
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://mariadb/latest/dev" \  --env github
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "sqlite://dev?mode=memory" \  --env github
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://sqlserver/2022-latest" \  --env github
```
```codeBlockLines_AdAo
$ atlas schema push app \  --dev-url "docker://clickhouse/23.11" \  --env github
```
Atlas will print a URL leading to your migrations on Atlas Cloud. You can visit this URL to view your migrations.

### Setting up GitHub Actions[​](#setting-up-github-actions "Direct link to Setting up GitHub Actions")


Create a `.github/workflows/ci-atlas.yaml` file with the following content, based on the type of your database.

*   PostgreSQL
*   MySQL
*   MariaDB
*   SQLite
*   SQL Server
*   ClickHouse

.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    services:        # Spin up a postgres:15 container to be used as the dev-database for analysis.        postgres:          image: postgres:15          env:            POSTGRES_DB: dev            POSTGRES_PASSWORD: pass          ports:            - 5432:5432          options: >-            --health-cmd pg_isready            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    services:        # Spin up a postgres:15 container to be used as the dev-database for analysis.        postgres:          image: postgres:15          env:            POSTGRES_DB: dev            POSTGRES_PASSWORD: pass          ports:            - 5432:5432          options: >-            --health-cmd pg_isready            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'postgres://postgres:pass@localhost:5432/dev?search_path=public&sslmode=disable'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    services:        # Spin up a mysql:8 container to be used as the dev-database for analysis.        mysql:          image: mysql:8          env:            MYSQL_DATABASE: dev            MYSQL_ROOT_PASSWORD: pass          ports:            - 3306:3306          options: >-            --health-cmd "mysqladmin ping -ppass"            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 10    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'mysql://root:pass@localhost:3306/dev'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    services:        # Spin up a mysql:8 container to be used as the dev-database for analysis.        mysql:          image: mysql:8          env:            MYSQL_DATABASE: dev            MYSQL_ROOT_PASSWORD: pass          ports:            - 3306:3306          options: >-            --health-cmd "mysqladmin ping -ppass"            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 10    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'mysql://root:pass@localhost:3306/dev'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'mysql://root:pass@localhost:3306/dev'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'mysql://root:pass@localhost:3306/dev'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    services:        # Spin up a mariadb:11 container to be used as the dev-database for analysis.        mariadb:          image: mariadb:11          env:            MYSQL_DATABASE: dev            MYSQL_ROOT_PASSWORD: pass          ports:            - 3306:3306          options: >-            --health-cmd "healthcheck.sh --su-mysql --connect --innodb_initialized"            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 10    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'maria://root:pass@localhost:3306/dev'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    services:        # Spin up a mariadb:11 container to be used as the dev-database for analysis.        mariadb:          image: mariadb:11          env:            MYSQL_DATABASE: dev            MYSQL_ROOT_PASSWORD: pass          ports:            - 3306:3306          options: >-            --health-cmd "healthcheck.sh --su-mysql --connect --innodb_initialized"            --health-interval 10s            --health-start-period 10s            --health-timeout 5s            --health-retries 10    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'maria://root:pass@localhost:3306/dev'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'maria://root:pass@localhost:3306/dev'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'maria://root@pass@localhost:3306/dev'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'sqlite://dev?mode=memory'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'sqlite://dev?mode=memory'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'sqlite://dev?mode=memory'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'sqlite://dev?mode=memory'
```
.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    services:        # Spin up a mcr.microsoft.com/mssql/server:2022-latest container to be used as the dev-database for analysis.        sqlserver:          image: mcr.microsoft.com/mssql/server:2022-latest          env:            ACCEPT_EULA: Y            MSSQL_PID: Developer            MSSQL_SA_PASSWORD: P@ssw0rd0995          ports:            - 1433:1433          options: >-            --health-cmd "/opt/mssql-tools/bin/sqlcmd -U sa -P P@ssw0rd0995 -Q \"SELECT 1\""            --health-interval 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    services:        # Spin up a mcr.microsoft.com/mssql/server:2022-latest container to be used as the dev-database for analysis.        sqlserver:          image: mcr.microsoft.com/mssql/server:2022-latest          env:            ACCEPT_EULA: Y            MSSQL_PID: Developer            MSSQL_SA_PASSWORD: P@ssw0rd0995          ports:            - 1433:1433          options: >-            --health-cmd "/opt/mssql-tools/bin/sqlcmd -U sa -P P@ssw0rd0995 -Q \"SELECT 1\""            --health-interval 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'sqlserver://sa:P@ssw0rd0995@localhost:1433/test?mode=schema'
```
📺 For a walkthrough of this CI/CD setup, watch our video demonstration: [Declarative Schema Management for ClickHouse with Atlas](https://youtu.be/7GDWWVBBuoM?utm_source=gh-actions-guide)

.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
name: Atlason:  push:    branches: [ master ]  pull_request:    branches: [ master ]  workflow_dispatch:# Permissions to write comments on the pull request.permissions:  contents: read  pull-requests: writejobs:  plan:    if: ${{ github.event_name == 'pull_request' }}    services:        # Spin up a clickhouse:23.10 container to be used as the dev-database for analysis.        clickhouse:          image: clickhouse/clickhouse-server:23.10          env:            CLICKHOUSE_DB: test            CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1            CLICKHOUSE_PASSWORD: pass            CLICKHOUSE_USER: root          ports:            - 9000:9000          options: >-            --health-cmd "clickhouse-client --host localhost --query 'SELECT 1'"            --health-interval 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan@v1        with:          env: github          dev-url: 'clickhouse://root:pass@localhost:9000/test'  push:    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    services:        # Spin up a clickhouse:23.10 container to be used as the dev-database for analysis.        clickhouse:          image: clickhouse/clickhouse-server:23.10          env:            CLICKHOUSE_DB: test            CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT: 1            CLICKHOUSE_PASSWORD: pass            CLICKHOUSE_USER: root          ports:            - 9000:9000          options: >-            --health-cmd "clickhouse-client --host localhost --query 'SELECT 1'"            --health-interval 10s            --health-timeout 5s            --health-retries 5    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v3        with:          fetch-depth: 0      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      - uses: ariga/atlas-action/schema/plan/approve@v1        id: plan-approve        with:          env: github          dev-url: 'clickhouse://root:pass@localhost:9000/test'      - uses: ariga/atlas-action/schema/push@v1        with:          env: github          dev-url: 'clickhouse://root:pass@localhost:9000/test'      - uses: ariga/atlas-action/schema/apply@v1        with:          env: github          dev-url: 'clickhouse://root:pass@localhost:9000/test'
```
info

Make sure the `DB_URL` environment variable is set to the URL of your database in your GitHub Actions environment.

Let's break down what this file is doing:

1.  When a new pull request is opened, the `plan` job will check if the desired state of the schema was changed. If it was, Atlas will generate a migration plan, lint it and post the report as a pull request comment.

2.  When the pull request is merged, two things happen: First, the updated schema is pushed to the schema registry by the `schema/push` step. Second, the plan created in the pull request will be approved.

3.  The `schema-apply` step will then use the approved plan to apply the new schema state to the database.

### Testing our workflow[​](#testing-our-workflow "Direct link to Testing our workflow")


Let's see our CI/CD workflow in action.

Add an "address" column to the users table:

schema.sql
```codeBlockLines_AdAo
	name varchar(100) NULL,+	address varchar(100) NULL,	PRIMARY KEY(id)
```
Commit the change to a new branch, push it to GitHub, and open a pull request. The `plan` job will use Atlas to create a migration plan from the current state of the database to the new desired state:

![GitHub Actions workflow showing Atlas schema plan and lint results in pull request comment](/u/github/plan-lint.png)

There are two things to note: The comment also includes instructions for editing the plan. This is useful when the plan has lint issues (for example, dropping a column will raise a "destructive changes" error).

*   The plan is created in a "pending" state, which means Atlas can't use it yet against the real database.

After merging the changes to the main branch, the workflow will run the `push` job:

1.  The `schema-plan-approve` step will approve the plan that was generated earlier.

2.  The `schema-push` step will update the schema registry with the new desired state.

3.  The `schema-apply` step will deploy the changes to our database.

![GitHub Actions workflow console output showing successful Atlas schema apply deployment to database](/u/github/schema-apply.png)

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


In this guide, we demonstrated how to use GitHub Actions with Atlas to set up a modern CI/CD pipeline for declarative schema migrations. Here's what we accomplished:

*   **Automated schema planning** on every pull request to visualize changes
*   **Centralized schema management** by pushing to Atlas Cloud's Schema Registry
*   **Approval workflow** ensuring only reviewed changes are applied
*   **Automated deployments** using approved plans

For more information on the declarative workflow, see the [Declarative Migrations documentation](/declarative/apply).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [schema testing](/testing/schema) to validate your schema
*   Read about [versioned migrations](/versioned/intro) as an alternative workflow
*   Check out the [Github Actions reference](/integrations/github-actions) for all available actions
*   Understand the differences between [Declarative vs Versioned](/concepts/declarative-vs-versioned) workflows

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token](#creating-a-bot-token)
    *   [Creating a secret for your database URL](#creating-a-secret-for-your-database-url)
    *   [Creating a GitHub personal access token (optional)](#creating-a-github-personal-access-token-optional)
*   [Declarative Migrations Workflow](#declarative-migrations-workflow)
    *   [Our goal](#our-goal)
    *   [Creating a simple SQL schema](#creating-a-simple-sql-schema)
    *   [Pushing the schema to Atlas Cloud](#pushing-the-schema-to-atlas-cloud)
    *   [Setting up GitHub Actions](#setting-up-github-actions)
    *   [Testing our workflow](#testing-our-workflow)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)