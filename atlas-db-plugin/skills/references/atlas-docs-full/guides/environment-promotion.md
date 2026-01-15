Environment Promotion and Compliance | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

## Overview[​](#overview "Direct link to Overview")


Environment promotion is a core part of modern database change management. It ensures that database and schema changes are tested and validated in lower environments, such as development and staging, before being applied to production. For teams operating under compliance frameworks like **SOC 2** or **ISO/IEC 27002**, enforcing environment segregation and controlled promotion is critical to maintaining security, integrity, and auditability.

Atlas, a [**SOC 2 Type II certified**](/blog/2024/12/24/soc2-atlas-certified-2024) product, provides built-in capabilities for defining and enforcing environment promotion workflows, validating schema states, and maintaining a complete, auditable history of every database change.

## Compliance Context[​](#compliance-context "Direct link to Compliance Context")


Several compliance frameworks emphasize strict environment segregation and controlled change management. For example:

*   **SOC 2** - Requires documented change management procedures that include testing, peer review, and approval before deploying to production. Auditors often expect SOC 2-audited organizations to maintain distinct environment stages (e.g., Dev, QA, Staging, Prod), enforce manual approval before production, and provide evidence of segregation and control. Database schema changes must follow the defined promotion flow and cannot skip lower environments.
*   **ISO/IEC 27002:2022 (Controls 8.31 and 8.32)** - Control 8.31 mandates separation between development, testing, and production environments, requiring explicit approval for any testing in production. Control 8.32 defines formal change management requirements, including planning, authorization, testing, and maintaining detailed records of all changes.
*   **PCI DSS** and **NIST SP 800-53** - Recommend comparable practices, including segregation of duties, controlled deployment pipelines, and validation of all changes prior to production rollout.

Although these frameworks do not mandate specific environment names or sequences (such as Dev → Staging → Prod), they share a common goal: enforcing **controlled environment promotion**. Atlas enables teams to meet these expectations with deterministic workflows, environment-aware policies, and auditable schema promotion.

## Implementing Environment Promotion in Atlas[​](#implementing-environment-promotion-in-atlas "Direct link to Implementing Environment Promotion in Atlas")


Atlas supports enforcing environment promotion workflows using [Data Sources](/atlas-schema/projects#data-sources), [Pre-Execution Checks](/versioned/apply#pre-execution-checks), and the [Atlas Registry](/cloud/features/registry). These capabilities allow teams to define workflows where production deployments depend on the state of lower environments. For example, a production deployment can be configured to only proceed if the deployed version matches the one applied in Staging or Dev. Similarly, attempts to promote a version that was not deployed and verified in Staging can be automatically blocked.

The guide below demonstrates how to configure an environment promotion workflow using the `cloud_databases` data source and pre-execution checks.

### Example: Promotion from Dev to Prod[​](#example-promotion-from-dev-to-prod "Direct link to Example: Promotion from Dev to Prod")


The following example shows two databases whose migration deployments have been reported to the Atlas Registry. Note that this reporting is disabled by default. To enable it, follow the instructions [here](/cloud/deployment).

![databases and their statuses in atlas registry](/u/compliance/db-status.png)

The following configuration defines an environment promotion workflow from `development` to `production`, ensuring production only advances to a version verified in development. The `cloud_databases` data source retrieves from the Atlas Registry the current migration version applied to development, and set it as the `to_version` for production. This ensures that production can only be promoted to the version currently deployed in development.

Why is this necessary? By default, `atlas migrate apply` will attempt to apply all pending migrations to the target database. If production is ahead of development, this could lead to untested migrations being applied directly to production.

atlas.hcl
```codeBlockLines_AdAo
data "cloud_databases" "dev" {  repo = "enviroment promotion"  env  = "development"}env "production" {  url = var.url  migration {    dir = "atlas://enviroment-promotion"    # Promote the migration version from the `development` environment.    to_version = data.cloud_databases.dev.targets[0].current_version  }}
```
How it works

*   The `cloud_databases` data source retrieves the current migration version of the Dev environment from Atlas Cloud.
*   The `to_version` attribute is set in the `production` environment to the version currently deployed in `development`.
*   This prevents direct production changes and enforces a progressive deployment workflow.

### Example: Pre-Execution Check for Promotion[​](#example-pre-execution-check-for-promotion "Direct link to Example: Pre-Execution Check for Promotion")


Another option to ensure controlled promotion is to use a pre-execution check that verifies the target version before applying migrations to production. If the planned migration version is higher than the version deployed in staging, the deployment is blocked with an informative error message.

atlas.hcl
```codeBlockLines_AdAo
data "cloud_databases" "staging" {  repo = "enviroment promotion"  env  = "staging"}locals {  staging_version = data.cloud_databases.staging.targets[0].current_version}env "production" {  url = var.url  migration {    dir = "atlas://enviroment-promotion"  }  check "migrate_apply" {    deny {      condition = self.planned_migration.target_version > local.staging_version      message   = <<-MSG  Production cannot be promoted to a version higher than staging environment.  Current staging version: ${local.staging_version}, planned production version: ${self.planned_migration.target_version}  MSG    }  }}
```
Running `atlas migrate apply --env production` will fail if the planned migration version exceeds the one in development:
```codeBlockLines_AdAo
Error: "migrate apply" was blocked by pre-execution check (deny rule: "check_version" at line: 17):  Production cannot be promoted to a version higher than development environment.  Current development version: 20230316085611, planned production version: 20230316090502
```
How it works

*   The pre-execution check retrieves the current migration version of the Staging environment from Atlas Cloud.
*   Before applying migrations to production, it compares the planned target version with the staging version.
*   If the planned version exceeds the staging version, the deployment is blocked with a clear error message.

### Auditing and Migration History[​](#auditing-and-migration-history "Direct link to Auditing and Migration History")


Atlas automatically maintains a detailed **audit trail of all database migrations**, ensuring full visibility and accountability across environments. This audit log is essential for meeting compliance requirements in frameworks like SOC 2, ISO 27002, PCI DSS, and HIPAA. It provides complete traceability, covering migration authoring, review, approval, and execution, so every change can be verified and audited with confidence.

The **Deployment Trace** view offers a clear, end-to-end record of how each migration was deployed across environments. You can track when and where each version was applied, which databases were affected, and whether all executions completed successfully. Every action is linked to its source pull request and CI run, allowing teams to easily trace who approved, merged, and deployed every change.

![Atlas deployment trace view showing migration audit history](/u/compliance/deployment-trace.png)

## Best Practices[​](#best-practices "Direct link to Best Practices")


1.  **Enforce promotion in CI/CD pipelines:** Configure your CI/CD workflows (e.g., GitHub Actions, GitLab CI, Azure DevOps, Argo CD, or Terraform) to ensure migrations are applied in the correct sequence. Use environment dependencies to block production deployments until lower environments have been successfully verified.

2.  **Use Atlas policies to block direct production changes:** Add pre-execution checks to prevent migrations from being applied directly to production without passing through lower environments. This enforces organizational change-control requirements and reduces the risk of human error.

3.  **Integrate audit evidence collection:** Configure Atlas to automatically report and track migration metadata, including deployment history, version information, and drift detection. This data can be exported to generate audit reports for SOC 2, ISO 27002, and similar compliance frameworks.

4.  **Validate migrations in lower environments:** Use [migration linting](/versioned/lint) and [testing](/testing/migrate) to detect issues early in the CI pipeline. Once validated, promote changes to lower environments such as _Dev_ or _Staging_ to catch runtime errors before reaching production.

5.  **Leverage Atlas Cloud for centralized visibility:** Atlas Cloud provides a unified view of schema states across all environments, making it easier to track promotion progress and detect drift. View migration history and deployment status in the [Atlas Cloud UI](https://gh.atlasgo.cloud/migrations).

## Summary[​](#summary "Direct link to Summary")


Environment promotion is a fundamental practice for secure and compliant database change management. Atlas provides native capabilities to automate and enforce progressive deployment workflows, ensuring that database and schema changes are validated before reaching production.

By using Atlas, teams can:

*   Satisfy compliance requirements for environment segregation and controlled deployments.
*   Reduce risk by validating changes in CI pipelines and lower environments first.
*   Maintain a complete, auditable history of all database and schema changes.
*   Automate promotion workflows through data sources and CI/CD integrations.

*   [Overview](#overview)
*   [Compliance Context](#compliance-context)
*   [Implementing Environment Promotion in Atlas](#implementing-environment-promotion-in-atlas)
    *   [Example: Promotion from Dev to Prod](#example-promotion-from-dev-to-prod)
    *   [Example: Pre-Execution Check for Promotion](#example-pre-execution-check-for-promotion)
    *   [Auditing and Migration History](#auditing-and-migration-history)
*   [Best Practices](#best-practices)
*   [Summary](#summary)