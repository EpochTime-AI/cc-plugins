Automatic TypeORM Migrations with Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

## TL;DR[​](#tldr "Direct link to TL;DR")


*   [TypeORM](https://typeorm.io/) is an ORM for TypeScript and JavaScript that can run in many platforms (Node.js, Browser, React Native, NativeScript, and more).
*   [Atlas](https://atlasgo.io) is an open-source tool for inspecting, planning, linting and executing schema changes to your database.
*   Developers using TypeORM can use Atlas to automatically plan schema migrations for them, based on the desired state of their schema instead of crafting them by hand.

## Automatic migration planning for TypeORM[​](#automatic-migration-planning-for-typeorm "Direct link to Automatic migration planning for TypeORM")


TypeORM is a popular ORM widely used in the Node.js community. TypeORM allows users to manage their database schemas using its [schema-synchronization](https://typeorm.io/sequelize-migration#schema-synchronization) feature, which is usually sufficient during development and in many simple cases.

However, at some point, teams need more control and decide to employ the [migrations](https://typeorm.io/migrations) methodology, which is a more robust way to manage your database schema. TypeORM allows users to [generate migration](https://typeorm.io/migrations#generating-migrations) files based on the difference between the current state of the database and the desired state.

In order to apply the schema migrations to the CI/CD pipeline, developers need to manually craft scripts that will execute the migrations against the production database and handle failures manually.

Atlas has many features that can help developers manage their database [CI/CD pipelines](https://atlasgo.io/guides/modern-database-ci-cd) more easily:

*   [Declarative migrations](https://atlasgo.io/declarative/apply) - use a Terraform-like `atlas schema apply` flow to manage your database.
*   [Diff policies](https://atlasgo.io/declarative/diff#diff-policy) - provide Atlas with policies for planning schema changes (such as "always create index concurrently" or "do not drop columns").
*   [CI for schema changes](https://atlasgo.io/cloud/setup-ci) - Atlas can be used during CI to make sure you never merge a pull request.
*   Modern CD integrations - Atlas integrates seamlessly with modern deployment tools such as [Kubernetes](/integrations/kubernetes), [Terraform](https://atlasgo.io/integrations/terraform-provider), [Helm](https://atlasgo.io/guides/deploying/helm), [Flux](https://atlasgo.io/guides/deploying/k8s-flux), and [ArgoCD](https://atlasgo.io/guides/deploying/k8s-argo). This allows you to deploy changes to your database schema as part of your existing deployment pipelines.
*   [Visualization](https://atlasgo.io/blog/2023/08/06/atlas-v-0-13#built-in-schema-visualization) - Atlas users can create beautiful, shareable ERDs of their application data model

In this guide, we will show how Atlas can be used to automatically plan schema migrations for TypeORM users.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


*   A local [TypeORM](https://github.com/typeorm/typeorm) project

If you don't have a TypeORM project handy, you can use [typeorm/typescript-express-example](https://github.com/typeorm/typescript-express-example) as a starting point:
```codeBlockLines_AdAo
git clone git@github.com:typeorm/typescript-express-example.git
```

## Using the Atlas TypeORM Provider[​](#using-the-atlas-typeorm-provider "Direct link to Using the Atlas TypeORM Provider")


In this guide, we will use the [TypeORM Atlas Provider](https://github.com/ariga/atlas-provider-typeorm) to automatically plan schema migrations for a TypeORM project.

### Installation[​](#installation "Direct link to Installation")


Install Atlas from macOS or Linux by running:
```codeBlockLines_AdAo
curl -sSf https://atlasgo.sh | sh
```
See [atlasgo.io](https://atlasgo.io/getting-started#installation) for more installation options.

Install the provider by running:
```codeBlockLines_AdAo
npm install @ariga/atlas-provider-typeorm
```
Make sure all your Node dependencies are installed by running:
```codeBlockLines_AdAo
npm install
```

### Standalone vs Script mode[​](#standalone-vs-script-mode "Direct link to Standalone vs Script mode")


The Atlas TypeORM Provider can be used in two modes:

*   **Standalone** - If all of your TypeORM entities exist in a single Node module, you can use the provider directly to load your TypeORM schema into Atlas.
*   **Script** - In other cases, you can use the provider as an npm package to write a script that loads your TypeORM schema into Atlas.

### Standalone mode[​](#standalone-mode "Direct link to Standalone mode")


In your project directory, create a new file named `atlas.hcl` with the following contents:
```codeBlockLines_AdAo
data "external_schema" "typeorm" {    program = [        "npx",        "@ariga/atlas-provider-typeorm",        "load",        "--path", "./path/to/entities",        "--dialect", "mysql", // mariadb | postgres | sqlite | mssql    ]}env "typeorm" {    src = data.external_schema.typeorm.url    dev = "docker://mysql/8/dev"    migration {        dir = "file://migrations"    }    format {        migrate {            diff = "{{ sql . \"  \" }}"        }    }}
```

### Script mode[​](#script-mode "Direct link to Script mode")


*   JavaScript
*   TypeScript

Create a new file named `load.js` with the following contents:
```codeBlockLines_AdAo
#!/usr/bin/env nodeconst loadEntities = require("@ariga/atlas-provider-typeorm/build/load").loadEntities;const EntitySchema = require("typeorm").EntitySchema;// require typeorm entities you want to loadconst user = new EntitySchema(require("./entities/User"));const blog = new EntitySchema(require("./entities/Blog"));loadEntities("mysql", [user, blog]).then((sql) => {  console.log(sql);});
```
Create a new file named `load.ts` with the following contents:
```codeBlockLines_AdAo
#!/usr/bin/env ts-node-scriptimport { loadEntities } from "@ariga/atlas-provider-typeorm/build/load";// import typeorm entities you want to loadimport { User } from "./entities/User";import { Blog } from "./entities/Blog";loadEntities("mysql", [User, Blog]).then((sql) => {  console.log(sql);});
```
Next, in your project directory, create a new file named `atlas.hcl` with the following contents:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
data "external_schema" "typeorm" {    program = [        "node",        "load.js",    ]}env "typeorm" {    src = data.external_schema.typeorm.url    dev = "docker://mysql/8/dev"    migration {        dir = "file://migrations"    }    format {        migrate {            diff = "{{ sql . \"  \" }}"        }    }}
```
```codeBlockLines_AdAo
data "external_schema" "typeorm" {    program = [        "npx",        "ts-node",        "load.ts",    ]}env "typeorm" {    src = data.external_schema.typeorm.url    dev = "docker://mysql/8/dev"    migration {        dir = "file://migrations"    }    format {        migrate {            diff = "{{ sql . \"  \" }}"        }    }}
```

## Usage[​](#usage "Direct link to Usage")


Atlas supports a [versioned migrations](/concepts/declarative-vs-versioned#versioned-migrations) workflow, where each change to the database is versioned and recorded in a migration file. You can use the `atlas migrate diff` command to automatically generate a migration file that will migrate the database from its latest revision to the current TypeORM schema.

Suppose we have the following TypeORM `entities` directory, with two entities `Blog` and `User`:

*   JavaScript
*   TypeScript

*   Blog.js
*   User.js
```codeBlockLines_AdAo
module.exports = {  name: "Blog",  columns: {    id: {      primary: true,      type: "int",      generated: true,    },    title: {      type: "varchar",    },  },  relations: {    project: {      type: "many-to-one",      target: "User",      joinColumn: {        name: "userId",      },      inverseSide: "blogs",    },  },};
```
```codeBlockLines_AdAo
module.exports = {  name: "User",  columns: {    id: {      primary: true,      type: "int",      generated: true,    },    firstName: {      type: "varchar",    },    lastName: {      type: "varchar",    },  },  relations: {    orders: {      type: "one-to-many",      target: "Blog",      cascade: true,      inverseSide: "user",    },  },};
```
*   Blog.ts
*   User.ts
```codeBlockLines_AdAo
import { Column, Entity, ManyToOne, PrimaryGeneratedColumn } from "typeorm";import { User } from "./User";@Entity()export class Blog {  @PrimaryGeneratedColumn()  id: number;  @Column()  title: string;  @ManyToOne(() => User, (user) => user.blogs)  user: User;}
```
```codeBlockLines_AdAo
import {  Entity,  PrimaryGeneratedColumn,  Column,  OneToMany,} from "typeorm";import { Blog } from "./Blog";@Entity()export class User {  @PrimaryGeneratedColumn()  id: number;  @Column()  firstName: string;  @Column()  lastName: string;  @OneToMany(() => Blog, (blog) => blog.user)  blogs: Blog[];}
```
Using the [Standalone mode](#standalone-mode) configuration file for the Atlas TypeORM Provider, we can generate a migration file by running this command:
```codeBlockLines_AdAo
atlas migrate diff --env typeorm
```
Running this command will generate files similar to this in the `migrations` directory:
```codeBlockLines_AdAo
migrations|-- 20230918143104.sql`-- atlas.sum0 directories, 2 files
```
Examining the contents of `20230918143104.sql`:
```codeBlockLines_AdAo
-- Create "user" tableCREATE TABLE `user` (  `id` int NOT NULL AUTO_INCREMENT,  `firstName` varchar(255) NOT NULL,  `lastName` varchar(255) NOT NULL,  PRIMARY KEY (`id`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-- Create "blog" tableCREATE TABLE `blog` (  `id` int NOT NULL AUTO_INCREMENT,  `title` varchar(255) NOT NULL,  `userId` int NULL,  PRIMARY KEY (`id`),  INDEX `FK_fc46ede0f7ab797b7ffacb5c08d` (`userId`),  CONSTRAINT `FK_fc46ede0f7ab797b7ffacb5c08d` FOREIGN KEY (`userId`) REFERENCES `user` (`id`) ON UPDATE NO ACTION ON DELETE NO ACTION) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```
Amazing! Atlas automatically generated a migration file that will create the `User` and `Blog` tables in our database!

Next, alter the `User` entity to add a new `age` field:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
    lastName: {      type: "varchar",    },+   age: {+     type: "int",+   }
```
```codeBlockLines_AdAo
  @Column()  lastName: string;+ @Column()+ age: number;
```
Re-run this command:
```codeBlockLines_AdAo
atlas migrate diff --env typeorm
```
Observe a new migration file is generated:
```codeBlockLines_AdAo
-- Modify "user" tableALTER TABLE `user` ADD COLUMN `age` int NOT NULL;
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


In this guide we demonstrated how projects using TypeORM can use Atlas to automatically plan schema migrations based only on their data model. To learn more about executing these migrations against your production database, read the documentation for the [`migrate apply`](/versioned/apply) command.

Have questions? Feedback? Find our team [on our Discord server](https://discord.gg/zZ6sWVg6NT)

*   [TL;DR](#tldr)
*   [Automatic migration planning for TypeORM](#automatic-migration-planning-for-typeorm)
*   [Prerequisites](#prerequisites)
*   [Using the Atlas TypeORM Provider](#using-the-atlas-typeorm-provider)
    *   [Installation](#installation)
    *   [Standalone vs Script mode](#standalone-vs-script-mode)
    *   [Standalone mode](#standalone-mode)
    *   [Script mode](#script-mode)
*   [Usage](#usage)
*   [Conclusion](#conclusion)