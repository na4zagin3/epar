# EPAR: Extensible Plain-text Archive (.epar) Format Version 0.1

**This document is under development!**

This is a specification of a extensible plain-text archive format.

Here is a sample EPAR that contains two files:

```epar
#EPAR: 0.1
delimiter: "---"
...

--- file1.txt

This is a text file.

--- windows.txt
line-break: CR
...

A text file with CR line breaks.
Typically created with Windows.

--- no-line-break-at-end.txt
line-break-at-end: no
...

A text file with CR line breaks.
Typically created with Windows.
```

## File format

A EPAR consist of a header, metadata, and file sections.
File encoding must be UTF-8.

```bnf
epar-archive ::= <header> <line-break> <header-metadata> <file-sections>
```

### Header

Every EPAR must contain header `#EPAR: 0.1` at the beginning.

```bnf
header = "#EPAR: 0.1"
```

### Header Metadata

Header metadata is a YAML document representing an YAML Object.
The object can have the following fields:

- `delimiter:  <string>`: A delimiter to split file sections. Default value is `--`. The value will be referenced as `<delimiter>` in the following part of BNF description.
    - A delimiter must contain more than one character.
    - Neither of the first or the last character of a delimiter should be a space (U+0020).
    - The sequence `<line-break> <line-break> <delimiter>` should not appear anywhere except in `file-section-preamble`s.

```bnf
header-metadata = <yaml-document>
```


### File Section

```bnf
file-section ::= <file-section-preamble> <filename> <line-break> <file-section-metadata>? <file-section-content>
file-section-preamble ::= <line-break> <line-break> <delimiter> <space>
filename ::= <non-space-character>+ | '"' <yaml:nb-double-one-line> '"' /* See https://yaml.org/spec/1.2/spec.html#nb-double-one-line */
```

#### File Section Metadata

File Section Metadata is a YAML document representing an YAML Object.
The object can have the following fields:

- `line-break: CR | LF | CRLF` Line break character. Default value is the same as the archive file.
- `line-break-at-end: <boolean>` If the file end with a line break. Default value is `yes`.

```bnf
file-section-metadata ::= <yaml-document>
```

#### File Section Content

File Section Content is a well-formed code unit sequence in UTF-8.

### Yaml Document

In EPAR, metadata is described with a [YAML 1.2](https://yaml.org/spec/1.2/spec.html) bare document.

```bnf
yaml-document = <yaml:l-bare-document> /* See https://yaml.org/spec/1.2/spec.html#l-bare-document */
             "..." <line-break>
```

## See Also
- [Human Readable Archive (.hrx)](https://github.com/google/hrx)
- [Plain Text Archive (ptar) Format](https://github.com/jtvaughan/ptar)
- [HAR - Human Archive Format](https://github.com/marler8997/har)
