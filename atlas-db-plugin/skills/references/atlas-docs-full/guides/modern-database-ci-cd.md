Modern Database CI/CD with Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

![Diagram showing the modern database CI/CD workflow: Develop, Review, Deliver, and Deploy.](/assets/images/database-ci-cd-workflow-415e36078f896a2b79477ace2e0e013c.png)

Continuous integration and continuous delivery (CI/CD) are tenets of modern software development. CI/CD enables teams to deliver software faster, with higher quality, and with less risk. However, CI/CD is not just for application code. Teams can also apply these principles to their databases, to extend their benefits to one of the most critical components of their stack.

This document describes the recommended workflow for teams using Atlas to achieve continuous integration and continuous delivery for their databases.

## Principles[​](#principles "Direct link to Principles")


1.  **Changes to the database are planned automatically**. Given the desired state of the database, the system should automatically generate a plan for how to get from the current state to the desired state.
2.  **Changes to the database schema are stored in a versioned migration directory.** All planned changes to the database are checked in to a versioned migration directory. This directory contains SQL scripts, which are executed in lexicographic order to apply the changes to the database.
3.  **Changes to the database are validated during CI.** All changes to the database are tested and evaluated against a set of governing policies.
4.  **Changes to the database are deployed via automation.** No manual steps are required to deploy changes to the database. All changes are deployed via a CI/CD pipeline.

## Workflow[​](#workflow "Direct link to Workflow")


This section follows a single change to your database schema from the time it is planned to the time it is deployed. The purpose of this section is not to provide extensive instructions for how to configure your CI/CD pipeline, but rather to provide a high-level overview of the recommended workflow.

### 1\. Create a new branch[​](#1-create-a-new-branch "Direct link to 1. Create a new branch")


Create a new branch to track a change to the database.

Example:
```codeBlockLines_AdAo
git checkout -b add-new-index
```

### 2\. Change the desired state of the database[​](#2-change-the-desired-state-of-the-database "Direct link to 2. Change the desired state of the database")


Similar to modern Infrastructure-as-Code tools, Atlas uses a declarative approach to database management. This means that you do not need to write SQL scripts to apply changes to the database. Instead, you simply declare the desired state of the database, and Atlas will automatically generate the SQL scripts required to apply the change.

In Atlas, the desired state of the database can be provided in many forms. For example, you can provide it as a plain SQL file, using [Atlas HCL](/atlas-schema/hcl), as a reference to your [ORM](/atlas-schema/external), or even as a connection to an existing database. For the purposes of this example, we will use a plain SQL file.

Example:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE users (  id INT AUTO_INCREMENT PRIMARY KEY,  name VARCHAR(255) NOT NULL,  email VARCHAR(255) NOT NULL,  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,+  INDEX idx_users_email (email));
```
Further reading

Atlas supports many ways of declaring the desired state of the database. Here is a list of resources for you to learn more about them:

*   [SQL Schemas](/atlas-schema/sql) - Define the desired state as plain `.sql` files or using [templates](/atlas-schema/sql#template-directory).
*   [Atlas HCL](/atlas-schema/hcl) - Define the desired state using a declarative language that is designed specifically for database schemas.
*   [External Schemas](/atlas-schema/external) - Define the desired state using a reference to your ORM or other external schema definition. For example, using [GORM](/guides/orms/gorm) (Go), [Sequelize](/guides/orms/sequelize) (Node.js), or [Beego](/guides/orms/beego) (Go).

### 3\. Automatically plan the change[​](#3-automatically-plan-the-change "Direct link to 3. Automatically plan the change")


Use the Atlas CLI to plan the change. This will generate a plan for how to get from the current state of the database to the desired state of the database.

Example:
```codeBlockLines_AdAo
atlas migrate diff --env local add_email_index
```
Atlas will create a new SQL file in the migration directory, which contains the SQL statements required to apply the change to the database.

migrations/20230919090302\_add\_email\_index.sql
```codeBlockLines_AdAo
ALTER TABLE `users` ADD INDEX `idx_users_email` (`email`);
```
Further reading

Atlas automates the planning process by generating SQL scripts that apply the desired changes to the database. To learn more about this process, review the [Automatic Migration Planning](/versioned/diff) documentation.

### 4\. Push your changes to trigger CI[​](#4-push-your-changes-to-trigger-ci "Direct link to 4. Push your changes to trigger CI")


Once you are satisfied with your change and the plan, push your changes to trigger CI:
```codeBlockLines_AdAo
git add -A .git commit -m "Add email index"git push origin add-email-index
```
Your CI pipeline should be configured to run the following steps whenever a new commit is pushed to the branch. Teams commonly configure these steps to run only when a change is made to the desired state of the database or to the migration directory.
```codeBlockLines_AdAo

# login to Atlas to report migration analysis reportatlas login --token $ATLAS_TOKEN# populate a JSON object with metadata about the Git repo, branch, commit and URL (usually the PR URL)ATLAS_CONTEXT=$(cat <<EOF{    "repo": "$CI_PROJECT_PATH",    "branch": "$CI_COMMIT_REF_NAME",    "commit": "$CI_COMMIT_SHA",    "url": "$URL"}EOF)# Run analysis against the latest revision of the database.atlas migrate lint --base atlas://dir-name --context $ATLAS_CONTEXT

```
This way, whenever a change is made to the desired state of the database, the CI pipeline will automatically simulate and analyze the changes, and verify that they are valid and safe. Atlas will generate a detailed analysis report, an example of which you can see [here](https://gh.atlasgo.cloud/dirs/4294967383/ci-runs/8589934684).

Further reading

It is possible to push your changes to Atlas Cloud directly with the CLI to your CI pipeline, but official integrations are also available. See the [Integrations](/integrations) page for more information.

### 5\. Merge your changes to push the latest revision of the database[​](#5-merge-your-changes-to-push-the-latest-revision-of-the-database "Direct link to 5. Merge your changes to push the latest revision of the database")


Once CI is green and the change is approved, the changes should be merged into the main branch. This merge should trigger another workflow, which will push the most recent version of the migration directory to [Atlas Cloud](https://atlasgo.cloud). You can think of this step as similar to pushing a Docker image to a container registry when code updates are merged into the main branch.

This is typically done by running the following steps in your workflow:
```codeBlockLines_AdAo

# login to Atlas to push the migration directoryatlas login --token $ATLAS_TOKEN# push the migration directory to Atlas Cloudatlas migrate push dir-name:$GIT_COMMITatlas migrate push dir-name:latest

```
Once the migration directory is pushed to Atlas Cloud, you can view a visual representation of the database schema and its revision history in the Atlas Cloud UI. You can see an example of this [here](https://gh.atlasgo.cloud/dirs/4294967383).

Further reading

It is possible to use the Atlas CLI directly in your CI pipeline, but official integrations are also available:

*   The `atlas-sync-action` [GitHub Action](/integrations/github-actions#arigaatlas-sync-action)
*   The [GitLab CI](/guides/ci-platforms/gitlab-versioned) guide, includes instructions for running `migrate push` in pipelines.

### 6\. Deploy the latest revision of the database[​](#6-deploy-the-latest-revision-of-the-database "Direct link to 6. Deploy the latest revision of the database")


Once the latest revision of the database is pushed to Atlas Cloud, it can be deployed to any environment. This is typically done by running the following steps in your workflow:
```codeBlockLines_AdAo

# login to Atlas to deploy the latest revision of the databaseatlas login --token $ATLAS_TOKEN# deploy the latest revision of the database to the staging environmentatlas migrate apply --url mysql://user:pass@host:port/dbname --dir atlas://dir-name

```
When you run the `atlas migrate apply` command, Atlas will connect to the target database, and apply any pending migrations. If there are no pending migrations, Atlas will do nothing. When Atlas finishes running, it will send a report to your Atlas Cloud account, with a summary of the changes that were applied, and any errors that occurred. You can view an example of this report [here](https://gh.atlasgo.cloud/deployments/sets/94489280523).

Further reading

Atlas offers integrations with many modern deployment tools. Here are some popular alternatives:

*   [Atlas CLI](/versioned/apply)
*   [GitHub Actions](/integrations/github-actions#arigaatlas-action-deploy)
*   [Terraform Provider](/integrations/terraform-provider)
*   [Kubernetes Operator](/integrations/kubernetes)
*   And many more such as [Argo CD](/guides/deploying/k8s-argo), [Flux CD](/guides/deploying/k8s-flux), [ECS Fargate](/guides/deploying/aws-ecs-fargate), [Helm](/guides/deploying/helm), and [more](/integrations)!

## Conclusion[​](#conclusion "Direct link to Conclusion")


This document described the recommended workflow for teams using Atlas to achieve continuous integration and continuous delivery (CI/CD) for their databases. As we've demonstrated, this process can generally be broken down into four phases:

1.  Develop - Create a new branch, change the desired state and plan the change.
2.  Review - Create a pull request, let Atlas analyze the change and report back.
3.  Deliver - Merge the change, push the latest revision of the database to Atlas Cloud.
4.  Deploy - Deploy the latest revision of the database to any environment.

By implementing this workflow, teams can achieve the following benefits:

*   **Faster development** - Developers can focus on writing application code, and let Atlas be responsible for planning schema changes.
*   **Lower defect rate** - By simulating and validating changes to the database during CI, teams can catch errors before they are deployed to production.
*   **Automation** - By integrating with your team's existing deployment tools, Atlas obviates the need for manual steps to deploy changes to the database, making your system more reliable and less error-prone.

*   [Principles](#principles)
*   [Workflow](#workflow)
    *   [1\. Create a new branch](#1-create-a-new-branch)
    *   [2\. Change the desired state of the database](#2-change-the-desired-state-of-the-database)
    *   [3\. Automatically plan the change](#3-automatically-plan-the-change)
    *   [4\. Push your changes to trigger CI](#4-push-your-changes-to-trigger-ci)
    *   [5\. Merge your changes to push the latest revision of the database](#5-merge-your-changes-to-push-the-latest-revision-of-the-database)
    *   [6\. Deploy the latest revision of the database](#6-deploy-the-latest-revision-of-the-database)
*   [Conclusion](#conclusion)