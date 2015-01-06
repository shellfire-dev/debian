# `debian`: functions module for [shellfire]

This module provides a simple framework for parsing Debian constructs. Currently, this consists of Debian control files, although specific support is only implemented for the [copyright](https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/) format. Other parsers are trivial to implement - look at the code in [copyright.functions](https://github.com/shellfire-dev/debian/blob/master/control/paragraph/copyright.functions). The parser validates format, syntax, checks for duplicates and verifies that mandatory fields are present.

An example user is [swaddle].

## Overview

Usage couldn't be simpler. To parse a Debian copyright file, try:-

```bash
debian_control_parser my_callback debian_control_parser_paragraph_copyright no /usr/share/doc/swaddle/copright
```

where `debian_control_parser_paragraph_copyright` is a paragraph-aware parser (Debian control files consist of one or more paragraphs of an expected kind) and `my_callback` is a function that accepts parse events:-

```bash
my_callback()
{
	local fileFormat="$1"
	local paragraphType="$2"
	local eventType="$3"
		
	# 'Copyright'
	echo "File Format $fileFormat"
	
	# 'Header', 'Files' or 'License'
	echo "$paragraphType"
	
	# 'ParagraphStart', 'ParagraphEnd' or 'Field'
	echo "$eventType"
	
	if [ "$eventType" = 'Field' ]; then

		local fieldName="$4"
		local fieldType="$5"
		
		# eg 'Format'
		echo "$fieldName"
		
		# eg 'singleLine' (see docs)
		echo "$fieldType"
		
		# Depending on fieldType, you can get to values with debian_control_parser_fieldValueFirstLine, debian_control_parser_rawLines (an array), debian_control_parser_fieldValue (for folded fields) or debian_control_parser_continuationLines (for fields with a synopsis or list)
	fi
}
```

## Importing

To import this module, add a git submodule to your repository. From the root of your git repository in the terminal, type:-
```bash
mkdir -p lib/shellfire
cd lib/shellfire
git submodule add "https://github.com/shellfire-dev/debian.git"
cd -
git submodule init --update
```

You may need to change the url `https://github.com/shellfire-dev/debian.git` above if using a fork.

You will also need to add paths - include the module [paths.d].

You will also need to import the [unicode] module.


## Namespace `debian_control_parser`

This namespace contains the core functionality needed to access the API.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn debian/control parser
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn debian/control parser
	…
}
```

### Functions

***
#### `debian_control_parser()`

|Parameter|Value|Optional|
|---------|-----|--------|
|`eventHandler`|Function to handle parser events.|_No_|
|`paragraph`|Function defined to handle file subformat (paragraphs).|_No_|
|`commentsAllowed`|Boolean. Comments are only normally allowed in `debian/control` package source files. If in doubt, use `yes`.|_No_|
|`controlDataFilePath`|Path to Debian control file to parse.|_No_|

The callback function is the following arguments:-

|Variable|Description|
|--------|-----------|
|`fileFormat`|The file format. Dictated by first paragraph kind. Not useful as you know this, unless writing a generic `paragraph` function or `eventHandler`.|
|`paragraphType`|The `start` or `end` of an object or array. The type of value: `boolean` (including null), `number` or `string`|
|`eventType`|The parse event name. One of `ParagraphStart`, `Field` and `ParagraphEnd`. If `ParagraphStart` is raised, one or more `Field` _will_ be raised next followed by `ParagraphEnd`.|
|`fieldName`|Only passed if `eventType` is `Field`.|
|`fieldType`|Only passed if `eventType` is `Field`.|

If `eventType` is `Field`, then the following variables are (locally) in scope depending on the value of `fieldType`:-

|Variable|`fieldType`|Description|
|--------|-----------|-----------|
|`debian_control_parser_fieldValueFirstLine`|Any|Value of first line, trimmed|
|`debian_control_parser_fieldValue`|Any|As above but with `foldedCommas` / `foldedWhitespace` unfolded.|
|`debian_control_parser_rawLines`|Any|All lines, including first|
|`debian_control_parser_continuationLines`|Any|Trimmed continuation lines, but includes first line for `multilineList` and `multilineWithoutSynopsis`|

`fieldType` may be one of:-

|Value|Description|
|-----|-----------|
|`singleLine`|A single line. The commonest type|
|`foldedCommas`|A folded line; folding uses commas. Only used in source control files, eg `Uploaders`|
|`foldedWhitespace`|A folded line; folding uses whitespace. Only used in source control files, eg `.changes' Binary`|
|`multilineListBlankFirstLine`|Multiple lines, first blank, eg `Checksums-Sha1`|
|`multilineList`|Multiple lines, first blank, eg `Upstream-Contact`|
|`multilineWithSynopsis`|Multiple lines, first synopsis, eg `Description`, `License`|
|`multilineWithoutSynopsis`|Multiple lines, no synopsis, eg `Comment`|

Additionally, the function `debian_control_parser_outputRawFieldLines'` can be called to write to standard out the current field exactly as read.

If `eventType` is `ParagraphEnd`, then the array variable `debian_control_parser_fieldsInParagraph` contains the fields found in the paragraph.

***
#### `debian_control_parser_outputRawFieldLines()`

Takes no arguments. Writes to standard out the current field exactly as read when the `eventType` is `Field`.

***

#### `debian_control_parser_continuationEventNotWanted()`

Intended for implementation of `paragraph` functions. Description omitted. May change.

***

#### `debian_control_parser_warning()`

Intended for implementation of `paragraph` functions. Description omitted. May change.



## Namespace `debian_control_parser`

This namespace contains the core functionality needed to access the API.

### To use in code

If calling from another [shellfire] module, add to your shell code the line
```bash
core_usesIn debian/control/parser/paragraph copyright
```
in the global scope (ie outside of any functions). A good convention is to put it above any function that depends on functions in this module. If using it directly in a program, put this line _inside_ the `_program()` function:-

```bash
_program()
{
	core_usesIn debian/control/parser/paragraph copyright
	…
}
```

### Functions

***
#### `debian_control_parser_paragraph_copyright()`

Arguments deliberately not described. Pass this function as the `paragraph` argument of `debian_control_parser()` to parse Debian copyright file.




[swaddle]: https://github.com/raphaelcohn/swaddle "Swaddle homepage"
[shellfire]: https://github.com/shellfire-dev "shellfire homepage"
[core]: https://github.com/shellfire-dev/core "shellfire core module homepage"
[paths.d]: https://github.com/shellfire-dev/paths.d "paths.d shellfire module homepage"
[github api]: https://github.com/shellfire-dev/github "github shellfire module homepage"
[jsonwriter]: https://github.com/shellfire-dev/jsonwriter "jsonwriter shellfire module homepage"
[debian]: https://github.com/shellfire-dev/debian "debian shellfire module homepage"
[urlencode]: https://github.com/shellfire-dev/urlencode "urlencode shellfire module homepage"
[unicode]: https://github.com/shellfire-dev/unicode "unicode shellfire module homepage"
[version]: https://github.com/shellfire-dev/version "version shellfire module homepage"