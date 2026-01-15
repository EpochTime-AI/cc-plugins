Generate GORM Models (Go Code) from a Database Schema | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Although Atlas supports reading GORM schemas (`gorm.Model`) as the desired state and planning migrations based on it, it can sometimes be useful to go the other way - generating GORM models from an existing schema, such as a live database, HCL or SQL schema files, or even another ORM.

## Overview[​](#overview "Direct link to Overview")


One way to generate custom code from an Atlas schema is to get a JSON representation of it using the `atlas schema inspect` command with the `--format '{{ json . }}'` flag, and then use a custom script to generate the desired code.

However, in this guide, we'll show how you can use Atlas templates to generate custom code from an Atlas schema. Atlas, like many other CLI tools such as `kubectl` and `docker`, supports templating the output of commands using Go templates. This lets you write "custom code" that is evaluated at runtime and generates the desired output.

If you're not familiar with the Go templates language, you can read more about it in the [Go documentation](https://pkg.go.dev/text/template).

The template execution context is the result of the schema inspection, which is an object with two fields: `URL` and `Realm`. The `URL` field holds the inspected URL, and the `Realm` field contains the schema information, such as schemas, tables, columns, and more. You can see this object [here](https://pkg.go.dev/ariga.io/atlas/cmd/atlas@master/internal/cmdlog#SchemaInspect), and the definition of the `Realm` object at this [link](https://pkg.go.dev/ariga.io/atlas@v0.35.0/sql/schema#Realm).

## Defining a Template[​](#defining-a-template "Direct link to Defining a Template")


Let's define a simple template that generates GORM model structs from the `schema.Realm` object. For this example, we assume the database contains a single schema, and all models will be generated into a single file named `models.go`.

Note that besides the main template, this example defines several small helper templates that act like functions (`"title"`, `"singular"`, etc.). These can be reused across the template using `exec` and `include` functions.

models.tmpl
```codeBlockLines_AdAo
{{- /* A helper function-like template for generating a title-case from a database-object name. */}}{{- define "title" }}{{ $v := "" }}{{- range $w := splitBy $ "_" }}    {{- if le (len $w) 1 }}        {{ $v = print $v (upper $w) }}    {{- else }}        {{ $v = print $v (upper (slice $w 0 1)) (lower (slice $w 1)) }}    {{- end }}{{- end }}{{ print $v }}{{- end }}{{- /* A helper function-like template for converting a plural noun to its singular form using basic English rules. */}}{{- define "singular" -}}    {{- $s := . }}    {{- if and (hasSuffix $s "ies") (gt (len $s) 3) -}}        {{- printf "%sy" (slice $s 0 (sub (len $s) 3)) }}    {{- else if hasSuffix $s "ses" -}}        {{- trimSuffix $s "es" }}    {{- else if hasSuffix $s "xes" -}}        {{- trimSuffix $s "es" }}    {{- else if and (hasSuffix $s "s") (gt (len $s) 1) -}}        {{- trimSuffix $s "s" }}    {{- else -}}        {{- $s }}    {{- end }}{{- end }}{{- /* A helper function-like template for determining the Go type of a column based on its type. */}}{{- define "gotype" }}    {{- $m := dict        "character" "string"        "character varying" "string"        "text" "string"        "boolean" "bool"        "smallint" "int"        "integer" "int"        "bigint" "int64"    }}    {{- with $t := columnType . }}        {{- if hasKey $m $t }}{{ get $m $t }}{{ else }}any{{ end }}    {{- end }}{{- end }}{{- /* A helper function-like template for generating a struct tag for a column. */}}{{- define "struct-tag" }}    {{- $tags := list (printf "column:%s" $.Name) }}    {{- if $.Type.Null }}        {{ $tags = append $tags "not null" }}    {{- end }}    {{- printf "`gorm:\"%s\"`" (join $tags ";") }}{{- end }}{{- /* The template for generating a Go struct for the given table (defined in $) */}}{{- define "model" }}{{- $name := exec "title" $.Name | exec "singular" }}// The {{ $name }} type holds the definition of the {{ $.Name }} table.type {{ $name }} struct {    {{- println "\n\tgorm.Model" }}    {{- range $c := $.Columns }}        {{- /* Ignore fields defined in gorm.Model */}}        {{- if not (eq $c.Name "id" "created_at" "updated_at" "deleted_at") }}            {{- printf "\t%s %s %s\n" (exec "title" $c.Name) (exec "gotype" $c) (exec "struct-tag" $c) }}        {{- end }}    {{- end -}}}{{- end }}{{- /* Main template for generating all models in the schema */}}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" -}}package modelsimport "gorm.io/gorm"{{ range $t := (index .Realm.Schemas 0).Tables }}    {{- /* Include the result of the "model" template for each table */}}    {{- include "model" $t }}    {{- println }}{{- end }}
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
package modelsimport "gorm.io/gorm"// The User type holds the definition of the users table.type User struct {	gorm.Model	Name string `gorm:"column:name;not null"`	// ...}
```
To write the output to a file, redirect stdout to `models.go` and run `goimports` to format it:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src  > models.go && goimports -w models.go
```

## `write` and `txtar` Functions[​](#write-and-txtar-functions "Direct link to write-and-txtar-functions")


The `write` function lets us write files directly from within the template. This is useful when we want to generate multiple files (e.g., one per model) and then run `goimports` on the entire directory.

To do this, we'll adapt the template to produce `txtar` output and use the `write` function to write the generated files to disk. The updated template will look like this:

models.tmpl
```codeBlockLines_AdAo
{{- /* The template for generating a Go struct for the given table (defined in $) */}}{{- define "model" }}{{- $name := exec "title" $.Name | exec "singular" }}-- {{ lower $name }}.go --package modelsimport "gorm.io/gorm"// The {{ $name }} type holds the definition of the {{ $.Name }} table.type {{ $name }} struct {    {{- println "\n\tgorm.Model" }}    {{- range $c := $.Columns }}        {{- /* Ignore fields defined in gorm.Model */}}        {{- if not (eq $c.Name "id" "created_at" "updated_at" "deleted_at") }}            {{- printf "\t%s %s %s\n" (exec "title" $c.Name) (exec "gotype" $c) (exec "struct-tag" $c) }}        {{- end }}    {{- end -}}}{{- end }}{{- /* Main template for generating all models in the schema */}}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" -}}{{- $txtar := list }}{{- range $t := (index .Realm.Schemas 0).Tables }}    {{- /* Append the result of the "model" template for each table */}}    {{- $txtar = append $txtar (include "model" $t) }}{{- end }}{{- /* Join the result of all table templates, convert to txtar, and then write all at once to the file-system. */}}{{- join $txtar "\n\n" | txtar | write "models" }}
```
Then, run the same command as before to execute the template:
```codeBlockLines_AdAo
atlas schema inspect \  --env generate \  --url env://src && goimports -w models
```
This will generate a `models` directory containing one file per model, all formatted and ready to use.
```codeBlockLines_AdAo
models├── friendship.go├── message.go├── post.go└── user.go
```

## Conclusion[​](#conclusion "Direct link to Conclusion")


This guide showed how to generate GORM model structs from a database schema using the Atlas `--format` flag and Go's `text/template` engine. For more, see the [Go templates documentation](https://pkg.go.dev/text/template) and the [Atlas template functions reference](/inspect#template-functions).

*   [Overview](#overview)
*   [Defining a Template](#defining-a-template)
*   [Executing the Template](#executing-the-template)
*   [`write` and `txtar` Functions](#write-and-txtar-functions)
*   [Conclusion](#conclusion)