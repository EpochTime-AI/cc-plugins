Secrets and IAM Authentication: Best Practices for Managing Database Credentials in Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Database credentials are considered sensitive information and should be treated as such. In this guide, we will show how to use Atlas to handle database credentials in a secure manner. We will present two strategies for handling database credentials, and show how to use Atlas to implement them: IAM Authentication and Secret Stores.

## Secret Stores[​](#secret-stores "Direct link to Secret Stores")


Secret stores are systems or services that allow users to store and retrieve sensitive information. The main features of secret stores are encryption, access control, and auditing. All cloud providers offer some form of secret store service, and there are also many open-source alternatives.

When working with secret stores, Atlas assumes that the secret store is already provisioned and configured. Atlas supports the following secret stores:

*   [AWS Secrets Manager](/guides/deploying/secrets?secret-store=aws-secret-manager#retrieving-credentials-from-a-secret-store)
*   [AWS Systems Manager Parameter Store](/guides/deploying/secrets?secret-store=aws-parameter-store#retrieving-credentials-from-a-secret-store)
*   [GCP Secret Manager](/guides/deploying/secrets?secret-store=gcp-secret-manager#retrieving-credentials-from-a-secret-store)
*   [HashiCorp Vault](/guides/deploying/secrets?secret-store=hashivault#retrieving-credentials-from-a-secret-store) [Atlas Pro](/features#pro)

Support for other secret stores is planned, if you have a specific request, please [open an issue](https://github.com/ariga/atlas/issues/new).

## IAM Authentication[​](#iam-authentication "Direct link to IAM Authentication")


IAM authentication is a mechanism that allows users to authenticate to a database using their cloud provider credentials. The main advantage of IAM authentication is that it allows users to avoid storing database credentials altogether. Although setting this up may be more cumbersome, it is considered a best practice for many cloud providers. IAM authentication is also more secure than using passwords. Even strong passwords stored in encrypted form can be leaked and used by attackers. IAM authentication allows users to avoid storing database credentials altogether.

IAM authentication is currently supported on GCP and AWS. Support for other cloud providers is planned as well, if you have a specific request, please [open an issue](https://github.com/ariga/atlas/issues/new).

## Retrieving Credentials from a Secret Store[​](#retrieving-credentials-from-a-secret-store "Direct link to Retrieving Credentials from a Secret Store")


Atlas can retrieve information from a secret store at runtime using the `runtimevar` data source. The `runtimevar` data source uses the [`runtimevar` package](https://gocloud.dev/howto/runtimevar/) from the Go [CDK](https://gocloud.dev/). To read more about using `runtimevar` with Atlas, view the [data source documentation](/atlas-schema/projects#data-source-runtimevar).

*   AWS Secret Manager
*   AWS Systems Manager Parameter Store
*   GCP Secret Manager
*   HashiCorp Vault

1.  Create a secret a secret to store the database password using the AWS CLI:

    ```codeBlockLines_AdAo
    aws secretsmanager create-secret \  --name db-pass-demo \  --secret-string "p455w0rd"
    ```

    The CLI prints out:

    ```codeBlockLines_AdAo
    {    "ARN": "arn:aws:secretsmanager:us-east-1:1111111111:secret:db-pass-demo-aBiM5k",    "Name": "db-pass-demo",    "VersionId": "b702431d-174f-4a8f-aee5-b80e687b8bf1"}
    ```

    Note the database secret name and the region (`us-east-1`), we will use them in the next part.

2.  Create a new file named `atlas.hcl` with the following contents:

    ```codeBlockLines_AdAo
    data "runtimevar" "pass" {  url = "awssecretsmanager://db-pass-demo?region=us-east-1"}env "dev" {  url = "mysql://root:${data.runtimevar.pass}@host:3306/database"}
    ```

    Let's breakdown the configuration:

    *   The `runtimevar` data source is used to retrieve the database password from AWS Secrets Manager.
    *   We define an `env` named `dev`. The value retrieved by the `runtimevar` data source is interpolated into the `url` attribute using the `${data.runtimevar.pass}` expression.
3.  Run `atlas schema inspect --env dev` to verify that Atlas is able to connect to the database.

note

If you using [RDS Password Management](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-secrets-manager.html#:~:text=RDS%20automatically%20generates%20database%20credentials,access%20and%20plain%20text%20view.), RDS will maintain a secret in JSON format similar to:
```codeBlockLines_AdAo
{  "username": "admin",  "password": "p455w0rd"}
```
To decode the json payload and retrieve the password from it, use the `jsondecode` standard lib func. also notice this password may contain special characters and therefore must be escaped using the `urlescape` func.

Use the next `atlas.hcl` file as an example:
```codeBlockLines_AdAo
data "runtimevar" "pass" {    url = "awssecretsmanager://<rds_secret_name>?region=us-east-1"}locals {    pass = jsondecode(data.runtimevar.pass).password}env "dev" {    url = "mysql://root:${urlescape(local.pass)}@host:3306/database"}
```
1.  Create a encrypted parameter to store the database password using the AWS CLI:

    ```codeBlockLines_AdAo
    aws ssm put-parameter \  --name "db-pass-demo" \  --value "p455w0rd" \  --region "us-east-1" \  --type "SecureString" \  --tags "Key=Env,Value=AtlasDemo"
    ```

    The CLI prints out:

    ```codeBlockLines_AdAo
    {    "Version": 1,    "Tier": "Standard"}
    ```

    Note the database parameter name and the region (`us-east-1`), we will use them in the next part.

2.  Create a new file named `atlas.hcl` with the following contents:

    ```codeBlockLines_AdAo
    data "runtimevar" "pass" {  url = "awsparamstore://db-pass-demo?region=us-east-1&decoder=string"}env "dev" {  url = "mysql://root:${data.runtimevar.pass}@host:3306/database"}
    ```

    Let's breakdown the configuration:

    *   The `runtimevar` data source is used to retrieve the database password from AWS Parameter Store.
    *   We define an `env` named `dev`. The value retrieved by the `runtimevar` data source is interpolated into the `url` attribute using the `${data.runtimevar.pass}` expression.
3.  Run `atlas schema inspect --env dev` to verify that Atlas is able to connect to the database.

1.  Create a secret a secret to store the database password using the GCP CLI:

    ```codeBlockLines_AdAo
    printf "p455w0rd" | gcloud secrets create db-pass-demo --data-file=-
    ```

    The CLI prints out:

    ```codeBlockLines_AdAo
    Created version [1] of the secret [db-pass-demo].
    ```

2.  Create a new file named `atlas.hcl` with the following contents:

    ```codeBlockLines_AdAo
    data "runtimevar" "pass" {  url = "gcpsecretmanager://projects/my-project/secrets/db-pass-demo"}env "dev" {  url = "mysql://root:${data.runtimevar.pass}@host:3306/database"}
    ```

    Let's breakdown the configuration:

    *   The `runtimevar` data source is used to retrieve the database password from GCP Secret Manager. The URL is composed of the project and secret name. If you are working locally in a multi-project environment, you can find out the name of the active project by running `gcloud config get-value project`.
    *   We define an `env` named `dev`. The value retrieved by the `runtimevar` data source is interpolated into the `url` attribute using the `${data.runtimevar.pass}` expression.
3.  Run `atlas schema inspect --env dev` to verify that Atlas is able to connect to the database.

[Atlas Pro Feature](/features#pro)

The HashiVault data source is available only to [Atlas Pro users](/features#pro). To use this feature, run:
```codeBlockLines_AdAo
atlas login
```
1.  Store a secret in HashiCorp Vault using the KV secrets engine:
```codeBlockLines_AdAo
vault kv put secret/database password=$th!s-!s-@-v3ry-$tr0ng-p@$$w0rd username=im-root
```
The CLI prints out:
```codeBlockLines_AdAo
==== Secret Path ====secret/data/database======= Metadata =======Key                Value..truncated..
```
2.  Create a new file named `atlas.hcl` with the following contents:

atlas.hcl
```codeBlockLines_AdAo
data "runtimevar" "vault" {  url = "hashivault://secret/data/database"}locals {  vault = jsondecode(data.runtimevar.vault)}env "dev" {  url = "mysql://${local.vault.username}:${local.vault.password}@aws-rds-instance.us-east-1.rds.amazonaws.com:3306/database"  dev = "docker://mysql/8/dev"}
```
Let's breakdown the configuration:

*   The `runtimevar` data source is used to retrieve secrets from HashiCorp Vault using the KV secrets engine.
*   We use `jsondecode()` to parse the JSON response from Vault and extract individual values.
*   We define an `env` named `dev`. The values retrieved from Vault are interpolated into the `url` attribute.

3.  Set the required environment variables and run the command:
```codeBlockLines_AdAo
VAULT_ADDR="https://vault.example.com:8200" VAULT_TOKEN="your-vault-token" atlas schema inspect --env dev
```
note

For KV v2, use the endpoint format `secret/data/your-secret-name`. For KV v1, use `secret/your-secret-name` (as mentioned in the [Vault documentation](https://developer.hashicorp.com/vault/docs/secrets/kv#version-comparison)).

## Using IAM Authentication[​](#using-iam-authentication "Direct link to Using IAM Authentication")


Atlas can retrieve short-lived credentials from the cloud provider and use them to connect to the database. The passwords are retrieved using special data sources that are specific to each cloud provider.

*   aws\_rds\_token
*   gcp\_cloudsql\_token

1.  Enable IAM Authentication for your database. For instructions on how to do this, [see the AWS documentation](https://aws.github.io/aws-sdk-go-v2/docs/sdk-utilities/rds/#iam-authentication).

2.  Create a database user and grant it permission to authenticate using IAM, see [the AWS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.DBAccounts.html) for instructions.

3.  Create an IAM role with the "rds-db:connect" permission for the specific database and user. For instructions on how to do this, [see the AWS documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.IAMPolicy.html).

4.  Create a new file named `atlas.hcl` with the following contents:

    *   postgres
    *   mysql

    ```codeBlockLines_AdAo
    locals {  user = "iamuser"  endpoint = "hostname-of-db.example9y7k.us-east-1.rds.amazonaws.com:5432"}data "aws_rds_token" "db" {  region = "us-east-1"  endpoint = local.endpoint  username = local.user}env "rds" {  url = "postgres://${local.user}:${urlescape(data.aws_rds_token.db)}@${local.endpoint}/postgres"}
    ```

    ```codeBlockLines_AdAo
    locals {  user = "iamuser"  endpoint = "hostname-of-db.example9y7k.us-east-1.rds.amazonaws.com:3306"}data "aws_rds_token" "db" {  region = "us-east-1"  endpoint = local.endpoint  username = local.user}env "rds" {  url = "mysql://${local.user}:${urlescape(data.aws_rds_token.db)}@${local.endpoint}?tls=preferred&allowCleartextPasswords=true"}
    ```

    Let's breakdown the configuration:

    *   The `aws_rds_token` data source is used to retrieve the database password from AWS Secrets Manager.
    *   We define an `env` named `rds`. The value retrieved by the `aws_rds_token` data source is interpolated into the `url` attribute using the `${data.aws_rds_token.db}` expression.

The `gcp_cloudsql_token` data source generates a short-lived token for an [GCP CloudSQL](https://cloud.google.com/sql) database using [IAM Authentication](https://cloud.google.com/sql/docs/mysql/authentication#manual).

To use this data source:

1.  Enable IAM Authentication for your database. For instructions on how to do this, [see the GCP documentation](https://cloud.google.com/sql/docs/mysql/create-edit-iam-instances).

2.  Create a database user and grant it permission to authenticate using IAM, see [the GCP documentation](https://cloud.google.com/sql/docs/mysql/add-manage-iam-users) for instructions.

3.  Create a file named `atlas.hcl` with the following contents:

    atlas.hcl

    ```codeBlockLines_AdAo
    locals {  user = "iamuser"  endpoint = "34.143.100.1:3306"}data "gcp_cloudsql_token" "db" {}env "cloudsql" {  url = "mysql://${local.user}:${urlescape(data.gcp_cloudsql_token.db)}@${local.endpoint}/?allowCleartextPasswords=1&tls=skip-verify&parseTime=true"}
    ```

    note

    The `allowCleartextPasswords` and `tls` parameters are required for the MySQL driver to connect to CloudSQL. For PostgreSQL, use `sslmode=require` to connect to the database.

    Let's breakdown the configuration:

    *   The `gcp_cloudsql_token` data source is used to retrieve the database password from GCP CloudSQL.
    *   We define an `env` named `cloudsql`. The value retrieved by the `gcp_cloudsql_token` data source is interpolated into the `url` attribute using the `${data.gcp_cloudsql_token.db}` expression.

*   [Secret Stores](#secret-stores)
*   [IAM Authentication](#iam-authentication)
*   [Retrieving Credentials from a Secret Store](#retrieving-credentials-from-a-secret-store)
*   [Using IAM Authentication](#using-iam-authentication)