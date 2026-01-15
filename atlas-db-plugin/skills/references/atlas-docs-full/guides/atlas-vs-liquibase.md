Atlas vs Liquibase: Why Modern Teams Choose Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Modern database development demands deterministic planning, end-to-end automation and strong safety rails. **Atlas** provides a schema-as-code engine that supports both declarative and versioned workflows and tightly integrates with CI/CD tooling. **Liquibase** is a classic migration runner that uses _changelogs_\-files where users manually define ordered changesets. This document summarizes Atlas's capabilities and contrasts them with Liquibase so that you can choose the right tool for your team.

Full Disclosure

This document is maintained by the Atlas team and was last updated in September 2025. It may contain outdated information or mistakes. For Liquibase's latest details and their own comparison, please refer to the official Liquibase website.

## Quick Comparison[​](#quick-comparison "Direct link to Quick Comparison")


Feature / Capability

Atlas

Liquibase

**Workflow Type**

Declarative (state-based) and versioned

Migration-based only - users maintain ordered change sets

**Source-of-Truth**

Schema files in HCL or SQL; can also load from ORMs (Hibernate, Django, etc.), database URLs or a mix of these sources

Changelog files in XML, YAML, JSON or formatted SQL

**Migration Planning**

Automatic diff-based planner; generates migrations from the difference between current and desired state

Manual - user writes change sets. Liquibase's `diff-changelog` command can generate a changelog from two databases, but it is a manual operation and part of the CLI

**Rollback**

Dynamic, state-aware rollback with safety checks; no down scripts required

Built-in rollback commands (by tag, date or count) but only certain change types support auto-generated rollback; others require custom definitions

**Validation & Linting**

Built-in analyzers and policy engine; enforces naming conventions, detects destructive or locking operations

Limited - Liquibase validates checksums but does not provide semantic linting or a policy engine

**Testing Framework**

Unit-style tests for schemas and data migrations; can auto-generate tests

None

**Policy Enforcement**

Declarative policy language to enforce custom rules

No native policy language

**CI/CD Integration**

Native GitHub Actions, GitLab components, CircleCI orbs, Azure DevOps; Kubernetes operator and Terraform provider for GitOps

CLI, Maven, Gradle and Ant plugins; official GitHub Action; Jenkins and GitLab integrations; Docker container and Spring Boot integration

**Drift Detection**

Built-in drift detection with ERD diffs and webhooks

`diff-changelog` can report drift manually; Liquibase does not provide continuous drift detection or integrated drift reporting

**Multi-tenant Support**

Built-in database-per-tenant and schema-per-tenant migrations

Not natively supported

**Secrets Management**

Use environment variables or external secret managers (AWS, GCP, Azure, Vault, etc.)

Use properties files or environment variables; no first-class integration with secret managers

**Database Support**

Supports PostgreSQL, MySQL/MariaDB, SQL Server, Oracle, ClickHouse, SQLite, CockroachDB, TiDB, Redshift, Snowflake, Spanner, [and more](/features#database-support)

Works with 60 relational, NoSQL and graph databases

**Requirements**

Distributed as a static Go binary and Docker image (~63 MB)

Java-based CLI; requires Java 8 or newer (the installer includes a JVM)

**AI Integration**

Atlas Copilot provides schema suggestions and test generation

None

## Migration Workflows[​](#migration-workflows "Direct link to Migration Workflows")


### Declarative vs Migration-based[​](#declarative-vs-migration-based "Direct link to Declarative vs Migration-based")


Atlas supports both [declarative (state-based)](/declarative/apply) and [versioned (migration-based)](/versioned/apply) workflows. In declarative mode, you express your desired schema in [HCL](/atlas-schema/hcl), [SQL](/atlas-schema/sql), an [ORM model](/orms), a database URL, or any mix of these sources. Atlas then computes a migration plan by diffing your desired state against the live database, using a deterministic engine to plan and apply the transition based on your company policies.

In versioned mode, Atlas compares the desired schema against the current migration directory and generates new migration files with [`atlas migrate diff`](/versioned/diff). The same deterministic engine powers both workflows, which can be executed as part of modern CI/CD pipelines.

Liquibase, by contrast, is strictly migration-based. You maintain a hand-written changelog of ordered _changesets_, each specifying a change (for example `createTable`, `addColumn`) and optional preconditions or labels. When you run `update`, Liquibase records each new changeset in the `DATABASECHANGELOG` table. If a changeset’s checksum differs from its previous record, Liquibase halts execution.

[

##### Declarative vs. Versioned Workflows


Read more about the differences between declarative and versioned migration workflows.

](/concepts/declarative-vs-versioned)

### Automatic Migration Planning[​](#automatic-migration-planning "Direct link to Automatic Migration Planning")


Atlas automatically plans migrations based on **schema diffing**. You run `atlas schema apply` against a live database or `atlas migrate diff` against a migration directory; Atlas inspects the current schema, compares it to the desired schema, and emits a series of SQL statements that respect team policies. This eliminates human error and ensures deterministic, idempotent migrations. Atlas can generate partial or complete migration plans and supports complex objects like triggers, functions and views.

Liquibase offers a `diff-changelog` command that compares two databases and produces a changelog with deployable changesets. This can help generate migrations when synchronizing environments or detecting drift. However, `diff-changelog` is an ad-hoc tool; the typical Liquibase workflow still requires developers to design each change manually and commit it to the changelog. Liquibase does not provide a declarative mode where you describe the desired end state and let the tool compute the plan.

Read more at:

*   [Automatic Migration Planning](/versioned/diff)
*   [Schema as Code: SQL syntax](/atlas-schema/sql)
*   [Schema as Code: HCL syntax](/atlas-schema/hcl)
*   [ORM integrations](/orms): Supports Python, Go, Java, JS/TS, C#, and more.

## Migration Safety, Policy and Governance[​](#migration-safety-policy-and-governance "Direct link to Migration Safety, Policy and Governance")


Atlas treats database schemas as code. Before applying changes, it runs **linting and policy checks** to detect destructive operations, table locks, potential constraint violations, and other risks. Teams can define and enforce **custom policies** (for example, naming conventions or no-FK rules) directly in CI/CD pipelines. Atlas also supports **[pre-migration checks](/versioned/checks)**\-SQL assertions that run before applying a migration-and enforces **migration directory integrity** to prevent conflicts or divergence between environments.

Liquibase validates migration integrity by computing a checksum for each changeset and storing it in the `MD5SUM` column of the `DATABASECHANGELOG` table. If a changeset has been modified after deployment, Liquibase recomputes the checksum and aborts the update on mismatch. It also supports preconditions inside changesets (similar to pre-migration checks in Atlas), allowing migrations to stop when certain conditions are not met. However, Liquibase does not provide semantic linting (for example, detecting locking or destructive operations) or a declarative policy engine for enforcing safety and compliance rules.

Read more at:

*   [Pre-migration Checks](/versioned/checks)
*   [Testing Schemas](/testing/schema)
*   [Testing Migration Changesets](/testing/migrate)
*   [Changeset Linting](/versioned/lint)
*   [Custom Policies](/lint/rules)

## Down Migrations and Rollback[​](#down-migrations-and-rollback "Direct link to Down Migrations and Rollback")


Rolling back schema changes safely is critical. In the Atlas ecosystem, rollback is **dynamic and state-aware**. The `atlas migrate down` command inspects the current database state and generates the exact SQL required to revert to a previous version or tag. It handles partial failures, runs pre-migration checks and supports transactional or step-by-step execution. Approval workflows can be enforced in CI/CD or Atlas Cloud.

Liquibase supports rollback via several commands - `rollback`, `rollback-to-date` and `rollback-count` - that revert changes after a specified tag, time or number of changesets. Liquibase also provides targeted rollback commands such as `rollback-one-changeset` and `rollback-one-update` and an option to automatically roll back on error using `--rollback-on-error`. **Automatic rollback generation** is limited: Liquibase can auto-generate rollback SQL only for some change types (e.g., `createTable`, `renameColumn`, `addColumn`), but operations like `dropTable` or formatted SQL require you to write custom rollback logic. Developers must maintain rollback statements in their changelog, increasing the chance of divergence between up and down migrations.

Read more at:

*   [Down Migrations](/versioned/down)
*   [Atlas Blog: _The Myth of Down Migrations_](/blog/2024/04/01/migrate-down)

## CI/CD Integration and Platform Fit[​](#cicd-integration-and-platform-fit "Direct link to CI/CD Integration and Platform Fit")


Atlas is built for modern CI/CD and GitOps workflows. There are official GitHub Actions, GitLab components, CircleCI orbs and Bitbucket pipelines for planning, linting, applying and rolling back migrations. Atlas also provides a **Kubernetes Operator** with CRDs (`AtlasSchema` and `AtlasMigration`) and a **Terraform provider** to manage schemas alongside infrastructure. Native webhooks, drift alerts and approval flows are available via Atlas Cloud.

Liquibase integrates with build tools through its CLI, Maven, Gradle and Ant plugins and can be embedded as a Java library. It also offers an official GitHub Action, a Jenkins plugin, and supports GitLab pipelines and Spinnaker. Liquibase provides a Docker container for integration into containerized environments and integrates with Spring Boot via configuration and customizers. However, it does not provide an official Kubernetes operator or Terraform provider; running in Kubernetes typically involves using the CLI in an init container.

Read more at:

*   CI integration: [GitHub Actions](/integrations/github-actions), [CircleCI](/integrations/circleci-orbs), [GitLab CI Components](/integrations/gitlab-ci-components), [Bitbucket Pipelines](/integrations/bitbucket-pipes).
*   [Azure DevOps](/integrations/azure-devops)

### Kubernetes Native[​](#kubernetes-native "Direct link to Kubernetes Native")


Atlas provides a production-ready Kubernetes Operator that uses Kubernetes CRDs (Custom Resource Definitions) to manage schema state as a first-class Kubernetes resource. You can choose between **declarative** or **versioned** workflows, backed by `AtlasSchema` and `AtlasMigration` CRDs respectively.
```codeBlockLines_AdAo
apiVersion: db.atlasgo.io/v1alpha1kind: AtlasSchemametadata:  name: myapp-schemaspec:  url: postgresql://myapp-db:5432/myapp  schema:    sql: |      CREATE TABLE users (        id UUID PRIMARY KEY DEFAULT gen_random_uuid(),        email VARCHAR(255) UNIQUE NOT NULL      );  policy:    lint:      destructive:        error: true
```
Features and capabilities:

*   Support for **both** declarative (desired-state) and versioned (migration file driven) schema workflows via Kubernetes.
*   Installation with flexible configuration including environments, project settings, SSL certificates, secret management.
*   Pre-approval and ad-hoc approval flows for declarative schema changes, so some changes can be gated or automatically approved depending on policy or environment.
*   Automatic drift reconciliation: the operator continuously monitors the actual database vs the declared CRD schema, and reconciles differences.
*   Kubernetes-native patterns: GitOps compatibility, RBAC, leveraging Kubernetes secrets, project configuration injection, separation of environments, etc.

Liquibase doesn't provide an official Kubernetes Operator.

[

##### !GitOps with Argo CD


Deploying to Kubernetes with the Atlas Operator and Argo CD

](/guides/deploying/k8s-argo)[

##### !GitOps with Flux CD


Deploying to Kubernetes with the Atlas Operator and Flux CD

](/guides/deploying/k8s-flux)[

##### !GitOps with Atlas Cloud


Deploying to Kubernetes from Atlas Schema Registry

](/guides/deploying/k8s-cloud-versioned)

### Terraform Provider[​](#terraform-provider "Direct link to Terraform Provider")


Atlas offers a first-class [Terraform provider](https://registry.terraform.io/providers/ariga/atlas/latest/docs) that makes database schemas part of your Infrastructure-as-Code workflows. With support for both declarative and versioned migration modes, teams can choose the workflow that best fits their delivery model.
```codeBlockLines_AdAo
resource "atlas_schema" "myapp" {  hcl        = file("schema.hcl")  url        = var.database_url  dev_db_url = "docker://postgres/15/dev"}
```
Capabilities and features:

*   **Two workflows supported** - Use the `atlas_schema` resource for declarative workflows or the `atlas_migration` resource for versioned workflows. This gives teams flexibility to manage the desired state or explicit migration histories within Terraform, respectively.
*   **Infrastructure-as-Code for schemas** - Schemas are defined as Terraform resources and managed alongside application and infrastructure code.
*   **State management and drift detection** - Atlas checks whether the live database matches the declared schema and reconciles differences automatically.
*   **Ad-hoc approvals** - Declarative schema changes can be gated with approval flows, ensuring sensitive or production environments require explicit review before applying changes.
*   **Native Terraform integration** - Works seamlessly with existing Terraform workflows, variables, external data sources, and modules.

Liquibase has no official Terraform provider.

## Deployment, Runtime and Toolchain[​](#deployment-runtime-and-toolchain "Direct link to Deployment, Runtime and Toolchain")


Atlas is distributed as a small statically linked Go binary (≈63 MB) and a Docker image. It requires no external runtime. You can run Atlas in containers, CI agents or embed it via the Go SDK.

Liquibase is implemented in Java and requires a JVM. According to Liquibase's system requirements, you must provide Java 8 or newer (the installer includes Java, but manual installations require you to supply your own JVM). Liquibase also offers a Java API to run migrations inside your application.

## Schema as Code vs Changelog Files[​](#schema-as-code-vs-changelog-files "Direct link to Schema as Code vs Changelog Files")


Atlas treats your database schema as **code**. You define the desired state in [HCL](/atlas-schema/hcl) or [SQL](/atlas-schema/sql) files, which serve as the single source of truth for your schema. This declarative approach means you can read a file and immediately understand what your database should look like. Atlas can also import schema definitions from [ORMs](/orms) (e.g., Hibernate, GORM, Django, SQLAlchemy) or from existing databases using the `atlas inspect` command, which exports a database schema to HCL/SQL. This makes it easy to adopt Atlas for existing projects and maintain your schema in version control alongside your application code.

Liquibase uses **changelog files** that store ordered changesets in XML, YAML, JSON or SQL formats. Unlike Atlas's declarative schemas, Liquibase has no single source of truth for the schema-the current schema is the accumulated result of applying all changesets in order. To understand what your database should look like, you must mentally (or programmatically) replay all historical changes. This makes it harder to reason about the current state and increases the risk of inconsistencies across environments.

## Atlas Cloud: Registry, Docs, and Drift Monitoring[​](#atlas-cloud-registry-docs-and-drift-monitoring "Direct link to Atlas Cloud: Registry, Docs, and Drift Monitoring")


Atlas Cloud centralizes schema management. When you push schemas and migrations to the registry, you get:

*   **Schema Registry** - A central store for schema versions and migration directories. Each push generates an ER diagram, searchable documentation, and SOC2-audited history of schema and migration changes.
*   **Always-up-to-date docs** - Cloud regenerates human-readable documentation and ERDs whenever a new version is pushed. Documentation is derived from schema and migration files only, ensuring accuracy and avoiding direct database or SCM connections.
*   **Pull-request checks** - Cloud allows running the same linters and policy checks in CI and can block merges if rules are violated.
*   **Drift detection** - Using Atlas Agent or CI actions, you can inspect production databases continuously. If the actual schema drifts from the expected state, Cloud sends alerts with detailed HCL/SQL diffs.
*   **Notifications** - Configure Slack and webhook alerts to be notified of drift, failed migrations, or other events.

Read more at:

*   [Atlas Schema Monitoring: Drift Detection](/monitoring/drift-detection)
*   [Atlas Schema Monitoring: Webhooks](/monitoring/webhooks)

## Multi-tenant Migrations[​](#multi-tenant-migrations "Direct link to Multi-tenant Migrations")


Atlas includes built-in support for managing multi-tenant database environments, commonly used in database-per-tenant and schema-per-tenant architectures. With Atlas, teams can define logical tenant groups to plan and apply schema changes across many databases in a single operation. This simplifies the management of large fleets while ensuring consistency and reducing the risk of drift or deployment errors.

To learn more, check out the [Database-per-Tenant guide](/guides/database-per-tenant/intro).

## Atlas and AI Tools[​](#atlas-and-ai-tools "Direct link to Atlas and AI Tools")


AI tools like GitHub Copilot, Cursor, and Claude Code are great at writing code, but generating database migrations is a different challenge. As schemas grow more complex, ensuring migrations are deterministic, predictable, and aligned with company policies becomes critical.

Atlas solves this problem by letting AI tools focus on editing the schema while Atlas provides the infrastructure for:

*   **Migration Generation** - Producing safe, deterministic migrations automatically.
*   **Migration Validation** - Ensuring migrations follow best practices.
*   **Policy Enforcement** - Enforcing organizational rules on schema changes.
*   **Unit Testing** - Executing test functions, views, and queries written by AI tools, reporting failures, and guiding fixes.
*   **Data Migration Testing** - Allowing AI tools to generate data migrations, seed data, run tests, and detect errors.

**Copilot and Ask Atlas** - Atlas also includes a built-in chat assistant that can answer questions about your project, explain migration errors, generate schema tests, and suggest safer patterns. All commands go through Atlas's deterministic engine - raw SQL is never executed directly.

To learn more, check out the [Atlas with AI Tools](/guides/ai-tools) docs.

[

##### !GitHub Copilot Instructions


Configure GitHub Copilot with Atlas-specific instructions.

](/guides/ai-tools/github-copilot-instructions)[

##### !Cursor Instructions


Set up Cursor with Atlas-specific rules.

](/guides/ai-tools/cursor-rules)[

##### !Claude Code Instructions


Set up Claude Code with Atlas-specific instructions.

](/guides/ai-tools/claude-code-instructions)

## Database Support: Depth vs Breadth[​](#database-support-depth-vs-breadth "Direct link to Database Support: Depth vs Breadth")


Liquibase supports a broad range of databases, many through community-maintained drivers designed primarily for running SQL scripts. It provides a straightforward model for teams that prefer to manage changes manually with changelog files.

Atlas takes a more comprehensive, integrated approach. It delivers deep, first-class support for each database, including schema inspection and diffing, automatic migration planning, policy enforcement, testing, drift detection, and security checks.

Supported databases include PostgreSQL, MySQL/MariaDB, ClickHouse, SQL Server, Oracle, SQLite, CockroachDB, TiDB, Redshift, Spanner, Snowflake, and others. By treating the schema as code, Atlas enables deterministic, policy-driven migrations, automates security and compliance validation, integrates seamlessly with AI development tools such as Cursor and Claude Code, and maintains consistent environments across development and production.

## Conclusion[​](#conclusion "Direct link to Conclusion")


Liquibase has been a well-known tool for managing database changes for many years. Its changelog-based format and broad database support make it suitable for teams that prefer manually authored migration scripts and a Java-based workflow.

Atlas takes a different approach. It treats the database schema as code, automatically plans migrations from a declared desired state, enforces safety, security and compliance policies, and provides a built-in testing framework. With native integrations into modern CI/CD, Kubernetes, and Terraform workflows, Atlas offers a more automated and deterministic alternative for teams adopting database-as-code practices.

*   [Quick Comparison](#quick-comparison)
*   [Migration Workflows](#migration-workflows)
    *   [Declarative vs Migration-based](#declarative-vs-migration-based)
    *   [Automatic Migration Planning](#automatic-migration-planning)
*   [Migration Safety, Policy and Governance](#migration-safety-policy-and-governance)
*   [Down Migrations and Rollback](#down-migrations-and-rollback)
*   [CI/CD Integration and Platform Fit](#cicd-integration-and-platform-fit)
    *   [Kubernetes Native](#kubernetes-native)
    *   [Terraform Provider](#terraform-provider)
*   [Deployment, Runtime and Toolchain](#deployment-runtime-and-toolchain)
*   [Schema as Code vs Changelog Files](#schema-as-code-vs-changelog-files)
*   [Atlas Cloud: Registry, Docs, and Drift Monitoring](#atlas-cloud-registry-docs-and-drift-monitoring)
*   [Multi-tenant Migrations](#multi-tenant-migrations)
*   [Atlas and AI Tools](#atlas-and-ai-tools)
*   [Database Support: Depth vs Breadth](#database-support-depth-vs-breadth)
*   [Conclusion](#conclusion)