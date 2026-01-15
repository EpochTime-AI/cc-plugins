Flyway Undo Alternative in Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Flyway's `undo` feature allows you to revert applied migrations by running manually written undo scripts. While this sounds useful in theory, in practice these pre-written undo (down) files come with significant limitations: they must be written before the migration is applied, they assume the migration succeeded fully, and they're rarely tested in production scenarios.

Atlas takes a fundamentally different approach with its `migrate down` command. Instead of requiring pre-written undo files, Atlas computes rollback plans dynamically based on the actual state of your database. This means Atlas can handle partial failures, work across non-transactional DDL operations, and provide built-in safety checks - all automatically.

## Quick Comparison[​](#quick-comparison "Direct link to Quick Comparison")


Capability

Flyway Undo

Atlas Migrate Down

Auto-generated undo scripts

✅ Enterprise only (via Flyway Desktop / Redgate Compare)

✅ Always, computed dynamically

Requires manual undo files

Yes

No

Works after partial/failed migration

❌ No

✅ Yes

Handles non-transactional DDL

⚠️ Limited

✅ Yes

Built-in safety checks

⚠️ Manual

✅ Automatic

CI/CD friendly

⚠️ Limited

✅ Fully supported

Available in free tier

❌ No

✅ Yes

## The Problem with Pre-written Undo (Down) Scripts[​](#the-problem-with-pre-written-undo-down-scripts "Direct link to The Problem with Pre-written Undo (Down) Scripts")


### They Assume Perfect Success[​](#they-assume-perfect-success "Direct link to They Assume Perfect Success")


Flyway's undo files are written before you know whether the migration will succeed. Consider this scenario:

V002\_\_add\_user\_columns.sql
```codeBlockLines_AdAo
ALTER TABLE users ADD COLUMN email VARCHAR(255);ALTER TABLE users ADD COLUMN phone VARCHAR(50);
```
The corresponding undo file would be:

U002\_\_add\_user\_columns.sql
```codeBlockLines_AdAo
ALTER TABLE users DROP COLUMN phone;ALTER TABLE users DROP COLUMN email;
```
But what if the migration fails after adding `email` but before adding `phone`? Running the undo script will fail because `phone` doesn't exist. Your database is now in an unknown state, and you need manual intervention to fix it.

### Production Usage Limitations[​](#production-usage-limitations "Direct link to Production Usage Limitations")


Undo files are rarely used in production as-is, if ever. Teams often maintain thousands of them "just in case," but when failures occur, these files either don't work (due to partial application) or cause data loss that teams aren't willing to accept.

As detailed in our blog post [The Myth of Down Migrations](/blog/2024/04/01/migrate-down), even teams at companies like Meta with thousands of down files, they were virtually never used in production environments.

### Incompatible with CD/GitOps Workflows[​](#incompatible-with-cdgitops-workflows "Direct link to Incompatible with CD/GitOps Workflows")


CD/GitOps workflows assume you can roll back to any previous version by deploying artifacts from an earlier commit. But those earlier commits don't contain the undo files needed to revert database changes - those files only exist in future commits.

This creates a mismatch between application rollbacks and database rollbacks, forcing teams to handle database reversions manually.

## How Atlas Migrate Down Works[​](#how-atlas-migrate-down-works "Direct link to How Atlas Migrate Down Works")


Atlas's `migrate down` command computes rollback plans based on your database's actual current state. Here's what makes it different:

### State-Aware Planning[​](#state-aware-planning "Direct link to State-Aware Planning")


Instead of pre-written scripts, Atlas:

1.  Inspects your current database state
2.  Determines what migrations were applied (fully or partially)
3.  Computes the exact sequence of statements needed to reach the target version
4.  Validates the plan with automatic safety checks, and follows your team policies

### Partial Failure Handling[​](#partial-failure-handling "Direct link to Partial Failure Handling")


For databases that support transactional DDLs (like PostgreSQL), Atlas wraps the entire rollback in a transaction. If anything fails, the database returns to its pre-rollback state.

For databases without transactional DDL support (like MySQL), Atlas reverts changes statement-by-statement, updating the revisions table after each successful step. If a failure occurs midway, you can simply re-run Atlas to continue from where it stopped.

### Built-in Safety Checks[​](#built-in-safety-checks "Direct link to Built-in Safety Checks")


By default, Atlas runs [pre-migration checks](/versioned/checks) before executing any rollback. For example, before dropping a column, Atlas verifies that it contains no data:
```codeBlockLines_AdAo
-- checks before reverting version 20240305171146  -> SELECT NOT EXISTS (SELECT 1 FROM `logs`) AS `is_empty`-- ok (50.472µs)-- reverting version 20240305171146  -> DROP TABLE `logs`-- ok (53.245µs)
```
If the check fails, the rollback is aborted, preventing unintended data loss.

## Basic Usage[​](#basic-usage "Direct link to Basic Usage")


### Reverting the Last Migration[​](#reverting-the-last-migration "Direct link to Reverting the Last Migration")


To revert the most recently applied migration:

*   MySQL
*   PostgreSQL
*   SQLite
```codeBlockLines_AdAo
atlas migrate down \  --dir "file://migrations" \  --url "mysql://root:pass@localhost:3306/example" \  --dev-url "docker://mysql/8/dev"
```
```codeBlockLines_AdAo
atlas migrate down \  --dir "file://migrations" \  --url "postgres://postgres:pass@:5432/example?search_path=public&sslmode=disable" \  --dev-url "docker://postgres/15/dev?search_path=public"
```
```codeBlockLines_AdAo
atlas migrate down \  --dir "file://migrations" \  --url "sqlite://file.db" \  --dev-url "sqlite://dev?mode=memory"
```

### Reverting Multiple Migrations[​](#reverting-multiple-migrations "Direct link to Reverting Multiple Migrations")


To revert a specific number of migrations:
```codeBlockLines_AdAo
atlas migrate down 3 \  --dir "file://migrations" \  --url "mysql://root:pass@localhost:3306/example" \  --dev-url "docker://mysql/8/dev"
```

### Reverting to a Specific Version[​](#reverting-to-a-specific-version "Direct link to Reverting to a Specific Version")


To roll back to a particular version:
```codeBlockLines_AdAo
atlas migrate down \  --to-version 20240301000000 \  --dir "file://migrations" \  --url "mysql://root:pass@localhost:3306/example" \  --dev-url "docker://mysql/8/dev"
```

### Reverting to a Git Tag[​](#reverting-to-a-git-tag "Direct link to Reverting to a Git Tag")


If you're using Atlas Cloud's [Schema Registry](/cloud/features/registry), you can tag migration directory states and revert to those tags:
```codeBlockLines_AdAo
atlas migrate down \  --to-tag "v1.2.0" \  --dir "file://migrations" \  --url "mysql://root:pass@localhost:3306/example" \  --dev-url "docker://mysql/8/dev"
```

## Production Safety Features[​](#production-safety-features "Direct link to Production Safety Features")


### Dry Run Mode[​](#dry-run-mode "Direct link to Dry Run Mode")


Preview what Atlas will do without executing any changes:
```codeBlockLines_AdAo
atlas migrate down --dry-run \  --dir "file://migrations" \  --url "mysql://root:pass@localhost:3306/example" \  --dev-url "docker://mysql/8/dev"
```
This shows the exact SQL statements that will be executed and which safety checks will run.

### Approval Workflows[​](#approval-workflows "Direct link to Approval Workflows")


For production environments, Atlas Cloud supports approval workflows. When a `migrate down` operation is triggered (via CLI or GitHub Actions), it can be configured to wait for approval from designated reviewers before execution.

This ensures that potentially destructive operations are reviewed by multiple team members before being applied.

![Review Required](/assets/images/require-approval-052a1c5f84fde67bbb44c6b7000e08f3.png)

## CI/CD Integration[​](#cicd-integration "Direct link to CI/CD Integration")


### GitHub Actions[​](#github-actions "Direct link to GitHub Actions")


Atlas provides a dedicated GitHub Action for migrate down operations. This allows you to:

1.  Connect your migration directory to the [Schema Registry](/cloud/features/registry)
2.  Set up a workflow that can be triggered on-demand or by tagging a commit
3.  Require approval before execution
4.  Apply the rollback once approved
```codeBlockLines_AdAo
name: Rollback Databaseon:  workflow_dispatch:    inputs:      tag:        description: 'Migration tag to roll back to'        required: truejobs:  rollback:    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v3      - uses: ariga/atlas-action/migrate/down@v1        with:          url: ${{ secrets.DATABASE_URL }}          to-tag: ${{ github.event.inputs.tag }}          dir: migrations
```
See the [Atlas Down Action documentation](https://github.com/ariga/atlas-action?tab=readme-ov-file#arigaatlas-actionmigratedown) for details.

## Comparison with Flyway Workflows[​](#comparison-with-flyway-workflows "Direct link to Comparison with Flyway Workflows")


### Flyway Undo Workflow[​](#flyway-undo-workflow "Direct link to Flyway Undo Workflow")


1.  Write your migration file: `V001__add_table.sql`
2.  Write the corresponding undo file: `U001__add_table.sql`
3.  Test the undo file separately (often skipped)
4.  Apply migration: `flyway migrate`
5.  If rollback needed: `flyway undo` (Teams/Enterprise only)
6.  Undo file assumes full success

**Limitations:**

*   Undo files must be written manually for every migration
*   No automatic safety checks
*   Doesn't handle partial failures
*   Undo feature requires Enterprise license
*   Rarely tested in production-like environments

### Atlas Migrate Down Workflow[​](#atlas-migrate-down-workflow "Direct link to Atlas Migrate Down Workflow")


1.  Define your desired schema in code (HCL, SQL, or ORM)
2.  Generate migration: `atlas migrate diff`
3.  Apply migration: `atlas migrate apply`
4.  If rollback needed: `atlas migrate down`
5.  Atlas automatically computes the rollback based on database current state

**Features:**

*   No need for manual undo (down) files to maintain
*   Automatic safety checks prevent data loss
*   Handles partial failures gracefully
*   Works in both free and paid tiers
*   Integrates with approval workflows for production environments

## Use Cases[​](#use-cases "Direct link to Use Cases")


### Local Development[​](#local-development "Direct link to Local Development")


Iterate on schema changes without manually writing undo (down) scripts:
```codeBlockLines_AdAo

# Try a schema changeatlas migrate diff --edit# Apply itatlas migrate apply --env local# Changed your mind? Revert itatlas migrate down --env local# Delete the migration fileatlas migrate rm 20240305171146

```

### Staging/Test Environment Resets[​](#stagingtest-environment-resets "Direct link to Staging/Test Environment Resets")


Reset a test environment to a specific schema version for testing:
```codeBlockLines_AdAo
atlas migrate down --to-version 20240301000000 --env staging
```

### Production Rollbacks[​](#production-rollbacks "Direct link to Production Rollbacks")


Roll back a production deployment that introduced database issues:
```codeBlockLines_AdAo

# Preview the rollbackatlas migrate down --dry-run --env production# Execute with approval (via Atlas Cloud)atlas migrate down --env production

```

## Migration from Flyway[​](#migration-from-flyway "Direct link to Migration from Flyway")


If you're currently using Flyway with undo files, migrating to Atlas is straightforward:

1.  **Import your existing migrations** using [Atlas migration import](/versioned/import):

    ```codeBlockLines_AdAo
    atlas migrate import \  --from "file://migrations?format=flyway" \  --to "file://atlas-migrations"
    ```

    Note: Undo files (`U__` prefix) are not imported because Atlas computes them dynamically.

2.  **Test migrate down** on a development database:

    ```codeBlockLines_AdAo
    atlas migrate down --env dev
    ```

3.  **Remove undo files** from your repository - they're no longer needed!

For a complete migration guide, see [Migrating from Flyway to Atlas](/guides/migrate-flyway-to-atlas).

## Summary[​](#summary "Direct link to Summary")


Atlas's `migrate down` command provides an alternative to Flyway's undo scripts by:

*   Computing rollback plans dynamically based on current database state
*   Handling partial migration failures automatically
*   Running safety checks before executing destructive operations
*   Supporting both transactional and non-transactional DDL databases
*   Integrating with approval workflows and CI/CD pipelines
*   Working with GitOps workflows through version tagging

## Read More[​](#read-more "Direct link to Read More")


*   [Down Migrations in Atlas](/versioned/down)
*   [Atlas Blog: The Myth of Down Migrations](/blog/2024/04/01/migrate-down)
*   [Compare Atlas and Flyway](/guides/atlas-vs-flyway)
*   [Migrate from Flyway to Atlas](/guides/migrate-flyway-to-atlas)
*   [Pre-migration Checks](/versioned/checks)
*   [GitHub Actions Integration](https://github.com/ariga/atlas-action?tab=readme-ov-file#arigaatlas-actionmigratedown)

*   [Quick Comparison](#quick-comparison)
*   [The Problem with Pre-written Undo (Down) Scripts](#the-problem-with-pre-written-undo-down-scripts)
    *   [They Assume Perfect Success](#they-assume-perfect-success)
    *   [Production Usage Limitations](#production-usage-limitations)
    *   [Incompatible with CD/GitOps Workflows](#incompatible-with-cdgitops-workflows)
*   [How Atlas Migrate Down Works](#how-atlas-migrate-down-works)
    *   [State-Aware Planning](#state-aware-planning)
    *   [Partial Failure Handling](#partial-failure-handling)
    *   [Built-in Safety Checks](#built-in-safety-checks)
*   [Basic Usage](#basic-usage)
    *   [Reverting the Last Migration](#reverting-the-last-migration)
    *   [Reverting Multiple Migrations](#reverting-multiple-migrations)
    *   [Reverting to a Specific Version](#reverting-to-a-specific-version)
    *   [Reverting to a Git Tag](#reverting-to-a-git-tag)
*   [Production Safety Features](#production-safety-features)
    *   [Dry Run Mode](#dry-run-mode)
    *   [Approval Workflows](#approval-workflows)
*   [CI/CD Integration](#cicd-integration)
    *   [GitHub Actions](#github-actions)
*   [Comparison with Flyway Workflows](#comparison-with-flyway-workflows)
    *   [Flyway Undo Workflow](#flyway-undo-workflow)
    *   [Atlas Migrate Down Workflow](#atlas-migrate-down-workflow)
*   [Use Cases](#use-cases)
    *   [Local Development](#local-development)
    *   [Staging/Test Environment Resets](#stagingtest-environment-resets)
    *   [Production Rollbacks](#production-rollbacks)
*   [Migration from Flyway](#migration-from-flyway)
*   [Summary](#summary)
*   [Read More](#read-more)