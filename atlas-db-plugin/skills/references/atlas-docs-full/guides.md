Atlas Guides | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Practical guides for using Atlas in your workflow. Learn how to deploy schema changes, integrate Atlas with your tools, and work with databases, ORMs, CI/CD platforms, testing, and compliance.

[

!

Databases

!

Guides for using Atlas with PostgreSQL, MySQL, ClickHouse, SQL Server, and more

](/databases)[

!

ORMs and Frameworks

!

Use your ORM of choice to define your desired schema

](/orms)[

!

AI Coding Assistants

!

Learn how to use Atlas with Cursor, Claude Code and GitHub Copilot

](/guides/ai-tools)

## Deploying to Kubernetes[​](#deploying-to-kubernetes "Direct link to Deploying to Kubernetes")


Deploy database migrations to Kubernetes clusters using GitOps principles with Argo CD, Flux CD, or directly from Atlas Cloud's Schema Registry.

[

##### !GitOps with Argo CD


Deploying to Kubernetes with the Atlas Operator and Argo CD

](/guides/deploying/k8s-argo)[

##### !GitOps with Flux CD


Deploying to Kubernetes with the Atlas Operator and Flux CD

](/guides/deploying/k8s-flux)[

##### !GitOps with Atlas Cloud


Deploying to Kubernetes from Atlas Schema Registry

](/guides/deploying/k8s-cloud-versioned)

## Deploying Schema Migrations[​](#deploying-schema-migrations "Direct link to Deploying Schema Migrations")


Comprehensive guides for deploying schema migrations across various platforms and environments, including cloud providers, container platforms, and integration with secrets management.

[

##### !Introduction


Getting started with deploying schema migrations

](/guides/deploying/intro)[

##### !Modern Database CI/CD


Complete guide to setting up CI/CD workflows for database schema changes with Atlas

](/guides/modern-database-ci-cd)[

##### !Atlas Registry


Using Atlas Registry to deploy, read, and view migrations

](/guides/deploying/remote-directories)[

##### !Container Images


Creating container images for migrations

](/guides/deploying/image)[

##### !AWS ECS (Fargate)


Deploying to AWS with ECS/Fargate

](/guides/deploying/aws-ecs-fargate)[

##### !Fly.io


Deploying to Kubernetes with the Atlas Operator and Fly.io

](/guides/deploying/fly-io)[

##### !GCP CloudSQL


Deploying schema migrations to Google CloudSQL using Atlas

](/guides/deploying/cloud-sql-via-github-actions)[

##### !Working with secrets


Using Atlas to implement IAM Authentication and Secret Stores

](/guides/deploying/secrets)[

##### !SSL Certs (Kubernetes)


Using SSL Certs with the Atlas Operator

](/guides/deploying/k8s-operator-certs)[

##### !Connect to DB from Actions


Connecting to your database from GitHub Actions

](/guides/deploying/connect-to-db-from-github-actions)

## CI/CD Platforms[​](#cicd-platforms "Direct link to CI/CD Platforms")


Integrate Atlas with your preferred CI/CD platform to automate database migrations as part of your deployment pipeline.

[

##### !GitHub Actions


Setting up CI/CD for databases on GitHub Actions

](/guides/ci-platforms/github-versioned)[

##### !GitLab CI


Setting up CI/CD for databases on GitLab

](/guides/ci-platforms/gitlab-versioned)[

##### !Azure DevOps (GitHub)


Setting up CI/CD with Azure DevOps and GitHub

](/guides/ci-platforms/azure-devops-github)[

##### !Azure DevOps (Repos)


Setting up CI/CD with Azure DevOps and Azure Repos

](/guides/ci-platforms/azure-devops-repos)

## AI Coding Assistants[​](#ai-coding-assistants "Direct link to AI Coding Assistants")


Learn how to use Atlas with AI coding assistants like Cursor, Claude Code, and GitHub Copilot to generate and manage database migrations more effectively.

[

##### !Introduction


Using Atlas with AI agents for database migration management

](/guides/ai-tools)[

##### !GitHub Copilot Instructions


Configure GitHub Copilot with Atlas-specific instructions

](/guides/ai-tools/github-copilot-instructions)[

##### !Cursor Instructions


Set up Cursor with Atlas-specific rules

](/guides/ai-tools/cursor-rules)[

##### !Claude Code Instructions


Set up Claude Code with Atlas-specific instructions

](/guides/ai-tools/claude-code-instructions)

## Security and Compliance[​](#security-and-compliance "Direct link to Security and Compliance")


Ensure your database migrations meet compliance requirements and follow security best practices with Atlas's approval workflows and audit capabilities.

[

##### !Reviewed and Approved Migrations


Enforcing reviewed and approved schema migrations for compliance

](/guides/reviewed-approved-migrations)[

##### !Environment Promotion


Managing environment promotion workflows for database changes

](/guides/environment-promotion)

## Database-per-Tenant[​](#database-per-tenant "Direct link to Database-per-Tenant")


Manage database schemas in multi-tenant architectures where each tenant has their own database. Learn how to define target groups, deploy migrations across multiple databases, and use the Atlas Cloud Control Plane for centralized management.

[

##### !Introduction


Utilizing Atlas to manage database schemas in Database-per-Tenant architectures

](/guides/database-per-tenant/intro)[

##### !Target Groups


Defining target groups

](/guides/database-per-tenant/target-groups)[

##### !Deploying


Deploying migrations to Database-per-Tenant architecture

](/guides/database-per-tenant/deploying)[

##### !Rollout Strategies


Staged deployments with canary patterns, parallelism, and error handling

](/guides/database-per-tenant/rollout)[

##### !Cloud Control Plane


Using the Atlas Cloud Control Plane to manage migrations across multiple databases

](/guides/database-per-tenant/control-plane)

## Migration Tools[​](#migration-tools "Direct link to Migration Tools")


Compare Atlas with other migration tools and learn how to migrate from existing tools to Atlas.

[

##### !Atlas vs Flyway


Comparing Atlas with Flyway and migrating from Flyway

](/guides/atlas-vs-flyway)[

##### !Atlas vs Liquibase


Comparing Atlas with Liquibase

](/guides/atlas-vs-liquibase)[

##### !Flyway Snapshot Alternative


Using Atlas schema inspect as an alternative to Flyway's snapshot command

](/guides/flyway-snapshot-alternative)[

##### !Flyway Undo Alternative


Using Atlas Migrate Down as an alternative to Flyway's undo scripts

](/guides/flyway-undo-alternative)[

##### !Migrating from Flyway


Step-by-step guide to migrate your Flyway-based project to Atlas

](/guides/migrate-flyway-to-atlas)[

##### !golang-migrate


Working with golang-migrate and Atlas

](/guides/migration-tools/golang-migrate)[

##### !Importing from goose


Importing existing migrations from goose

](/guides/migration-tools/goose-import)

## Testing[​](#testing "Direct link to Testing")


Comprehensive testing guides for ensuring your database migrations work correctly across different scenarios and database objects.

[

##### !docker-compose


Creating Atlas integration tests with docker-compose

](/guides/testing/docker-compose)[

##### !GitHub Actions


Creating Atlas integration tests with GitHub Actions

](/guides/testing/github-actions)[

##### !Data Migrations


Testing data migrations with Atlas

](/guides/testing/data-migrations)[

##### !Views


Testing views with Atlas

](/guides/testing/views)[

##### !Functions


Testing functions with Atlas

](/guides/testing/functions)[

##### !Domains


Testing domains with Atlas

](/guides/testing/domains)[

##### !Stored Procedures


Testing stored procedures with Atlas

](/guides/testing/procedures)[

##### !Triggers


Testing triggers with Atlas

](/guides/testing/triggers)

## Additional Guides[​](#additional-guides "Direct link to Additional Guides")


Additional resources for specific use cases and advanced features, including Docker deployment, Terraform integration, template directories, and programmatic inspection.

[

##### !Atlas in Docker


Learn how to run Atlas in Docker containers

](/guides/atlas-in-docker)[

##### !Named Databases with Terraform


Managing named databases with Terraform and Atlas

](/guides/terraform/named-databases)[

##### !Template Directories


Using template directories for dynamic migration generation

](/guides/migration-dirs/template-directory)[

##### !Programmatic Inspection Output


Using Go templates for programmatic inspection output

](/guides/go-templates)

*   [Deploying to Kubernetes](#deploying-to-kubernetes)
*   [Deploying Schema Migrations](#deploying-schema-migrations)
*   [CI/CD Platforms](#cicd-platforms)
*   [AI Coding Assistants](#ai-coding-assistants)
*   [Security and Compliance](#security-and-compliance)
*   [Database-per-Tenant](#database-per-tenant)
*   [Migration Tools](#migration-tools)
*   [Testing](#testing)
*   [Additional Guides](#additional-guides)