CI/CD for Databases on CircleCI - Versioned Workflow | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

CircleCI is a popular CI/CD platform that allows you to automatically build, test, and deploy your code. Combined with Atlas, you can manage database schema changes with confidence.

In this guide, we will demonstrate how to use [CircleCI](https://circleci.com/) and Atlas to set up CI/CD pipelines for your database schema changes using the **versioned migrations** workflow.

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

## Versioned Migrations Workflow[​](#versioned-migrations-workflow "Direct link to Versioned Migrations Workflow")


In the [versioned workflow](/versioned/intro), changes to the schema are represented by a _migration directory_ in your codebase. Each file in this directory represents a transition to a new version of the schema.

Based on our blueprint for [Modern CI/CD for Databases](/guides/modern-database-ci-cd), our pipeline will:

1.  [Lint](/versioned/lint) new migration files whenever a pull request is opened.
2.  [Push](/versioned/intro#pushing-migrations-to-the-schema-registry) the migration directory to the [Schema Registry](/cloud/features/registry) when changes are merged to the main branch.
3.  [Apply](/versioned/apply) new migrations to our database.

In this guide, we will walk through each of these steps and set up a CircleCI configuration to automate them.

The full source code for this example can be found in the [atlas-examples/versioned](https://github.com/ariga/atlas-examples/tree/master/cicd/circleci/versioned) repository.

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
atlas migrate push circleci-atlas-action-versioned-demo --env dev
```
This command pushes the migrations directory linked in the migration `dir` field in the `dev` environment defined in our `atlas.hcl` to a project in the Schema Registry called `circleci-atlas-action-versioned-demo`.

Atlas will print a URL leading to your migrations on Atlas Cloud. You can visit this URL to view your migrations.

### Setting up CircleCI[​](#setting-up-circleci "Direct link to Setting up CircleCI")


Create a `.circleci/config.yml` file in the root of your repository with the following content:

.circleci/config.yml
```codeBlockLines_AdAo
version: 2.1orbs:  atlas-orb: ariga/atlas-orb@0.3.1jobs:  lint-migrations:    docker:      - image: cimg/base:current      - image: cimg/postgres:15.0        environment:          POSTGRES_DB: postgres          POSTGRES_USER: postgres          POSTGRES_PASSWORD: postgres    steps:      - checkout      - run:          name: Wait for Postgres          command: dockerize -wait tcp://127.0.0.1:5432 -timeout 60s      - atlas-orb/setup:          version: "latest"      - atlas-orb/migrate_lint:          env: dev          dir_name: "circleci-atlas-action-versioned-demo"          dev_url: "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=public"  push-and-apply-migrations:    docker:      - image: cimg/base:current      - image: cimg/postgres:15.0        environment:          POSTGRES_DB: postgres          POSTGRES_USER: postgres          POSTGRES_PASSWORD: postgres    steps:      - checkout      - run:          name: Wait for Postgres          command: dockerize -wait tcp://127.0.0.1:5432 -timeout 60s      - atlas-orb/setup:          version: "latest"      - atlas-orb/migrate_push:          env: dev          dir_name: "circleci-atlas-action-versioned-demo"          dev_url: "postgres://postgres:postgres@localhost:5432/postgres?sslmode=disable&search_path=public"      - atlas-orb/migrate_apply:          env: dev          dir: "atlas://circleci-atlas-action-versioned-demo"workflows:  version: 2  atlas-workflow:    jobs:      - lint-migrations:          context: dev          filters:            branches:              ignore: main      - push-and-apply-migrations:          context: dev          filters:            branches:              only: main
```
note

This configuration uses `main` as the default branch name. If your GitHub repository uses a different default branch (such as `master`), update the workflow filters accordingly:
```codeBlockLines_AdAo
filters:  branches:    only: master  # Change to match your default branch
```
Let's break down what this pipeline configuration does:

1.  The `lint-migrations` job runs on every pull request (branches that are not `main`). When new migrations are detected, Atlas will lint them and post a report as a comment on the pull request. This helps you catch issues early in the development process.

![CircleCI pull request comment showing Atlas migration lint results](/u/circleci/versioned-lint-pr-comment.png)

2.  After the pull request is merged into the main branch, the `push-and-apply-migrations` job will push the new state of the migration directory to the [Schema Registry](/cloud/features/registry) on Atlas Cloud.

3.  The `migrate_apply` step will then deploy the new migrations to your database.

### Testing our pipeline[​](#testing-our-pipeline "Direct link to Testing our pipeline")


Let's take our pipeline for a spin:

1.  Locally, create a new branch and add a new migration with `atlas migrate new --edit`. Paste the following in the editor:

    schema.sql

    ```codeBlockLines_AdAo
    DROP INDEX "users_active";
    ```

2.  Commit and push the changes.
3.  In GitHub, create a pull request for the branch you just pushed.
4.  View the lint report generated by Atlas. Follow the links to see the changes visually on Atlas Cloud.
5.  Merge the pull request.
6.  When the pipeline has finished running, check your database to verify that the changes were applied.

## Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to use CircleCI with Atlas to set up a modern CI/CD pipeline for versioned database migrations. Here's what we accomplished:

*   **Automated migration linting** on every pull request to catch issues early
*   **Centralized migration management** by pushing to Atlas Cloud's Schema Registry
*   **Automated deployments** to your target database when changes are merged

For more information on the versioned workflow, see the [Versioned Migrations documentation](/versioned/intro).

### Next steps[​](#next-steps "Direct link to Next steps")


*   Learn about [Atlas Cloud](/cloud/getting-started) for enhanced collaboration
*   Explore [migration testing](/testing/migrate) to validate your migrations
*   Read about [declarative migrations](/declarative/apply) as an alternative workflow
*   Check out the [CircleCI Orbs reference](/integrations/circleci-orbs) for all available actions

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Creating a bot token and CircleCI context](#creating-a-bot-token-and-circleci-context)
*   [Versioned Migrations Workflow](#versioned-migrations-workflow)
    *   [Defining the desired schema](#defining-the-desired-schema)
    *   [Creating the Atlas configuration file](#creating-the-atlas-configuration-file)
    *   [Generating your first migration](#generating-your-first-migration)
    *   [Pushing a migration directory to Atlas Cloud](#pushing-a-migration-directory-to-atlas-cloud)
    *   [Setting up CircleCI](#setting-up-circleci)
    *   [Testing our pipeline](#testing-our-pipeline)
*   [Wrapping up](#wrapping-up)
    *   [Next steps](#next-steps)