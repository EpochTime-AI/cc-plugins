Manage MySQL on RDS with Terraform and Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

### TL;DR[​](#tldr "Direct link to TL;DR")


*   Many teams use [Terraform](https://terraform.io/), a popular, open-source, infrastructure-as-code (IaC) tool created by [HashiCorp](https://www.hashicorp.com/) to provision their cloud infrastructure.
*   [RDS](https://aws.amazon.com/rds/) is a fully managed relational database service by AWS.
*   The popular [AWS Terraform Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) can be used to easily provision database instances and manage them.
*   This guide shows how to use the [Atlas Terraform Provider](https://atlasgo.io/integrations/terraform-provider) to manage the schema of an RDS-managed MySQL database using Terraform as part of their IaC pipelines.

### Schema management in IaC[​](#schema-management-in-iac "Direct link to Schema management in IaC")


[Terraform](https://terraform.io/) is widely used in the industry to provision and manage resources in AWS. One of the popular use-cases teams use Terraform for is to provision the database backing their application on RDS (a fully managed relational database service) using the AWS Terraform Provider. The provider supports a resource named [`aws_db_instance`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance) to provision databases using RDS.

This resource can be used to configure many attributes of the database like the amount of storage, the engine and version of the database, backup settings, etc. However, when it comes to provisioning the database's schema itself, the AWS provider does not offer any support. As a result, many teams do not manage their database schemas as part of their infrastructure-as-code pipeline in Terraform.

In this guide, we will demonstrate how to use Terraform to both provision a MySQL database on AWS with RDS _and_ manage its schema in a single pipeline.

### Prerequisites[​](#prerequisites "Direct link to Prerequisites")


*   Install Terraform ([guide](https://learn.hashicorp.com/tutorials/terraform/install-cli))
*   An [AWS account](https://aws.amazon.com/free/) (free-tier available) with credentials [configured for Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication-and-configuration)

### Source code[​](#source-code "Direct link to Source code")


The full source code for this guide can be found on the [Atlas Examples](https://github.com/ariga/atlas-examples/tree/master/terraform/rds-mysql-example) repository on GitHub.

### Dev database[​](#dev-database "Direct link to Dev database")


To plan a migration from the current to the desired state, Atlas uses a [Dev Database](/concepts/dev-database), which is usually provided by a locally running container with an empty database of the type you work with (such as MySQL or PostgreSQL).

To spin up a local MySQL database that will be used as a dev-database in our example, run:
```codeBlockLines_AdAo
docker run --rm --name atlas-db-dev -d -p 3306:3306 -e MYSQL_DATABASE=example -e MYSQL_ROOT_PASSWORD=pass mysql:8
```
As reference for the next steps, the URL for the Dev Database will be:
```codeBlockLines_AdAo
mysql://root:pass@localhost:3306/example
```

### Getting started[​](#getting-started "Direct link to Getting started")


Create a new directory for our new Terraform project and in it a file named `main.tf`. In this file we will define all the required resources for our project. Start by configuring the providers which we will use in our project:
```codeBlockLines_AdAo
terraform {  required_providers {    aws = {      source  = "hashicorp/aws"      version = "~> 4.0"    }    atlas = {      source  = "ariga/atlas"      version = "0.3.0"    }  }}# Configure the AWS Providerprovider "aws" {  region = "us-east-1"}
```

### Networking[​](#networking "Direct link to Networking")


Next, our database must reside in a network topology. We will use the AWS-maintained Terraform module `terraform-aws-modules/vpc/aws` for creating a new VPC for our demo.
```codeBlockLines_AdAo
// Fetch the list of availability zones from the current region.data "aws_availability_zones" "available" {  state = "available"}// Provision a VPC and subnets in these AZs.module "vpc" {  source  = "terraform-aws-modules/vpc/aws"  version = "3.16.1"  name                 = "atlas-rds-demo"  cidr                 = "10.0.0.0/16"  azs                  = data.aws_availability_zones.available.names  public_subnets       = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]  enable_dns_hostnames = true  enable_dns_support   = true}// Create a DB subnet to provision the database. resource "aws_db_subnet_group" "atlas" {  name       = "atlas-rds-demo"  subnet_ids = module.vpc.public_subnets  tags = {    Name = "Demo"  }}
```
A few things worth noting:

1.  In a more realistic scenario we will not provision a dedicated VPC for our database, rather our database will reside in a more general-purpose VPC which will include compute and other resources.
2.  For the sake of simplicity we are placing our instance in _public_ subnets. In a production use case, it is usually recommended to provision specific subnets for databases which should not be reachable from the internet.

### Security groups[​](#security-groups "Direct link to Security groups")


Next, we provision the security group which governs access to our database instance. _Notice_: in this demo we are creating a database which is _accessible from the public internet_. The reason we need this is that we are running Terraform locally and need it to be able to connect to the database directly in order to manage our schema. In a realistic scenario, we will run Terraform from a server which is located in our VPC and can access our database.
```codeBlockLines_AdAo
// Security group which allows *public access* to our database.// DO NOT use this in production.resource "aws_security_group" "rds" {  name   = "atlas-demo"  vpc_id = module.vpc.vpc_id  ingress {    from_port   = 3306    to_port     = 3306    protocol    = "tcp"    cidr_blocks = ["0.0.0.0/0"]  }  egress {    from_port   = 3306    to_port     = 3306    protocol    = "tcp"    cidr_blocks = ["0.0.0.0/0"]  }  tags = {    Name = "atlas"  }}
```

### Provision the database[​](#provision-the-database "Direct link to Provision the database")


Next, we use the `aws_db_instance` to provision a new database instance:
```codeBlockLines_AdAo
// Generate a random password for our db user.resource "random_password" "password" {  length  = 16  special = true}// Our RDS-based MySQL 8 instance.resource "aws_db_instance" "atlas-demo" {  identifier             = "atlas-demo"  instance_class         = "db.t3.micro"  allocated_storage      = 5  engine                 = "mysql"  engine_version         = "8.0.28"  username               = "atlas"  password               = random_password.password.result  db_subnet_group_name   = aws_db_subnet_group.atlas.name  vpc_security_group_ids = [aws_security_group.rds.id]  parameter_group_name   = "default.mysql8.0"  publicly_accessible    = true  skip_final_snapshot    = true}
```

### Define the desired schema[​](#define-the-desired-schema "Direct link to Define the desired schema")


In a separate file named `schema.hcl` define the desired database schema. To learn more about defining SQL resources with the Atlas language, [see the docs](https://atlasgo.io/atlas-schema/hcl).
```codeBlockLines_AdAo
// Create a new database named "hello"schema "hello" {}// Create a table named "users".table "users" {  schema = schema.hello  column "id" {    type = int  }  column "name" {    type = varchar(255)  }}
```

### Connect everything together[​](#connect-everything-together "Direct link to Connect everything together")


Finally, configure the Atlas Terraform Provider to apply the schema in our `schema.hcl` file on the RDS-managed database instance.
```codeBlockLines_AdAo
locals {  dev_db_url = "mysql://root:pass@localhost:3306/example"}// Load the schema from file and normalize it using the dev database.data "atlas_schema" "hello" {  dev_db_url = local.dev_db_url  src        = file("schema.hcl")}// Apply the normalized schema to the RDS-managed database.resource "atlas_schema" "hello" {  hcl        = data.atlas_schema.hello.hcl  dev_db_url = local.dev_db_url    // The connection string will be: mysql://user:pass@endpoint/  url = "mysql://${aws_db_instance.atlas-demo.username}:${urlencode(random_password.password.result)}@${aws_db_instance.atlas-demo.endpoint}/"}
```

### Time to provision[​](#time-to-provision "Direct link to Time to provision")


We're now ready to provision the resources in our AWS account. Start by initializing the project:
```codeBlockLines_AdAo
terraform init
```
Observe that Terraform downloads the necessary modules and plugins:
```codeBlockLines_AdAo
Initializing modules...Downloading registry.terraform.io/terraform-aws-modules/vpc/aws 3.16.1 for vpc...- vpc in .terraform/modules/vpcInitializing the backend...Initializing provider plugins...- Finding ariga/atlas versions matching "0.3.0-pre.1"...- Finding hashicorp/aws versions matching ">= 3.73.0, ~> 4.0"...- Finding latest version of hashicorp/random...- Installing ariga/atlas v0.3.0-pre.1...- Installed ariga/atlas v0.3.0-pre.1 (self-signed, key ID 45441FCEAAC3770C)- Installing hashicorp/aws v4.35.0...- Installed hashicorp/aws v4.35.0 (signed by HashiCorp)- Installing hashicorp/random v3.4.3...- Installed hashicorp/random v3.4.3 (signed by HashiCorp)Partner and community providers are signed by their developers.If you'd like to know more about provider signing, you can read about it here:https://www.terraform.io/docs/cli/plugins/signing.htmlTerraform has created a lock file .terraform.lock.hcl to record the providerselections it made above. Include this file in your version control repositoryso that Terraform can guarantee to make the same selections by default whenyou run "terraform init" in the future.Terraform has been successfully initialized!
```
Finally, let's provision our infrastructure. Run:
```codeBlockLines_AdAo
terraform apply
```
Terraform offers a plan similar to:
```codeBlockLines_AdAo
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:  + createTerraform will perform the following actions:  # atlas_schema.hello will be created  + resource "atlas_schema" "hello" {      + dev_db_url = (sensitive value)      + hcl        = <<-EOT            table "users" {              schema = schema.hello              column "id" {                null = false                type = int              }              column "name" {                null = false                type = varchar(255)              }            }            schema "hello" {              charset = "latin1"              collate = "latin1_swedish_ci"            }        EOT      + id         = (known after apply)      + url        = (sensitive value)    }        // ... A long list of more changesPlan: 15 to add, 0 to change, 0 to destroy.Do you want to perform these actions?  Terraform will perform the actions described above.  Only 'yes' will be accepted to approve.  Enter a value:
```
If the plan looks right to you, write "yes" and hit Enter.

After a few minutes Terraform should finish provisioning everything and print:
```codeBlockLines_AdAo
Apply complete! Resources: 15 added, 0 changed, 0 destroyed.
```

### Evolving the schema[​](#evolving-the-schema "Direct link to Evolving the schema")


The Atlas provider is useful also in cases where your schema evolves, and you wish to change it. Let's see how we can add a new, unique, column to our `users` table.

Edit `schema.hcl`:
```codeBlockLines_AdAo
schema "hello" {}table "users" {  schema = schema.hello  column "id" {    type = int  }  column "name" {    type = varchar(255)  }  column "email" {    type = varchar(255)  }  index "idx_email" {    columns = [      column.email    ]    unique = true  }}
```
Now, re-apply with Terraform:
```codeBlockLines_AdAo
terraform apply
```
Terraform understands the diff in the desired schema for our database and produces a plan accordingly:
```codeBlockLines_AdAo
Terraform will perform the following actions:  # atlas_schema.hello will be updated in-place  ~ resource "atlas_schema" "hello" {      ~ hcl        = <<-EOT            table "users" {              schema = schema.hello              column "id" {                null = false                type = int              }              column "name" {                null = false                type = varchar(255)              }          +   column "email" {          +     null = false          +     type = varchar(255)          +   }          +   index "idx_email" {          +     unique  = true          +     columns = [column.email]          +   }            }            schema "hello" {              charset = "latin1"              collate = "latin1_swedish_ci"            }        EOT        id         = "mysql://atlas:7%28%3Ej%24rQ%3Ez%3C%3DPrqU%5D@atlas-demo.cripwukl9y7k.us-east-1.rds.amazonaws.com:3306/"        # (2 unchanged attributes hidden)    }Plan: 0 to add, 1 to change, 0 to destroy.Do you want to perform these actions?  Terraform will perform the actions described above.  Only 'yes' will be accepted to approve.  Enter a value:
```
After reviewing the plan, write "yes" at the prompt and hit Enter:
```codeBlockLines_AdAo
Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

### Wrapping up[​](#wrapping-up "Direct link to Wrapping up")


In this guide, we demonstrated how to provision and manage a MySQL instance using Terraform and the AWS and Atlas providers. Following the example here, teams can finally integrate schema management with their full Infrastructure-as-Code workflows.

## Need More Help?[​](#need-more-help "Direct link to Need More Help?")


[Join the Ariga Discord Server](https://discord.gg/zZ6sWVg6NT) for early access to features and the ability to provide exclusive feedback that improves your Database Management Tooling.

*   [TL;DR](#tldr)
*   [Schema management in IaC](#schema-management-in-iac)
*   [Prerequisites](#prerequisites)
*   [Source code](#source-code)
*   [Dev database](#dev-database)
*   [Getting started](#getting-started)
*   [Networking](#networking)
*   [Security groups](#security-groups)
*   [Provision the database](#provision-the-database)
*   [Define the desired schema](#define-the-desired-schema)
*   [Connect everything together](#connect-everything-together)
*   [Time to provision](#time-to-provision)
*   [Evolving the schema](#evolving-the-schema)
*   [Wrapping up](#wrapping-up)
*   [Need More Help?](#need-more-help)