Managing Sequelize Schemas in Script Mode | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

This document describes how to set up Atlas to load your Sequelize schema in **Script Mode**. This mode is suitable for cases requiring finer control when loading Sequelize models.

If your Sequelize models are all exposed via a single Node module, **Standalone Mode** might be more suitable. Refer to [Standalone Mode](/guides/orms/sequelize/standalone) for more details.

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

### Extract Sequelize model via SQL[​](#extract-sequelize-model-via-sql "Direct link to Extract Sequelize model via SQL")


*   JavaScript
*   TypeScript

Create a new file named `load.js` with the following contents:
```codeBlockLines_AdAo
#!/usr/bin/env node// require sequelize models you want to loadconst user = require("./models/user");const task = require("./models/task");const loadModels = require("@ariga/atlas-provider-sequelize");console.log(loadModels("mysql", user, task));
```
Create a new file named `load.ts` with the following contents:
```codeBlockLines_AdAo
#!/usr/bin/env ts-node-script// import sequelize models you want to loadimport User from "./models/user";import Task from "./models/task";import { loadModels } from "@ariga/ts-atlas-provider-sequelize/src/sequelize_schema";console.log(loadModels("mysql", [User, Task]));
```

### Setup Atlas[​](#setup-atlas "Direct link to Setup Atlas")


In your project directory, create a file named `atlas.hcl` with the following contents:

*   JavaScript
*   TypeScript
```codeBlockLines_AdAo
data "external_schema" "sequelize" {  program = [    "node",    "load.js"  ]}env "sequelize" {  src = data.external_schema.sequelize.url  dev = "docker://mysql/8/dev"  migration {    dir = "file://migrations"  }  format {    migrate {      diff = "{{ sql . \"  \" }}"    }  }}
```
```codeBlockLines_AdAo
data "external_schema" "sequelize" {  program = [    "npx",    "ts-node",    "load.ts"  ]}env "sequelize" {  src = data.external_schema.sequelize.url  dev = "docker://mysql/8/dev"  migration {    dir = "file://migrations"  }  format {    migrate {      diff = "{{ sql . \"  \" }}"    }  }}
```
note

Be sure to update the `dialect` in the config file, as well as the selected `dev` configuration, depending on which database you are using. For more information, see [Dev Database Concept](/concepts/dev-database).

Next, verify Atlas is able to read our desired schema, by running the [`schema inspect`](/inspect) command.
```codeBlockLines_AdAo
atlas schema inspect --env sequelize --url "env://src"
```
info

For in-depth details on the `atlas schema inspect` command, covering aspects like inspecting specific schemas, handling multiple schemas concurrently, excluding tables, and more, refer to our documentation [here](/inspect).

### Example[​](#example "Direct link to Example")


Consider the following Sequelize `models` directory, which includes two models: `user` and `task`:

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
import {  Table,  Column,  Model,  PrimaryKey,  ForeignKey,  BelongsTo,  AutoIncrement,  DataType,  AllowNull,  Default,} from "sequelize-typescript";import User from "./user";@Table({ tableName: "Tasks" })class Task extends Model {  @PrimaryKey  @AutoIncrement  @Column(DataType.INTEGER)  id!: number;  @Default(false)  @Column(DataType.BOOLEAN)  complete!: boolean;  @AllowNull(false)  @ForeignKey(() => User)  @Column(DataType.INTEGER)  userID!: number;  @BelongsTo(() => User)  user!: User;}export default Task;
```
```codeBlockLines_AdAo
import {  Table,  Column,  Model,  PrimaryKey,  AutoIncrement,  DataType,  AllowNull,  HasMany,} from "sequelize-typescript";import Task from "./task";@Table({ tableName: "Users" })class User extends Model {  @PrimaryKey  @AutoIncrement  @Column(DataType.INTEGER)  id!: number;  @AllowNull(false)  @Column  name!: string;  @AllowNull(false)  @Column  email!: string;  @HasMany(() => Task)  task!: Task[];}export default User;
```
We should get the following output after running the `inspect` command above:
```codeBlockLines_AdAo
table "Tasks" {  schema = schema.dev  column "id" {    null           = false    type           = int    auto_increment = true  }  column "complete" {    null    = true    type    = bool    default = 0  }  column "createdAt" {    null = false    type = datetime  }  column "updatedAt" {    null = false    type = datetime  }  column "userID" {    null = false    type = int  }  primary_key {    columns = [column.id]  }  foreign_key "Tasks_ibfk_1" {    columns     = [column.userID]    ref_columns = [table.Users.column.id]    on_update   = CASCADE    on_delete   = NO_ACTION  }  index "userID" {    columns = [column.userID]  }}table "Users" {  schema = schema.dev  column "id" {    null           = false    type           = int    auto_increment = true  }  column "name" {    null = false    type = varchar(255)  }  column "email" {    null = false    type = varchar(255)  }  column "createdAt" {    null = false    type = datetime  }  column "updatedAt" {    null = false    type = datetime  }  primary_key {    columns = [column.id]  }}schema "dev" {  charset = "utf8mb4"  collate = "utf8mb4_0900_ai_ci"}
```

## Generating SQL migration files from a Sequelize Schema[​](#generating-sql-migration-files-from-a-sequelize-schema "Direct link to Generating SQL migration files from a Sequelize Schema")


We can now generate a migration file by running this command:
```codeBlockLines_AdAo
atlas migrate diff --env sequelize
```
Running this command will generate files similar to this in the `migrations` directory:
```codeBlockLines_AdAo
migrations|-- 20241205100006.sql`-- atlas.sum0 directories, 2 files
```
Examining the contents of `20241205100006.sql`:
```codeBlockLines_AdAo
-- Create "Users" tableCREATE TABLE `Users` (  `id` int NOT NULL AUTO_INCREMENT,  `name` varchar(255) NOT NULL,  `email` varchar(255) NOT NULL,  `createdAt` datetime NOT NULL,  `updatedAt` datetime NOT NULL,  PRIMARY KEY (`id`)) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;-- Create "Tasks" tableCREATE TABLE `Tasks` (  `id` int NOT NULL AUTO_INCREMENT,  `complete` bool NULL DEFAULT 0,  `createdAt` datetime NOT NULL,  `updatedAt` datetime NOT NULL,  `userID` int NOT NULL,  PRIMARY KEY (`id`),  INDEX `userID` (`userID`),  CONSTRAINT `Tasks_ibfk_1` FOREIGN KEY (`userID`) REFERENCES `Users` (`id`) ON UPDATE CASCADE ON DELETE NO ACTION) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

### Applying the migration directory to a database[​](#applying-the-migration-directory-to-a-database "Direct link to Applying the migration directory to a database")


Let's apply our generated migration files to a development database.

First, create a MySQL development database with Docker:
```codeBlockLines_AdAo
docker run --name mysql -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=example -p 3306:3306 -d mysql:latest
```
Then, run the following command to apply the schema to the database:
```codeBlockLines_AdAo
atlas migrate apply --env sequelize --url "mysql://root:pass@localhost:3306/example"
```
Atlas should print an output similar to this:
```codeBlockLines_AdAo
Migrating to version 20241205100006 (1 migrations in total):  -- migrating version 20241205100006    -> CREATE TABLE `Users` (         `id` int NOT NULL AUTO_INCREMENT,         `name` varchar(255) NOT NULL,         `email` varchar(255) NOT NULL,         `createdAt` datetime NOT NULL,         `updatedAt` datetime NOT NULL,         PRIMARY KEY (`id`)       ) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;    -> CREATE TABLE `Tasks` (         `id` int NOT NULL AUTO_INCREMENT,         `complete` bool NULL DEFAULT 0,         `createdAt` datetime NOT NULL,         `updatedAt` datetime NOT NULL,         `userID` int NOT NULL,         PRIMARY KEY (`id`),         INDEX `userID` (`userID`),         CONSTRAINT `Tasks_ibfk_1` FOREIGN KEY (`userID`) REFERENCES `Users` (`id`) ON UPDATE CASCADE ON DELETE NO ACTION       ) CHARSET utf8mb4 COLLATE utf8mb4_0900_ai_ci;  -- ok (65.529168ms)  -------------------------  -- 102.386668ms  -- 1 migration  -- 2 sql statements
```
Apply the Schema Directly on the Database

Sometimes, there is a need to apply the schema directly to the database without generating a migration file. For example, when experimenting with schema changes, spinning up a database for testing, etc. In such cases, you can use the command below to apply the schema directly to the database:
```codeBlockLines_AdAo
atlas schema apply \  --env local \  --url "mysql://root:pass@localhost:3306/example"
```

## Next Steps[​](#next-steps "Direct link to Next Steps")


Now that your project is set up, start by choosing between the two workflows offered by Atlas for generating and planning migrations. Select the one you prefer that works best for you:

*   **Declarative Migrations**: Set up a Terraform-like workflow where each migration is calculated as the diff between your desired state and the current state of the database. See [Declarative Schema Migrations](/declarative/apply)

*   **Versioned Migrations**: Set up a migration directory for your project, creating a version-controlled source of truth of your database schema. See [Quick Introduction](/versioned/intro)

*   [Installation](#installation)
*   [Extract Sequelize model via SQL](#extract-sequelize-model-via-sql)
*   [Setup Atlas](#setup-atlas)
*   [Example](#example)
*   [Generating SQL migration files from a Sequelize Schema](#generating-sql-migration-files-from-a-sequelize-schema)
    *   [Applying the migration directory to a database](#applying-the-migration-directory-to-a-database)
*   [Next Steps](#next-steps)