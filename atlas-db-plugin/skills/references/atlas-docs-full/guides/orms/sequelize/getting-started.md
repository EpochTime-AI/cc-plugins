Automatic migration planning for Sequelize | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

## TL;DR[​](#tldr "Direct link to TL;DR")


*   [Sequelize](https://sequelize.org/) is an ORM library that's widely used in the Node.js community.
*   [Atlas](https://atlasgo.io) is an open-source tool for inspecting, planning, linting and executing schema changes to your database.
*   Developers using Sequelize can use Atlas to automatically plan schema migrations for them, based on the desired state of their schema instead of crafting them by hand.

## Automatic migration planning for Sequelize[​](#automatic-migration-planning-for-sequelize "Direct link to Automatic migration planning for Sequelize")


Sequelize is a popular ORM widely used in the Node.js community. Sequelize allows users to manage their database schemas using its [sync](https://sequelize.org/docs/v6/core-concepts/model-basics/#model-synchronization/) feature, which is usually sufficient during development and in many simple cases.

However, at some point, teams need more control and decide to employ the [migrations](https://sequelize.org/master/manual/migrations.html) methodology, which is a more robust way to manage your database schema. The problem with [creating migrations](https://sequelize.org/docs/v6/other-topics/migrations/#creating-the-first-model-and-migration) in Sequelize is that they are usually written by hand, which is error-prone and time-consuming.

Atlas can automatically plan database schema migrations for developers using Sequelize. Atlas plans migrations by calculating the diff between the _current_ state of the database, and its _desired_ state.

In the context of [versioned migrations](/concepts/declarative-vs-versioned#versioned-migrations), the current state can be thought of as the database schema that would have been created by applying all previous migration scripts.

The desired schema of your application can be provided to Atlas via an [External Schema](/atlas-schema/projects#data-source-external_schema) data source, which is any program that can output a SQL schema definition to stdout.

To use Atlas with Sequelize, users can utilize the [Sequelize Atlas Provider](https://github.com/ariga/atlas-provider-sequelize), a small program that can be used to load the schema of a Sequelize project into Atlas.

In this guide, we will show how Atlas can be used to automatically plan schema migrations for Sequelize users.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


*   A local [Sequelize](https://github.com/sequelize/sequelize) project.

If you don't have a Sequelize project handy, you can use [sequelize/express-example](https://github.com/sequelize/express-example) as a starting point:
```codeBlockLines_AdAo
git clone git@github.com:sequelize/express-example.git
```

## Using the Atlas Sequelize Provider[​](#using-the-atlas-sequelize-provider "Direct link to Using the Atlas Sequelize Provider")


In this guide, we will use the [Sequelize Atlas Provider](https://github.com/ariga/atlas-provider-sequelize) to automatically plan schema migrations for a Sequelize project.

The Atlas Sequelize Provider can be used in two modes:

*   **Standalone** - If all of your Sequelize models exist in a single Node module, you can use the provider directly to load your Sequelize schema into Atlas.
*   **Script** - In other cases, you can use the provider as a Node module to write a script that loads your Sequelize schema into Atlas.

This guide covers Standalone mode. For Script mode, please see [the relevant guide](/guides/orms/sequelize/script).

### Installation[​](#installation "Direct link to Installation")


1.  Install Atlas from macOS or Linux by running:

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

2.  Install the provider by running:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
npm install @ariga/atlas-provider-sequelize
```
```codeBlockLines_AdAo
npm install @ariga/ts-atlas-provider-sequelize
```
Make sure all your Node dependencies are installed by running:
```codeBlockLines_AdAo
npm install
```

### Setup[​](#setup "Direct link to Setup")


In your project directory, create a new file named `atlas.hcl` with the following contents:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
data "external_schema" "sequelize" {    program = [        "npx",        "@ariga/atlas-provider-sequelize",        "load",        "--path", "./path/to/models",        "--dialect", "mysql", // mariadb | postgres | sqlite | mssql    ]}env "sequelize" {    src = data.external_schema.sequelize.url    dev = "docker://mysql/8/dev"    migration {        dir = "file://migrations"    }    format {        migrate {            diff = "{{ sql . \"  \" }}"        }    }}
```
```codeBlockLines_AdAo
data "external_schema" "sequelize" {    program = [        "npx",        "@ariga/ts-atlas-provider-sequelize",        "load",        "--path", "./path/to/models",        "--dialect", "mysql", // mariadb | postgres | sqlite | mssql    ]}env "sequelize" {    src = data.external_schema.sequelize.url    dev = "docker://mysql/8/dev"    migration {        dir = "file://migrations"    }    format {        migrate {            diff = "{{ sql . \"  \" }}"        }    }}
```
note

Be sure to update the `--path` in the config file with the correct path, as well as `--dialect` and `dev` depending on which database you are using.

## Usage[​](#usage "Direct link to Usage")


Atlas supports a [versioned migrations](/concepts/declarative-vs-versioned#versioned-migrations) workflow, where each change to the database is versioned and recorded in a migration file. You can use the `atlas migrate diff` command to automatically generate a migration file that will migrate the database from its latest revision to the current Sequelize schema.

Suppose we have the following Sequelize `models` directory, with two models `user` and `task`:

*   JavaScript
*   TypeScript

*   task.js
*   user.js
```codeBlockLines_AdAo
'use strict';module.exports = (sequelize, DataTypes) => {  const Task = sequelize.define('Task', {    complete: {      type: DataTypes.BOOLEAN,      defaultValue: false,    }  });  Task.associate = (models) => {    Task.belongsTo(models.User, {      foreignKey: {        name: 'userID',        allowNull: false      },      as: 'tasks'    });  };  return Task;};
```
```codeBlockLines_AdAo
'use strict';module.exports = function(sequelize, DataTypes) {  const User = sequelize.define('User', {    name: {      type: DataTypes.STRING,      allowNull: false    },    email: {      type: DataTypes.STRING,      allowNull: false,      validate: {        isEmail: true      },    }  });  User.associate = (models) => {    User.hasMany(models.Task, {      foreignKey: {        name: 'userID',        allowNull: false      },      as: 'tasks'    });  };  return User;};
```
*   task.ts
*   user.ts
```codeBlockLines_AdAo
import { Table, Column, Model, PrimaryKey, ForeignKey, BelongsTo, AutoIncrement, DataType, AllowNull, Default } from "sequelize-typescript";import User from "./user";@Table({ tableName: "Tasks" })class Task extends Model {  @PrimaryKey  @AutoIncrement  @Column(DataType.INTEGER)  id!: number; @Default(false) @Column(DataType.BOOLEAN) complete!: boolean;  @AllowNull(false)  @ForeignKey(() => User)  @Column(DataType.INTEGER)  userID!: number;  @BelongsTo(() => User)  user!: User;}export default Task;
```
```codeBlockLines_AdAo
import { Table, Column, Model, PrimaryKey, AutoIncrement, DataType, AllowNull, HasMany} from "sequelize-typescript";import Task from "./task";@Table({ tableName: "Users" })class User extends Model {  @PrimaryKey  @AutoIncrement  @Column(DataType.INTEGER)  id!: number;  @AllowNull(false)  @Column  name!: string;  @AllowNull(false)  @Column  email!: string;  @HasMany(() => Task)  task!: Task[];}export default User;
```
Using the [Standalone mode](#standalone-mode) configuration file for the Atlas Sequelize Provider, we can generate a migration file by running this command:
```codeBlockLines_AdAo
atlas migrate diff --env sequelize
```
Running this command will generate files similar to this in the `migrations` directory:
```codeBlockLines_AdAo
migrations|-- 20230918143104.sql`-- atlas.sum0 directories, 2 files
```
Examining the contents of `20230918143104.sql`:
```codeBlockLines_AdAo
-- Create "Users" tableCREATE TABLE `Users` (  `id` int NOT NULL AUTO_INCREMENT,  `name` varchar(255) NOT NULL,  `email` varchar(255) NOT NULL,  `createdAt` datetime NOT NULL,  `updatedAt` datetime NOT NULL,  PRIMARY KEY (`id`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-- Create "Tasks" tableCREATE TABLE `Tasks` (  `id` int NOT NULL AUTO_INCREMENT,  `complete` bool NULL DEFAULT 0,  `createdAt` datetime NOT NULL,  `updatedAt` datetime NOT NULL,  `userID` int NOT NULL,  PRIMARY KEY (`id`),  INDEX `userID` (`userID`),  CONSTRAINT `Tasks_ibfk_1` FOREIGN KEY (`userID`) REFERENCES `Users` (`id`) ON UPDATE CASCADE ON DELETE NO ACTION) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```
Amazing! Atlas automatically generated a migration file that will create the `Users` and `Tasks` tables in our database!

Next, alter the `User` model to add a new `age` field:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
    name: {      type: DataTypes.STRING,      allowNull: false    },+   age: {+     type: DataTypes.INTEGER,+     allowNull: false+   },
```
```codeBlockLines_AdAo
    @AllowNull(false)    @Column    name!: string;+   @AllowNull(false)+   @Column+   age!: number;
```
Re-run this command:
```codeBlockLines_AdAo
atlas migrate diff --env sequelize
```
Observe a new migration file is generated:
```codeBlockLines_AdAo
-- Modify "Users" tableALTER TABLE `Users` ADD COLUMN `age` int NOT NULL;
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


In this guide we demonstrated how projects using Sequelize can use Atlas to automatically plan schema migrations based only on their data model. To learn more about executing these migrations against your production database, read the documentation for the [`migrate apply`](/versioned/apply) command.

Have questions? Feedback? Find our team [on our Discord server](https://discord.gg/zZ6sWVg6NT)

*   [TL;DR](#tldr)
*   [Automatic migration planning for Sequelize](#automatic-migration-planning-for-sequelize)
*   [Prerequisites](#prerequisites)
*   [Using the Atlas Sequelize Provider](#using-the-atlas-sequelize-provider)
    *   [Installation](#installation)
    *   [Setup](#setup)
*   [Usage](#usage)
*   [Conclusion](#conclusion)