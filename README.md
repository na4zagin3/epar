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
line-break: crlf
...

A text file with CRLF line breaks.
Typically created with Windows.

--- no-line-break-at-end.txt
line-breaks-at-end: 0
...

A text file without a trailing line break.

--- excess-line-break-at-end.txt
line-breaks-at-end: 2
...

A text file with an excess empty line.
```

## Goals

- Both human-readable and human-writable.
- Simple look for typical usages.
- Comprehensive as an archive format. In other words, EPAR must eventually be able to handle extreme cases as [non-UTF-8 encoded filenames](https://github.com/na4zagin3/epar/issues/4), [binary files](https://github.com/na4zagin3/epar/issues/2) and sparse files.

## File format

A EPAR consist of a header, metadata, and file sections.
File encoding must be UTF-8.

```bnf
epar-archive ::= <header> <line-break> ( <header-metadata> <file-sections>? )?
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
- `defaults:  <object>`: An object representing default options of each file section metadata.

```bnf
header-metadata = <yaml-document> | <line-break>
```


### File Section

```bnf
file-sections ::= <file-section> ( <line-break> <line-break> <file-section> )*
file-section ::= <file-section-preamble> <filename> <line-break> <file-section-metadata>? <file-section-content>
file-section-preamble ::= <delimiter> <space>
filename ::= <non-space-character>+ | '"' <yaml:nb-double-one-line> '"' /* See https://yaml.org/spec/1.2/spec.html#nb-double-one-line */
```

The file separator in a filename is `/` regardless of environments, including Windows.

A filename MUST NOT contain current-directory references (i.e., `.`) or parent-directory references (i.e., `..`).  For example, `a/./b.txt` is NOT a valid filename; meanwhile `a/.b.txt` IS a valid filename.

Each file section SHOULD have a unique filename.

#### File Section Metadata

File Section Metadata is a YAML document representing an YAML Object.
The object can have the following fields:

- `line-break: cr | lf | crlf` Line break character. Default value is the same as the archive file.
- `line-breaks-at-end: <integer>` Number of line breaks at the end of the content. Default value is `1`.

```bnf
file-section-metadata ::= <yaml-document>
```

#### File Section Content

File Section Content is a well-formed code unit sequence (in UTF-8).
Trailing line breaks are not encoded.

```bnf
file-section-content ::= <file-content> <line-break>*
```

### Yaml Document

In EPAR, metadata is described with a [YAML 1.2](https://yaml.org/spec/1.2/spec.html) bare document.

```bnf
yaml-document = <yaml:l-bare-document> /* See https://yaml.org/spec/1.2/spec.html#l-bare-document */
             "..." <line-break>
```

## Implementations

- [ocaml-epar](https://github.com/na4zagin3/ocaml-epar)

## See Also
- [Human Readable Archive (.hrx)](https://github.com/google/hrx)
- [Plain Text Archive (ptar) Format](https://github.com/jtvaughan/ptar)
- [HAR - Human Archive Format](https://github.com/marler8997/har)
