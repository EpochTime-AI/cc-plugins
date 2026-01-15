Managing Multi-Tenant Migrations with Atlas Cloud Control Plane | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

In the previous section, we demonstrated how to use the Atlas CLI to manage migrations for a database-per-tenant architecture. Next, we will see how to use the Atlas Cloud Control Plane to manage migrations across multiple databases.

## Setting up[​](#setting-up "Direct link to Setting up")


In this section, we will be continuing our minimal example from before, so if you are just joining us, please follow the steps in the previous section to set up your project.

Additionally, you will need an Atlas Cloud account. If you don't have one, you can sign up for free by running the following command and following the instructions on the screen:
```codeBlockLines_AdAo
atlas login
```

### Pushing our project to Atlas Cloud[​](#pushing-our-project-to-atlas-cloud "Direct link to Pushing our project to Atlas Cloud")


In order to manage our migrations across multiple databases, we need push our project to the Atlas Cloud Schema Registry. But first, let's set up a local `env` block in our `atlas.hcl` file. Append the following to the file:
```codeBlockLines_AdAo
env "local" {  dev = "sqlite://?mode=memory"  migration {    dir = "file://migrations"  }}
```
Next, push the project to the Atlas Cloud Schema Registry by running the following command:
```codeBlockLines_AdAo
atlas migrate push --env prod db-per-tenant
```
Atlas will push our migration directory to the Schema Registry and print the URL of the project, for example:
```codeBlockLines_AdAo
https://rotemtam85.atlasgo.cloud/dirs/4294967396
```

## Working with Atlas Cloud[​](#working-with-atlas-cloud "Direct link to Working with Atlas Cloud")


### Deploying from the Registry[​](#deploying-from-the-registry "Direct link to Deploying from the Registry")


Once we have successfully pushed our project to the Schema Registry, we can deploy from it to our target databases. To do this, let's make a small change to our `prod` env in `atlas.hcl`:
```codeBlockLines_AdAo
env "prod" {  for_each = toset(local.tenant)  url = "sqlite://${each.value}.db"  migration {    dir = "atlas://db-per-tenant"  }}
```
Now, we can deploy the migrations to our target databases by running:
```codeBlockLines_AdAo
atlas migrate apply --env prod
```
Atlas will read the most recent version of our migration directory from the schema registry, apply the migrations to each target database, report the results to Atlas Cloud, and print the results:
```codeBlockLines_AdAo
No migration files to executeNo migration files to executehttps://rotemtam85.atlasgo.cloud/deployments/sets/94489280593
```
In this case, we see that there were no new migrations to apply to the target databases. Let's show how this flow works when there is work to be done in the next section.

### Another migration[​](#another-migration "Direct link to Another migration")


Let's plan another migration to our project. Create a new migration file by running:
```codeBlockLines_AdAo
atlas migrate new --edit seed_users
```
In the editor, add the following SQL statements:
```codeBlockLines_AdAo
INSERT INTO users (id, name) VALUES (1, "a8m");INSERT INTO users (id, name) VALUES (2, "rotemtam");
```
Save the file and exit the editor. Let's push the new migration to the Schema Registry:
```codeBlockLines_AdAo
atlas migrate push --env prod db-per-tenant
```

### Deploying the new migration[​](#deploying-the-new-migration "Direct link to Deploying the new migration")


After successfully pushing the new migration, we can deploy it to our target databases by running:
```codeBlockLines_AdAo
atlas migrate apply --env prod
```
Atlas will apply the new migration to each target database and print the results:
```codeBlockLines_AdAo
Migrating to version 20240721111345 from 20240721101205 (1 migrations in total):  -- migrating version 20240721111345    -> INSERT INTO users (id, name) VALUES (1, "a8m");    -> INSERT INTO users (id, name) VALUES (2, "rotemtam");  -- ok (1.106417ms)  -------------------------  -- 7.441584ms  -- 1 migration  -- 2 sql statementsMigrating to version 20240721111345 from 20240721101205 (1 migrations in total):  -- migrating version 20240721111345    -> INSERT INTO users (id, name) VALUES (1, "a8m");    -> INSERT INTO users (id, name) VALUES (2, "rotemtam");  -- ok (1.061709ms)  -------------------------  -- 3.272584ms  -- 1 migration  -- 2 sql statementshttps://rotemtam85.atlasgo.cloud/deployments/sets/94489280594
```
Following the link will take you to the Atlas Cloud UI, where you can see the details of the deployment:

![Screenshot of the Atlas Cloud multi-tenant deployment report, showing two successful migrations.](/assets/images/deployment-set-b73618872f0db29c0a500f05bd173ae4.png)

## Gaining Visibility[​](#gaining-visibility "Direct link to Gaining Visibility")


The Atlas Cloud Control Plane provides a centralized view of all your deployments across multiple databases. You can see the status of each deployment, the target databases, and the results of each migration.

### Database Status[​](#database-status "Direct link to Database Status")


![Screenshot of the Atlas Cloud &#39;Databases&#39; tab, showing the status of multiple tenant databases.](/assets/images/databases-screen-1204f8021475ad64d969411dd5724eee.png)

To view the status of the different databases in your project, navigate to the "Databases" tab in the Atlas Cloud UI. Here, you can see the status of each database, the most recent migration applied, and the results of the migration.

Databases can be in one of three states:

*   Synced - The database is up-to-date with the most recent migration.
*   Pending - The database is waiting for a new migration to be applied.
*   Error - An error occurred while applying the migration.

### Troubleshooting[​](#troubleshooting "Direct link to Troubleshooting")


If an error occurs during a migration, having a centralized view of all your deployments can help you quickly identify the issue and take corrective action. You can view the error message, the target database, and the migration that caused the error.

Suppose we run a deployment that fails during the schema migration phase, we can easily locate the error in the Atlas Cloud UI by navigating to the "Migrations" tab:

![Screenshot of the Atlas Cloud &#39;Migrations&#39; tab, showing a list of deployments with one marked as &#39;Failed&#39;.](/assets/images/migrations-screen-1ec56f3b0bbdede0f6e542a53f7120f3.png)

We quickly find the failed deployment and drill down to diagnose the issue:

![Screenshot of a failed multi-tenant deployment report, showing one tenant migration failed while others passed.](/assets/images/deployment-set-error-fe2968f05ceab34a585064ce44602eda.png)

From the logs, we see that 3 out of 4 migrations passed without action, but the last one failed. We see that it failed on `tenant_4.db` with the error message:
```codeBlockLines_AdAo
Error: sql/migrate: executing statement "INSERT INTO users (id, name) VALUES (1, \"a8m\");" from version "20240721111345": UNIQUE constraint failed: users.id
```
We can further drill down into the specific database target migration:

![Screenshot of the detailed error report for a single failed tenant migration, showing a &#39;UNIQUE constraint failed&#39; error.](/assets/images/deployment-error-fb5a2d03f1de3c1359019c9f468fe050.png)

We now clearly see the issue, our data migration failed due to a unique constraint violation. Now, we can take corrective action to fix the issue and reapply the migration - usually by fixing the problematic data in our target database.

## Conclusion[​](#conclusion "Direct link to Conclusion")


In this section, we demonstrated how to use the Atlas Cloud Control Plane to manage migrations across multiple target databases. We showed how to push our project to the Atlas Cloud Schema Registry, deploy migrations to target databases, and gain visibility into the status of our deployments.

While it is possible to manage migrations using the Atlas CLI, the Atlas Cloud Control Plane provides a centralized view of all your deployments, making it easier to manage and troubleshoot issues across multiple databases.

*   [Setting up](#setting-up)
    *   [Pushing our project to Atlas Cloud](#pushing-our-project-to-atlas-cloud)
*   [Working with Atlas Cloud](#working-with-atlas-cloud)
    *   [Deploying from the Registry](#deploying-from-the-registry)
    *   [Another migration](#another-migration)
    *   [Deploying the new migration](#deploying-the-new-migration)
*   [Gaining Visibility](#gaining-visibility)
    *   [Database Status](#database-status)
    *   [Troubleshooting](#troubleshooting)
*   [Conclusion](#conclusion)