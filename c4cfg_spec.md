# The C4CFG Language Specification

| | |
|---|---|
| **Version** | 1.0 |
| **Status** | Draft / Experimental |
| **Author** | Achior Labs |
| **Last Updated** | June 2026 |
| **File Extension** | `.c4cfg` |
| **Encoding** | UTF-8 |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Source Encoding](#2-source-encoding)
3. [Lexical Structure](#3-lexical-structure)
4. [File Structure](#4-file-structure)
5. [Statements](#5-statements)
6. [Keys](#6-keys)
7. [Assignments](#7-assignments)
8. [Tables](#8-tables)
9. [Array Tables](#9-array-tables)
10. [Expressions](#10-expressions)
11. [String Literals](#11-string-literals)
12. [Integer Literals](#12-integer-literals)
13. [Floating-Point Literals](#13-floating-point-literals)
14. [Boolean Literals](#14-boolean-literals)
15. [Null Literal](#15-null-literal)
16. [Arrays](#16-arrays)
17. [Inline Tables](#17-inline-tables)
18. [Semantic Rules](#18-semantic-rules)
19. [Grammar Summary](#19-grammar-summary)
20. [Future Extensions](#20-future-extensions)

---

## 1. Introduction

C4CFG is a human-readable configuration language designed for the C4 ecosystem. It is intended to give tools, compilers, and build systems a single, predictable format for describing structured configuration data.

### 1.1 Design Goals

- **Simplicity** — a small grammar that is easy to learn and easy to implement.
- **Deterministic parsing** — every valid document has exactly one parse.
- **Strong typing** — values carry an explicit, unambiguous type.
- **Human readability** — documents should be easy to read and edit by hand.
- **Tooling friendliness** — the grammar is regular enough for editors, linters, and formatters to process reliably.
- **Fast parsing** — single-pass, non-backtracking parsing is achievable.
- **Minimal ambiguity** — the grammar avoids constructs with more than one valid interpretation.

### 1.2 Intended Use Cases

C4CFG is intended for, among other things:

- Project manifests
- Build configuration
- Package management metadata
- Compiler configuration
- General tool configuration
- Deployment manifests

### 1.3 Conformance

The key words **MUST**, **MUST NOT**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

A document is a **conforming C4CFG document** if and only if it satisfies the grammar in this specification and all rules in [§18 Semantic Rules](#18-semantic-rules). A **conforming implementation** MUST accept every conforming document and MUST reject every document that violates a normative rule.

---

## 2. Source Encoding

All C4CFG files **MUST** be encoded using UTF-8 without a byte-order mark (BOM). Implementations encountering a BOM **SHOULD** strip it before parsing.

The recommended file extension is `.c4cfg`.

```text
project.c4cfg
compiler.c4cfg
package.c4cfg
```

---

## 3. Lexical Structure

### 3.1 Whitespace

Whitespace separates tokens and is otherwise insignificant. The recognized whitespace characters are:

| Character | Description |
|---|---|
| `U+0020` | space |
| `U+0009` | tab |
| `U+000D` | carriage return |
| `U+000A` | newline |

### 3.2 Comments

Comments extend from the comment marker to the end of the line. Both `#` and `//` are recognized as comment markers.

```c4cfg
# compiler settings
// build configuration
```

### 3.3 Identifiers

Identifiers are restricted to ASCII characters.

```ebnf
identifier        = identifier_start , { identifier_continue } ;
identifier_start   = "A"…"Z" | "a"…"z" | "_" ;
identifier_continue = identifier_start | "0"…"9" | "-" ;
```

```c4cfg
project
project_name
build-output
target_arch
```

---

## 4. File Structure

A C4CFG file consists of a sequence of zero or more statements.

```ebnf
file = { statement } ;
```

---

## 5. Statements

```ebnf
statement = assignment | table | array_table ;
```

---

## 6. Keys

Keys identify values and may be hierarchical, with components separated by `.`.

```ebnf
key = identifier , { "." , identifier } ;
```

```c4cfg
compiler
compiler.optimization
build.output.directory
```

---

## 7. Assignments

An assignment associates a key with a value.

```ebnf
assignment = key , "=" , expression ;
```

```c4cfg
name          = "kernel"
version       = "0.1.0"
debug         = true
optimization  = 3
```

---

## 8. Tables

Tables introduce a hierarchical scope under which subsequent assignments are nested, until the next table or array-table header.

```ebnf
table = "[" , key , "]" , { assignment } ;
```

```c4cfg
[build]
optimization = 3
debug = true
```

Nested tables are addressed with dotted keys:

```c4cfg
[compiler.options]
warnings = true
```

---

## 9. Array Tables

Array tables define a sequence of repeated, structurally similar objects under the same key.

```ebnf
array_table = "[[" , key , "]]" , { assignment } ;
```

```c4cfg
[[dependencies]]
name    = "raylib"
version = "5.0"

[[dependencies]]
name    = "openssl"
version = "3.5"
```

---

## 10. Expressions

```ebnf
expression = string
           | integer
           | float
           | boolean
           | null
           | array
           | inline_table ;
```

---

## 11. String Literals

### 11.1 Basic Strings

```ebnf
string      = '"' , { string_char } , '"' ;
string_char = escape_sequence | ? any character except '"' or '\' ? ;
```

Supported escape sequences:

| Escape | Meaning |
|---|---|
| `\\` | backslash |
| `\"` | double quote |
| `\n` | newline |
| `\r` | carriage return |
| `\t` | tab |
| `\0` | null character |
| `\xHH` | byte by two-digit hex code |
| `\uHHHH` | Unicode code point by four-digit hex code |

```c4cfg
name = "C4 Compiler"
path = "C:\\Projects\\c4"
```

### 11.2 Multiline Strings

```ebnf
multiline_string = '"""' , { character } , '"""' ;
```

```c4cfg
description = """
A systems programming language
focused on performance,
safety and simplicity.
"""
```

---

## 12. Integer Literals

```ebnf
integer = decimal_integer
        | hexadecimal_integer
        | binary_integer
        | octal_integer ;
```

### 12.1 Decimal

```ebnf
decimal_integer = [ "+" | "-" ] , digit , { digit | "_" } ;
```

```c4cfg
42
-100
1_000_000
```

### 12.2 Hexadecimal

```ebnf
hexadecimal_integer = "0x" , hex_digit , { hex_digit | "_" } ;
```

```c4cfg
0xFF
0xDEADBEEF
```

### 12.3 Binary

```ebnf
binary_integer = "0b" , binary_digit , { binary_digit | "_" } ;
```

```c4cfg
0b10101010
```

### 12.4 Octal

```ebnf
octal_integer = "0o" , octal_digit , { octal_digit | "_" } ;
```

```c4cfg
0o755
```

Underscores in integer literals are purely visual separators and **MUST** be ignored by the implementation when computing the literal's value. An underscore **MUST NOT** appear as the first or last character of the digit sequence, nor adjacent to another underscore.

---

## 13. Floating-Point Literals

```ebnf
float = decimal_integer , "." , decimal_integer , [ exponent ]
      | decimal_integer , exponent ;

exponent = ( "e" | "E" ) , [ "+" | "-" ] , decimal_integer ;
```

```c4cfg
3.14
1.0e6
2.5E-4
```

---

## 14. Boolean Literals

```ebnf
boolean = "true" | "false" ;
```

```c4cfg
debug   = true
release = false
```

---

## 15. Null Literal

```ebnf
null = "null" ;
```

```c4cfg
cache = null
```

---

## 16. Arrays

```ebnf
array = "[" , [ expression , { "," , expression } , [ "," ] ] , "]" ;
```

```c4cfg
ports = [80, 443]

sources = [
    "main.c4",
    "lexer.c4",
    "parser.c4",
]
```

A trailing comma before the closing bracket is permitted. Per [§18.3](#18-semantic-rules), all elements of an array **MUST** share the same type.

---

## 17. Inline Tables

```ebnf
inline_table  = "{" , [ inline_field , { "," , inline_field } , [ "," ] ] , "}" ;
inline_field  = identifier , "=" , expression ;
```

```c4cfg
author = {
    name = "Ariel",
    organization = "Achior Labs",
}
```

---

## 18. Semantic Rules

The following rules are normative and **MUST** be enforced by conforming implementations:

1. Duplicate keys within the same scope **MUST** be rejected.
2. Duplicate table declarations (re-opening the same table path) **MUST** be rejected.
3. Arrays **MUST** be homogeneous — all elements **MUST** share the same expression type.
4. Keys **MUST** be treated as case-sensitive.
5. Integer overflow (a literal exceeding the implementation's integer representation) **MUST** be treated as an error.
6. Unknown escape sequences within a string **MUST** be treated as invalid.
7. Circular references (where supported by an extension) **MUST** be rejected.
8. Existing values **MUST NOT** be redefined once set.
9. Parsing **MUST** be deterministic: a given document **MUST** always produce the same parse tree.
10. Parsers **MUST NOT** require backtracking; the grammar is designed to be parseable with a single token of lookahead.

---

## 19. Grammar Summary

The complete grammar, collected for reference, in EBNF:

```ebnf
file                = { statement } ;
statement           = assignment | table | array_table ;

key                 = identifier , { "." , identifier } ;
assignment          = key , "=" , expression ;
table               = "[" , key , "]" , { assignment } ;
array_table         = "[[" , key , "]]" , { assignment } ;

expression          = string | integer | float | boolean | null
                     | array | inline_table ;

identifier          = identifier_start , { identifier_continue } ;
identifier_start    = "A"…"Z" | "a"…"z" | "_" ;
identifier_continue = identifier_start | "0"…"9" | "-" ;

string              = '"' , { string_char } , '"' ;
multiline_string    = '"""' , { character } , '"""' ;

integer             = decimal_integer | hexadecimal_integer
                     | binary_integer | octal_integer ;
decimal_integer     = [ "+" | "-" ] , digit , { digit | "_" } ;
hexadecimal_integer = "0x" , hex_digit , { hex_digit | "_" } ;
binary_integer      = "0b" , binary_digit , { binary_digit | "_" } ;
octal_integer       = "0o" , octal_digit , { octal_digit | "_" } ;

float               = decimal_integer , "." , decimal_integer , [ exponent ]
                     | decimal_integer , exponent ;
exponent            = ( "e" | "E" ) , [ "+" | "-" ] , decimal_integer ;

boolean             = "true" | "false" ;
null                = "null" ;

array               = "[" , [ expression , { "," , expression } , [ "," ] ] , "]" ;
inline_table        = "{" , [ inline_field , { "," , inline_field } , [ "," ] ] , "}" ;
inline_field        = identifier , "=" , expression ;
```

---

## 20. Future Extensions

The following features are under consideration for future versions of C4CFG:

- Imports
- Environment variable expansion
- Compile-time expressions
- Conditional sections
- Schema validation
- Build profiles
- Package references
- Tool metadata

Any future extension **MUST** preserve backward compatibility with C4CFG v1.0: a document valid under v1.0 **MUST** remain valid under any subsequent minor version.

---

## Appendix A: Full Example

```c4cfg
# Example project manifest

name    = "kernel"
version = "0.1.0"
debug   = true

description = """
A systems programming language
focused on performance,
safety and simplicity.
"""

[build]
optimization = 3
debug        = true

[compiler.options]
warnings = true

author = {
    name = "xxxx",
    organization = "xxxx",
}

[[dependencies]]
name    = "raylib"
version = "5.0"

[[dependencies]]
name    = "openssl"
version = "3.5"
```
