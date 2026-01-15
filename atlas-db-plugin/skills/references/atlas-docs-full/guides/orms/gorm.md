Automatic GORM Migrations with Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[GORM](https://gorm.io/index.html), a popular ORM in the Go community, provides basic schema migration capabilities using its `AutoMigrate` feature, suitable mostly for local development and small projects. For more advanced schema control, Atlas automates the process by comparing and managing database states, integrating seamlessly with GORM projects.

[

Quick Start

!

Get started with Atlas and GORM.

](/guides/orms/gorm/getting-started)

### Loading GORM Models Into Atlas[​](#loading-gorm-models-into-atlas "Direct link to Loading GORM Models Into Atlas")


To use Atlas with GORM, there are two modes in which the Atlas GORM Provider can load your schema. Choose the one that suits your project setup.

[

##### Standalone Mode


The common case. Use it if all of your GORM models exist in a single package and either embed gorm.Model or contain gorm struct tags.

](/guides/orms/gorm/standalone)[

##### Go Program Mode


Use in more advanced scenarios where you need more control specifying which structs to consider as models.

](/guides/orms/gorm/program)

### Managing Database Objects[​](#managing-database-objects "Direct link to Managing Database Objects")


Like many ORMs, GORM provides a way to define the most common database objects, such as tables, columns, and indexes using Go structs and tags. Atlas extends this capability by allowing you to define more advanced database objects such as composite types, domain types, and triggers.

[

##### !Composite Types


Incorporate composite types into your GORM schema.

](/guides/orms/gorm/composite-types)[

##### !Domain Types


Define and use custom domain types within your GORM schema.

](/guides/orms/gorm/domain-types)[

##### !Enum Types


Implement enum types to represent fixed sets of values in your GORM schema.

](/guides/orms/gorm/enum-types)[

##### !Extensions


Enhance your GORM schema with PostgreSQL extensions.

](/guides/orms/gorm/extensions)[

##### !Row-Level Security


Apply RLS policies to safeguard data in your GORM schema.

](/guides/orms/gorm/row-level-security)[

##### !Triggers


Automate actions in your GORM schema using triggers.

](/guides/orms/gorm/triggers)[

##### !Views


Create views in your GORM schema using Atlas migrations.

](/guides/orms/gorm/views)

### More Guides[​](#more-guides "Direct link to More Guides")


[

##### !Visualizing GORM Schemas


Generate an ERD of your GORM types.

](/guides/orms/gorm/visualize)[

##### !Generate GORM Models


Generate GORM Models (Go Code) from a Database Schema

](/guides/orms/gorm/generate-models)

*   [Loading GORM Models Into Atlas](#loading-gorm-models-into-atlas)
*   [Managing Database Objects](#managing-database-objects)
*   [More Guides](#more-guides)