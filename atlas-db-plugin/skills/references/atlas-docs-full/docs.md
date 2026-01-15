Welcome to the Atlas Documentation | Atlas

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas is a language-independent tool for managing and migrating database schemas using modern DevOps principles. It offers two workflows:

*   **Declarative**: Similar to Terraform, Atlas compares the current state of the database to the desired state, as defined in an [HCL](/atlas-schema/hcl), [SQL](/atlas-schema/sql), or [ORM](/atlas-schema/external) schema. Based on this comparison, it generates and executes a migration plan to transition the database to its desired state.

*   **Versioned**: Unlike other tools, Atlas automatically plans schema migrations for you. Users can describe their desired database schema in [HCL](/atlas-schema/hcl), [SQL](/atlas-schema/sql), or their chosen [ORM](/atlas-schema/external), and by utilizing Atlas, they can plan, lint, and apply the necessary migrations to the database.

[

Quick Start

!

Get started with Atlas in under 5 minutes.

](/getting-started)

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

### Schema-as-Code[​](#schema-as-code "Direct link to Schema-as-Code")


Choose your preferred way to represent the desired state of your database.

[![SQL illustration](/icons-docs/docs-sql.svg)

##### SQL


Describe your database schema with native SQL.

](/atlas-schema/sql)[![Atlas HCL illustration](/icons-docs/docs-hcl.svg)

##### Atlas HCL


The HCL-based language allows developers to describe database schemas in a declarative manner.

](/atlas-schema/hcl)[![ORM illustration](/icons-docs/docs-orm.svg)

##### ORM


Represent your database with your ORM.

](/orms)

### Generating Migration[​](#generating-migration "Direct link to Generating Migration")


Atlas offers two workflows to generate and plan migration changes, choose which one works best for you. In both workflows, Atlas automatically plans schema migrations based on your desired schema.

[![Declarative migrations illustration](/icons-docs/migration-declarative.svg)

##### Declarative Migrations


Set up a Terraform-like workflow where each migration is calculated as the diff between your desired state and the current state of the database.

](/declarative/apply)[![Versioned migrations illustration](/icons-docs/migration-versioned.svg)

##### Versioned Migration


Set up a migration directory for your project, creating a version-controlled source of truth of your database schema.

](/versioned/intro)

### Verifying Migration Safety[​](#verifying-migration-safety "Direct link to Verifying Migration Safety")


Atlas verifies your migrations by running them against [various checks](/lint/analyzers) to ensure their safety and validity. Verify these changes locally, or as part of your CI process.

[

##### ![SQL icon](/icons-docs/sql-icon.svg)CLI


Verify your migration files in your CLI.

](/versioned/lint)[

##### ![GitHub logo](/icons-docs/github-icon.svg)GitHub


Setup automatic migration linting with GitHub Actions.

](/integrations/github-actions#arigaatlas-actionmigratelint-ci)[

##### ![GitLab logo](/icons-docs/gitlab-icon.svg)Gitlab


Setup automatic migration linting with GitLab CI.

](/integrations/gitlab-ci-components#migrate-lint)

### Deploying Migrations[​](#deploying-migrations "Direct link to Deploying Migrations")


Deploy migrations to target databases in production by integrating natively with the rest of your infrastructure management.

[

##### ![SQL icon](/icons-docs/sql-icon.svg)CLI


Deploy migrations from your CLI.

](/versioned/apply)[

##### ![GitHub logo](/icons-docs/github-icon.svg)GitHub


Setup deployments with GitHub Actions.

](/integrations/github-actions#arigaatlas-actionmigrateapply)[

##### ![GitLab logo](/icons-docs/gitlab-icon.svg)Gitlab


Setup deployments with GitLab CI.

](/integrations/gitlab-ci-components#migrate-apply)[

##### ![Terraform logo](/icons-docs/terraform-icon.svg)Terraform Provider


Use Atlas with Terraform to manage database schema changes.

](/integrations/terraform-provider)[

##### ![Kubernetes logo](/icons-docs/kubernetes-icon.svg)Kubernetes Operator


Use the Atlas Kubernetes Operator to manage database schema changes.

](/integrations/kubernetes)

### Testing Framework[​](#testing-framework "Direct link to Testing Framework")


Atlas provides a database testing framework that allows you to test database schemas and migrations much like you would test your own code.

[![Schema test illustration](/icons-docs/schema-test.svg)

##### Schema Test


Test your database schema using familiar software testing paradigms.

](/testing/schema)[![Migrate test illustration](/icons-docs/migrate-test.svg)

##### Migrate Test


Write tests for schema migrations.

](/testing/migrate)

### Atlas Cloud[​](#atlas-cloud "Direct link to Atlas Cloud")


Gain full visibility into the databases in your development and production environments with visualizations, detailed logs and troubleshooting capabilities.

[![Atlas Cloud illustration](/icons-docs/atlas-cloud.svg)

##### Getting Started


Push your database schema to the Cloud to maintain a single source of truth for database schemas with auto-generated documentation.

](/cloud/getting-started)[![Setup CI illustration](/icons-docs/setup-ci-icon.svg)

##### Setup CI


Simulate and analyze changes to catch destructive changes, backward-incompatibility issues, accidental table locks, and constraint violations way before they reach production.

](/cloud/setup-ci)[![Deployments illustration](/icons-docs/deployments-icon.svg)

##### Deployments


Gain full visibility into the database schema in your production environments with detailed logs and troubleshooting capabilities.

](/cloud/deployment)

### Supported Databases


[All Databases !](/guides)

[

![PostgreSQL logo](/icons-docs/postgresql.svg)

PostgreSQL

](/guides/postgres/automatic-migrations)[

![MySQL logo](/icons-docs/mysql.svg)

MySQL

](/guides/mysql/mysql-automatic-migrations)[

![SQLite logo](/icons-docs/sqlite.svg)

SQLite

](/guides/sqlite/partial-indexes)

![MariaDB logo](/icons-docs/mariadb.svg)

MariaDB

[

![SQL Server logo](/icons-docs/sql-server.svg)

SQL Server

](/guides/mssql)[

![ClickHouse logo](/icons-docs/clickhouse.svg)

ClickHouse

](/guides/clickhouse)[

![Redshift logo](/icons-docs/redshift.svg)

Redshift

](/guides/redshift)[

![Oracle logo](/icons-docs/oracle.svg)

Oracle

](/guides/oracle/automatic-migrations)[

![Spanner logo](/icons-docs/spanner.svg)

Spanner

](/guides/spanner/automatic-migrations)[

![Snowflake logo](/icons-docs/snowflake.svg)

Snowflake

](/guides/snowflake/automatic-migrations)

### Latest Guides


[View all guides !](/guides)

[

##### ![Guide icon](/icons-docs/guide-icon.svg)Modern Database CI/CD


Learn about Atlas's modern approach to Database CI/CD

](/guides/modern-database-ci-cd)[

##### ![Guide icon](/icons-docs/guide-icon.svg)Migration for SQLAlchemy


Automatic migration planning for SQLAlchemy applications

](/guides/orms/sqlalchemy)[

##### ![Guide icon](/icons-docs/guide-icon.svg)Working with secrets


Working with sensitive info like database passwords

](/guides/deploying/secrets)

*   [Installation](#installation)
*   [Schema-as-Code](#schema-as-code)
*   [Generating Migration](#generating-migration)
*   [Verifying Migration Safety](#verifying-migration-safety)
*   [Deploying Migrations](#deploying-migrations)
*   [Testing Framework](#testing-framework)
*   [Atlas Cloud](#atlas-cloud)