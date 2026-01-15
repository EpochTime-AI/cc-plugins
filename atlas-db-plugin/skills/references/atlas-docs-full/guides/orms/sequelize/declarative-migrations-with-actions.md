Declarative Migrations for Sequelize using Atlas and GitHub Actions | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In this guide, we'll show you how to set up a declarative migrations pipeline with Sequelize and Atlas on GitHub Actions. This pipeline will allow you to manage your database schema using a declarative approach, where you define your schema in one place - your Sequelize models - and let Atlas handle the rest.

The code for this guide can be found in the [`atlas-examples`](https://github.com/ariga/atlas-examples/tree/master/orms/sequelize/declarative) repository.

### Prerequisites[​](#prerequisites "Direct link to Prerequisites")


1.  An Atlas Cloud account, if you don't have one, you can [sign up here](https://auth.atlasgo.cloud/signup).

2.  A GitHub repository with your Sequelize project.

3.  A target database that you want to deploy your Sequelize schema to. **Note:** Your database should be accessible from the GitHub Actions runner. To learn more about connectivity between GitHub Actions and your database, refer to [our documentation](/guides/deploying/connect-to-db-from-github-actions).

    In this guide we will use a PostgreSQL database, but you can use any database supported by both Atlas and Sequelize.

### 1\. Atlas Project Configuration[​](#1-atlas-project-configuration "Direct link to 1. Atlas Project Configuration")


Before we set up the pipeline in GitHub Actions, let's first make sure our Atlas configuration file is ready. If you haven't already, follow the instructions in the [Sequelize Getting Started guide](/guides/orms/sequelize) to install the Atlas Sequelize provider and load your schema into Atlas.

For more detailed instructions:

*   If all of your Sequelize models are placed in a single directory, you can use [Standalone Mode](/guides/orms/sequelize/standalone). Otherwise, use [Script Mode](/guides/orms/sequelize/script).
*   Be sure to use configuration specific to your target database in the external schema block as well as the `dev` URL in the `env` block.

At the root of your Sequelize project (next to `package.json`) You should have an `atlas.hcl` file similar to the one below:
```codeBlockLines_AdAo
data "external_schema" "sequelize" {    program = [        // ... replace with the configuration for your Sequelize project    ]}env {  name = atlas.env  dev = "docker://postgres/16/dev?search_path=public"  schema {    src = data.external_schema.sequelize.url    repo {      name = "sequelize-declarative-demo"    }  }}
```
Notice a couple of edits we made to the `atlas.hcl` file:

1.  As we are using a declarative flow, we can erase the `migration` and `format` blocks from the `env` block.
2.  We added a `repo` block to the `schema` block. This is required as this workflow relies on storing plans in the Atlas Schema Registry.

Verify that your `atlas.hcl` file is correct by running the following command:
```codeBlockLines_AdAo
atlas schema inspect --env local --url env://schema.src --format '{{ sql . }}'
```
If everything is set up correctly, you should see the SQL representation of your schema.

### 2\. Push your Project Schema to the Schema Registry[​](#2-push-your-project-schema-to-the-schema-registry "Direct link to 2. Push your Project Schema to the Schema Registry")


To start building our CI/CD pipeline, we need to push our project schema to the Atlas Schema Registry.

Before pushing the schema to the registry, make sure you are logged in to Atlas. If you are not logged in, run the following command:
```codeBlockLines_AdAo
atlas login
```
After logging in, run the following command:
```codeBlockLines_AdAo
atlas schema push --env local
```
Atlas will print something like this:
```codeBlockLines_AdAo
Schema: sequelize-declarative-demo  -- Atlas URL: atlas://sequelize-declarative-demo  -- Cloud URL: https://rotemtam85.atlasgo.cloud/schemas/141733920836
```
The `Atlas URL` is the URL you will use to reference your schema in the pipeline and the `Cloud URL` is the URL you can use to view your schema in the Atlas Cloud UI.

### 3\. Set up secrets in GitHub Actions[​](#3-set-up-secrets-in-github-actions "Direct link to 3. Set up secrets in GitHub Actions")


Our pipeline will need to utilize two secrets in order to function correctly, one for the database URL and one for an Atlas token.

First, let's create a secret for the database URL. To learn how to format a connection string to your database correctly, refer to the [relevant documentation](/concepts/url).

Once you have the connection string, add the secret to your repository as `DATABASE_URL`. To learn how, refer to the [GitHub documentation](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions).

Next, create a Bot Token for our pipeline to use. To learn how to do this, refer to the [Atlas documentation](/cloud/bots). After acquiring the token, add it to your repository as a secret named `ATLAS_TOKEN`.

### 4\. The GitHub Actions Workflow for Planning and Approving Changes[​](#4-the-github-actions-workflow-for-planning-and-approving-changes "Direct link to 4. The GitHub Actions Workflow for Planning and Approving Changes")


Now that we have our secrets set up, let's create the GitHub Actions workflow.

Create a new file in your repository at `.github/workflows/sequelize-atlas-plan.yml` with the following content:
```codeBlockLines_AdAo
name: "Sequelize: Plan Declarative Migrations"on:  workflow_dispatch:  push:    branches:      - master    paths:      - .github/workflows/sequelize-atlas-plan.yaml      - 'path/to/models/**/*'  pull_request:    paths:      - .github/workflows/sequelize-atlas-plan.yaml      - 'path/to/models/**/*'permissions:  contents: read  pull-requests: writejobs:  plan:    name: plan    if: ${{ github.event_name == 'pull_request' }}    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4      - name: Set up Node        uses: actions/setup-python@v4      - name: Install dependencies        working-directory: path/to/project        run: |          npm ci      - name: Setup Atlas        uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_TOKEN }}      - name: Run schema plan        uses: ariga/atlas-action/schema/plan@v1        env:          GITHUB_TOKEN: ${{ github.token }}        with:          working-directory: path/to/project          from: ${{ secrets.DATABASE_URL }}          env: ci  approve-push:    name: approve-push    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/master' }}    runs-on: ubuntu-latest    env:      GITHUB_TOKEN: ${{ github.token }}    steps:      - uses: actions/checkout@v4      - name: Set up Node        uses: actions/setup-node@v4      - name: Install dependencies        working-directory: path/to/project        run: |            npm ci      - name: Setup Atlas        uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_TOKEN }}      - name: Approve the plan        id: plan-approve        uses: ariga/atlas-action/schema/plan/approve@v1        with:          env: ci          working-directory: path/to/project          from: ${{ secrets.SEQUELIZE_NEON_URL }}      # Push the schema after the plan is approved.      - name: Push the schema        id: schema-push        uses: ariga/atlas-action/schema/push@v1        with:          env: ci          working-directory: path/to/project
```
Before committing the workflow, make sure to replace the `path/to/project` with the path to the root of your Sequelize project (next to `atlas.hcl`) directory and that the secrets are correctly named.

Here's a brief overview of what the workflow does:

1.  The `plan` job runs on every push to the `master` branch and on every pull request where models are changed or the workflow is modified. It checks out the code, installs dependencies, sets up the Atlas CLI, and runs the schema plan.

    The `schema/plan` action will generate a plan for the changes in your Sequelize models and store it in the Atlas Schema Registry. It will also create a detailed PR comment with the changes and any potential issues.

2.  The `approve-push` job runs only on pushes to the `master` branch. It checks out the code, installs dependencies, and sets up the Atlas CLI

    The `schema/push` action will push the schema to the schema registry and marks the plan as approved as well.

To see this in action, refer to the video at the top of this guide.

### 5\. The Workflow for Deploying Changes[​](#5-the-workflow-for-deploying-changes "Direct link to 5. The Workflow for Deploying Changes")


The previous workflow covers the planning and approval of changes as well as pushing them to the schema registry from where they can be deployed.

To deploy the changes, create a new file in your repository at `.github/workflows/sequelize-atlas-apply.yml` with the following content:
```codeBlockLines_AdAo
name: "Sequelize: Deploy Declarative"on:  workflow_dispatch:    inputs:      tag:        description: 'Tag to deploy'        required: true        default: 'latest'permissions:  contents: read  pull-requests: writejobs:  deploy:    runs-on: ubuntu-latest    steps:      - uses: actions/checkout@v4      - name: Setup Atlas        uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_TOKEN }}      - name: Run schema apply        uses: ariga/atlas-action/schema/apply@v1        env:          GITHUB_TOKEN: ${{ github.token }}        with:          working-directory: path/to/models          url: ${{ secrets.DATABASE_URL }}          env: prod          to: "atlas://sequelize-declarative-demo/?tag=${{ github.event.inputs.tag }}"
```
This workflow will deploy the schema to the target database. To trigger the deployment, run the workflow manually and provide a tag for the deployment.

To see this in action, refer to the video at the top of this guide.

### Conclusion[​](#conclusion "Direct link to Conclusion")


In this guide, we showed you how to set up a declarative migrations pipeline with Sequelize and Atlas on GitHub Actions. This pipeline allows you to manage your database schema using a declarative approach, where you define your schema in one place - your Sequelize models - and let Atlas handle the rest.

*   [Prerequisites](#prerequisites)
*   [1\. Atlas Project Configuration](#1-atlas-project-configuration)
*   [2\. Push your Project Schema to the Schema Registry](#2-push-your-project-schema-to-the-schema-registry)
*   [3\. Set up secrets in GitHub Actions](#3-set-up-secrets-in-github-actions)
*   [4\. The GitHub Actions Workflow for Planning and Approving Changes](#4-the-github-actions-workflow-for-planning-and-approving-changes)
*   [5\. The Workflow for Deploying Changes](#5-the-workflow-for-deploying-changes)
*   [Conclusion](#conclusion)