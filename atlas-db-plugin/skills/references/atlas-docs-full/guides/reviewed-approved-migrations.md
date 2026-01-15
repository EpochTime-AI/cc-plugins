Enforcing Reviewed and Approved Schema Migrations | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In regulated and compliance-driven environments, ensuring that only thoroughly reviewed and approved database changes reach production is critical. Organizations operating under frameworks like SOC 2, ISO/IEC 27002, PCI DSS, or HIPAA are expected to demonstrate that all code changes follow a controlled review process before deployment. This includes database work: schema changes and migration scripts are code that runs in production, and they must be validated, tested, peer reviewed, and formally approved before release.

Atlas, a [**SOC 2 Type II certified**](/blog/2024/12/24/soc2-atlas-certified-2024) product, provides a comprehensive workflow that enforces these requirements at every stage-from initial development through production deployment. This guide shows how Atlas ensures that no unapproved changes slip through, maintaining security, compliance, and auditability throughout the migration lifecycle.

## Compliance Context[​](#compliance-context "Direct link to Compliance Context")


Multiple compliance frameworks require formal change management for database migrations:

*   **SOC 2** - Requires documented change management procedures, including testing in non-production environments, peer review, and formal approval before production deployment. Auditors expect clear evidence of review processes and approval mechanisms.
*   **ISO/IEC 27002:2022 (Control 8.32)** - Defines requirements for formal change management, including planning, testing, authorization, and maintaining detailed records of all changes.
*   **PCI DSS (Requirements 6.4 and 6.5)** - Mandates separation of duties, change authorization, and documented testing before production deployment of code and configuration changes.
*   **HIPAA Security Rule** - Requires information system activity review and integrity controls to ensure only authorized changes are made to systems containing protected health information.

Atlas helps you meet these requirements through automated validation, policy enforcement, and complete audit trails.

## The Atlas Migration Lifecycle[​](#the-atlas-migration-lifecycle "Direct link to The Atlas Migration Lifecycle")


Atlas enforces a multi-stage workflow that ensures every migration is validated, reviewed, and approved before reaching production:

![databases and their statuses in atlas registry](/u/compliance/migration-lifecycle.png)

## Step 1: Pull Request Validation and Drift Detection[​](#step-1-pull-request-validation-and-drift-detection "Direct link to Step 1: Pull Request Validation and Drift Detection")


When a developer opens a pull request, Atlas inspects the proposed code changes to detect database schema modifications.

*   If migration files were created locally, Atlas automatically detects and validates them, ensuring there's no drift between the declared migrations and the actual schema (for example, ORM models are aligned with the migration files).
*   If no migrations were introduced, Atlas plans the required schema changes automatically and commits the new migration files into the pull request.

Atlas also detects conflicting migrations created by different team members working in parallel, preventing situations where multiple developers create migration files that conflict with each other. This ensures all versions are synchronized and the migration history remains linear, avoiding deployment failures and unexpected behavior.

This workflow automatically validates every proposed code change to the database schema or the migrations directory, posting detailed results directly to the pull request. Developers get immediate feedback on any issues before requesting review.

## Step 2: Automatic Analysis, Linting, and Policy Enforcement[​](#step-2-automatic-analysis-linting-and-policy-enforcement "Direct link to Step 2: Automatic Analysis, Linting, and Policy Enforcement")


During the pull request phase, Atlas automatically analyzes and simulates the proposed migrations to catch issues before they occur. It runs built-in linters that flag destructive or backward-incompatible changes, long-locking operations, and potential SQL injection risks.

### Migration Linting[​](#migration-linting "Direct link to Migration Linting")


Atlas includes a set of [migration analyzers](/versioned/lint) with dozens of built-in checks that flag risky or non-compliant patterns, including:

*   **Destructive changes** - Detects `DROP TABLE`, `DROP COLUMN`, or `TRUNCATE` operations that can cause data loss.
*   **Backward incompatibility** - Flags changes that may break existing code, like renaming columns or adding `NOT NULL` constraints without defaults.
*   **Constraint violations** - Ensures new constraints (unique, foreign key, check) won't fail on existing data.
*   **Performance issues** - Warns about operations that can lock tables, trigger full scans, or copy large datasets.
*   **Incorrect transaction use** - Detects transaction statements that conflict with Atlas's transaction handling or non-transactional migrations.
*   **Data-dependent changes** - Identifies changes that may fail depending on existing data.
*   **Security vulnerabilities** - Detects potential SQL injection risks in dynamic SQL code paths.
*   And more.

![atlas migrate lint](/u/plan-action.png)

### Custom Policy Rules[​](#custom-policy-rules "Direct link to Custom Policy Rules")


In addition to the built-in checks, teams can define [custom schema policies](/lint/rules) to enforce their own standards. Custom rules are written in `.rule.hcl` files using an HCL-based language with `predicate` and `rule` blocks. Common examples include:

*   Requiring every table to have a primary key
*   Ensuring columns are `NOT NULL` or have default values
*   Enforcing `ON DELETE CASCADE` on foreign keys
*   Marking PII columns with specific comments for compliance tracking
*   Requiring indexes on foreign key columns for performance
*   Enforcing naming conventions for tables, columns, and indexes

schema.rule.hcl
```codeBlockLines_AdAo
predicate "table" "has_primary_key" {  primary_key {    condition = self != null  }}rule "schema" "require-primary-key" {  description = "All tables must have a primary key for data integrity and replication"  table {    assert {      predicate = predicate.table.has_primary_key      message   = "Table ${self.name} must have a primary key"    }  }}
```

## Step 3: Continuous Delivery of Approved Migrations[​](#step-3-continuous-delivery-of-approved-migrations "Direct link to Step 3: Continuous Delivery of Approved Migrations")


Once a pull request passes all automated checks and is approved, the migration is ready to be packaged and stored in Atlas Registry, AWS S3, your codebase, or any other storage. Atlas ensures that only approved migrations reach production through its [Schema Registry](/cloud/features/registry), the central repository for all verified migration artifacts.

The Atlas Registry provides immutable, schema-aware, versioned storage that guarantees only approved migrations can be deployed. When a migration is packaged and pushed to the Registry, it becomes the single source of truth for that migration version.

![atlas migrate push](/u/cloud/images/dir-overview-v1.png)

Migration Directory created with `atlas migrate push`

### Packaging Verified Artifacts[​](#packaging-verified-artifacts "Direct link to Packaging Verified Artifacts")


Once a pull request passes all validation checks and is approved, the migration is ready for deployment. When the PR is merged into the main branch, Atlas automatically packages the approved migrations and pushes them to the Atlas Registry, creating an immutable, versioned artifact that becomes the single source of truth for that migration version.

For example, the code below shows how to push approved migrations to the Atlas Registry using GitHub Actions:

.github/workflows/ci-atlas.yaml
```codeBlockLines_AdAo
- uses: ariga/atlas-action/migrate/push@v1  if: github.ref == 'refs/heads/main'  with:    dir: 'file://migrations'    dev-url: ${{ secrets.DEV_DATABASE_URL }}    dir-name: 'my-app'
```
This immutable artifact ensures that the exact migration version that passed review is the only one deployable to production, preventing unauthorized changes and preserving a complete audit trail for compliance reporting.

## Step 4: Deploying Only Approved Migrations[​](#step-4-deploying-only-approved-migrations "Direct link to Step 4: Deploying Only Approved Migrations")


When deploying to production, Atlas verifies that migrations originate from the approved artifact created in earlier steps. This process ensures that only reviewed and approved migrations can be applied to production databases, preserving compliance and blocking unauthorized changes.

Atlas supports multiple deployment methods and storage backends. You can deploy from the Atlas Registry, AWS S3, your SCM system (e.g., Azure Repos or GitHub), or any other supported storage. It integrates seamlessly with popular CI/CD tools and platforms such as [GitHub Actions](/integrations/github-actions), [GitLab CI/CD](/guides/ci-platforms/gitlab-versioned), [Azure DevOps](/integrations/azure-devops), [Kubernetes Operator](/integrations/kubernetes) with [Argo CD](/guides/deploying/k8s-argo), [Terraform Provider](/integrations/terraform-provider), [Helm](/guides/deploying/helm), and [Flux CD](/guides/deploying/k8s-flux).

Example deployment using GitHub Actions:

.github/workflows/deploy-prod.yaml
```codeBlockLines_AdAo
name: Deploy to Productionon:  push:    branches:      - mainjobs:  deploy:    runs-on: ubuntu-latest    environment: production  # Requires approval in GitHub    steps:      - uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      # Apply migrations from the verified registry      - uses: ariga/atlas-action/migrate/apply@v1        with:          dir: 'atlas://my-app'  # References the registry          url: ${{ secrets.PROD_DATABASE_URL }}
```
By referencing `atlas://my-app` (or your configured storage location) instead of local files, deployments pull migrations directly from verified artifact storage. This ensures immutability and traceability, as only approved migrations can be applied.

How it works

*   After PR approval and merge, migrations are pushed to the Atlas Registry (or your configured storage) as verified artifacts.
*   Production deployments reference `atlas://repo-name` (or your storage URL) to pull from that verified location.
*   Atlas validates that only migrations from the approved artifact are applied, blocking any unauthorized changes.

### Auditing and History[​](#auditing-and-history "Direct link to Auditing and History")


Atlas automatically maintains a comprehensive audit trail of every migration applied to your databases. This audit trail is essential for compliance frameworks and provides full traceability - not just for migration authoring and approval workflows, but also for the actual execution of migrations in each environment.

Below, the **Deployment Trace** view provides a clear, end-to-end record of how a migration progressed through your environments. You can see when and where each version was applied, which databases were affected, and whether all instances completed successfully. Each step is linked to its originating pull request and CI run, giving teams full visibility into who approved, merged, and deployed every change.

![deployment trace](/u/compliance/deployment-trace.png)

## Next Steps[​](#next-steps "Direct link to Next Steps")


### Webhook Integrations for Notifications[​](#webhook-integrations-for-notifications "Direct link to Webhook Integrations for Notifications")


To improve visibility and enable real-time monitoring, Atlas supports [webhook integrations](/cloud/directories#slack-integration) that notify your team about key CI/CD events. You can configure webhooks to send notifications to Slack or custom endpoints when:

*   A new migration is pushed to the Schema Registry
*   A migration is successfully applied to a database
*   A deployment fails or encounters errors
*   Schema drift is detected in an environment
*   Policy violations are found during linting
*   And more

## Summary[​](#summary "Direct link to Summary")


Enforcing reviewed and approved migrations is a cornerstone of compliant database change management. Atlas provides a comprehensive, automated workflow that:

*   **Validates every proposed change** through automated drift detection and consistency checks
*   **Applies organizational policies** using customizable linting rules and pre-execution checks
*   **Ensures only approved migrations reach production** via verified artifacts in the Schema Registry
*   **Maintains complete audit trails** for compliance reporting and incident investigation

By adopting this workflow, teams can:

*   Meet SOC 2, ISO 27002, PCI DSS, HIPAA, and similar compliance requirements
*   Minimize the risk of data loss or outages caused by unsafe database changes
*   Speed up development by automating review and validation tasks
*   Gain full visibility into schema changes and migrations across environments

*   [Compliance Context](#compliance-context)
*   [The Atlas Migration Lifecycle](#the-atlas-migration-lifecycle)
*   [Step 1: Pull Request Validation and Drift Detection](#step-1-pull-request-validation-and-drift-detection)
*   [Step 2: Automatic Analysis, Linting, and Policy Enforcement](#step-2-automatic-analysis-linting-and-policy-enforcement)
    *   [Migration Linting](#migration-linting)
    *   [Custom Policy Rules](#custom-policy-rules)
*   [Step 3: Continuous Delivery of Approved Migrations](#step-3-continuous-delivery-of-approved-migrations)
    *   [Packaging Verified Artifacts](#packaging-verified-artifacts)
*   [Step 4: Deploying Only Approved Migrations](#step-4-deploying-only-approved-migrations)
    *   [Auditing and History](#auditing-and-history)
*   [Next Steps](#next-steps)
    *   [Webhook Integrations for Notifications](#webhook-integrations-for-notifications)
*   [Summary](#summary)