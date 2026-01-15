Liquibase Diff Alternative in Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Liquibase provides `diff` and `diff-changelog` commands to compare database schemas and generate changelogs from differences, respectively. While useful, these commands have limitations: they require two live database connections, produce verbose output, and don't integrate well with modern CI/CD workflows.

Atlas provides powerful alternatives with `atlas schema diff` and `atlas schema inspect` that offer more flexibility, better output formats, and seamless integration with GitOps workflows.

**Atlas can compare any schema sources**: databases, SQL files, HCL files, ORM models, and migration directories can all be diffed against each other. In addition, Atlas produces the migration file needed to migrate the database to the desired state in standard, native SQL format.

## Quick Comparison[​](#quick-comparison "Direct link to Quick Comparison")


Capability

Atlas schema diff/inspect

Liquibase diff/diff-changelog

Compare any source to any

✅ DB, SQL, HCL, ORM, migrations

⚠️ Databases and offline snapshots only

Compare to schema files

✅ SQL and HCL files directly

❌ Not supported

Compare to ORM models

✅ GORM, Ent, Django, Sequelize, etc.

❌ Not supported

Output formats

✅ SQL, HCL, JSON, ERD, custom

✅ XML/YAML/JSON/SQL changelog

Visual ERD diff

✅ Interactive ERD with diff view

❌ Not supported

Drift detection

✅ Continuous monitoring

⚠️ Manual

CI/CD integration

✅ GitHub Actions, GitLab, K8s, Argo CD, Terraform

⚠️ CLI and plugins only

Full Disclosure

This document is maintained by the Atlas team. It may contain outdated information or mistakes. For Liquibase's latest details, please refer to the official [Liquibase website](https://docs.liquibase.com/).

## Comparing Database Schemas with Atlas[​](#comparing-database-schemas-with-atlas "Direct link to Comparing Database Schemas with Atlas")


Atlas's `schema diff` command accepts any combination of schema sources for `--from` and `--to`. All sources can be compared to each other in any combination.

[

##### !Database URLs


Connect directly to MySQL, PostgreSQL, SQL Server, ClickHouse, SQLite, and more using standard connection URLs.

](/concepts/url)[

##### !SQL Schema Code


Define your schema as plain SQL DDL statements. Great for teams familiar with SQL who want version-controlled schemas.

](/atlas-schema/sql)[

##### !HCL Schema Code


Use Atlas's declarative HCL language for type-safe schema definitions with IDE support and validation.

](/atlas-schema/hcl)[

##### !Migration Directories


Compare against existing migration directories to detect drift or generate new migrations from schema changes.

](/versioned/diff)[

##### !ORM Models


Load schemas directly from your ORM: GORM, Ent, Django, SQLAlchemy, Sequelize, TypeORM, Hibernate, and more.

](/orms)

### Compare Two Databases[​](#compare-two-databases "Direct link to Compare Two Databases")


Here are some examples comparing two database schemas across different database engines:

*   MySQL
*   PostgreSQL
*   SQL Server
*   ClickHouse
*   SQLite
*   Redshift
*   Databricks
```codeBlockLines_AdAo
atlas schema diff \  --from "mysql://root:pass@localhost:3306/prod" \  --to "mysql://root:pass@localhost:3306/staging"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "postgres://user:pass@localhost:5432/prod?sslmode=disable" \  --to "postgres://user:pass@localhost:5432/staging?sslmode=disable"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "sqlserver://sa:pass@localhost:1433?database=prod" \  --to "sqlserver://sa:pass@localhost:1433?database=staging"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "clickhouse://default:pass@localhost:9000/prod" \  --to "clickhouse://default:pass@localhost:9000/staging"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "sqlite://prod.db" \  --to "sqlite://staging.db"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "redshift://user:pass@cluster.region.redshift.amazonaws.com:5439/prod" \  --to "redshift://user:pass@cluster.region.redshift.amazonaws.com:5439/staging"
```
```codeBlockLines_AdAo
atlas schema diff \  --from "databricks://token:dapi...@workspace.cloud.databricks.com:443/prod" \  --to "databricks://token:dapi...@workspace.cloud.databricks.com:443/staging"
```
Atlas outputs the SQL statements needed to transform the `from` schema into the `to` schema:
```codeBlockLines_AdAo
-- Add new columnALTER TABLE "users" ADD COLUMN "phone" varchar(50);-- Create new indexCREATE INDEX "idx_users_email" ON "users" ("email");
```

### Compare Database to Schema Code[​](#compare-database-to-schema-code "Direct link to Compare Database to Schema Code")


Unlike Liquibase, Atlas can compare a live database directly to a schema file - no reference database needed:
```codeBlockLines_AdAo

# Compare database to SQL schema fileatlas schema diff \  --from "mysql://root:pass@localhost:3306/prod" \  --to "file://schema.sql" \  --dev-url "docker://mysql/8/dev"# Compare database to HCL schema fileatlas schema diff \  --from "mysql://root:pass@localhost:3306/prod" \  --to "file://schema.hcl" \  --dev-url "docker://mysql/8/dev"

```
About `--dev-url`

The `--dev-url` flag specifies a temporary database connection used by Atlas to normalize and compare schemas. This can be a local Docker container or any development database instance. Atlas uses this connection internally and doesn't modify your source or target schemas. Read more at: [Dev Database](/concepts/dev-database).

### Compare Any Source to Any Other[​](#compare-any-source-to-any-other "Direct link to Compare Any Source to Any Other")


Atlas can compare any schema sources to any other. Here are some examples:
```codeBlockLines_AdAo

# Compare database to ORM modelsatlas schema diff \  --from "mysql://root:pass@localhost:3306/prod" \  --to "ent://ent/schema" \  --dev-url "docker://mysql/8/dev"# Compare SQL file to HCL fileatlas schema diff \  --from "file://old-schema.sql" \  --to "file://new-schema.hcl" \  --dev-url "docker://mysql/8/dev"# Compare migration directory to databaseatlas schema diff \  --from "file://migrations" \  --to "mysql://root:pass@localhost:3306/prod" \  --dev-url "docker://mysql/8/dev"# Compare two migration directoriesatlas schema diff \  --from "file://migrations-v1" \  --to "file://migrations-v2" \  --dev-url "docker://mysql/8/dev"

```

## Inspecting and Snapshotting Schemas[​](#inspecting-and-snapshotting-schemas "Direct link to Inspecting and Snapshotting Schemas")


Similar to Liquibase's snapshot command, Atlas can capture database schema states for any supported source, including PostgreSQL, MySQL, MariaDB, SQL Server, SQLite, ClickHouse, Redshift, Snowflake, and [more](/features#database-support).

*   MySQL
*   PostgreSQL
*   SQL Server
*   ClickHouse
*   SQLite
*   Databricks
```codeBlockLines_AdAo
atlas schema inspect -u "mysql://root:pass@localhost:3306/mydb" --format '{{ sql . }}'
```
```codeBlockLines_AdAo
atlas schema inspect -u "postgres://user:pass@localhost:5432/mydb?sslmode=disable" --format '{{ sql . }}'
```
```codeBlockLines_AdAo
atlas schema inspect -u "sqlserver://sa:pass@localhost:1433?database=mydb" --format '{{ sql . }}'
```
```codeBlockLines_AdAo
atlas schema inspect -u "clickhouse://default:pass@localhost:9000/mydb" --format '{{ sql . }}'
```
```codeBlockLines_AdAo
atlas schema inspect -u "sqlite://mydb.db" --format '{{ sql . }}'
```
```codeBlockLines_AdAo
atlas schema inspect -u "databricks://token:dapi...@workspace.cloud.databricks.com:443/mydb" --format '{{ sql . }}'
```

### Output Formats[​](#output-formats "Direct link to Output Formats")


Atlas supports multiple output formats for schema inspection:

Format

Command

Use Case

SQL

`--format '{{ sql . }}'`

Version control, migrations

HCL

(default)

Atlas schema-as-code

JSON

`--format '{{ json . }}'`

Programmatic processing

ERD

`--web`

Visual documentation

## Visualize with ERD[​](#visualize-with-erd "Direct link to Visualize with ERD")


Atlas can generate interactive Entity Relationship Diagrams when you use the `--web` flag:
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" --web
```
This opens an interactive ERD in Atlas Cloud where you can explore tables, relationships, and schema documentation.

### Visual Schema Diff[​](#visual-schema-diff "Direct link to Visual Schema Diff")


Atlas can also display schema differences as a visual ERD diff, making it easy to see exactly what changed:
```codeBlockLines_AdAo
atlas schema diff \  --from "mysql://root:pass@localhost:3306/prod" \  --to "file://schema.sql" \  --dev-url "docker://mysql/8/dev" \  --web
```
This opens an interactive view showing added, modified, and removed tables/columns highlighted in the ERD.

## Drift Detection[​](#drift-detection "Direct link to Drift Detection")


While Liquibase requires manually running `diff-changelog` to detect drift, Atlas provides both CLI-based checks and continuous automated monitoring.

### CLI Drift Check[​](#cli-drift-check "Direct link to CLI Drift Check")

```codeBlockLines_AdAo

# Check if production matches expected schemaatlas schema diff \  --from "mysql://root:pass@prod:3306/app" \  --to "file://schema.sql" \  --dev-url "docker://mysql/8/dev"

```
This check can also run automatically in CI with native integrations for [GitHub Actions](/integrations/github-actions), [GitLab CI](/integrations/gitlab-ci-components), [CircleCI](/integrations/circleci-orbs), [Bitbucket Pipelines](/integrations/bitbucket-pipes), and [Azure DevOps](/integrations/azure-devops).

### Automated Drift Detection using Atlas Schema Monitoring[​](#automated-drift-detection-using-atlas-schema-monitoring "Direct link to Automated Drift Detection using Atlas Schema Monitoring")


Atlas [Schema Monitoring](/monitoring) provides continuous, automated drift detection. It continuously compares your database schema with the expected state in real-time with minimal performance impact. You can compare database-to-database or database-to-deployment to catch drift from any source.

When drift is detected, Atlas provides instant alerts via Slack notifications or webhook integrations. You get visual ERD diffs showing exactly what drifted, along with HCL and SQL representations of the differences. Atlas also generates an SQL plan to migrate either the code or the database to the desired state, making remediation straightforward. A complete audit trail tracks all detected drifts over time.

![SQL Diff View for a Detected Drift](/assets/images/diff-sql-bdca4cf8764026a70be5167a63862a60.png)

### Schema Monitoring Features[​](#schema-monitoring-features "Direct link to Schema Monitoring Features")


Beyond drift detection, Atlas Schema Monitoring provides:

*   **Live schema visibility** with auto-generated ER diagrams and documentation for every monitored database
*   **Schema statistics** including table sizes, row counts, and index usage to track database growth over time
*   **Schema changelog** tracking all changes over time, making it easy to triage schema-related issues
*   **Flexible deployment** via both agent-less ([GitHub Actions](/monitoring/github-quickstart), [GitLab CI](/monitoring/gitlab-quickstart), [Bitbucket Pipelines](/monitoring/bitbucket-quickstart)) and agent-based ([Atlas Agent](/monitoring/quickstart) with [Helm](/monitoring/helm) or [Terraform](/monitoring/terraform)) setups
*   **Webhook integrations** for custom alerting and automation workflows

See [Schema Monitoring](/monitoring) for the full feature set and [Drift Detection](/monitoring/drift-detection) for setup instructions.

## Generating Migrations[​](#generating-migrations "Direct link to Generating Migrations")


Unlike Liquibase's `diff-changelog` which generates changelog files, Atlas automatically generates clean SQL migration scripts using the `atlas migrate diff` command. You define the desired state and Atlas computes the exact SQL needed to get there.

### Automatic Migration Planning[​](#automatic-migration-planning "Direct link to Automatic Migration Planning")


Atlas can generate migrations from any schema source. Instead of manually writing migration scripts, you define the desired state and Atlas computes the exact SQL needed to transition your database:

*   From SQL Schema
*   From HCL Schema
*   From Database
*   From ORM
```codeBlockLines_AdAo
atlas migrate diff <migration_name> \  --dir "file://migrations" \  --to "file://schema.sql" \  --dev-url "docker://postgres/16/dev"
```
```codeBlockLines_AdAo
atlas migrate diff <migration_name> \  --dir "file://migrations" \  --to "file://schema.hcl" \  --dev-url "docker://mysql/8/dev"
```
```codeBlockLines_AdAo
atlas migrate diff <migration_name> \  --dir "file://migrations" \  --to "mysql://user:pass@localhost:3306/source_db" \  --dev-url "docker://mysql/8/dev"
```
Atlas can load schemas directly from your ORM models and generate migrations automatically:
```codeBlockLines_AdAo
atlas migrate diff <migration_name> \  --dir "file://migrations" \  --to "<orm_url>" \  --dev-url "docker://postgres/16/dev"
```
Supported ORMs include:

*   **Go**: [GORM](/guides/orms/gorm), [Ent](https://entgo.io/docs/versioned-migrations), [Beego](/guides/orms/beego), [Bun](/guides/orms/bun)
*   **Python**: [SQLAlchemy](/guides/orms/sqlalchemy), [Django](/guides/orms/django)
*   **Node.js**: [Sequelize](/guides/orms/sequelize), [TypeORM](/guides/orms/typeorm), [Prisma](/guides/orms/prisma), [Drizzle](/guides/orms/drizzle)
*   **Other**: [Hibernate](/guides/orms/hibernate), [Doctrine](/guides/orms/doctrine), [EF Core](/guides/orms/efcore)

See [ORM Integrations](/orms) for setup instructions.

This creates a versioned migration file with the exact SQL statements:

migrations/20241225120000\_add\_users\_table.sql
```codeBlockLines_AdAo
CREATE TABLE "users" (  "id" bigint NOT NULL,  "name" varchar(255) NOT NULL,  PRIMARY KEY ("id"));
```
See [Versioned Migrations](/versioned/intro) for the complete workflow including linting and applying migrations.

## Summary[​](#summary "Direct link to Summary")


Atlas provides modern, flexible alternatives to Liquibase's diff and diff-changelog commands. Unlike Liquibase which requires two database connections and produces changelog files, Atlas can compare any schema source to any other databases, SQL files, HCL files, ORM models, and migration directories — all interchangeably.

Key capabilities include:

*   **Schema Comparison (`atlas schema diff`)** - Compare any two schema sources and produce a detailed diff. Supports multiple output formats including SQL migrations, HCL schema files, JSON for programmatic processing, and interactive ERD visualizations with the `--web` flag.

*   **Schema Inspection (`atlas schema inspect`)** - Capture the current state of any database and export it to version-controlled schema files in SQL or HCL format. This enables treating your database schema as code, with full support for Git workflows and code review.

*   **Automatic Migration Generation (`atlas migrate diff`)** - Generate versioned SQL migration files automatically by comparing your desired schema (from code, ORM, or HCL) against your migration directory. Atlas computes the exact SQL statements needed to reach the target state.

*   **Continuous Schema Monitoring** - Detect schema drift in real-time with Atlas Cloud's monitoring capabilities. Get instant alerts when production schemas diverge from expected states, view visual diffs between environments, and generate SQL plans to remediate drift.

*   **Native CI/CD Integration** - Built-in support for GitHub Actions, GitLab CI, CircleCI, Bitbucket Pipelines, Azure DevOps, Kubernetes Operators, Argo CD, and Terraform. Automate schema comparisons, drift detection, and migration deployment as part of your existing workflows.

*   [Quick Comparison](#quick-comparison)
*   [Comparing Database Schemas with Atlas](#comparing-database-schemas-with-atlas)
    *   [Compare Two Databases](#compare-two-databases)
    *   [Compare Database to Schema Code](#compare-database-to-schema-code)
    *   [Compare Any Source to Any Other](#compare-any-source-to-any-other)
*   [Inspecting and Snapshotting Schemas](#inspecting-and-snapshotting-schemas)
    *   [Output Formats](#output-formats)
*   [Visualize with ERD](#visualize-with-erd)
    *   [Visual Schema Diff](#visual-schema-diff)
*   [Drift Detection](#drift-detection)
    *   [CLI Drift Check](#cli-drift-check)
    *   [Automated Drift Detection using Atlas Schema Monitoring](#automated-drift-detection-using-atlas-schema-monitoring)
    *   [Schema Monitoring Features](#schema-monitoring-features)
*   [Generating Migrations](#generating-migrations)
    *   [Automatic Migration Planning](#automatic-migration-planning)
*   [Summary](#summary)