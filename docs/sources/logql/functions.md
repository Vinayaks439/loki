---
title: Template functions
---

The [text template](https://golang.org/pkg/text/template) format used in `| line_format` and `| label_format` support functions the usage of functions.

All labels are added as variables in the template engine. They can be referenced using they label name prefixed by a `.`(e.g `.label_name`) for example the following template will output the value of the path label:

```template
{{ .path }}
```

You can take advantage of [pipeline](https://golang.org/pkg/text/template/#hdr-Pipelines) to chain multiple functions.
In a chained pipeline, the result of each command is passed as the last argument of the following command.

Example:

```template
{{ .path | replace " " "_" | trunc 5 | upper }}
```

## ToLower & ToUpper

Convert the entire string to lowercase or uppercase.

Signatures:

- `ToLower(string) string`
- `ToUpper(string) string`

Examples:

```template
"{{.request_method | ToLower}}"
"{{.request_method | ToUpper}}"
`{{ToUpper "This is a string" | ToLower}}`
```

> **Note:** In Loki 2.1 you can also use respectively [`lower`](#lower) and [`upper`](#upper) shortcut, e.g `{{.request_method | lower }}`

## Replace string

> **Note:** In Loki 2.1 [`replace`](#replace) (as opposed to `Replace`) is available with a different signature but easier to chain within pipeline.

Perform simple string replacement.

Signature:

`Replace(s, old, new string, n int) string`

It takes four arguments:

- `s` source string
- `old` string to replace
- `new` string to replace with
- `n` the maximun amount of replacement (-1 for all)

Example:

```template
`{{ Replace "This is a string" " " "-" -1 }}`
```

The above will produce `This-is-a-string`.

## Trim, TrimLeft, TrimRight and TrimSpace

> **Note:** In Loki 2.1 [trim](#trim), [trimAll](#trimAll), [trimSuffix](#trimSuffix) and [trimPrefix](trimPrefix) have been added with a different signature for better pipeline chaining.

`Trim` returns a slice of the string s with all leading and
trailing Unicode code points contained in cutset removed.

Signature: `Trim(value, cutset string) string`

`TrimLeft` and `TrimRight` are the same as `Trim` except that it respectively trim only leading and trailing characters.

```template
`{{ Trim .query ",. " }}`
`{{ TrimLeft .uri ":" }}`
`{{ TrimRight .path "/" }}`
```

`TrimSpace` TrimSpace returns string s with all leading
and trailing white space removed, as defined by Unicode.

Signature: `TrimSpace(value string) string`

```template
{{ TrimSpace .latency }}
```

`TrimPrefix` and `TrimSuffix` will trim respectively the prefix or suffix supplied.

Signature:

- `TrimPrefix(value string, prefix string) string`
- `TrimSuffix(value string, suffix string) string`

```template
{{ TrimPrefix .path "/" }}
```

## regexReplaceAll and regexReplaceAllLiteral

`regexReplaceAll` returns a copy of the input string, replacing matches of the Regexp with the replacement string replacement. Inside string replacement, $ signs are interpreted as in Expand, so for instance $1 represents the text of the first sub-match. See the golang [docs](https://golang.org/pkg/regexp/#Regexp.ReplaceAll) for detailed examples.

```template
`{{ regexReplaceAllLiteral "(a*)bc" .some_label "${1}a" }}`
```

`regexReplaceAllLiteral` returns a copy of the input string, replacing matches of the Regexp with the replacement string replacement The replacement string is substituted directly, without using Expand.

```template
`{{ regexReplaceAllLiteral "(ts=)" .timestamp "timestamp=" }}`
```

You can combine multiple function using pipe. For example, if you want to strip out spaces and make the request method in capital, then you would write the following template: `{{ .request_method | TrimSpace | ToUpper }}`.

## lower

> Added in Loki 2.1

Convert to lower case.

Signature:

`lower(string) string`

Examples:

```template
"{{ .request_method | lower }}"
`{{ lower  "HELLO"}}`
```

The last example will return `hello`.

## upper

> Added in Loki 2.1

Convert to upper case.

Signature:

`upper(string) string`

Examples:

```template
"{{ .request_method | upper }}"
`{{ upper  "hello"}}`
```

The last example will return `HELLO`.

## title

> **Note:** Added in Loki 2.1.

Convert to title case.

Signature:

`title(string) string`

Examples:

```template
"{{.request_method | title}}"
`{{ title "hello world"}}`
```

The last example will return `Hello World`.

## trunc

> **Note:** Added in Loki 2.1.

Truncate a string and add no suffix.

Signature:

`trunc(count int,value string) string`

Examples:

```template
"{{ .path | trunc 2 }}"
`{{ trunc 5 "hello world"}}`   // output: hello
`{{ trunc -5 "hello world"}}`  // output: world
```

## substr

> **Note:** Added in Loki 2.1.

Get a substring from a string.

Signature:

`trunc(start int,end int,value string) string`

If start is < 0, this calls value[:end].
If start is >= 0 and end < 0 or end bigger than s length, this calls value[start:]
Otherwise, this calls value[start, end].

Examples:

```template
"{{ .path | substr 2 5 }}"
`{{ substr 0 5 "hello world"}}`  // output: hello
`{{ substr 6 11 "hello world"}}` // output: world
```

## replace

> **Note:** Added in Loki 2.1.

Perform simple string replacement.

Signature: `replace(old string, new string, src string) string`

It takes three arguments:

- `old` string to replace
- `new` string to replace with
- `src` source string

Examples:

```template
{{ .cluster | replace "-cluster" "" }}
{{ replace "hello" "world" "hello world" }}
```

The last example will return `world world`.

## trim

> Added in Loki 2.1

The trim function removes space from either side of a string.

Signature: `trim(string) string`

Examples:

```template
{{ .ip | trim }}
{{ trim "   hello    " }} // output: hello
```

## trimAll

> Added in Loki 2.1

Remove given characters from the front or back of a string.

Signature: `trimAll(chars string,src string) string`

Examples:

```template
{{ .path | trimAll "/" }}
{{ trimAll "$" "$5.00" }} // output: 5.00
```

## trimSuffix

> Added in Loki 2.1

Trim just the suffix from a string.

Signature: `trimSuffix(suffix string, src string) string`

Examples:

```template
{{  .path | trimSuffix "/" }}
{{ trimSuffix "-" "hello-" }} // output: hello
```

## trimPrefix

> Added in Loki 2.1

Trim just the prefix from a string.

Signature: `trimPrefix(suffix string, src string) string`

Examples:

```template
{{  .path | trimPrefix "/" }}
{{ trimPrefix "-" "-hello" }} // output: hello
```

## indent

> **Note:** Added in Loki 2.1.

The indent function indents every line in a given string to the specified indent width. This is useful when aligning multi-line strings.

Signature: `indent(spaces int,src string) string`

```template
{{ indent 4 .query }}
```

The following will indent by 4 spaces each lines contained in `.query`.

## nindent

> **Note:** Added in Loki 2.1.

The nindent function is the same as the indent function, but prepends a new line to the beginning of the string.

Signature: `nindent(spaces int,src string) string`

```template
{{ nindent 4 .query }}
```

The above will indent every line of text by 4 space characters and add a new line to the beginning.

## repeat

> **Note:** Added in Loki 2.1.

Repeat a string multiple times.

Signature: `repeat(c int,value string) string`

```template
{{ repeat 3 "hello" }} // output: hellohellohello
```

## contains

> **Note:** Added in Loki 2.1.

Test to see if one string is contained inside of another.

Signature: `contains(s string, src string) bool`

Examples:

```template
{{ if .err contains "ErrTimeout" }} timeout {{end}}
{{ if contains "he" "hello" }} yes {{end}}
```

## hasPrefix and hasSuffix

> **Note:** Added in Loki 2.1.

The `hasPrefix` and `hasSuffix` functions test whether a string has a given prefix or suffix.

Signatures:

- `hasPrefix(prefix string, src string) bool`
- `hasSuffix(suffix string, src string) bool`

Examples:

```template
{{ if .err hasSuffix "Timeout" }} timeout {{end}}
{{ if hasPrefix "he" "hello" }} yes {{end}}
```