---
title: grep
date: 2025-05-05
tags:
  - linux
  - geek
  - tools
  - shell
categories: handbook
CountWords: false
---

## grep

ref: [www.hackingnote.com](https://www.hackingnote.com/en/cheatsheets/grep/)

## Count Occurrence: `-c`

```
$ cat example.txt | grep -c good
2
```

## Get Context: `-C`

Set `--context=0` to print that line alone

```
$ cat example.txt | grep --context=0 "good luck"
good luck
```

Set `--context=1` to print 1 line below and 1 line above

```
$ cat example.txt | grep --context=1 "good luck"
hello world
good luck
good day
```

or use `-C 1`

```
$ cat example.txt | grep -C 1 "good luck"
hello world
good luck
good day
```

## Ignore: `-v`

Use `-v` to exclude some lines(i.e. NOT)

```
$ cat example.txt | grep good | grep -v day
good luck
```

## Case Insensitive: `-i`

```
$ cat example.txt | grep GOOD
$ cat example.txt | grep -i GOOD
good luck
good day
```

## Show Match in Color: `--color`

```
$ cat example.txt | grep good --color
```

## Show Matched Line Number: `-n`

```
$ cat example.txt | grep good -n
2:good luck
3:good day
```

## Show Matched File Name: `-l`

`grep` is not limited to searching a single file, compare the results below

```
$ grep good example.txt
good luck
good day
```

to search from multiple files:

```
$ grep good *
example.txt:good luck
example.txt:good day
```

filename will be shown along with the matched lines; to show the filename only:

```
$ grep -l good *
example.txt
```

what happens to the "pipe" version?

```
$ cat example.txt | grep good -l
(standard input)
```

## Search for Whole Words Only: `-w`

```
$ grep -w goo example.txt
```

this returns nothing since `goo` is a pattern though not a whole word

```
$ grep goo example.txt
good luck
good day
```

## Recursive grep: `-R`

This will search all the directory and sub-directories recursively

```
$ grep -R pattern *
```

## Set Maximum Matches: `-m`

```
$ cat example.txt | grep -m 1 good
good luck
```

## Match

Show `ssh` processes

```
$ ps | grep ssh
```

Specify max count by `-m`:

```
$ cat foo.log | grep -m 10 ERROR
```

## Show file name

```
grep -H
```

## grep vs egrap vs fgrep

`grep`, `egrep` and `fgrep` are used to match patterns in files, here are the differences:

- `grep`: basic regular expressions
- `egrep`: extended regular expressions(`?`, `+`, `|`), equivalent to `grep -E`
- `fgrep`: fixed patterns, no regular expression; faster than grep and egrep; equivalent to `grep -F`
  Checkout the [Regular_expression wikipedia page](https://en.wikipedia.org/wiki/Regular_expression#POSIX_basic_and_extended) for the definitions of POSIX basic and extended regular expressions
  Assume there's a `examle.txt` file containing 4 lines:

```
$ cat example.txt
hello world
good luck
good day
linux
```

### `grep` vs `fgrep`

`fgrep` does not support regular expression at all, this will return nothing

```
$ cat example.txt | fgrep g..d
```

use `grep` instead

```
$ cat example.txt | grep g..d
good luck
good day
```

### `grep` vs `egrep`

`grep` does not support `|`, so this will return nothing

```
$ cat example.txt | grep "good|linux"
```

however `egrep` can recognize `|` as OR

```
$ cat example.txt | egrep "good|linux"
good luck
good day
linux
```
