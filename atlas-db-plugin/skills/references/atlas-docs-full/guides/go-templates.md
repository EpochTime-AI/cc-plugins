Customizing Inspection Output Programmatically with Go Templates | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

Atlas supports templating inspection output using Go templates, similar to tools like `kubectl` and `docker`. This lets you generate custom output at runtime.

If you're new to Go templates, see the [Go documentation](https://pkg.go.dev/text/template).

Templates are evaluated against the result of a schema inspection, which is an object with two fields:

*   `URL` - the inspected source
*   `Realm` - the schema structure, including schemas, tables, columns, etc.

See [SchemaInspect](https://pkg.go.dev/ariga.io/atlas/cmd/atlas@master/internal/cmdlog#SchemaInspect) and [`Realm`](https://pkg.go.dev/ariga.io/atlas@v0.35.0/sql/schema#Realm) for full details.

### Example: Generating ORM Models[​](#example-generating-orm-models "Direct link to Example: Generating ORM Models")


For complete examples of using Go templates to generate ORM models, see:

*   [Generate GORM models](/guides/orms/gorm/generate-models)
*   [Generate Sequelize models](/guides/orms/sequelize/generate-models)
*   [Generate TypeORM entities](/guides/orms/typeorm/generate-entities)

### Template Functions[​](#template-functions "Direct link to Template Functions")


Atlas includes several built-in template functions to help format and manipulate the output:

Name

Description

`fail`

Stops the template execution with the given error message.
```codeBlockLines_AdAo
{{- if ne (len .Realm.Schemas) 1 }}  {{- fail "expect exactly one schema" }}{{- end }}
```
`assert`

Asserts that the condition is true, otherwise stops the template execution with the given error message.
```codeBlockLines_AdAo
{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" }}
```
`lower`

Converts the given string to lower case.

`upper`

Converts the given string to upper case.

`replace`

Replaces all occurrences of old in s with new.
```codeBlockLines_AdAo
{{- $v := replace .Realm.Name " " "_" | lower }}
```
`trim`

Trims leading and trailing whitespace from the given string.

`join`

Joins the elements of the given slice of strings into a single string, separated by the given separator.

`splitBy`

Splits the given string by the specified separator and returns a slice of strings.

`trimAll`

Trims leading and trailing whitespace from each string in the given slice.

`hasPrefix`

Checks if the given string starts with the specified prefix.

`hasSuffix`

Checks if the given string ends with the specified suffix.

`trimPrefix`

Removes the specified prefix from the given string.

`trimSuffix`

Removes the specified suffix from the given string.

`sub`

Returns the result of subtracting the second integer from the first.

`add`

Returns the sum of the provided integers. If no integers are provided, returns 0.

`inc`

Increments the given integer by 1 and returns the result.

`mul`

Returns the product of two integers.

`div`

Returns the result of dividing the first integer by the second. Returns 0 on division by zero.

`mod`

Returns the remainder of dividing the first integer by the second.

`txtar`

Parses the given string as a txtar archive and returns it as an Archive object.
```codeBlockLines_AdAo
{{- (include "sub-template" .) | txtar |  write }}
```
`exec`

Executes the given template with the provided context and returns the result as a trimmed string.
```codeBlockLines_AdAo
{{- /* A helper function-like template for generating a title-case from a database-object name. */}}{{- define "title" }}{{ $v := "" }}{{- range $w := splitBy $ "_" }}  {{- if le (len $w) 1 }}    {{ $v = print $v (upper $w) }}  {{- else }}    {{ $v = print $v (upper (slice $w 0 1)) (lower (slice $w 1)) }}  {{- end }}{{- end }}{{ print $v }}{{- end }}{{- /* Call for the title template and assign it to a variable. */}}{{ $title := exec "title" $t.Name }}{{- /* Call the title template and use it in the output. */}}{{- exec "title" .Schema.Name }}
```
Note, unlike the **include** function, the **exec** function returns the result of the template as a trimmed strings.

`include`

Executes the named template with the provided context and returns the result as a string.
```codeBlockLines_AdAo
{{- /* A helper template for generating model definitions from an Atlas inspected schema. */}}{{- define "models" -}}package models{{- range $t := (index .Realm.Schemas 0).Tables }}{{- $name := exec "title" $t.Name }}// {{ $name }} holds the definition of the {{ $t.Name }} table.type {{ $name }} struct {  {{- range $c := $t.Columns }}  ...  {{- end }}}{{- end }}{{- end }}{{- assert (eq (len .Realm.Schemas) 1) "only one schema is supported in this example" }}{{- (include "models" .) | write "models.go" -}}
```
`columnType`

Returns the SQL type of the given column as a string.
```codeBlockLines_AdAo
{{- range $c := $t.Columns }}  {{ - $c.Name }}: {{ columnType $c.Type }},{{- end }}
```
`dict`

Creates a dictionary from a list of key-value pairs.
```codeBlockLines_AdAo
{{- $d := dict "key1" "value1" "key2" "value2" }}{{- $value1 := get $d "key1" }}
```
`get`

Retrieves the value associated with the given key from the dictionary.

`set`

Sets the value for the given key in the dictionary and returns the updated dictionary.

`unset`

Deletes the key from the dictionary and returns the updated dictionary.

`hasKey`

Checks if the dictionary contains the specified key.

`list`

Creates a list from the provided values.

`append`

Appends the given values to the list and returns a new list.

*   [Example: Generating ORM Models](#example-generating-orm-models)
*   [Template Functions](#template-functions)