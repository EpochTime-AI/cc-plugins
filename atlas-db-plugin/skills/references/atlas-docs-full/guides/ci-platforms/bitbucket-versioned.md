CI/CD for Databases on Bitbucket Pipelines - Versioned Workflow | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Bitbucket Pipelines is a built-in CI/CD service in Bitbucket that allows you to automatically build, test, and deploy your code. Combined with Atlas, you can manage database schema changes with confidence.

In this guide, we will demonstrate how to use [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) and Atlas to set up CI/CD pipelines for your database schema changes using the **versioned migrations** workflow.

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


Atlas can post lint reports as comments on pull requests. To enable this, create a [Bitbucket app password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) with `pullrequest:write` permissions. Then, add it as a secured variable named `BITBUCKET_ACCESS_TOKEN`.

## Versioned Migrations Workflow[​](#versioned-migrations-workflow "Direct link to Versioned Migrations Workflow")


In the [versioned workflow](/versioned/intro), changes to the schema are represented by a _migration directory_ in your codebase. Each file in this directory represents a transition to a new version of the schema.

Based on our blueprint for [Modern CI/CD for Databases](/guides/modern-database-ci-cd), our pipeline will:

1.  [Lint](/versioned/lint) new migration files whenever a pull request is opened.
2.  [Push](/versioned/intro#pushing-migrations-to-the-schema-registry) the migration directory to the [Schema Registry](/cloud/features/registry) when changes are merged to the main branch.
3.  [Apply](/versioned/apply) new migrations to our database.

In this guide, we will walk through each of these steps and set up a Bitbucket Pipelines configuration to automate them.

The full source code for this example can be found in the [atlas-examples/versioned](https://github.com/ariga/atlas-examples/tree/master/cicd/bitbucket/versioned) repository.

### Defining the desired schema[​](#defining-the-desired-schema "Direct link to Defining the desired schema")


First, define your desired database schema. Create a file named `schema.sql` with the following content:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE "users" (  "id" bigserial PRIMARY KEY,  "name" text NOT NULL,  "active" boolean NOT NULL,  "address" text NOT NULL,  "nickname" text NOT NULL,  "nickname2" text NOT NULL,  "nickname3" text NOT NULL);CREATE INDEX "users_active" ON "users" ("active");
```

### Creating the Atlas configuration file[​](#creating-the-atlas-configuration-file "Direct link to Creating the Atlas configuration file")


Create a configuration file for Atlas named `atlas.hcl` with the following content:

atlas.hcl
```codeBlockLines_AdAo
variable "database_url" {  type        = string  default     = getenv("DATABASE_URL")  description = "URL to the target database to apply changes"}env "dev" {  src = "file://schema.sql"  url = var.database_url  dev = "docker://postgres/15/dev?search_path=public"  migration {    dir = "file://migrations"  }  diff {    concurrent_index {      add  = true      drop = true    }  }}
```

### Generating your first migration[​](#generating-your-first-migration "Direct link to Generating your first migration")


Now, generate your first migration by comparing your desired schema with the current (empty) migration directory:
```codeBlockLines_AdAo
atlas migrate diff initial --env dev
```
This command will automatically create a `migrations` directory with a migration file containing the SQL statements needed to create the `users` table and index, as defined in our file linked at `src` in the `dev` environment.

### Pushing a migration directory to Atlas Cloud[​](#pushing-a-migration-directory-to-atlas-cloud "Direct link to Pushing a migration directory to Atlas Cloud")


Run the following command from the parent directory of your migration directory to create a "migration directory" repository in your Atlas Cloud organization:
```codeBlockLines_AdAo
atlas migrate push bitbucket-atlas-action-versioned-demo --env dev
```
This command pushes the migrations directory linked in the migration `dir` field in the `dev` environment defined in our `atlas.hcl` to a project in the Schema Registry called `bitbucket-atlas-action-versioned-demo`.

Atlas will print a URL leading to your migrations on Atlas Cloud. You can visit this URL to view your migrations.

### Setting up Bitbucket Pipelines[​](#setting-up-bitbucket-pipelines "Direct link to Setting up Bitbucket Pipelines")


Create a `bitbucket-pipelines.yml` file in the root of your repository with the following content:

bitbucket-pipelines.yml
```codeBlockLines_AdAo

# This is an example Starter pipeline configuration# Use a skeleton to build, test and deploy using manual and parallel steps# -----# You can specify a custom docker image from Docker Hub as your build environment.image: atlassian/default-image:3# List of available ATLAS_ACTION can be found at:# https://github.com/ariga/atlas-action## For input variables, please refer to the README.md of the action# then add the prefix `ATLAS_INPUT_` to the input name.## For example, if the input name is `working-directory`, then the variable name should be `ATLAS_INPUT_WORKING_DIRECTORY`# The output will be saved in the file `.atlas-action/outputs.sh` and can be sourced to get the output variables# And the directory can be changed by setting the `ATLAS_OUTPUT_DIR` variable.## The `ATLAS_TOKEN` is required for all actions.definitions:  services:    # Run Postgres database to simulate the production database    # This is only for demo purposes.    # We can replace this with the proxy to connect to the production database.    fake-db:      image: postgres:15      environment:        POSTGRES_DB: postgres        POSTGRES_USER: postgres        POSTGRES_PASSWORD: postgres  commonItems:    atlas-action-variables: &atlasVars      ATLAS_TOKEN: $ATLAS_TOKEN # Required      # The access token is required to comment on the PR      # It must have Pull requests/Write permission      BITBUCKET_ACCESS_TOKEN: $BITBUCKET_ACCESS_TOKEN      # To connect to the service running outside the pipeline,      # we need to use the host.docker.internal to connect to the host machine      DATABASE_URL: postgres://postgres:postgres@host.docker.internal:5432/postgres?sslmode=disable&search_path=publicpipelines:  pull-requests:    '**':      - step:          name: 'Lint Migrations'          script:            - name: Lint migrations              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: migrate/lint # Required                ATLAS_INPUT_ENV: dev                ATLAS_INPUT_DIR_NAME: "bitbucket-atlas-action-versioned-demo"            - cat .atlas-action/outputs.sh            - source .atlas-action/outputs.sh          services:            - fake-db  branches:    master:      - step:          name: 'Push and Apply Migrations'          script:            - name: Push migrations              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: migrate/push # Required                ATLAS_INPUT_ENV: dev                ATLAS_INPUT_DIR_NAME: "bitbucket-atlas-action-versioned-demo"            - name: Apply migrations              pipe: docker://arigaio/atlas-action:v1              variables:                <<: *atlasVars                ATLAS_ACTION: migrate/apply # Required                ATLAS_INPUT_ENV: dev                ATLAS_INPUT_DIR: "atlas://bitbucket-atlas-action-versioned-demo"            - cat .atlas-action/outputs.sh            - source .atlas-action/outputs.sh          services:            - fake-db

```
note

This configuration uses `master` as the default branch name. If your Bitbucket repository uses a different default branch (such as `main`), update the `branches:` section accordingly:
```codeBlockLines_AdAo
branches:  main:  # Change to match your default branch
```
Let's break down what this pipeline configuration does:

1.  The `migrate/lint` step runs on every pull request. When new migrations are detected, Atlas will lint them and post a report as a comment on the pull request. This helps you catch issues early in the development process.

![Bitbucket pull request comment showing Atlas migration lint results](/u/bitbucket/versioned-lint-pr-comment.png)

2.  After the pull request is merged into the master branch, the `migrate/push` step will push the new state of the migration directory to the [Schema Registry](/cloud/features/registry) on Atlas Cloud.

3.  The `migrate/apply` step will then deploy the new migrations to your database.

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's take our pipeline for a spin:

1.  Locally, create a new branch and modify your `schema.sql` file to remove the index:

    schema.sql

    ```codeBlockLines_AdAo
    CREATE TABLE "users" (  "id" bigserial PRIMARY KEY,  "name" text NOT NULL,  "active" boolean NOT NULL,  "address" text NOT NULL,  "nickname" text NOT NULL,  "nickname2" text NOT NULL,  "nickname3" text NOT NULL);
    ```

2.  Generate a new migration with `atlas migrate diff drop_users_active --env dev`. This will create a migration file that drops the index.
3.  Commit and push the changes (both `schema.sql` and the new migration file in the `migrations` directory).
4.  In Bitbucket, create a pull request for the branch you just pushed.
5.  View the lint report generated by Atlas. Follow the links to see the changes visually on Atlas Cloud.
6.  Merge the pull request.
7.  When the pipeline has finished running, check your database to verify that the changes were applied.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use Bitbucket Pipelines with Atlas to set up a modern CI/CD pipeline for versioned database migrations. Here's what we accomplished:

*   **Automated migration linting** on every pull request to catch issues early
*   **Centralized migration management** by pushing to Atlas Cloud's Schema Registry
*   **Automated deployments** to your target database when changes are merged

For more information on the versioned workflow, see the [Versioned Migrations documentation](/versioned/intro).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [migration testing](/testing/migrate) to validate your migrations
*   Read about [declarative migrations](/declarative/apply) as an alternative workflow
*   Check out the [Bitbucket Pipes reference](/integrations/bitbucket-pipes) for all available actions

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token](#creating-a-bot-token)
    *   [Creating a secret for your database URL](#creating-a-secret-for-your-database-url)
    *   [Creating a Bitbucket access token (optional)](#creating-a-bitbucket-access-token-optional)
*   [Versioned Migrations Workflow](#versioned-migrations-workflow)
    *   [Defining the desired schema](#defining-the-desired-schema)
    *   [Creating the Atlas configuration file](#creating-the-atlas-configuration-file)
    *   [Generating your first migration](#generating-your-first-migration)
    *   [Pushing a migration directory to Atlas Cloud](#pushing-a-migration-directory-to-atlas-cloud)
    *   [Setting up Bitbucket Pipelines](#setting-up-bitbucket-pipelines)
    *   [Testing our pipeline](#testing-our-pipeline)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)