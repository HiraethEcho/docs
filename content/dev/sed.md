---
title: sed cheatsheet
tags:
  - linux
  - geek
  - tools
  - shell
categories: handbook
date: 2025-05-01
---

# sed

## tldr

```sh
sed -i "[range]s/[patern]/[replacement]/[options]" file_*
sed -i "2,5s|[patern]|[replacement]|[options]" file_*
```

### Search for text

Search for a string and only print the lines that were matched

```sh
$ sed -n '/hello/p' file.txt
```

Case insensitive search

```sh
$ sed -n '/hello/Ip' file.txt
```

### Sed Usage

Syntax

```shell
$ sed [options] command [input-file]
```

With pipeline

```sh
$ cat report.txt | sed 's/Nick/John/g'
```

```sh
$ echo '123abc' | sed 's/[0-9]+//g'
```

### Option Examples {.col-span-2}

| Option | Example                                    | Description                             |
| ------ | ------------------------------------------ | --------------------------------------- |
| `-i`   | sed -ibak 's/On/Off/' php.ini              | Backup and modify input file directly   |
| `-E`   | sed -E 's/[0-9]+//g' input-file            | Use extended regular expressions        |
| `-n`   | sed -n '3 p' config.conf                   | Suppress default pattern space printing |
| `-f`   | sed -f script.sed config.conf              | Execute sed script file                 |
| `-e`   | sed -e 'command1' -e 'command2' input-file | Execute multiple sed commands           |

### Multiple commands

```sh {.wrap}
$ echo "hello world" | sed -e 's/h/H/g' -e 's/w/W/g'
Hello World
```

Use `-e` to execute multiple sed commands

### Sed script

```sh
$ echo 's/h/H/g' >> hello.sed
$ echo 's/w/W/g' >> hello.sed
$ echo "hello world" | sed -f hello.sed
Hello World
```

Use `-f` to execute sed script file

### Examples

```sh
$ sed 's/old/new/g' file.txt
$ sed 's/old/new/g' file.txt > new.txt
$ sed 's/old/new/g' -i file.txt
$ sed 's/old/new/g' -i.backup file.txt
```

### Sed Usage

Syntax

```shell
$ sed [options] command [input-file]
```

With pipeline

```sh
$ cat report.txt | sed 's/Nick/John/g'
```

```sh
$ echo '123abc' | sed 's/[0-9]+//g'
```

### Option Examples {.col-span-2}

| Option | Example                                    | Description                             |
| ------ | ------------------------------------------ | --------------------------------------- |
| `-i`   | sed -ibak 's/On/Off/' php.ini              | Backup and modify input file directly   |
| `-E`   | sed -E 's/[0-9]+//g' input-file            | Use extended regular expressions        |
| `-n`   | sed -n '3 p' config.conf                   | Suppress default pattern space printing |
| `-f`   | sed -f script.sed config.conf              | Execute sed script file                 |
| `-e`   | sed -e 'command1' -e 'command2' input-file | Execute multiple sed commands           |

{.show-header}

### Multiple commands

```sh {.wrap}
$ echo "hello world" | sed -e 's/h/H/g' -e 's/w/W/g'
Hello World
```

Use `-e` to execute multiple sed commands

### Sed script

```sh
$ echo 's/h/H/g' >> hello.sed
$ echo 's/w/W/g' >> hello.sed
$ echo "hello world" | sed -f hello.sed
Hello World
```

Use `-f` to execute sed script file

### Examples

```sh
$ sed 's/old/new/g' file.txt
$ sed 's/old/new/g' file.txt > new.txt
$ sed 's/old/new/g' -i file.txt
$ sed 's/old/new/g' -i.backup file.txt
```

## Sed commands

### Commands {.col-span-2}

| Command | Example                                | Description                 |
| ------- | -------------------------------------- | --------------------------- |
| `p`     | sed -n '1,4 p' input.txt               | Print lines 1-4             |
| `p`     | sed -n -e '1,4 p' -e '6,7 p' input.txt | Print lines 1-4 and 6-7     |
| `d`     | sed '1,4 d' input.txt                  | Print lines except 1-4      |
| `w`     | sed -n '1,4 w output.txt' input.txt    | Write pattern space to file |
| `a`     | sed '2 a new-line' input.txt           | Append line after           |
| `i`     | sed '2 i new-line' input.txt           | Insert line before          |

{.show-header}

### Space commands

| Command | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `n`     | Print pattern space, empty pattern space, and read next line |
| `x`     | Swap pattern space with hold space                           |
| `h`     | Copy pattern space to hold space                             |
| `H`     | Append pattern space to hold space                           |
| `g`     | Copy hold space to pattern space                             |
| `G`     | Append hold space to pattern space                           |

### Flags

```sh
$ sed 's/old/new/[flags]' [input-file]
```

---

| Flag     | Description                                |
| -------- | ------------------------------------------ |
| `g`      | Global substitution                        |
| `1,2...` | Substitute the nth occurrence              |
| `p`      | Print only the substituted line            |
| `w`      | Write only the substituted line to a file  |
| `I`      | Ignore case while searching                |
| `e`      | Substitute and execute in the command line |

### Loops commands

| Command   | Description                                                        |
| --------- | ------------------------------------------------------------------ |
| `b label` | Branch to a label (for looping)                                    |
| `t label` | Branch to a label only on successful substitution<br>(for looping) |
| `:label`  | Label for the b and t commands (for looping)                       |
| `N`       | Append next line to pattern space                                  |
| `P`       | Print 1st line in multi-line                                       |
| `D`       | Delete 1st line in multi-line                                      |

### Misc Flags

| Flag                      | Description                                                                  |
| ------------------------- | ---------------------------------------------------------------------------- |
| <code>/ \| ^ @ ! #</code> | Substitution delimiter can be any character                                  |
| `&`                       | Gets the matched pattern                                                     |
| `( ) \1 \2 \3`            | Group using `(` and `)`.<br>Use `\1`, `\2` in replacement to refer the group |

## Sed examples

### Replacing text {.row-span-2}

Replace all occurrences of a string

```sh
$ sed 's/old/new/g' file.txt
```

Replace only the nth occurrence of a string

```sh
$ sed 's/old/new/2' file.txt
```

Replace a string only on the 5th line

```sh
$ sed '5 s/old/new/' file.txt
```

Replace "world" with "universe" but only if the line begins with "hello"

```sh
$ sed '/hello/s/world/universe/' file.txt
```

Remove "\" from the end of each line

```sh
$ sed 's/\\$//' file.txt
```

Remove all whitespace from beginning of each line

```sh
$ sed 's/^\s*//' file.txt
```

Remove comments. Even those that are at the end of a line

```sh
$ sed 's/#.*$//' file.txt
```

### Search for text

Search for a string and only print the lines that were matched

```sh
$ sed -n '/hello/p' file.txt
```

Case insensitive search

```sh
$ sed -n '/hello/Ip' file.txt
```

Search for a string but only output lines that do not match

```sh
$ sed -n '/hello/!p' file.txt
```

### Appending lines

Append line after line 2

```sh
$ sed '2a Text after line 2' file.txt
```

Append line at the end of the file

```sh
$ sed '$a THE END!' file.txt
```

Append line after every 3rd line starting from line 3

```sh
$ sed '3~3a Some text' file.txt
```

### Numbering {.col-span-2}

Number line of a file (simple left alignment)

```sh
$ sed = file.txt | sed 'N;s/\n/\t/'
```

Number line of a file (number on left, right-aligned)

```sh
$ sed = file.txt | sed 'N; s/^/   /; s/ *\(.\{6,\}\)\n/\1  /'
```

Number line of file, but only print numbers if line is not blank

```sh
$ sed '/./=' file.txt | sed '/./N; s/\n/ /'
```

Count lines (emulates "wc -l")

```sh
$ sed -n '$='
```

### Prepending lines

Insert text before line 5

```sh
$ sed '5i line number five' file.txt
```

Insert "Example: " before each line that contains "hello"

```sh
$ sed '/hello/i Example: ' file.txt
```

### Deleting lines

Delete line 5-7 in file

```sh
$ sed '5,7d' file.txt
```

Delete every 2nd line starting with line 3

```sh
$ sed '3~2d' file.txt
```

Delete the last line in file

```sh
$ sed '$d' file.txt
```

Delete lines starting with "Hello"

```sh
$ sed '/^Hello/d' file.txt
```

Delete all empty lines

```sh
$ sed '/^$/d' file.txt
```

Delete lines starting with "#"

```sh
$ sed '/^#/d' file.txt
```

### File spacing

Double space

```sh
$ sed G
```

Delete all blank lines and double space

```sh
$ sed '/^$/d;G'
```

Triple space a file

```sh
$ sed 'G;G'
```

Undo double-spacing

```sh
$ sed 'n;d'
```

Insert a blank line above line which matches "regex"

```sh
$ sed '/regex/{x;p;x;}'
```

Insert a blank line below line which matches "regex"

```sh
$ sed '/regex/G'
```

Insert a blank line around line which matches "regex"

```sh
$ sed '/regex/{x;p;x;G;}'
```

## Also see {.cols-1}

- [sed cheatsheet](https://gist.github.com/ssstonebraker/6140154) _(gist.github.com)_
  > 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [quickref.me](https://quickref.me/sed.html)
  > Sed is a stream editor, this sed cheat sheet contains sed commands and some common sed tricks.

### Replacing text

Replace all occurrences of a string

```
$ sed 's/old/new/g' file.txt
```

Replace only the nth occurrence of a string

```
$ sed 's/old/new/2' file.txt
```

Replace replace a string only on the 5th line

```
$ sed '5 s/old/new/' file.txt
```

Replace "world" with "universe" but only if the line begins with "hello"

```
$ sed '/hello/s/world/universe/' file.txt
```

Remove "" from the end of each line

```
$ sed 's/\\$//' file.txt
```

Remove all whitespace from beginning of each line

```
$ sed 's/^\s*//' file.txt
```

Remove comments. Even those that are at the end of a line

```
$ sed 's/#.*$//' file.txt
```

### Search for text

Search for a string and only print the lines that were matched

```
$ sed -n '/hello/p' file.txt
```

Case insensitive search

```
$ sed -n '/hello/Ip' file.txt
```

Search for a string but only output lines that do not match

```
$ sed -n '/hello/!p' file.txt
```

### Appending lines

Append line after line 2

```
$ sed '2a Text after line 2' file.txt
```

Append line at the end of the file

```
$ sed '$a THE END!' file.txt
```

Append line after every 3rd line starting from line 3

```
$ sed '3~3a Some text' file.txt
```

### Numbering

Number line of a file (simple left alignment)

```
$ sed = file.txt | sed 'N;s/\n/\t/'
```

Number line of a file (number on left, right-aligned)

```
$ sed = file.txt | sed 'N; s/^/   /; s/ *\(.\{6,\}\)\n/\1  /'
```

Number line of file, but only print numbers if line is not blank

```
$ sed '/./=' file.txt | sed '/./N; s/\n/ /'
```

Count lines (emulates "wc -l")

```
$ sed -n '$='
```

### Prepending lines

Insert text before line 5

```
$ sed '5i line number five' file.txt
```

Insert "Example:" before each line that contains "hello"

```
$ sed '/hello/i Example: ' file.txt
```

### Deleting lines

Delete line 5-7 in file

```
$ sed '5,7d' file.txt
```

Delete every 2nd line starting with line 3

```
$ sed '3~2d' file.txt
```

Delete the last line in file

```
$ sed '$d' file.txt
```

Delete lines starting with "Hello"

```
$ sed '/^Hello/d' file.txt
```

Delete all empty lines

```
$ sed '/^$/d' file.txt
```

Delete lines starting with "#"

```
$ sed '/^#/d' file.txt
```

### File spacing

Double space

```
$ sed G
```

Delete all blank lines and double space

```
$ sed '/^$/d;G'
```

Triple space a file

```
$ sed 'G;G'
```

Undo double-spacing

```
$ sed 'n;d'
```

Insert a blank line above line which matches "regex"

```
$ sed '/regex/{x;p;x;}'
```

Insert a blank line below line which matches "regex"

```
$ sed '/regex/G'
```

Insert a blank line around line which matches "regex"

```
$ sed '/regex/{x;p;x;G;}'
```
