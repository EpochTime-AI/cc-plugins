Generate TypeORM Entities (TypeScript Code) from a Database Schema | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Although Atlas supports using TypeORM schemas as the desired state and planning migrations accordingly, it can sometimes be useful to do the reverse - generate TypeORM entities from an existing schema, such as a live database, HCL or SQL files, or even another ORM.

## Overview[​](#overview "Direct link to Overview")


One way to generate custom code from an Atlas schema is to get a JSON representation of it using the `atlas schema inspect` command with the `--format '{{ json . }}'` flag, and then use a custom script to generate the desired code.

However, in this guide, we'll show how you can use Atlas templates to generate custom code from an Atlas schema. Atlas, like many other CLI tools such as `kubectl` and `docker`, supports templating the output of commands using Go templates. This lets you write "custom code" that is evaluated at runtime and generates the desired output.

If you're not familiar with the Go templates language, you can read more about it in the [Go documentation](https://pkg.go.dev/text/template).

The `schema inspect` command results in a [`SchemaInspect` object](https://pkg.go.dev/ariga.io/atlas/cmd/atlas@master/internal/cmdlog#SchemaInspect) with a [`Realm` field](https://pkg.go.dev/ariga.io/atlas@v0.35.0/sql/schema#Realm) containing the schema information, such as schemas, tables, columns, and more. This object is used as the context for the template

## Defining a Template[​](#defining-a-template "Direct link to Defining a Template")


Let's define a template that properly detects auto-generated columns and generates the appropriate TypeORM decorators:

Note that besides the main template, this example defines several small helper templates that act like functions (`"title"`, `"singular"`, etc.). These can be reused across the template using `exec` and `include` functions.

entities.tmpl
```codeBlockLines_AdAo
{{- /* Helper function-like template for generating TitleCase from a database object name. */}}{{- define "title" }}  {{- $v := "" }}  {{- range $w := splitBy $ "_" }}    {{- if le (len $w) 1 }}      {{- $v = print $v (upper $w) }}    {{- else }}      {{- $v = print $v (upper (slice $w 0 1)) (lower (slice $w 1)) }}    {{- end }}  {{- end }}  {{- print $v }}{{- end }}{{- /* Helper function-like template for generating singular form of a plural noun using basic English rules. */}}{{- define "singular" }}  {{- $s := . }}  {{- if and (hasSuffix $s "ies") (gt (len $s) 3) }}    {{- printf "%sy" (slice $s 0 (sub (len $s) 3)) }}  {{- else if or (hasSuffix $s "ses") (hasSuffix $s "xes") }}    {{- trimSuffix $s "es" }}  {{- else if and (hasSuffix $s "s") (gt (len $s) 1) }}    {{- trimSuffix $s "s" }}  {{- else }}    {{- $s }}  {{- end }}{{- end }}{{- /* Helper function-like template for converting a column type to a TypeORM column type. */}}{{- define "typeorm-type" }}  {{- $m := dict    "character" "varchar"    "text" "text"    "boolean" "boolean"    "smallint" "smallint"    "integer" "int"    "bigint" "bigint"    "real" "real"    "numeric" "decimal"    "decimal" "decimal"    "date" "date"    "time" "time"    "timestamp" "timestamp"    "uuid" "uuid"    "json" "json"    "jsonb" "jsonb"  }}  {{- with $t := columnType . }}    {{- if hasKey $m $t }}      {{- get $m $t }}    {{- else }}      varchar    {{- end }}  {{- end }}{{- end }}{{- /* Helper function-like template for mapping database column types to TypeScript types. */}}{{- define "typescript-type" }}  {{- $m := dict    "character" "string"    "text" "string"    "boolean" "boolean"    "smallint" "number"    "integer" "number"    "bigint" "number"    "real" "number"    "numeric" "number"    "decimal" "number"    "date" "Date"    "time" "string"    "timestamp" "Date"    "uuid" "string"    "json" "any"    "jsonb" "any"  }}  {{- with $t := columnType . }}    {{- if hasKey $m $t }}      {{- get $m $t }}    {{- else }}      string    {{- end }}  {{- end }}{{- end }}{{- /* Helper function to check if column is auto-incremented */}}{{- define "autoInc" }}  {{- $isAuto := false }}    {{- range $attr := .Attrs }}      {{- $t := printf "%T" $attr }}      {{- if hasSuffix $t "AutoIncrement" }}        {{- $isAuto = true }}      {{- end }}    {{- end }}  {{- $isAuto }}{{- end }}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" -}}import { Entity, PrimaryGeneratedColumn, PrimaryColumn, Column } from 'typeorm';{{- range $t := (index .Realm.Schemas 0).Tables }}@Entity('{{ $t.Name }}')export class {{ $name := (exec "singular" (exec "title" $t.Name)) }}{{ $name }} {{{- range $c := $t.Columns }}  {{- $isPrimary := false }}  {{- if $t.PrimaryKey }}    {{- range $part := $t.PrimaryKey.Parts }}      {{- if eq $part.C.Name $c.Name }}        {{- $isPrimary = true }}      {{- end }}    {{- end }}  {{- end }}  {{- $autoInc := exec "autoInc" . }}    {{- if and $isPrimary $autoInc }}  @PrimaryGeneratedColumn()  {{- else if $isPrimary }}  @PrimaryColumn()  {{- else }}  @Column({    type: '{{ exec "typeorm-type" . }}',    nullable: {{ .Type.Null }},  })  {{- end }}  {{ $c.Name }}: {{ exec "typescript-type" . }}{{ if .Type.Null }} | null{{ end }};{{- end }}}{{- end }}
```
After defining the template, we can configure the `atlas.hcl` file to run the template whenever we execute schema inspection with the `generate` environment.

*   atlas.hcl
*   schema.sql

atlas.hcl
```codeBlockLines_AdAo
env "generate" {  src = "file://schema.sql"  dev = "docker://mysql/8/dev"  format {    schema {      inspect = file("entities.tmpl")    }  }}
```
```codeBlockLines_AdAo
CREATE TABLE users (  id INT AUTO_INCREMENT PRIMARY KEY,  name VARCHAR(100) NULL);CREATE TABLE blog_posts (  id INT AUTO_INCREMENT PRIMARY KEY,  title VARCHAR(100) NULL,  body TEXT NULL,  author_id INT NULL,  CONSTRAINT author_fk FOREIGN KEY (author_id) REFERENCES users(id));
```

## Executing the Template[​](#executing-the-template "Direct link to Executing the Template")


After we defined the template and configured it in our `atlas.hcl`, we can execute the template using the `atlas schema inspect` command using the `--env` flag to select the environment and the `--url` flag to specify the schema source. In this case, `env://src` refers to the `src` attribute defined in the selected environment.
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src
```
The output of this command will look something like this:
```codeBlockLines_AdAo
import { Entity, PrimaryGeneratedColumn, PrimaryColumn, Column } from 'typeorm';@Entity('blog_posts')export class BlogPost {  @PrimaryGeneratedColumn()  id: number;  @Column({    type: 'varchar',    nullable: true,  })  ...  author_id: number | null;}@Entity('users')export class User {  @PrimaryGeneratedColumn()  id: number;  @Column({    type: 'varchar',    nullable: true,  })  name: string | null;}
```
To write the output to a file, redirect stdout to `entities.ts`:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src > entities.ts
```

## Multi-File Generation with `write` and `txtar`[​](#multi-file-generation-with-write-and-txtar "Direct link to multi-file-generation-with-write-and-txtar")


The `write` function lets us write files directly from within the template. This is useful when we want to generate multiple files (e.g., one per entity).

To do this, we'll adapt the template to produce `txtar` output and use the `write` function to write the generated files to disk. The updated template will look like this:

entities-multi.tmpl
```codeBlockLines_AdAo
{{- /* Include all the helper functions from above */}}{{- /* ... (helper functions omitted for brevity) ... */}}{{- /* The template for generating a TypeORM entity file for the given table */}}{{- define "entity" }}{{- $name := exec "title" $.Name | exec "singular" }}-- {{ lower $name }}.entity.ts --import { Entity, PrimaryGeneratedColumn, PrimaryColumn, Column } from 'typeorm';@Entity('{{ $.Name }}')export class {{ $name }} {{{- range $c := $.Columns }}  {{- $isPrimary := false }}  {{- if $.PrimaryKey }}    {{- range $part := $.PrimaryKey.Parts }}      {{- if eq $part.C.Name $c.Name }}        {{- $isPrimary = true }}      {{- end }}    {{- end }}  {{- end }}  {{- $autoInc := exec "autoInc" . }}    {{- if and $isPrimary $autoInc }}  @PrimaryGeneratedColumn()  {{- else if $isPrimary }}  @PrimaryColumn()  {{- else }}  @Column({    type: '{{ exec "typeorm-type" . }}',    nullable: {{ .Type.Null }},  })  {{- end }}  {{ $c.Name }}: {{ exec "typescript-type" . }}{{ if .Type.Null }} | null{{ end }};{{- end }}}{{- end }}{{- /* Template for generating the index file that exports all entities */}}{{- define "index" }}-- index.ts --{{- range $t := (index .Realm.Schemas 0).Tables }}{{- $name := exec "title" $t.Name | exec "singular" }}export { {{ $name }} } from './{{ lower $name }}.entity';{{- end }}{{- end }}{{- /* Main template: build txtar archive with all entity files and index.ts */}}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" }}{{- $txtar := list }}{{- range $t := (index .Realm.Schemas 0).Tables }}  {{- $txtar = append $txtar (include "entity" $t) }}{{- end }}{{- $txtar = append $txtar (include "index" .) }}{{- $output := "" }}{{- range $i, $content := $txtar }}  {{- $output = printf "%s%s" $output $content }}{{- end }}{{- $output | txtar | write "entities" }}
```
Then, run the same command as before to execute the template:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src
```
This will generate a `entities` directory containing one file per table, plus an index file that loads all entities:
```codeBlockLines_AdAo
entities/├── user.entity.ts├── blogpost.entity.ts└── index.ts
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


This guide showed how to generate TypeORM entity definitions from a database schema using the Atlas `--format` flag and Go's `text/template` engine. For more, see the [Go templates documentation](https://pkg.go.dev/text/template) and the [Atlas template functions reference](/inspect#template-functions).

*   [Overview](#overview)
*   [Defining a Template](#defining-a-template)
*   [Executing the Template](#executing-the-template)
*   [Multi-File Generation with `write` and `txtar`](#multi-file-generation-with-write-and-txtar)
*   [Conclusion](#conclusion)