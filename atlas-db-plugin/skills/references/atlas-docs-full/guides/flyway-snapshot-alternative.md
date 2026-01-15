Flyway Snapshot Alternative in Atlas | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

The `flyway snapshot` command captures the structure of a database and stores it as a JSON file that represents its state at a specific point in time. This is often used for drift detection, change reviews, or as a baseline for further comparisons.

In Atlas, you can achieve the same workflow with a few key advantages using the `atlas schema inspect` command.

## Inspect and Snapshot Your Schema[​](#inspect-and-snapshot-your-schema "Direct link to Inspect and Snapshot Your Schema")


Unlike Flyway's `snapshot` feature, this capability is free in Atlas. With the `schema inspect` command, you can capture your database schema and export it in any format you need. The examples below show how to export a database to a SQL (DDL format), HCL, JSON, or even a Mermaid diagram. Next sections show more advanced exports, such as generating ORM models or structured code folders from your database schema.

*   SQL Format
*   HCL Format
*   JSON Format
*   Mermaid (ERD)
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" --format '{{ sql . }}' > schema.sql
```
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" > schema.hcl
```
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" --format '{{ json . }}' > schema.json
```
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" --format '{{ mermaid . }}' > schema.mmd
```
This command connects to your database, inspects its structure, and produces a complete, version-controlled representation of your schema. The output can be stored in your Git repository, compared against other states, or used to detect drift between environments.

## What the Snapshot Includes[​](#what-the-snapshot-includes "Direct link to What the Snapshot Includes")


Atlas currently supports introspection for the following objects in the free version:

*   Schemas, tables, and columns
*   Indexes and unique constraints
*   Foreign keys and relationships
*   Check constraints
*   Enums (PostgreSQL, MySQL, etc.)
*   Comments

These are the core elements required for most schema-as-code and drift-detection workflows.

Using Atlas Pro, you can extend the schema snapshot to provide the full representation of your database's state, with complete dependency-graph and relationships between the different objects, including:

*   Views and materialized views
*   Functions and procedures
*   Triggers
*   Sequences
*   Custom types, such as domains, composite-types, etc.
*   Extensions, event-triggers, RLS policies, and more.

## Customizing Snapshot Output[​](#customizing-snapshot-output "Direct link to Customizing Snapshot Output")


Atlas supports exporting the schema to custom formats using Go templates. This allows you to generate tailored outputs for your specific needs. For example, you can create documentation, custom reports, or export ORM definitions. For example:

*   [Generate TypeORM Entities](/guides/orms/typeorm/generate-entities)
*   [Generate GORM Models](/guides/orms/gorm/generate-models)
*   [Generate Sequelize Models](/guides/orms/sequelize/generate-models)

📺 For a step-by-step example walk-through, watch our tutorial: [Inspect Your Database Schema with Atlas](https://www.youtube.com/watch?v=WILZMLr0REw)

### Visualize with ERD[​](#visualize-with-erd "Direct link to Visualize with ERD")


You can also visualize your schema as an interactive Entity Relationship Diagram (ERD). Use the `--web` flag to open an interactive ERD in Atlas Cloud:
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" --web
```
Try it out in the component below:

Loading ERD...

## Export Database Schema to Code[​](#export-database-schema-to-code "Direct link to Export Database Schema to Code")


Users who want to export the database to structured folders can follow the [Export Schema to Code](/inspect/database-to-code) documentation. Atlas can organize your schema into a modular directory structure like this:
```codeBlockLines_AdAo
├── extensions│   ├── hstore.sql│   └── citext.sql├── schemas│   └── public│       ├── public.sql│       ├── tables│       │   ├── profiles.sql│       │   └── users.sql│       ├── functions│       └── types└── main.sql
```
To export your schema into this structure, use the following command:
```codeBlockLines_AdAo
atlas schema inspect -u "<DATABASE_URL>" \  --format '{{ sql . | split | write }}'
```
This will create a structured directory with your schema organized by object type (tables, views, functions, etc.), with a `main.sql` file as an entry point.

## Comparing Snapshots (Drift Detection)[​](#comparing-snapshots-drift-detection "Direct link to Comparing Snapshots (Drift Detection)")


Once you have a snapshot of your schema, you can compare it to another state - a live database, a migration directory, or another snapshot file.
```codeBlockLines_AdAo
atlas schema diff \  --from "file://schema.hcl" \  --to "postgres://localhost:5432/prod?sslmode=disable"
```
This command produces a deterministic diff between two schema states, allowing you to identify and review changes before applying them.

Learn more about [drift detection](/monitoring/drift-detection) and [comparing schemas](/declarative/diff).

## Use Cases Covered (Compared to Flyway)[​](#use-cases-covered-compared-to-flyway "Direct link to Use Cases Covered (Compared to Flyway)")


Use Case

Flyway Command

Atlas Equivalent

Notes

Snapshot live database

`flyway snapshot -source=<db>` (Enterprise)

`atlas schema inspect -u <url>`

Atlas allows exporting to SQL, HCL, JSON, custom templates, etc.

Drift detection between snapshots

`flyway checkDrift`

`atlas schema diff`

Atlas supports comparing any sources

Advanced filtering

`-source` / `-schemaModelSchemas`

`--schema`, `--include`, `--exclude`

Flyway is limited to filter by schemas; Atlas allows filtering by schemas, type, name, glob, etc.

Full object graph (functions, triggers, etc.)

✅ (Enterprise)

✅ (Pro / Enterprise )

Atlas free limited to tables, indexes, FKs

Visualize schema as ERD

❌

✅ (Pro for private, free for public)

Atlas also supports generating schema docs

Export schema to structured code folders

❌

✅

Atlas supports exporting to organized folder structures

Generate ORM models from schema

❌

✅

Atlas supports multiple ORMs (TypeORM, GORM, Sequelize, etc.)

Custom export formats using templates

❌

✅

Atlas supports Go templates for custom outputs

## Example: Compare without Snapshot[​](#example-compare-without-snapshot "Direct link to Example: Compare without Snapshot")


The `atlas schema diff` command can also be used to compare two live databases without creating intermediate snapshot files.
```codeBlockLines_AdAo
atlas schema diff \  --from "postgres://localhost:5432/staging?sslmode=disable" \  --to "postgres://localhost:5432/production?sslmode=disable"
```
See [comparing schemas](/declarative/diff) for more details. Atlas supports comparing any source to any source.

## Key Advantages over Flyway Snapshot[​](#key-advantages-over-flyway-snapshot "Direct link to Key Advantages over Flyway Snapshot")


*   **Free**: basic schema inspection is available in the free version.
*   **Advanced filtering**: use glob patterns to include/exclude resources by name or type.
*   **Multiple output formats**: choose between HCL, SQL, JSON, Mermaid, custom templating, etc.
*   **Declarative model**: output can be used directly in versioned workflows (`atlas schema apply`, `atlas migrate diff`).
*   **Pro extensions**: introspection for functions, triggers, procedures, and extensions.
*   **Drift detection built-in**: compare any two states deterministically.
*   **CI/CD ready**: easily automate snapshot generation and comparison in pipelines.
*   **Export to code**: generate organized folder structures with your schema definition.
*   **ERD visualization**: create interactive diagrams from your database schema.
*   **ORM model generation**: generate models for Go, Python, Java, and more from your schema.

## Read More[​](#read-more "Direct link to Read More")


*   [Inspect your first schema](/inspect)
*   [Learn about Drift Detection](/monitoring/drift-detection)
*   [Compare Atlas and Flyway](/guides/atlas-vs-flyway)
*   [Migrate from Flyway to Atlas](/guides/migrate-flyway-to-atlas)
*   [Export Database Schema to Code](/inspect/database-to-code)
*   [Programmatic Inspection Output with Templates](/guides/go-templates)

*   [Inspect and Snapshot Your Schema](#inspect-and-snapshot-your-schema)
*   [What the Snapshot Includes](#what-the-snapshot-includes)
*   [Customizing Snapshot Output](#customizing-snapshot-output)
    *   [Visualize with ERD](#visualize-with-erd)
*   [Export Database Schema to Code](#export-database-schema-to-code)
*   [Comparing Snapshots (Drift Detection)](#comparing-snapshots-drift-detection)
*   [Use Cases Covered (Compared to Flyway)](#use-cases-covered-compared-to-flyway)
*   [Example: Compare without Snapshot](#example-compare-without-snapshot)
*   [Key Advantages over Flyway Snapshot](#key-advantages-over-flyway-snapshot)
*   [Read More](#read-more)