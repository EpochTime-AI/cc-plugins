Automatic Sequelize Migrations with Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[Sequelize](https://sequelize.org/) is a popular ORM widely used in the Node.js community. Sequelize supports automatic database schema management using its [sync](https://sequelize.org/docs/v6/core-concepts/model-basics/#model-synchronization/) feature, which is not considered safe for production use.

The Sequelize project recommends using [versioned migrations](https://sequelize.org/docs/v6/other-topics/migrations) for production workloads, using the `sequelize-cli` tool. However, `sequelize-cli` relies on manual planning of migrations which can be error-prone and time-consuming.

Atlas provides automatic migration planning which seamlessly integrates with Sequelize. This allows you to keep your Sequelize schema in sync with your database without the need to manually plan migrations.

[

Quick Start

!

Get started with Atlas and Sequelize.

](/guides/orms/sequelize/getting-started)

### Loading Sequelize Models Into Atlas[​](#loading-sequelize-models-into-atlas "Direct link to Loading Sequelize Models Into Atlas")


To use Atlas with Sequelize, there are two modes in which the Atlas Sequelize Provider can load your schema. Choose the one that suits your project setup.

[

##### Standalone Mode


If all of your Sequelize models exist in a single Node moudle, you can use the provider directly to load your Sequelize schema into Atlas.

](/guides/orms/sequelize/standalone)[

##### Script Mode


Use in more advanced scenarios where you need more control specifying which models should be included.

](/guides/orms/sequelize/script)

### Managing Database Objects[​](#managing-database-objects "Direct link to Managing Database Objects")


Like many ORMs, Sequelize provides a way to define the most common database objects, such as tables, columns, and indexes using Javascript. Atlas extends this capability by allowing you to define more advanced database objects such as composite types, domain types, and triggers.

[

##### !Composite Types


Incorporate composite types into your Sequelize schema.

](/guides/orms/sequelize/composite-types)[

##### !Domain Types


Define and use custom domain types within your Sequelize schema.

](/guides/orms/sequelize/domain-types)[

##### !Row-Level Security


Apply RLS policies to safeguard data in your Sequelize schema.

](/guides/orms/sequelize/row-level-security)[

##### !Triggers


Automate actions in your Sequelize schema using triggers.

](/guides/orms/sequelize/triggers)[

##### !Views


Incorporate Views into your schemas.

](/guides/orms/sequelize/views)

### More Guides[​](#more-guides "Direct link to More Guides")


[

##### !Visualizing Sequelize Schemas


Generate an ERD of your Sequelize types.

](/guides/orms/sequelize/visualize)[

##### !Declarative Migrations


Build a declarative migrations pipeline with Sequelize and Atlas on GitHub Actions.

](/guides/orms/sequelize/declarative-migrations-with-actions)[

##### !Generate Sequelize Models


Generate Sequelize Models (JS/TS Code) from a Database Schema

](/guides/orms/sequelize/generate-models)

*   [Loading Sequelize Models Into Atlas](#loading-sequelize-models-into-atlas)
*   [Managing Database Objects](#managing-database-objects)
*   [More Guides](#more-guides)