Automatic SQLAlchemy Migrations with Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[SQLAlchemy](https://www.sqlalchemy.org) is one of the most popular ORMs in the Python ecosystem. While many projects use [Alembic](https://alembic.sqlalchemy.org/en/latest/), developers often seek a more powerful and automated tool for **SQLAlchemy migrations**. Atlas provides a seamless integration with SQLAlchemy, offering a robust alternative to Alembic. It allows you to keep defining your schema as SQLAlchemy models while enjoying Atlas's advanced schema management features. Some examples are:

1.  **Advanced database features**: Atlas can recognize and detect changes in many database features that are not natively supported by SQLAlchemy and Alembic, such as:

    *   Views
    *   Functions & Procedures
    *   Triggers
    *   Sequences
    *   Row-Level Security
    *   And [much more!](https://atlasgo.io/features#database-features)

    With [Composite Schemas](https://atlasgo.io/atlas-schema/projects#data-source-composite_schema), these object can be now managed as part of your code.

2.  **CI/CD support**: Catch issues before they hit production with robust GitHub Actions, GitLab, and CircleCI Orbs integrations. Detect risky migrations, test data migrations, database functions, and more. In addition, Atlas can be integrated into your pipelines to provide native integrations with your deployment machinery (e.g. Kubernetes Operator, Terraform, etc.).

3.  **Declarative workflow**: Atlas can be used to manage your schema as code - the schema is continuously synced to [Atlas Cloud](https://atlasgo.cloud/) and Atlas generates a migration plan for every new change, removing the need for a migration directory.

[

Quick Start

!

Get started with Atlas and SQLAlchemy.

](/guides/orms/sqlalchemy/getting-started)

### Loading SQLAlchemy Models Into Atlas[​](#loading-sqlalchemy-models-into-atlas "Direct link to Loading SQLAlchemy Models Into Atlas")


To use Atlas with SQLAlchemy, there are two modes in which the Atlas SQLAlchemy Provider can load your schema. Choose the one that suits your project setup.

[

##### Standalone Mode


The common case. If all of your SQLAlchemy models exist in a single module, you can use the provider directly to load your SQLAlchemy schema into Atlas.

](/guides/orms/sqlalchemy/standalone)[

##### Python Script


Use Atlas as a Python script to load and manage your SQLAlchemy schema in Atlas.

](/guides/orms/sqlalchemy/script)

### Managing Database Objects[​](#managing-database-objects "Direct link to Managing Database Objects")


Like many ORMs, SQLAlchemy provides a way to define the most common database objects, such as tables, columns, and indexes using Python classes and decorators. Atlas extends this capability by allowing you to define more advanced database objects such as composite types, domain types, and triggers.

[

##### !Row-Level Security


Implement Row-Level Security (RLS) policies in your SQLAlchemy models to safeguard data and control access.

](/guides/orms/sqlalchemy/row-level-security)[

##### !Triggers


Learn how to automate database actions and logic with triggers in your SQLAlchemy models.

](/guides/orms/sqlalchemy/triggers)[

##### !Extenstions


Use composite schema to integrate relevant extensions to your SQLAlchemy models.

](/guides/orms/sqlalchemy/extensions)

*   [Loading SQLAlchemy Models Into Atlas](#loading-sqlalchemy-models-into-atlas)
*   [Managing Database Objects](#managing-database-objects)