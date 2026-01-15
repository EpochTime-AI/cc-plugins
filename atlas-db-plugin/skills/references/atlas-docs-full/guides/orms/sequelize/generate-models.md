Generate Sequelize Models (JavaScript Code) from a Database Schema | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Although Atlas supports using Sequelize schemas as the desired state and planning migrations accordingly, it can sometimes be useful to do the reverse - generate Sequelize models from an existing schema, such as a live database, HCL or SQL files, or even another ORM.

## Overview[​](#overview "Direct link to Overview")


One way to generate custom code from an Atlas schema is to get a JSON representation of it using the `atlas schema inspect` command with the `--format '{{ json . }}'` flag, and then use a custom script to generate the desired code.

However, in this guide, we'll show how you can use Atlas templates to generate custom code from an Atlas schema. Atlas, like many other CLI tools such as `kubectl` and `docker`, supports templating the output of commands using Go templates. This lets you write "custom code" that is evaluated at runtime and generates the desired output.

If you're not familiar with the Go templates language, you can read more about it in the [Go documentation](https://pkg.go.dev/text/template).

The template execution context is the result of the schema inspection, which is an object with two fields: `URL` and `Realm`. The `URL` field holds the inspected URL, and the `Realm` field contains the schema information, such as schemas, tables, columns, and more. You can see this object [here](https://pkg.go.dev/ariga.io/atlas/cmd/atlas@master/internal/cmdlog#SchemaInspect), and the definition of the `Realm` object at this [link](https://pkg.go.dev/ariga.io/atlas@v0.35.0/sql/schema#Realm).

## Defining a Template[​](#defining-a-template "Direct link to Defining a Template")


Let's define a simple template that generates Sequelize model definitions from the `schema.Realm` object. For this example, we assume the database contains a single schema, and all models will be generated into a single file named `models.js`.

Note that besides the main template, this example defines several small helper templates that act like functions (`"title"`, `"singular"`, etc.). These can be reused across the template using `exec` and `include` functions.

models.tmpl
```codeBlockLines_AdAo
{{- /* Helper function-like template for generating TitleCase from a database object name. */}}{{- define "title" }}  {{- $v := "" }}  {{- range $w := splitBy $ "_" }}    {{- if le (len $w) 1 }}      {{- $v = print $v (upper $w) }}    {{- else }}      {{- $v = print $v (upper (slice $w 0 1)) (lower (slice $w 1)) }}    {{- end }}  {{- end }}  {{- print $v }}{{- end }}{{- /* Helper function-like template for generating singular form of a plural noun using basic English rules. */}}{{- define "singular" }}  {{- $s := . }}  {{- if and (hasSuffix $s "ies") (gt (len $s) 3) }}    {{- printf "%sy" (slice $s 0 (sub (len $s) 3)) }}  {{- else if or (hasSuffix $s "ses") (hasSuffix $s "xes") }}    {{- trimSuffix $s "es" }}  {{- else if and (hasSuffix $s "s") (gt (len $s) 1) }}    {{- trimSuffix $s "s" }}  {{- else }}    {{- $s }}  {{- end }}{{- end }}{{- /* Helper function-like template for converting a column type to a Sequelize data type. */}}{{- define "sequelize-type" }}  {{- $m := dict    "character" "DataTypes.STRING"    "character varying" "DataTypes.STRING"    "text" "DataTypes.TEXT"    "boolean" "DataTypes.BOOLEAN"    "smallint" "DataTypes.INTEGER"    "integer" "DataTypes.INTEGER"    "bigint" "DataTypes.BIGINT"    "real" "DataTypes.FLOAT"    "double precision" "DataTypes.DOUBLE"    "numeric" "DataTypes.DECIMAL"    "decimal" "DataTypes.DECIMAL"    "date" "DataTypes.DATEONLY"    "time" "DataTypes.TIME"    "timestamp" "DataTypes.DATE"    "timestamp with time zone" "DataTypes.DATE"    "timestamp without time zone" "DataTypes.DATE"    "uuid" "DataTypes.UUID"    "json" "DataTypes.JSON"    "jsonb" "DataTypes.JSONB"  }}  {{- with $t := columnType . }}    {{- if hasKey $m $t }}      {{- get $m $t }}    {{- else }}      DataTypes.STRING    {{- end }}  {{- end }}{{- end }}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" -}}module.exports = (sequelize, DataTypes) => {  const models = {};{{- range $t := (index .Realm.Schemas 0).Tables }}  {    const {{ $name := (exec "singular" (exec "title" $t.Name)) }}{{ $name }} = sequelize.define('{{ $name }}', {{{- range $i, $c := $t.Columns }}      {{ $c.Name }}: {        type: {{ exec "sequelize-type" . }},        allowNull: {{ .Type.Null }},        {{- range $idx := $c.Indexes }}            {{- if and (eq $idx $t.PrimaryKey) (eq (len $idx.Parts) 1) }}                {{- print "\n        primaryKey: true," }}            {{- end }}        {{- end }}      },{{- end }}    }, {      tableName: '{{ $t.Name }}',      timestamps: false    });    {{ $name }}.associate = (models) => {      // Define associations here    };    models.{{ $name }} = {{ $name }};  }{{- end }}  return models;};
```
After defining the template, we can configure the `atlas.hcl` file to run the template whenever we execute schema inspection with the `generate` environment.

atlas.hcl
```codeBlockLines_AdAo
env "generate" {  src = "file://schema.sql"  dev = "docker://postgres/16/dev?search_path=public"  format {    schema {      inspect = file("models.tmpl")    }  }}
```

## Executing the Template[​](#executing-the-template "Direct link to Executing the Template")


After we defined the template and configured it in our `atlas.hcl`, we can execute the template using the `atlas schema inspect --env generate --url env://src` command. The `--env` flag selects the environment, and the `--url` flag specifies the schema source. In this case, `env://src` references the `src` attribute defined in the selected environment.
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src
```
The output of this command will look like something like this:
```codeBlockLines_AdAo
module.exports = (sequelize, DataTypes) => {  const models = {};  {    const Friendship = sequelize.define('Friendship', {      id: {        type: DataTypes.BIGINT,        allowNull: false,        primaryKey: true,      },      // ...    }, {      tableName: 'friendships',      timestamps: false    });    Friendship.associate = (models) => {      // Define associations here    };    models.Friendship = Friendship;  }  {    const User = sequelize.define('User', {      // ...    }, {      tableName: 'users',      timestamps: false    });    User.associate = (models) => {      // Define associations here    };    models.User = User;  }  return models;};
```
To write the output to a file, redirect stdout to `models.js`:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src > models.js
```

## `write` and `txtar` Functions[​](#write-and-txtar-functions "Direct link to write-and-txtar-functions")


The `write` function lets us write files directly from within the template. This is useful when we want to generate multiple files (e.g., one per model).

To do this, we'll adapt the template to produce `txtar` output and use the `write` function to write the generated files to disk. The updated template will look like this:

models.tmpl
```codeBlockLines_AdAo
{{- /* The template for generating a Sequelize model file for the given table (defined in $) */}}{{- define "model" }}{{- $name := exec "title" $.Name | exec "singular" }}-- {{ lower $name }}.js --const { DataTypes } = require('sequelize');module.exports = (sequelize) => {  const {{ $name }} = sequelize.define('{{ $name }}', {    {{- range $i, $c := $.Columns }}    {{- if $i }},{{ end }}    {{ $c.Name }}: {      type: {{ exec "sequelize-type" . }},      allowNull: {{ .Type.Null }},      {{- range $idx := $c.Indexes }}          {{- if and (eq $idx $.PrimaryKey) (eq (len $idx.Parts) 1) }}              {{- print "\n      primaryKey: true," }}          {{- end }}      {{- end }}    }    {{- end }}  }, {    tableName: '{{ $.Name }}',    timestamps: false  });  return {{ $name }};};{{- end }}{{- /* Template for generating the index file that loads all models */}}{{- define "index" }}-- index.js --const { Sequelize } = require('sequelize');// Initialize Sequelize instance (adjust connection parameters as needed)const sequelize = new Sequelize('database', 'username', 'password', {  host: 'localhost',  dialect: 'postgres' // or 'mysql', 'mariadb', 'sqlite', 'mssql'});const models = {};{{- range $t := (index .Realm.Schemas 0).Tables }}{{- $name := exec "title" $t.Name | exec "singular" }}models.{{ $name }} = require('./{{ lower $name }}')(sequelize);{{- end }}module.exports = {  sequelize,  ...models};{{- end }}{{- /* Main template: build txtar archive with all model files and index.js */}}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" }}{{- $txtar := list }}{{- range $t := (index .Realm.Schemas 0).Tables }}  {{- $txtar = append $txtar (include "model" $t) }}{{- end }}{{- $txtar = append $txtar (include "index" .) }}{{- $output := "" }}{{- range $i, $content := $txtar }}  {{- $output = printf "%s%s" $output $content }}{{- end }}{{- $output | txtar | write "models" }}
```
Then, run the same command as before to execute the template:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src
```
This will generate a `models` directory containing one file per model, plus an index file that loads all models:
```codeBlockLines_AdAo
models├── friendship.js├── index.js├── message.js├── post.js└── user.js
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


This guide showed how to generate Sequelize model definitions from a database schema using the Atlas `--format` flag and Go's `text/template` engine. For more, see the [Go templates documentation](https://pkg.go.dev/text/template) and the [Atlas template functions reference](/inspect#template-functions).

*   [Overview](#overview)
*   [Defining a Template](#defining-a-template)
*   [Executing the Template](#executing-the-template)
*   [`write` and `txtar` Functions](#write-and-txtar-functions)
*   [Conclusion](#conclusion)