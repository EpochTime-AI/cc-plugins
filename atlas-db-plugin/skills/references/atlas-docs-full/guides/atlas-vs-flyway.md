Atlas vs Flyway (Redgate): Why Modern Teams Choose Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Modern database development requires deterministic planning, end-to-end automation, and guardrails that prevent outages. Atlas is a schema-as-code system that supports both declarative and versioned workflows, integrates deeply with CI/CD, and offers integrated policy and testing frameworks. Flyway is a traditional migration runner that executes SQL scripts in order.

This document summarizes Atlas's capabilities and compares them with Flyway's to help you decide which tool is best for your team.

Full Disclosure

This document is maintained by the Atlas team and was last updated in September 2025. It may contain outdated information or mistakes. For Flyway's latest details and their own comparison, please refer to the official Flyway (Redgate) website.

## Quick Comparison[​](#quick-comparison "Direct link to Quick Comparison")


Feature / Capability

Atlas

Flyway (Redgate)

**Workflows**

Declarative (state-based) and versioned

Versioned (manual SQL files). State-based supported in Teams/Enterprise editions, implemented as schema diffing via Schema Model

**Migration Planning**

Automatic diff-based planning with policy-awareness and checks

Manual SQL. Diffing and script generation available only in state-based Enterprise workflow

**Rollback / Down Migrations**

Dynamic, state-aware rollback with safety checks

Optional manual down scripts. Partial failures not handled automatically

**Validation & Linting**

Built-in analyzers, linting commands, policy checks, CI/CD enforcement

Not available

**Testing Framework**

Unit-style schema/data-migration tests. Tests can run locally and in CI/CD, with auto-generation supported

Not available

**Policy Enforcement**

Write custom rules using Atlas syntax (e.g., naming conventions, constraints, no-FK policies)

Not available

**CI/CD Integration**

Native GitHub, GitLab, Azure DevOps, Bitbucket, CircleCI actions

Requires custom scripting or plugins

**Kubernetes Integration**

Official Kubernetes Operator with CRDs (`AtlasSchema`, `AtlasMigration`), GitOps-ready, Argo CD supported

No official operator

**Terraform Integration**

Official Terraform Provider (`atlas_schema`, `atlas_migration`)

No provider

**Cloud Registry & Docs**

Atlas Cloud with ERDs, SOC2-audited history, drift detection, PR checks

Not available

**Multi-tenant Migrations**

Built-in support for DB-per-tenant and schema-per-tenant

Not supported

**Secrets Manager Integration**

Supports environment variables and standard secret managers (AWS, GCP, Azure, etc.) in free version (Vault support in Pro)

Enterprise feature only

**Database Support**

PostgreSQL, MySQL/MariaDB, SQL Server, Oracle, ClickHouse, SQLite, CockroachDB, TiDB, Redshift, Spanner, Snowflake, and more

Broad database compatibility, but support is mostly limited to executing SQL scripts without deeper schema management functionality

**AI Integration**

Atlas Copilot built-in assistant for schema guidance and testing, plus integration with Copilot, Cursor, and Claude Code (all executed through Atlas's deterministic engine)

Not available

## Migration Workflows: Declarative and Versioned[​](#migration-workflows-declarative-and-versioned "Direct link to Migration Workflows: Declarative and Versioned")


Atlas supports two workflows for managing schema changes: declarative (state-based) and versioned (migrations-based). In both workflows, Atlas inspects the "current state," compares it to the "desired state," and plans the necessary statements to reach the desired state using a single deterministic migration planner.

### Current State vs Desired State[​](#current-state-vs-desired-state "Direct link to Current State vs Desired State")


*   **Current state**: In the declarative workflow (Terraform-like workflow), the current state is typically a live database. In the versioned workflow, the current state is the result of applying all migration files in order. Atlas supports both workflows.
*   **Desired state**: The desired state defines the target schema. It can be defined using HCL schema files, SQL schema files, another database, ORM providers (such as Hibernate, GORM, Drizzle, Django, or SQLAlchemy), or a combination of these sources. For example, you can define your core schema in an ORM and augment it with SQL files that add triggers, RLS policies, functions, or other constructs.

### Key Differences Between Atlas and Flyway[​](#key-differences-between-atlas-and-flyway "Direct link to Key Differences Between Atlas and Flyway")


Unlike Flyway, where you write and maintain ordered SQL migration files manually, Atlas automates migration planning based on the difference between the current and desired states.

*   **Declarative** - Atlas compares the desired state to the current database, plans safe migrations based on defined policies, and applies them automatically on the target database.
*   **Versioned** - You define the desired schema in code (or another database), but instead of writing SQL yourself (as in Flyway), you run `atlas migrate diff` to generate migration files that transition the current state of the migrations directory to the desired state.

Both workflows can be automated in CI/CD using Atlas Actions (GitHub, GitLab, Azure DevOps, etc.) or the Atlas CLI. After changes are planned, Atlas validates them with migration linting and policy checks before execution.

Read more at:

*   [Automatic Migration Planning](/versioned/diff)
*   [Schema as Code: SQL syntax](/atlas-schema/sql)
*   [Schema as Code: HCL syntax](/atlas-schema/hcl)
*   [ORM integrations](/orms): Supports, Python, Go, Java, JS/TS, C#, and more.

## Migration Safety, Policy, and Governance[​](#migration-safety-policy-and-governance "Direct link to Migration Safety, Policy, and Governance")


Atlas pioneered a **code-first** methodology for database management. Database logic is treated like application code: it can be linted, validated, and unit-tested after each change. By bringing modern software engineering practices such as static analysis, validation, and automated testing into schema management, Atlas provides a level of safety and reliability not found in traditional migration runners like Flyway:

*   **Testing framework** - Atlas's testing framework lets you write unit-style tests for schema and data migrations. Tests run locally and in CI to catch issues before deployment. Using Atlas Copilot, you can also generate tests automatically.
*   **Policy-aware planning** - Changes are checked against team policies (e.g., create indexes concurrently). Planning is not just about moving from state A to state B, but doing so in a way that respects your team's conventions and requirements.
*   **Validation and analyzers** - The `atlas migrate lint` and `atlas schema lint` commands check migrations for semantic correctness, warn about locking operations, table copies, destructive changes, and incompatible modifications to suggest safer alternatives. Linters run locally and in CI/CD. They respect company policies and work on both migration files and schema definitions. Atlas Actions in CI can block merges if linting fails and support code comments in SQL, HCL, Go, Python, Java, and more.
*   **Policy enforcement** - Write custom rules in the Atlas Rules Language to enforce company policies (e.g., naming conventions, no foreign keys). Policies run during diff, lint, and apply.
*   **Pre-migration checks** - Embed SQL assertions in a migration to verify conditions (e.g., a table is empty before dropping it). If a check fails, the migration aborts.
*   **Migration directory integrity** - Atlas enforces migration history integrity automatically to detect and prevent conflicts that arise when multiple branches introduce overlapping changes.

Read more at:

*   [Pre-migration Checks](/versioned/checks)
*   [Testing Schemas](/testing/schema)
*   [Testing Migration Changesets](/testing/migrate)
*   [Changeset Linting](/versioned/lint)
*   [Custom Policies](/lint/rules)

## Down Migrations and Rollback[​](#down-migrations-and-rollback "Direct link to Down Migrations and Rollback")


### Rolling Back vs Going Down[​](#rolling-back-vs-going-down "Direct link to Rolling Back vs Going Down")


Rolling back schema changes is often confused with _"going down"_, but the concepts are not the same:

*   **Rollback** - Happens at the transaction level. If a migration fails in a database that supports transactional DDLs (like PostgreSQL), the entire transaction is aborted and the schema returns to its pre-migration state. On databases without transactional DDLs (like MySQL), a failed migration may leave the schema partially applied.
*   _**Going Down**_ - Refers to intentionally reverting previously applied migrations to reach an earlier version or tag. This requires the migration tool to understand the database state and apply reverse changes across multiple files. The tool must generate a short, safe sequence of statements that revert the schema to the desired state, even if the last migration was partially applied or failed.

### Flyway vs Atlas[​](#flyway-vs-atlas "Direct link to Flyway vs Atlas")


*   **Flyway** - Down scripts are optional and must be written manually. They assume the "up" migration succeeded fully and do not handle partial failures. In practice, they are rarely used and can be destructive.
*   **Atlas** - The `atlas migrate down` command computes a rollback plan dynamically. It inspects the current database state and generates the exact sequence of statements required to revert to a chosen version, tag, or number of steps back.
    *   Works even if the last migration was partially applied or failed.
    *   Runs pre-migration checks by default to prevent destructive operations.
    *   Supports `--dry-run` for previews, and in Atlas Cloud you can enforce review/approval policies before execution.
    *   Executes transactionally when supported, or statement-by-statement with proper history tracking.
    *   Approval workflows can be enforced in CI/CD or Atlas Cloud for down migrations.

Read more at:

*   [Down Migrations](/versioned/down)
*   [Atlas Blog: _The Myth of Down Migrations_](/blog/2024/04/01/migrate-down)

## CI/CD Integration and Platform Fit[​](#cicd-integration-and-platform-fit "Direct link to CI/CD Integration and Platform Fit")


Unlike Flyway and other traditional tools, Atlas is designed for modern CI/CD workflows and integrates deeply with popular tools:

*   **Kubernetes and Terraform** - A Kubernetes operator and Terraform provider let you manage schema changes declaratively alongside your infrastructure. This enables GitOps-style deployments where the database state is version-controlled and automatically applied, just like application code or infrastructure settings.
*   **CI/CD integrations** - GitHub, GitLab, CircleCI, Bitbucket, and Azure DevOps actions for easy integration. These integrations support all Atlas workflows, provide PR/code comments, run summaries, and can enforce policies or block merges if migration linting fails. This ensures schema changes are reviewed and tested before deployment.
*   **Webhooks and alerts** - Integrate schema drift and migration events into your existing monitoring or approval systems. Atlas Cloud can notify Slack channels, trigger webhooks, or connect to incident management tools when issues arise, giving teams visibility and control over production changes.
*   **GitOps with Argo CD** - Manage schemas declaratively with the Atlas Kubernetes Operator and sync them via Argo CD. Use Git as the single source of truth, apply schema manifests in sync waves (database → schema → app), and define custom health checks for `AtlasSchema` resources to ensure safe, ordered rollouts of database changes. This approach makes schema deployments reproducible, observable, and fully aligned with the GitOps workflow.

Read more at:

*   CI integration: [GitHub Actions](/integrations/github-actions), [CircleCI](/integrations/circleci-orbs), [GitLab CI Components](/integrations/gitlab-ci-components), [BitBucket Pipelines](/integrations/bitbucket-pipes).
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

Flyway doesn't provide an official Kubernetes Operator.

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


Atlas offers a first-class Terraform provider that makes database schemas part of your Infrastructure-as-Code workflows. With support for both declarative and versioned migration modes, teams can choose the workflow that best fits their delivery model.
```codeBlockLines_AdAo
resource "atlas_schema" "myapp" {  hcl        = file("schema.hcl")  url        = var.database_url  dev_db_url = "docker://postgres/15/dev"}
```
Capabilities and features:

*   **Two workflows supported** - Use the `atlas_schema` resource for declarative workflows or the `atlas_migration` resource for versioned workflows. This gives teams flexibility to manage the desired state or explicit migration histories within Terraform, respectively.
*   **Infrastructure-as-Code for schemas** - Schemas are defined as Terraform resources and managed alongside application and infrastructure code.
*   **State management and drift detection** - Atlas checks whether the live database matches the declared schema and reconciles differences automatically.
*   **Ad-hoc approvals** - Declarative schema changes can be gated with approval flows, ensuring sensitive or production environments require explicit review before applying changes.
*   **Native Terraform integration** - Works seamlessly with existing Terraform workflows, variables, external data sources, and modules.

Flyway has no official Terraform provider.

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


Flyway offers compatibility with a wide range of databases, many of which are supported through community-provided drivers that focus primarily on executing SQL scripts.

Atlas, by design, supports a more focused set of databases but provides deeper functionality for each: schema inspection and diffing, automatic migration planning, policy enforcement, testing, and drift detection.

Databases supported by Atlas currently include PostgreSQL, MySQL/MariaDB, ClickHouse, SQL Server, Oracle, SQLite, CockroachDB, TiDB, Redshift, Spanner, Snowflake, and others.

## Why Teams Replace Flyway with Atlas[​](#why-teams-replace-flyway-with-atlas "Direct link to Why Teams Replace Flyway with Atlas")


Teams adopt Atlas for deterministic planning and end-to-end safety:

1.  **Declarative state** - Define the desired schema once (HCL, SQL, or ORM) and let Atlas plan the migration.
2.  **Dynamic rollback** - Compute state-aware down plans and handle partial failures.
3.  **Policy as code** - Enforce conventions and block unsafe changes with custom rules.
4.  **Integrated testing and checks** - Catch errors before deployment with linting, assertions, and tests.
5.  **Centralized docs and drift monitoring** - Keep a single source of truth, generate ERDs, and receive drift alerts.
6.  **Multi-tenant migrations** - Manage migrations for thousands of databases with a single workflow.
7.  **Conflict detection** - Ensure migration history integrity across branches and avoid drift caused by merge conflicts.
8.  **AI tooling** - Safely leverage Copilot, Cursor, and Claude Code with deterministic execution.
9.  **Better configuration ergonomics** - The `atlas.hcl` configuration supports flexible environment definitions, dev and prod separation, support for Terraform-like data-sources, external schema sources, composite schemas, etc. This makes managing multiple environments easier.
10.  **GitOps & Kubernetes Operator support** - Use the Atlas Operator in Kubernetes with tools like ArgoCD to treat schema changes as part of your declarative infrastructure. You can version the schema in Git, tag migration directories, and deploy via Kubernetes resources.
11.  **Rigorous validation & linting rules** - Reduce the risk of unintended consequences in production using Atlas's many built-in analyzers and linters that check for destructive changes, table locking, and data-dependent alterations.

## Summary[​](#summary "Direct link to Summary")


Both Atlas and Flyway serve their purpose in the ecosystem. Flyway established important patterns for database migrations. Atlas builds on the versioned migrations foundations to provide a modern, cloud-native approach with comprehensive safety features and infrastructure integration.

*   [Quick Comparison](#quick-comparison)
*   [Migration Workflows: Declarative and Versioned](#migration-workflows-declarative-and-versioned)
    *   [Current State vs Desired State](#current-state-vs-desired-state)
    *   [Key Differences Between Atlas and Flyway](#key-differences-between-atlas-and-flyway)
*   [Migration Safety, Policy, and Governance](#migration-safety-policy-and-governance)
*   [Down Migrations and Rollback](#down-migrations-and-rollback)
    *   [Rolling Back vs Going Down](#rolling-back-vs-going-down)
    *   [Flyway vs Atlas](#flyway-vs-atlas)
*   [CI/CD Integration and Platform Fit](#cicd-integration-and-platform-fit)
    *   [Kubernetes Native](#kubernetes-native)
    *   [Terraform Provider](#terraform-provider)
*   [Atlas Cloud: Registry, Docs, and Drift Monitoring](#atlas-cloud-registry-docs-and-drift-monitoring)
*   [Multi-tenant Migrations](#multi-tenant-migrations)
*   [Atlas and AI Tools](#atlas-and-ai-tools)
*   [Database Support: Depth vs Breadth](#database-support-depth-vs-breadth)
*   [Why Teams Replace Flyway with Atlas](#why-teams-replace-flyway-with-atlas)
*   [Summary](#summary)