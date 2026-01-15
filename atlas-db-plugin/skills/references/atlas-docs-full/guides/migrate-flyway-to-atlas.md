Migrating from Flyway to Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

## Migrating from Flyway to Atlas[​](#migrating-from-flyway-to-atlas "Direct link to Migrating from Flyway to Atlas")


Flyway is one of the earliest tools in the database migration space, but its design has barely evolved in over a decade. It relies on sequential, hand-written SQL files and metadata tables that make schema management brittle at scale.

Atlas is a modern, open-source tool that treats database schemas as code, enabling teams to inspect, plan, lint, visualize, test, and apply schema changes safely and deterministically. It combines declarative and versioned workflows, drift detection, CI/CD automation, and policy enforcement in a single platform designed for modern engineering teams.

In this guide, we'll walk you through the steps for migrating your project from Flyway to Atlas.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


Before you begin, make sure you have:

*   Access to your Flyway migration directory and database
*   Docker installed and running
*   Atlas CLI installed

If you do not have Atlas installed yet, choose your platform below:

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

## Step 1 – Import Your Existing Migration Files[​](#step-1--import-your-existing-migration-files "Direct link to Step 1 – Import Your Existing Migration Files")


Atlas can convert Flyway migration files to Atlas format using the [Migration Directory Import](/versioned/import) feature:
```codeBlockLines_AdAo

# Convert Flyway migrations to Atlas formatatlas migrate import \  --from "file://migrations?format=flyway" \  --to "file://atlas-migrations"

```
What this does:

*   Converts `V__` (versioned) and `R__` (repeatable) Flyway files into Atlas migrations
*   Generates an `atlas.sum` file to track checksums
*   Skips `U__` (undo) files, as Atlas computes [down migrations](/versioned/down) dynamically

Additional details about the import process:

*   **Repeatable migrations** (`R__` prefix) are converted to versioned migrations that end with an `R` suffix (for example, `R__view.sql` → `1R_view.sql`).
*   **Undo migrations** (`U__` prefix) are _not_ imported because Atlas calculates down migrations automatically.
*   **Comments** that don’t directly precede SQL statements may be lost during conversion. Review the generated files if you rely on inline documentation.
*   If you need repeatable-like behaviour after import, use [SQL hooks](/versioned/apply#sql-hooks) to recreate views, procedures, or other objects on each deployment.

## Step 2 – Configure Atlas[​](#step-2--configure-atlas "Direct link to Step 2 – Configure Atlas")


Before we look at the Atlas configuration, here’s a typical Flyway configuration that manages multiple environments:

flyway.toml
```codeBlockLines_AdAo
databaseType = "MySql"id = "8b52a0b7-07aa-46ba-883c-fdf6ef5cfe3d"name = "FlywayAutoPilot"[flyway]locations = [ "filesystem:migrations" ]mixed = trueoutOfOrder = true[flyway.check]majorTolerance = 0[flywayDesktop]developmentEnvironment = "development"shadowEnvironment = "shadow"[flywayDesktop.generate]undoScripts = true[redgateCompare]filterFile = "filter.rgf"[environments.development]url = "jdbc:mysql://localhost:3306"schemas = [ "dev" ]displayName = "Development database"[environments.test]url = "jdbc:mysql://localhost:3308"schemas = [ "test" ]displayName = "Test database"[environments.prod]url = "jdbc:mysql://localhost:3306"schemas = [ "prod" ]displayName = "Production database"[environments.shadow]url = "jdbc:mysql://localhost:3306"schemas = [ "shadow" ]displayName = "Shadow database"[environments.check]url = "jdbc:mysql://localhost:3306"schemas = [ "check" ]displayName = "Check database"[environments.build]url = "jdbc:mysql://localhost:3306"schemas = [ "build" ]displayName = "Build database"
```
**Atlas simplifies this significantly.** Since Atlas utilizes a [dev-database](/concepts/dev-database), an ephemeral Docker container used for migration validation and planning, you only need to configure your real application environments:

atlas.hcl
```codeBlockLines_AdAo
env "local" {  url  = "mysql://root:admin@localhost:3306/dev"  dev  = "docker://mysql/8/dev"  migration {    dir = "file://atlas-migrations"  }}env "production" {  url  = "mysql://user:pass@prod-server:3306/prod"  dev  = "docker://mysql/8/dev"  migration {    dir = "file://atlas-migrations"  }}
```
note

Atlas uses [standard URL format](/concepts/url) (`mysql://user:pass@host:port/database`) instead of JDBC URLs (`jdbc:mysql://host:port`).

## Step 3 – Set the Baseline on Your Existing Database[​](#step-3--set-the-baseline-on-your-existing-database "Direct link to Step 3 – Set the Baseline on Your Existing Database")


Like many other database schema management tools, Atlas uses a [metadata table](/concepts/migration-directory-integrity) on the target database to keep track of which migrations were already applied. When starting to use Atlas on an existing database that was previously managed by Flyway, we must inform Atlas that all migrations up to a certain version were already applied.

Let's try to run Atlas's `migrate apply` command on a database that is currently managed by Flyway:
```codeBlockLines_AdAo
atlas migrate apply --env local
```
Atlas will return an error because it detects that the database is not "clean" (it already contains Flyway's migration history):
```codeBlockLines_AdAo
Error: sql/migrate: connected database is not clean: found table "flyway_schema_history" in schema "your_db". baseline version or allow-dirty is required
```
To fix this, use the `--baseline` flag to tell Atlas that the database is already at a certain version. Use the latest migration version that has been applied by Flyway:
```codeBlockLines_AdAo
atlas migrate apply --env local --baseline 004
```
Atlas reports that there's nothing new to run:
```codeBlockLines_AdAo
No migration files to execute
```
Perfect! Next, let's verify that Atlas is aware of what migrations were already applied by using the `migrate status` command:
```codeBlockLines_AdAo
atlas migrate status --env local
```
Atlas reports:
```codeBlockLines_AdAo
Migration Status: OK  -- Current Version: 004  -- Next Version:    Already at latest version  -- Executed Files:  1  -- Pending Files:   0
```
Great! Atlas now recognizes your existing database state and is ready to manage future migrations.

## Next Steps[​](#next-steps "Direct link to Next Steps")


*   Manage your migrations with Atlas: [versioned workflows](/versioned/intro) or [declarative workflows](/inspect)
*   Explore [migration linting](/versioned/lint) and [schema monitoring](/monitoring) to strengthen your workflow.
*   Plug Atlas into your [CI/CD pipeline](/guides/modern-database-ci-cd) for automatic migration checks.

## Need Help?[​](#need-help "Direct link to Need Help?")


Click the Intercom bubble on the site or [schedule a demo](https://calendly.com/d/cq3j-ktn-2rc) with our team.

*   [Migrating from Flyway to Atlas](#migrating-from-flyway-to-atlas)
*   [Prerequisites](#prerequisites)
*   [Step 1 – Import Your Existing Migration Files](#step-1--import-your-existing-migration-files)
*   [Step 2 – Configure Atlas](#step-2--configure-atlas)
*   [Step 3 – Set the Baseline on Your Existing Database](#step-3--set-the-baseline-on-your-existing-database)
*   [Next Steps](#next-steps)
*   [Need Help?](#need-help)