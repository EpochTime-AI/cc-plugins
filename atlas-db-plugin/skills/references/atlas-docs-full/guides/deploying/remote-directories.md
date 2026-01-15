Working with Atlas Registry | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In the past, we have recommended users to [build a migrations Docker image](/guides/deploying/image) as part of their CI pipeline and then use that image during their deployment process. This is still a valid approach, as it bundles together the Atlas binary needed to run migrations with the migrations themselves. However, over the last years we have received feedback from many users that this approach is cumbersome and requires a lot of boilerplate code to be written.

To address this, we have introduced the [Atlas Schema Registry](/cloud/features/registry), which allows you to store schemas and migrations in the cloud and make them available later to your deployment pipelines by their tags.

On a high-level this approach works as follows:

1.  Users sync their migration directory to the Atlas Registry whenever a new migration is merged to the main branch. Learn more about it in the [Syncing Migration Directories](/cloud/directories) doc.
2.  During deployment, Atlas fetches the migration directory from the Atlas Registry by its tag (defaults to `latest`) and applies the migrations to the database.

This guide shows you how to set up this approach for your project.

### Prerequisites[​](#prerequisites "Direct link to Prerequisites")


1.  An Atlas Cloud account with administrator access. If you don't have an account, you can [sign up for free](https://atlasgo.cloud/).
2.  Sync your migration directory from GitHub to your Atlas Cloud account. See [Syncing Migration Directories](/cloud/directories) for more information.
3.  A token for an Atlas Cloud Bot user with permissions report CI/CD runs and read the migration directory. See [Creating a Bot User](/cloud/bots) for more information.

### Deploying migrations using Atlas Registry[​](#deploying-migrations-using-atlas-registry "Direct link to Deploying migrations using Atlas Registry")


Once your migration directory is pushed to the Registry, you can use the `atlas` CLI to fetches the migration directory from the Atlas Registry and apply the migrations to the database.

To get started, create a project configuration file named `atlas.hcl`:
```codeBlockLines_AdAo
env {  name = atlas.env  url  = getenv("DATABASE_URL")  migration {    dir = "atlas://<name of dir>"  }}
```
Let's review what this configuration file does:

1.  We define an environment using the `env` block. To avoid setting database credentials in the configuration file, we use the `DATABASE_URL` environment variable.
2.  To fetch the migration directory from the Atlas Registry we use the `atlas://<name of dir>` URL in the `migration.dir` attribute. The name is the same as the name you used when you [synced your migration directory](/cloud/directories).

### Read Migrations from Atlas Registry[​](#read-migrations-from-atlas-registry "Direct link to Read Migrations from Atlas Registry")


Once you have created your configuration file, you can read the available migrations from the Atlas Registry and apply them to the database using the `atlas` CLI using the following commands:

*   In Local Development
*   In CI/CD Pipelines
```codeBlockLines_AdAo

# Login first to Atlas Cloud.atlas login# Run migrations. Give an environment name, such as --env local.atlas migrate apply --env local

```
Let's review what these commands do:

1.  We run the `atlas login` command to authenticate with Atlas Cloud.
2.  We run the `atlas migrate apply` command to apply migrations to the database. The `--env` flag is used to specify the name of the environment we defined in the configuration file.
```codeBlockLines_AdAo
ATLAS_TOKEN="{{ YOUR_ATLAS_TOKEN }}" atlas migrate apply --env production
```
Let's review what these commands do:

1.  We set the `ATLAS_TOKEN` environment variable to the token we created earlier in the CI/CD pipeline.
2.  We run the `atlas migrate apply` command to apply migrations to the database. The `--env` flag is used to specify the name of the environment we defined in the configuration file.

The `atlas migrate apply` command will run all migrations that have not been applied to the database yet:
```codeBlockLines_AdAo
Migrating to version 20230306221009 (1 migrations in total):  -- migrating version 20230306221009    -> create table users (         id int primary key       );  -- ok (8.60933ms)  -------------------------  -- 68.037117ms  -- 1 migrations  -- 1 sql statements
```

### Viewing migration logs in Atlas Cloud[​](#viewing-migration-logs-in-atlas-cloud "Direct link to Viewing migration logs in Atlas Cloud")


After the migrations have been applied, you can view them in Atlas Cloud by heading to the `/deployments` page in your Atlas Cloud account. You should see a new migration log with the name of the environment you specified in the configuration file. Clicking on the migration-log will show you the details of the migration, including the statements and checks that were applied:

![Screenshot of an Atlas Cloud deployment report showing a successful migration with all checks passed.](/assets/images/check-passed-v1-898ebfd2ad73443c5218309a96288c53.png)

*   [Prerequisites](#prerequisites)
*   [Deploying migrations using Atlas Registry](#deploying-migrations-using-atlas-registry)
*   [Read Migrations from Atlas Registry](#read-migrations-from-atlas-registry)
*   [Viewing migration logs in Atlas Cloud](#viewing-migration-logs-in-atlas-cloud)