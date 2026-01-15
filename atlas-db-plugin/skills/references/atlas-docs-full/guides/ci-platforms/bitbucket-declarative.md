CI/CD for Databases on Bitbucket Pipelines - Declarative Workflow | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Bitbucket Pipelines is a built-in CI/CD service in BitBucket to run automated workflows directly from your repository. When used with Atlas, it lets you validate and plan database schema changes as part of your regular CI process, keeping updates predictable and safe as they move through your pipeline.

In this guide, we will demonstrate how to use [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) and Atlas to set up CI/CD pipelines for your database schema changes using the **declarative migrations** workflow.

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

### Creating a bot token[​](#creating-a-bot-token "Direct link to Creating a bot token")


To report CI run results to Atlas Cloud, create an Atlas Cloud bot token by following [these instructions](/cloud/bots) and copy it.

Next, in your Bitbucket repository, go to **Repository settings** -> **Repository variables** and create a new secured variable named `ATLAS_TOKEN`. Paste your token in the value field.

### Creating a secret for your database URL[​](#creating-a-secret-for-your-database-url "Direct link to Creating a secret for your database URL")


To avoid having plain-text database URLs that may contain sensitive information in your configuration files, create another secured variable named `DATABASE_URL` and populate it with the URL (connection string) of your database.

To learn more about formatting URLs for different databases, see the [URL documentation](/concepts/url).

### Creating a Bitbucket access token (optional)[​](#creating-a-bitbucket-access-token-optional "Direct link to Creating a Bitbucket access token (optional)")


Atlas can post schema plans as comments on pull requests. To enable this, create a [Bitbucket app password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) with `pullrequest:write` permissions. Then, add it as a secured variable named `BITBUCKET_ACCESS_TOKEN`.

## Declarative Migrations Workflow[​](#declarative-migrations-workflow "Direct link to Declarative Migrations Workflow")


In the declarative workflow, developers provide the desired state of the database as code. Atlas can read database schemas from various formats such as [plain SQL](/atlas-schema/sql), [Atlas HCL](/atlas-schema/hcl), [ORM models](/atlas-schema/external), and even another live database. Atlas then connects to the target database and calculates the diff between the current state and the desired state. It then generates a migration plan to bring the database to the desired state.

In this guide, we will use the SQL schema format.

The full source code for this example can be found in the [atlas-examples/declarative](https://github.com/ariga/atlas-examples/tree/master/cicd/bitbucket/declarative) repository.

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
variable "database_url" {  type        = string  default     = getenv("DATABASE_URL")  description = "URL to the target database to apply changes"}env "dev" {  url = var.database_url  dev = "docker://postgres/15/dev?search_path=public"  schema {    src = "file://schema.sql"    repo {      name = "bitbucket-atlas-action-declarative-demo"    }  }  diff {    concurrent_index {      add  = true      drop = true    }  }}
```

### Pushing the schema to Atlas Cloud[​](#pushing-the-schema-to-atlas-cloud "Direct link to Pushing the schema to Atlas Cloud")


To push our initial schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud, run the following command:
```codeBlockLines_AdAo
atlas schema push bitbucket-atlas-action-declarative-demo --env dev
```
This command pushes the content linked in the schema `src` field in the `dev` environment defined in our `atlas.hcl` to a project in the Schema Registry called `bitbucket-atlas-action-declarative-demo`.

Atlas will print a URL to your schema on Atlas Cloud. You can visit this URL to view your schema.

### Setting up Bitbucket Pipelines[​](#setting-up-bitbucket-pipelines "Direct link to Setting up Bitbucket Pipelines")


Create a `bitbucket-pipelines.yml` file in the root of your repository with the following content:

bitbucket-pipelines.yml
```codeBlockLines_AdAo

# This is an example Starter pipeline configuration# Use a skeleton to build, test and deploy using manual and parallel steps# -----# You can specify a custom docker image from Docker Hub as your build environment.image: atlassian/default-image:3# List of available ATLAS_ACTION can be found at:# https://github.com/ariga/atlas-action## For input variables, please refer to the README.md of the action# then add the prefix `ATLAS_INPUT_` to the input name.## For example, if the input name is `working-directory`, then the variable name should be `ATLAS_INPUT_WORKING_DIRECTORY`# The output will be saved in the file `.atlas-action/outputs.sh` and can be sourced to get the output variables# And the directory can be changed by setting the `ATLAS_OUTPUT_DIR` variable.## The `ATLAS_TOKEN` is required for all actions.definitions:  services:    # Run Postgres database to simulate the production database    # This is only for demo purposes.    # We can replace this with the proxy to connect to the production database.    fake-db:      image: postgres:15      environment:        POSTGRES_DB: postgres        POSTGRES_USER: postgres        POSTGRES_PASSWORD: postgres  commonItems:    atlas-action-variables: &atlasVars      ATLAS_TOKEN: $ATLAS_TOKEN # Required      # The access token is required to comment on the PR      # It must have Pull requests/Write permission      BITBUCKET_ACCESS_TOKEN: $BITBUCKET_ACCESS_TOKEN      # To connect to the service running outside the pipeline,      # we need to use the host.docker.internal to connect to the host machine      DATABASE_URL: postgres://postgres:postgres@host.docker.internal:5432/postgres?sslmode=disable&search_path=publicpipelines:  pull-requests:    '**':      - step:          name: 'Plan schema changes'          script:            - name: Plan schema changes              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: schema/plan # Required                ATLAS_INPUT_ENV: dev            - cat .atlas-action/outputs.sh            - source .atlas-action/outputs.sh          services:            - fake-db  branches:    master:      - step:          name: 'Deploy schema changes'          script:            - name: Approve the plan              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: schema/plan/approve # Required                ATLAS_INPUT_ENV: dev            - name: Push the schema              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: schema/push # Required                ATLAS_INPUT_LATEST: "true" # Required                ATLAS_INPUT_ENV: dev            - name: Apply schema changes              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: schema/apply # Required                ATLAS_INPUT_ENV: dev            - cat .atlas-action/outputs.sh            - source .atlas-action/outputs.sh          services:            - fake-db

```
note

This configuration uses `master` as the default branch name. If your Bitbucket repository uses a different default branch (such as `main`), update the `branches:` section accordingly:
```codeBlockLines_AdAo
branches:  main:  # Change to match your default branch
```
Let's break down what this pipeline configuration does:

1.  The `schema/plan` step runs on every pull request. Atlas generates a migration plan showing the changes needed to move from the current state to the desired state, and posts it as a comment on the pull request.

![Bitbucket pull request comment showing Atlas schema plan results](/u/bitbucket/declarative-lint-pr-comment.png)

2.  After the pull request is merged into the master branch, the following steps are executed:
    *   The plan created in the pull request is approved by the `schema/plan/approve` step.
    *   The `schema/push` step pushes the new schema to the [Schema Registry](/cloud/features/registry) on Atlas Cloud.
    *   The `schema/apply` step applies the changes to the database.

Run this workflow by pushing the `schema.sql`, `atlas.hcl`, and `bitbucket-pipelines.yml` to a Bitbucket repository to ensure the configuration is correct and apply the initial schema to your database.

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's see our CI/CD pipeline in action!

#### Step 1: Make a schema change[​](#step-1-make-a-schema-change "Direct link to Step 1: Make a schema change")


Let's modify the `schema.sql` file to drop the index we created earlier:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE "users" (  "id" bigserial PRIMARY KEY,  "name" text NOT NULL,  "active" boolean NOT NULL,  "address" text NOT NULL,  "nickname" text NOT NULL,  "nickname2" text NOT NULL,  "nickname3" text NOT NULL);
```
Now, let's commit the change to a new branch, push it to Bitbucket, and create a pull request. The `schema/plan` step will use Atlas to create a migration plan from the current state of the database to the new desired state.

There are two things to note:

*   The comment also includes instructions to edit the plan. This is useful when the plan has lint issues (for example, dropping a column will raise a "destructive changes" error).
*   The plan is created in a "pending" state, which means Atlas can't use it yet against the real database.

### Merging the changes[​](#merging-the-changes "Direct link to Merging the changes")


Once you're satisfied with the plan, merge the pull request. A new pipeline will run with the remaining steps:

1.  The `schema/plan/approve` step approves the plan that was generated earlier
2.  The `schema/push` step syncs the new desired state to the schema registry on Atlas Cloud
3.  The `schema/apply` step deploys the changes to your database, using the approved plan.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use Bitbucket Pipelines with Atlas to set up a modern CI/CD pipeline for declarative schema migrations. Here's what we accomplished:

*   **Automated schema planning** on every pull request to visualize changes
*   **Centralized schema management** by pushing to Atlas Cloud's Schema Registry
*   **Approval workflow** ensuring only reviewed changes are applied
*   **Automated deployments** using approved plans

For more information on the declarative workflow, see the [Declarative Migrations documentation](/declarative/apply).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [schema testing](/testing/schema) to validate your schema
*   Read about [versioned migrations](/versioned/intro) as an alternative workflow
*   Check out the [Bitbucket Pipes reference](/integrations/bitbucket-pipes) for all available actions
*   Understand the differences between [Declarative vs Versioned](/concepts/declarative-vs-versioned) workflows

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token](#creating-a-bot-token)
    *   [Creating a secret for your database URL](#creating-a-secret-for-your-database-url)
    *   [Creating a Bitbucket access token (optional)](#creating-a-bitbucket-access-token-optional)
*   [Declarative Migrations Workflow](#declarative-migrations-workflow)
    *   [Our goal](#our-goal)
    *   [Creating a simple SQL schema](#creating-a-simple-sql-schema)
    *   [Pushing the schema to Atlas Cloud](#pushing-the-schema-to-atlas-cloud)
    *   [Setting up Bitbucket Pipelines](#setting-up-bitbucket-pipelines)
    *   [Testing our pipeline](#testing-our-pipeline)
    *   [Merging the changes](#merging-the-changes)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)