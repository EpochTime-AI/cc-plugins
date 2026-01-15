CI/CD for Databases with Azure DevOps and GitHub | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Many teams use GitHub for source control but prefer Azure DevOps Pipelines for CI/CD. Azure Pipelines can seamlessly trigger from GitHub repositories, giving you the best of both platforms.

This guide walks you through setting up Atlas's automated database schema migrations with code hosted on GitHub and pipelines running on Azure DevOps.

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

### Setting up Azure DevOps[​](#setting-up-azure-devops "Direct link to Setting up Azure DevOps")


1.  **Create an Azure DevOps organization** if you don't have one already.
2.  **Create a new project** in your Azure DevOps organization.
3.  **Add the Atlas extension** to your organization from the [Azure DevOps Marketplace](https://marketplace.visualstudio.com/items?itemName=Ariga.atlas-action).

### Connecting GitHub to Azure DevOps[​](#connecting-github-to-azure-devops "Direct link to Connecting GitHub to Azure DevOps")


To trigger Azure DevOps pipelines from GitHub repositories, you need to create a service connection:

1.  In your Azure DevOps project, go to **Project Settings** → **Service connections**.
2.  Click **New service connection** and select **GitHub**.
3.  Choose **OAuth** or **Personal Access Token** authentication method.
4.  If using Personal Access Token, create a [GitHub personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with `repo` scope.
5.  Name your service connection (e.g., "GitHub Connection").

GitHub Integration

The GitHub service connection allows the `AtlasAction` task to post migration lint results directly as comments on your GitHub pull requests. This provides immediate feedback to developers without requiring them to navigate to Azure DevOps to view the results. Make sure to use the exact name of your service connection in the `githubConnection` parameter below.

### Creating an Atlas Cloud bot token[​](#creating-an-atlas-cloud-bot-token "Direct link to Creating an Atlas Cloud bot token")


To report CI run results to Atlas Cloud, create an Atlas Cloud bot token by following [these instructions](/cloud/bots). Copy the token and store it as a secret using the following steps.

### Creating secrets in Azure DevOps[​](#creating-secrets-in-azure-devops "Direct link to Creating secrets in Azure DevOps")


In your Azure DevOps project, go to **Pipelines** → **Library**:

1.  Create a variable group named "atlas-vars".
2.  Add the following variables:
    *   `ATLAS_TOKEN` - Your Atlas Cloud bot token (mark as secret)
    *   `DB_URL` - Your [database connection string](/concepts/url) (mark as secret)

## Choose a workflow[​](#choose-a-workflow "Direct link to Choose a workflow")


Atlas supports two types of schema management workflows:

*   [**Versioned Migrations**](#versioned-migrations-workflow) - Changes to the schema are defined as _migrations_ (SQL scripts) and applied in sequence to reach the desired state.
*   **Declarative Migrations** - The desired state of the database is defined as code, and Atlas calculates the migration plan to apply it.

This guide focuses on the Versioned Migrations workflow. To learn more about the differences and tradeoffs between these approaches, see the [Declarative vs Versioned](/concepts/declarative-vs-versioned) article.

## Versioned Migrations Workflow[​](#versioned-migrations-workflow "Direct link to Versioned Migrations Workflow")


In the [versioned workflow](/versioned/intro), changes to the schema are represented by a _migration directory_ in your codebase. Each file in this directory represents a transition to a new version of the schema.

Based on our blueprint for [Modern CI/CD for Databases](/guides/modern-database-ci-cd), our pipeline will:

1.  [Lint](/versioned/lint) new migration files whenever a pull request is opened.
2.  [Push](/versioned/intro#pushing-migrations-to-the-schema-registry) the migration directory to the [Schema Registry](/cloud/features/registry) when changes are merged to the main branch.
3.  [Apply](/versioned/apply) new migrations to our database.

### Pushing a migration directory to Atlas Cloud[​](#pushing-a-migration-directory-to-atlas-cloud "Direct link to Pushing a migration directory to Atlas Cloud")


Running the following command from the parent directory of your migration directory creates a "migration directory" repo in your Atlas Cloud organization (substitute "app" with the name you want to give the new Atlas repository before running):
```codeBlockLines_AdAo
atlas migrate push app \  --dev-url "docker://postgres/16/dev?search_path=public"
```
Dev Database

Replace `docker://postgres/16/dev` with the appropriate dev database URL for your database. For more information on the dev database, see the [dev database](/concepts/dev-database) article.

Atlas will print a URL leading to your migrations on Atlas Cloud. You can visit this URL to view your migrations.

### Setting up GitHub[​](#setting-up-github "Direct link to Setting up GitHub")


Create an `azure-pipelines.yml` file in the root of your GitHub repository with the following content. Remember to replace "app" with the real name of your Atlas Cloud repository.

azure-pipelines.yml
```codeBlockLines_AdAo
trigger:  branches:    include:      - main  paths:    include:      - 'migrations/*'      - 'azure-pipelines.yml'pr:  branches:    include:      - main  paths:    include:      - 'migrations/*'pool:  vmImage: ubuntu-latestvariables:  - group: atlas-varssteps:  - checkout: self    persistCredentials: true    fetchDepth: 0    fetchTags: true  - script: |    echo "Configuring git user for commits..."    git config user.email "azure-pipelines[bot]@users.noreply.github.com"    git config user.name "azure-pipelines[bot]"  displayName: 'Configure Git User for Commits'  - script: curl -sSf https://atlasgo.sh | sh    displayName: Install Atlas  - script: atlas version    displayName: Atlas Version  - script: atlas login --token $(ATLAS_TOKEN)    displayName: Atlas Login  # Lint migrations on pull requests  - task: AtlasAction@1    condition: eq(variables['Build.Reason'], 'PullRequest')    inputs:      action: 'migrate lint'      dir: 'file://migrations'      dir_name: 'app'      config: 'file://atlas.hcl'      env: 'ci'      githubConnection: 'GitHub Connection'    displayName: Lint Migrations  # Push migrations to Atlas Cloud on main branch  - task: AtlasAction@1    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))    inputs:      action: 'migrate push'      dir: 'file://migrations'      dir_name: 'app'      latest: true      env: 'ci'    displayName: Push Migrations  # Apply migrations to database on main branch  - task: AtlasAction@1    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))    inputs:      action: 'migrate apply'      dir: 'file://migrations'      url: $(DB_URL)    displayName: Apply Migrations
```
Also, create an `atlas.hcl` file in the root of your GitHub repository with the following content:

atlas.hcl
```codeBlockLines_AdAo
env {    name = atlas.env    dev = "docker://postgres/16/dev?search_path=public" # Replace if necessary (see "Dev Database" above)    migration {      repo {        name = "app" # Replace with the name of your repository in previous step      }    }}
```

### Creating the pipeline in Azure DevOps[​](#creating-the-pipeline-in-azure-devops "Direct link to Creating the pipeline in Azure DevOps")


1.  In your Azure DevOps project, go to **Pipelines** → **Pipelines**.
2.  Click **New pipeline**.
3.  Select **GitHub** as your source.
4.  Authenticate and select your GitHub repository.
5.  Choose **Existing Azure Pipelines YAML file**.
6.  Select the `azure-pipelines.yml` file you created.
7.  Click **Save and run**.

Let's break down what this pipeline does:

1.  **Lint on Pull Requests**: The `migrate lint` step runs automatically whenever a pull request is opened that modifies the `migrations/` directory. Atlas analyzes the new migrations for potential issues like destructive changes, backward incompatibility, or syntax errors. Because we configured the `githubConnection` parameter, lint results appear as a comment directly on the GitHub pull request.

2.  **Push to Registry**: When changes are merged into the main branch, the `migrate push` step pushes the migration directory to Atlas Cloud's [Schema Registry](/cloud/features/registry). This creates a versioned snapshot of your migrations that can be referenced and deployed across environments.

3.  **Apply to Database**: The `migrate apply` step deploys pending migrations to your database using the connection string stored in the `DB_URL` secret.

### Testing the workflow[​](#testing-the-workflow "Direct link to Testing the workflow")


Let's take our new pipeline for a spin. Assume we have an existing migration file in our repository:

migrations/20251019111\_create\_t1\_table.sql
```codeBlockLines_AdAo
CREATE TABLE t1(  c1 serial NOT NULL,  c2 integer NOT NULL,  c3 integer NOT NULL,  CONSTRAINT pk PRIMARY KEY (c1));
```
Now let's add a new migration:

1.  Create a new branch in your GitHub repository and add a new migration locally with `atlas migrate new drop_c3 --edit`. Paste the following in the editor:

migrations/20251019222\_drop\_c3.sql
```codeBlockLines_AdAo
ALTER TABLE "t1" DROP COLUMN "c3";
```
2.  Commit and push the changes to GitHub.

3.  Open a pull request in GitHub. This will trigger the Azure DevOps pipeline and run the `migrate lint` step.

![Atlas migration lint results](/assets/images/migrate-lint-488a431ea817d6dea68d959e0250a110.png)

4.  Check the lint report. Follow any instructions to fix the issues.

5.  Merge the pull request into the main branch. This will trigger the `migrate push` and `migrate apply` steps.

6.  When the pipeline finishes running, check your database to see if the changes were applied.

*   [Prerequisites](#prerequisites)
    *   [Installing Atlas](#installing-atlas)
    *   [Setting up Azure DevOps](#setting-up-azure-devops)
    *   [Connecting GitHub to Azure DevOps](#connecting-github-to-azure-devops)
    *   [Creating an Atlas Cloud bot token](#creating-an-atlas-cloud-bot-token)
    *   [Creating secrets in Azure DevOps](#creating-secrets-in-azure-devops)
*   [Choose a workflow](#choose-a-workflow)
*   [Versioned Migrations Workflow](#versioned-migrations-workflow)
    *   [Pushing a migration directory to Atlas Cloud](#pushing-a-migration-directory-to-atlas-cloud)
    *   [Setting up GitHub](#setting-up-github)
    *   [Creating the pipeline in Azure DevOps](#creating-the-pipeline-in-azure-devops)
    *   [Testing the workflow](#testing-the-workflow)