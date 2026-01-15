Using Atlas Terraform Provider with OpenTaco (Digger) | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

OpenTaco (formerly Digger) is an all-in-one Terraform toolkit that provides PR automation, state management, and drift detection for your Terraform workflows. When combined with the [Atlas Terraform Provider](/integrations/terraform-provider), you can automate database schema migrations directly from your pull requests—getting plan previews as PR comments and applying changes on merge.

This guide walks you through setting up Atlas Terraform Provider with OpenTaco to create a complete database migration CI/CD pipeline.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


Before you begin, ensure you have:

1.  A GitHub repository for your project
2.  [Terraform](https://developer.hashicorp.com/terraform/install) installed
3.  [Atlas CLI](/getting-started#installation) installed
4.  Access to a target database (we'll use PostgreSQL in this example)

## How It Works[​](#how-it-works "Direct link to How It Works")


OpenTaco integrates with your GitHub repository to:

1.  **Run `terraform plan`** on pull requests and post results as comments
2.  **Apply changes** when PRs are merged (or via CommentOps)
3.  **Handle locking** to prevent concurrent changes
4.  **Manage state** securely

When combined with Atlas, this means your database migrations are:

*   **Reviewed in PRs** - See exactly what SQL will run before merging
*   **Applied automatically** - Migrations run on merge without manual intervention
*   **Version controlled** - All changes tracked in Git

## Project Setup[​](#project-setup "Direct link to Project Setup")


### Step 1: Define Your Database Schema[​](#step-1-define-your-database-schema "Direct link to Step 1: Define Your Database Schema")


Create a `schema.sql` file that defines your desired database schema:

schema.sql
```codeBlockLines_AdAo
-- Create users tableCREATE TABLE users (  id SERIAL PRIMARY KEY,  email VARCHAR(255) NOT NULL UNIQUE,  name VARCHAR(255) NOT NULL,  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
```

### Step 2: Create the Terraform Configuration[​](#step-2-create-the-terraform-configuration "Direct link to Step 2: Create the Terraform Configuration")


Now create your Terraform configuration. This consists of several files:

#### Provider Configuration[​](#provider-configuration "Direct link to Provider Configuration")


Create a `main.tf` file with the Atlas provider configuration:

main.tf
```codeBlockLines_AdAo
terraform {  required_providers {    atlas = {      source  = "ariga/atlas"      version = "~> 0.10.0"    }  }}provider "atlas" {  # The dev database is used by Atlas to normalize schemas and plan migrations.  # Using docker:// automatically spins up a temporary container.  dev_url = "docker://postgres/16/dev?search_path=public"  cloud {    repo  = "openTaco"    token = var.atlas_token  }}
```

#### Variables Configuration[​](#variables-configuration "Direct link to Variables Configuration")


Create a `variables.tf` file to define input variables:

variables.tf
```codeBlockLines_AdAo
variable "database_url" {  description = "The connection URL to the target database"  type        = string  sensitive   = true  # Marks this as sensitive to hide in logs}variable "atlas_token" {  description = "Atlas token for authentication"  type        = string  sensitive   = true}
```

#### Migration Resources[​](#migration-resources "Direct link to Migration Resources")


Create a `migrations.tf` file to define the Atlas migration resources:

migrations.tf
```codeBlockLines_AdAo

# Data source to read the current state of migrations# This reads from Atlas Cloud Registry and inspects the target databasedata "atlas_migration" "app" {  # URL to the migration directory in Atlas Cloud Registry  dir = "atlas://open-taco"  # Connection URL to the target database  url = var.database_url}# Resource to apply migrations to the target databaseresource "atlas_migration" "app" {  # Reference the migrations directory from the data source  dir     = data.atlas_migration.app.dir  # Apply up to the latest migration version  version = data.atlas_migration.app.latest  # Target database URL  url     = data.atlas_migration.app.url}

```
Using Atlas Cloud Registry

The `atlas://open-taco` URL points to a migration directory stored in the [Atlas Cloud Registry](/cloud/features/registry). This ensures that your Terraform configuration always uses the latest approved migrations from your CI/CD pipeline.

### Step 3: Generate Your First Migration[​](#step-3-generate-your-first-migration "Direct link to Step 3: Generate Your First Migration")


With your schema defined, generate the initial migration:
```codeBlockLines_AdAo

# Generate migration from your schemaatlas migrate diff initial_schema \  --to "file://schema.sql" \  --dev-url "docker://postgres/16/dev?search_path=public"

```
This creates a migration file in the `migrations/` directory:

migrations/20240115120000\_initial\_schema.sql
```codeBlockLines_AdAo
-- Create "users" tableCREATE TABLE "users" (  "id" serial NOT NULL,  "email" character varying(255) NOT NULL,  "name" character varying(255) NOT NULL,  "created_at" timestamp NULL DEFAULT CURRENT_TIMESTAMP,  PRIMARY KEY ("id"),  CONSTRAINT "users_email_key" UNIQUE ("email"));
```
Your project structure should now look like:
```codeBlockLines_AdAo
.├── main.tf├── variables.tf├── migrations.tf├── outputs.tf├── schema.sql└── migrations/    ├── 20240115120000_initial_schema.sql    └── atlas.sum
```

### Step 4: Initialize Terraform Locally[​](#step-4-initialize-terraform-locally "Direct link to Step 4: Initialize Terraform Locally")


Test that your Terraform configuration is valid:
```codeBlockLines_AdAo
terraform init
```
You should see output indicating the Atlas provider was installed:
```codeBlockLines_AdAo
Initializing the backend...Initializing provider plugins...- Finding ariga/atlas versions matching "~> 0.10.0"...- Installing ariga/atlas v0.10.0...- Installed ariga/atlas v0.10.0Terraform has been successfully initialized!
```

### Step 5: Set Up OpenTaco in Your Repository[​](#step-5-set-up-opentaco-in-your-repository "Direct link to Step 5: Set Up OpenTaco in Your Repository")


OpenTaco can be set up in two ways: using the hosted service or self-hosting. We'll cover the hosted approach which is quickest to get started.

#### 5.1: Install the OpenTaco GitHub App[​](#51-install-the-opentaco-github-app "Direct link to 5.1: Install the OpenTaco GitHub App")


1.  Go to [github.com/apps/digger-cloud](https://github.com/apps/digger-cloud) (or your self-hosted instance)
2.  Click **Install**
3.  Select your repository (or all repositories)
4.  Authorize the app

The GitHub App needs these permissions:

*   **Read** access to code and metadata
*   **Read and write** access to pull requests and issues
*   **Read and write** access to actions (for workflow triggering)

#### 5.2: Create the OpenTaco Configuration File[​](#52-create-the-opentaco-configuration-file "Direct link to 5.2: Create the OpenTaco Configuration File")


Create a `digger.yml` file in your repository root. This file tells OpenTaco about your Terraform projects:

digger.yml
```codeBlockLines_AdAo

# Define your Terraform projectsprojects:  - name: database-migrations      # Unique name for this project    dir: .                         # Directory containing Terraform files    workflow: default              # Which workflow to use    terragrunt: false              # Set to true if using Terragrunt    # Uncomment to auto-apply when PR is merged:    # apply_on_merge: true# Define reusable workflowsworkflows:  default:    # Steps to run on pull request (plan)    plan:      steps:        - init                     # terraform init        - plan                     # terraform plan    # Steps to run on apply    apply:      steps:        - init                     # terraform init        - apply                    # terraform apply    # Optional: Configure plan behavior    configuration:      on_pull_request_pushed: ["digger plan"]      on_pull_request_closed: ["digger unlock"]      on_commit_to_default: ["digger apply"]

```

#### 5.3: Create the GitHub Actions Workflow[​](#53-create-the-github-actions-workflow "Direct link to 5.3: Create the GitHub Actions Workflow")


Create the workflow file that runs OpenTaco on demand (manual trigger):
```codeBlockLines_AdAo
mkdir -p .github/workflows
```
Create `.github/workflows/digger.yml`:

.github/workflows/digger.yml
```codeBlockLines_AdAo
name: OpenTaco Database Migrationson:  workflow_dispatch:    inputs:      spec:        required: true      run_name:        required: false# Limit concurrent runs to prevent state conflictsconcurrency:  group: digger-${{ github.head_ref || github.ref }}  cancel-in-progress: falsejobs:  digger:    runs-on: ubuntu-latest    # Required permissions for OpenTaco    permissions:      contents: write        # To checkout code      pull-requests: write   # To post PR comments      issues: write          # For issue comments (CommentOps)      id-token: write        # For OIDC authentication (if using)      statuses: write        # To update commit statuses    steps:      # Checkout the repository      - name: Checkout        uses: actions/checkout@v4      # Install Atlas CLI for database migrations      - name: Setup Atlas        uses: ariga/setup-atlas@v0        with:          cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}      # Push migrations to Atlas Cloud Registry      - name: Push Migrations        uses: ariga/atlas-action/migrate/push@v1        with:          dir: 'file://migrations'          dir-name: 'open-taco'          dev-url: 'docker://postgres/16/dev?search_path=public'      # Run OpenTaco      - name: Run OpenTaco        uses: diggerhq/digger@vLatest        with:          setup-aws: false           # Set to true if using AWS          setup-terraform: true          terraform-version: 1.6.0          disable-locking: false     # Enable PR-level locking        env:          # GitHub token for PR comments and API access          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}          GITHUB_CONTEXT: ${{ toJson(github) }}          # Pass database URL as Terraform variable          TF_VAR_database_url: ${{ secrets.DATABASE_URL }}          TF_VAR_atlas_token: ${{ secrets.ATLAS_CLOUD_TOKEN }}          # Optional: Pass additional Terraform variables          # TF_VAR_environment: "production"
```

#### 5.4: Configure State Backend (Recommended)[​](#54-configure-state-backend-recommended "Direct link to 5.4: Configure State Backend (Recommended)")


For production use, configure a remote state backend. Add this to your `main.tf`:

*   AWS S3
*   Google Cloud Storage
*   OpenTaco Backend

main.tf
```codeBlockLines_AdAo
terraform {  required_providers {    atlas = {      source  = "ariga/atlas"      version = "~> 0.10.0"    }  }  # Store state in S3 with DynamoDB locking  backend "s3" {    bucket         = "your-terraform-state-bucket"    key            = "database-migrations/terraform.tfstate"    region         = "us-east-1"    dynamodb_table = "terraform-locks"    encrypt        = true  }}provider "atlas" {  dev_url = "docker://postgres/16/dev?search_path=public"  cloud {    repo  = "openTaco"    token = var.atlas_token  }}
```
main.tf
```codeBlockLines_AdAo
terraform {  required_providers {    atlas = {      source  = "ariga/atlas"      version = "~> 0.10.0"    }  }  # Store state in GCS  backend "gcs" {    bucket = "your-terraform-state-bucket"    prefix = "database-migrations"  }}provider "atlas" {  dev_url = "docker://postgres/16/dev?search_path=public"  cloud {    repo  = "openTaco"    token = var.atlas_token  }}
```
OpenTaco provides its own state management backend. See [OpenTaco State Management](https://docs.opentaco.dev/state-management/introduction) for setup instructions.

main.tf
```codeBlockLines_AdAo
terraform {  required_providers {    atlas = {      source  = "ariga/atlas"      version = "~> 0.10.0"    }  }  # Use OpenTaco's built-in state backend  backend "http" {    # Configured via environment variables by OpenTaco  }}provider "atlas" {  dev_url = "docker://postgres/16/dev?search_path=public"  cloud {    repo  = "openTaco"    token = var.atlas_token  }}
```

### Step 6: Configure GitHub Secrets[​](#step-6-configure-github-secrets "Direct link to Step 6: Configure GitHub Secrets")


Add any required secrets:

1.  Navigate to your repository on GitHub
2.  Go to **Settings** → **Secrets and variables** → **Actions**
3.  Click **New repository secret**
4.  Add the following secrets:

Secret Name

Description

Example Value

`ATLAS_CLOUD_TOKEN`

Atlas Cloud bot token used by `ariga/setup-atlas`

Follow [these instructions](/cloud/bots)

`DATABASE_URL`

Connection URL to your database

`postgres://user:pass@host:5432/db?sslmode=require`

`AWS_ACCESS_KEY_ID`

AWS credentials (if using S3 backend)

`AKIA...`

`AWS_SECRET_ACCESS_KEY`

AWS credentials (if using S3 backend)

`secret...`

Database URL Format

The database URL format depends on your database type:

*   **PostgreSQL**: `postgres://user:password@host:5432/dbname?sslmode=require`
*   **MySQL**: `mysql://user:password@host:3306/dbname`
*   **MariaDB**: `maria://user:password@host:3306/dbname`

### Step 7: Commit and Push[​](#step-7-commit-and-push "Direct link to Step 7: Commit and Push")


Commit all files to your repository:
```codeBlockLines_AdAo
git add .git commit -m "Set up Atlas migrations with OpenTaco"git push origin main
```
Your repository should now have this structure:
```codeBlockLines_AdAo
.├── .github/│   └── workflows/│       └── digger.yml├── migrations/│   ├── 20240115120000_initial_schema.sql│   └── atlas.sum├── digger.yml├── main.tf├── variables.tf├── migrations.tf├── outputs.tf└── schema.sql
```

## Workflow in Action[​](#workflow-in-action "Direct link to Workflow in Action")


### Creating a Migration[​](#creating-a-migration "Direct link to Creating a Migration")


1.  **Modify your schema** - Update `schema.sql` with your desired changes:

schema.sql
```codeBlockLines_AdAo
CREATE TABLE users (  id SERIAL PRIMARY KEY,  email VARCHAR(255) NOT NULL UNIQUE,  name VARCHAR(255) NOT NULL,  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);-- Add a new tableCREATE TABLE posts (  id SERIAL PRIMARY KEY,  user_id INT REFERENCES users(id),  title VARCHAR(255) NOT NULL,  content TEXT,  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
```
2.  **Generate the migration**:
```codeBlockLines_AdAo
atlas migrate diff add_posts \  --to "file://schema.sql" \  --dev-url "docker://postgres/16/dev?search_path=public"
```
3.  **Push the migration to Atlas Cloud**:
```codeBlockLines_AdAo
atlas migrate push open-taco \  --dev-url "docker://postgres/16/dev?search_path=public"
```
4.  **Create a pull request** with your changes

### PR Preview[​](#pr-preview "Direct link to PR Preview")


When you open a pull request, OpenTaco will automatically:

1.  Run `terraform init` and `terraform plan`
2.  Post the plan as a PR comment showing exactly what migrations will be applied

You'll see a comment like:

![OpenTaco PR comment](/u/opentaco/comment.png)

Then, commenting `digger apply` on the PR will trigger the apply step:

![OpenTaco PR Apply](/u/opentaco/apply.png)

And in atlas cloud you can see the migration run log:

![OpenTaco Atlas Cloud](/u/opentaco/cloud.png)

### Applying Migrations[​](#applying-migrations "Direct link to Applying Migrations")


Depending on your OpenTaco configuration, migrations are applied either:

*   **On merge**: Automatically when the PR is merged to main
*   **Via CommentOps**: By commenting `digger apply` on the PR

## Advanced Configuration[​](#advanced-configuration "Direct link to Advanced Configuration")


### Using Declarative Workflow[​](#using-declarative-workflow "Direct link to Using Declarative Workflow")


If you prefer the declarative workflow over versioned migrations:

main.tf
```codeBlockLines_AdAo
terraform {  required_providers {    atlas = {      source  = "ariga/atlas"      version = "~> 0.10.0"    }  }}variable "database_url" {  description = "The connection URL to the target database"  type        = string  sensitive   = true}variable "atlas_token" {  description = "Atlas token for authentication"  type        = string  sensitive   = true}provider "atlas" {  dev_url = "docker://postgres/16/dev?search_path=public"  cloud {    repo  = "openTaco"    token = var.atlas_token  }}data "atlas_schema" "app" {  src = "file://${path.module}/schema.sql"}resource "atlas_schema" "app" {  url = var.database_url  hcl = data.atlas_schema.app.hcl}
```

### Multiple Databases[​](#multiple-databases "Direct link to Multiple Databases")


For projects with multiple databases, define separate projects in `digger.yml`:

digger.yml
```codeBlockLines_AdAo
projects:  - name: users-db    dir: databases/users    workflow: default  - name: products-db    dir: databases/products    workflow: defaultworkflows:  default:    plan:      steps:        - init        - plan    apply:      steps:        - init        - apply
```

### Apply on Merge[​](#apply-on-merge "Direct link to Apply on Merge")


Configure OpenTaco to automatically apply on merge:

digger.yml
```codeBlockLines_AdAo
projects:  - name: database-migrations    dir: .    workflow: default    apply_on_merge: trueworkflows:  default:    plan:      steps:        - init        - plan    apply:      steps:        - init        - apply
```

## Best Practices[​](#best-practices "Direct link to Best Practices")


1.  **Always review migration plans** - Check the SQL in PR comments before merging
2.  **Use Atlas Cloud** - For additional features like [pre-migration checks](/cloud/features/pre-migration-checks) and [migration troubleshooting](/cloud/features/troubleshooting)
3.  **Test locally first** - Run migrations against a local database before pushing
4.  **Use environment-specific configs** - Separate configurations for staging and production
5.  **Enable drift detection** - Use OpenTaco's drift detection to monitor schema drift

## Troubleshooting[​](#troubleshooting "Direct link to Troubleshooting")


### Docker Not Available[​](#docker-not-available "Direct link to Docker Not Available")


If you get errors about Docker not being available for the dev database, ensure the GitHub runner has Docker access or use an external dev database:
```codeBlockLines_AdAo
variable "dev_database_url" {  type = string}variable "atlas_token" {  type = string}provider "atlas" {  dev_url = var.dev_database_url  # Use a dedicated dev database  cloud {    repo  = "openTaco"    token = var.atlas_token  }}
```

### State Locking Issues[​](#state-locking-issues "Direct link to State Locking Issues")


If you encounter state locking issues, check OpenTaco's [PR-level locks](https://docs.opentaco.dev/features/pr-level-locks) documentation for configuration options.

### Plan Shows No Changes[​](#plan-shows-no-changes "Direct link to Plan Shows No Changes")


If `terraform plan` shows no changes but you expect migrations:

1.  Ensure migration files are properly generated with `atlas migrate hash`
2.  Check that the `atlas.sum` file is committed
3.  Verify the `dir` path in your Terraform configuration

## Conclusion[​](#conclusion "Direct link to Conclusion")


By combining Atlas Terraform Provider with OpenTaco, you get a powerful database migration pipeline that:

*   Provides visibility into changes through PR comments
*   Automates migration application on merge
*   Handles concurrency and locking
*   Integrates seamlessly with your existing Terraform workflow

## Need More Help?[​](#need-more-help "Direct link to Need More Help?")


*   [Atlas Terraform Provider Registry](https://registry.terraform.io/providers/ariga/atlas/latest/docs)
*   [Join the Ariga Discord Server](https://discord.com/invite/zZ6sWVg6NT)

*   [Prerequisites](#prerequisites)
*   [How It Works](#how-it-works)
*   [Project Setup](#project-setup)
    *   [Step 1: Define Your Database Schema](#step-1-define-your-database-schema)
    *   [Step 2: Create the Terraform Configuration](#step-2-create-the-terraform-configuration)
    *   [Step 3: Generate Your First Migration](#step-3-generate-your-first-migration)
    *   [Step 4: Initialize Terraform Locally](#step-4-initialize-terraform-locally)
    *   [Step 5: Set Up OpenTaco in Your Repository](#step-5-set-up-opentaco-in-your-repository)
    *   [Step 6: Configure GitHub Secrets](#step-6-configure-github-secrets)
    *   [Step 7: Commit and Push](#step-7-commit-and-push)
*   [Workflow in Action](#workflow-in-action)
    *   [Creating a Migration](#creating-a-migration)
    *   [PR Preview](#pr-preview)
    *   [Applying Migrations](#applying-migrations)
*   [Advanced Configuration](#advanced-configuration)
    *   [Using Declarative Workflow](#using-declarative-workflow)
    *   [Multiple Databases](#multiple-databases)
    *   [Apply on Merge](#apply-on-merge)
*   [Best Practices](#best-practices)
*   [Troubleshooting](#troubleshooting)
    *   [Docker Not Available](#docker-not-available)
    *   [State Locking Issues](#state-locking-issues)
    *   [Plan Shows No Changes](#plan-shows-no-changes)
*   [Conclusion](#conclusion)
*   [Need More Help?](#need-more-help)