CI/CD for Databases on CircleCI - Declarative Workflow | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

CircleCI is a popular CI/CD platform that allows you to automatically build, test, and deploy your code. When used with Atlas, it lets you validate and plan database schema changes as part of your regular CI process, keeping updates predictable and safe as they move through your pipeline.

In this guide, we will demonstrate how to use [CircleCI](https://circleci.com/) and Atlas to set up CI/CD pipelines for your database schema changes using the **declarative migrations** workflow.

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

### Creating a bot token and CircleCI context[​](#creating-a-bot-token-and-circleci-context "Direct link to Creating a bot token and CircleCI context")


To report CI run results to Atlas Cloud, create an Atlas Cloud bot token by following [these instructions](/cloud/bots) and copy it.

Next, we'll create a CircleCI [context](https://circleci.com/docs/contexts/) to securely store environment variables that will be shared across jobs:

1.  In CircleCI, go to **Organization Settings** -> **Contexts**
2.  Click **Create Context** and name it `dev` (this matches the context used in our example configuration)
3.  Click on the newly created `dev` context
4.  Add the following environment variables:
    *   **`ATLAS_TOKEN`**: Your Atlas Cloud bot token ([how to create](/cloud/bots)) - required
    *   **`DATABASE_URL`**: The URL (connection string) of your target database ([URL format guide](/concepts/url)) - required
    *   **`GITHUB_TOKEN`**: GitHub personal access token with `repo` scope ([how to create](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)) - optional, needed for PR comments
    *   **`GITHUB_REPOSITORY`**: Your GitHub repository in the format `owner/repo` (e.g., [`ariga/atlas`](https://github.com/ariga/atlas)) - required for PR comments

Using a context allows you to manage these sensitive variables in one place and reuse them across multiple projects and workflows.

## Declarative Migrations Workflow[​](#declarative-migrations-workflow "Direct link to Declarative Migrations Workflow")


In the declarative workflow, developers provide the desired state of the database as code. Atlas can read database schemas from various formats such as [plain SQL](/atlas-schema/sql), [Atlas HCL](/atlas-schema/hcl), [ORM models](/atlas-schema/external), and even another live database. Atlas then connects to the target database and calculates the diff between the current state and the desired state. It then generates a migration plan to bring the database to the desired state.

In this guide, we will use the SQL schema format.

The full source code for this example can be found in the [atlas-examples/declarative](https://github.com/ariga/atlas-examples/tree/master/cicd/circleci/declarative) repository.

### Our goal[​](#our-goal "Direct link to Our goal")


When a pull request contains changes to the schema, we want Atlas to:

*   Compare the current state (your database) with the new desired state
*   Create a migration plan to show the user for approval
*   Mark the plan as approved when the pull request is approved and merged
*   Use the approved plan to apply the changes to the database during deployment

### Creating a simple SQL schema[​](#creating-a-simple-sql-schema "Direct link to Creating a simple SQL schema")


Create a file named `schema.sql` and fill it with the following content:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE "users" (  "id" bigserial PRIMARY KEY,  "name" text NOT NULL,  "active" boolean NOT NULL,  "address" text NOT NULL,  "nickname" text NOT NULL,  "nickname2" text NOT NULL,  "nickname3" text NOT NULL);CREATE INDEX "users_active" ON "users" ("active");
```
Then, create a configuration file for Atlas named `atlas.hcl` as follows:

atlas.hcl
```codeBlockLines_AdAo
variable "database_url" {  type        = string  default     = getenv("DATABASE_URL")  description = "URL to the target database to apply changes"}env "dev" {  url = var.database_url  dev = "docker://postgres/15/dev?search_path=public"  schema {    src = "file://schema.sql"    repo {      name = "circleci-atlas-action-declarative-demo"    }  }  diff {    concurrent_index {      add  = true      drop = true    }  }}
```

### Pushing the schema to Atlas Cloud[​](#pushing-the-schema-to-atlas-cloud "Direct link to Pushing the schema to Atlas Cloud")


To push our initial schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud, run the following command:
```codeBlockLines_AdAo
atlas schema push circleci-atlas-action-declarative-demo --env dev
```
This command pushes the content linked in the schema `src` field in the `dev` environment defined in our `atlas.hcl` to a project in the Schema Registry called `circleci-atlas-action-declarative-demo`.

Atlas will print a URL to your schema on Atlas Cloud. You can visit this URL to view your schema.

### Setting up CircleCI[​](#setting-up-circleci "Direct link to Setting up CircleCI")


Create a `.circleci/config.yml` file in the root of your repository with the following content:

.circleci/config.yml
```codeBlockLines_AdAo
version: 2.1orbs:  atlas-orb: ariga/atlas-orb@0.3.1jobs:  plan-schema-changes:    docker:      - image: cimg/base:current      - image: cimg/postgres:15.0        environment:          POSTGRES_DB: postgres          POSTGRES_USER: postgres          POSTGRES_PASSWORD: postgres    steps:      - checkout      - run:          name: Wait for Postgres          command: dockerize -wait tcp://127.0.0.1:5432 -timeout 60s      - atlas-orb/setup:          version: "latest"      - atlas-orb/schema_plan:          env: dev          dev_url: postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=public  deploy-schema-changes:    docker:      - image: cimg/base:current      - image: cimg/postgres:15.0        environment:          POSTGRES_DB: postgres          POSTGRES_USER: postgres          POSTGRES_PASSWORD: postgres    steps:      - checkout      - run:          name: Wait for Postgres          command: dockerize -wait tcp://127.0.0.1:5432 -timeout 60s      - atlas-orb/setup:          version: "latest"      - atlas-orb/schema_plan_approve:          env: dev          dev_url: postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=public      - atlas-orb/schema_push:          env: dev          dev_url: postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=public      - atlas-orb/schema_apply:          env: dev          dev_url: postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=publicworkflows:  version: 2  atlas-workflow:    jobs:      - plan-schema-changes:          context: dev          filters:            branches:              ignore: main      - deploy-schema-changes:          context: dev          filters:            branches:              only: main
```
Why do we need 2 `dev_url` configurations?

Our `atlas.hcl` file contains a `dev_url` configuration that points to a Docker container:
```codeBlockLines_AdAo
dev = "docker://postgres/15/dev?search_path=public"
```
This works locally where Docker-in-Docker is supported. However, CircleCI Docker images don't support Docker-in-Docker.

By adding the `dev_url` parameter to all relevant commands (`schema_plan`, `schema_plan_approve`, `schema_push`, and `schema_apply`), we override the `atlas.hcl` configuration and point to the PostgreSQL service container running at `localhost:5432` in the CI environment. While some commands may not strictly require `dev_url`, we provide it to all for consistency and to avoid configuration issues.

Environment Variables

The environment variables we configured in the CircleCI context (`ATLAS_TOKEN`, `DATABASE_URL`, `GITHUB_TOKEN`, `GITHUB_REPOSITORY`) are automatically available in all jobs that use the `dev` context. These variables are securely managed by CircleCI and don't need to be specified in individual jobs.

note

This configuration uses `main` as the default branch name. If your GitHub repository uses a different default branch (such as `master`), update the workflow filters accordingly:
```codeBlockLines_AdAo
filters:  branches:    only: master  # Change to match your default branch
```
Let's break down what this pipeline configuration does:

1.  The `plan-schema-changes` job runs on every pull request (branches that are not `main`). Atlas generates a migration plan showing the changes needed to move from the current state to the desired state, and posts it as a comment on the pull request.

![CircleCI pull request comment showing Atlas schema plan results](/u/circleci/declarative-lint-pr-comment.png)

2.  After the pull request is merged into the main branch, three things happen: First, the plan created in the pull request is approved by the `schema_plan_approve` step. Second, the `schema_push` step pushes the new schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud. Finally, the `schema_apply` step applies the changes to the database.

note

In this example, we use `schema_apply` to apply changes to the same test database that we used for planning. In a production environment, you would typically:

*   Set up a separate production database with its own `DATABASE_URL` in a production context
*   Run the `schema_apply` step against your production database
*   Consider using [deployment approvals](/declarative/apply#approval-policies) for additional control

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's see our CI/CD pipeline in action!

#### Step 1: Make a schema change[​](#step-1-make-a-schema-change "Direct link to Step 1: Make a schema change")


Let's modify the `schema.sql` file to drop the index we created earlier:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE "users" (  "id" bigserial PRIMARY KEY,  "name" text NOT NULL,  "active" boolean NOT NULL,  "address" text NOT NULL,  "nickname" text NOT NULL,  "nickname2" text NOT NULL,  "nickname3" text NOT NULL);
```
Now, let's commit the change to a new branch, push it to GitHub, and create a pull request. The `plan-schema-changes` job will use Atlas to create a migration plan from the current state of the database to the new desired state.

There are two things to note:

*   The comment also includes instructions to edit the plan. This is useful when the plan has lint issues (for example, dropping a column will raise a "destructive changes" error).
*   The plan is created in a "pending" state, which means Atlas can't use it yet against the real database.

### Merging the changes[​](#merging-the-changes "Direct link to Merging the changes")


Once you're satisfied with the plan, merge the pull request. A new pipeline will run with the remaining steps:

1.  The `schema_plan_approve` step approves the plan that was generated earlier
2.  The `schema_push` step syncs the new desired state to the schema registry on Atlas Cloud
3.  The `schema_apply` step applies the approved changes to the database

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use CircleCI with Atlas to set up a modern CI/CD pipeline for declarative schema migrations. Here's what we accomplished:

*   **Automated schema planning** on every pull request to visualize changes
*   **Centralized schema management** by pushing to Atlas Cloud's Schema Registry
*   **Approval workflow** ensuring only reviewed changes are applied
*   **Automated deployments** using approved plans

For more information on the declarative workflow, see the [Declarative Migrations documentation](/declarative/apply).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [schema testing](/testing/schema) to validate your schema
*   Read about [versioned migrations](/versioned/intro) as an alternative workflow
*   Check out the [CircleCI Orbs reference](/integrations/circleci-orbs) for all available actions
*   Understand the differences between [Declarative vs Versioned](/concepts/declarative-vs-versioned) workflows

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token and CircleCI context](#creating-a-bot-token-and-circleci-context)
*   [Declarative Migrations Workflow](#declarative-migrations-workflow)
    *   [Our goal](#our-goal)
    *   [Creating a simple SQL schema](#creating-a-simple-sql-schema)
    *   [Pushing the schema to Atlas Cloud](#pushing-the-schema-to-atlas-cloud)
    *   [Setting up CircleCI](#setting-up-circleci)
    *   [Testing our pipeline](#testing-our-pipeline)
    *   [Merging the changes](#merging-the-changes)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)