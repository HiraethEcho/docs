---
title: Archlinux 上的包管理
date: 2025-06-08
summary: pacman的一些使用技巧
tags:
  - archlinux
  - tools
categories: handbook
---

# Package manage on Arch

## pacman

```sh
pacman -Qqe | fzf --preview 'pacman -Qiil {}' --layout=reverse --bind 'enter:execute(pacman -Qiil {} | less)'
```

```sh
pacman -Slq | fzf --preview 'pacman -Si {}' --layout=reverse
```

```sh
pacman -D --asdeps $(pacman -Qqe)
```

```sh
pacman -D --asexplicit base linux linux-firmware
```

```sh
cat explicit | sudo pacman -D --asexplicit -
```

```sh
pacman -Qii | awk '/^MODIFIED/ {print $2}'
```

```sh
pacman -Qi | awk '/^Name/{name=$3} /^Install Date/{print $4,$5,$6,name}' | sort
```

## paru

## tools

### pacfiles

### expac

A data extraction tool for alpm databases, offering printf-like flexibility for pacman-based utilities.
[More information](https://github.com/falconindy/expac)

```man

SYNOPSIS
       Usage: expac [options] <format> targets...

OPTIONS
       -Q, --query
           Search  the local database for provided targets. This is the default
           behavior.

       -S, --sync
           Search the sync databases for provided targets.

       -s, --search
           Search for packages matching the strings specified by targets.  This
           is a boolean AND query and regex is allowed.

       -g, --group
           Return packages matching the specified targets as package groups.

       --config <file>
           Read from file for alpm initialization instead of /etc/pacman.conf.

       -H, --humansize <size>
           Format  package  sizes  in SI units according to size. Valid options
           are:

             B, K, M, G, T, P, E, Z, Y

       -1, --readone
           Stop searching after the first result. This only has an effect on -S
           operations without -s.

       -d, --delim <string>
           Separate each package with the specified string. The  default  value
           is a newline character.

       -l, --listdelim <string>
           Separate  each  list  item  with the specified string. Lists are any
           interpreted sequence specified with a capital  letter.  The  default
           value is two spaces.

       -p, --file
           Interpret targets as paths to local files.

       -t, --timefmt <format>
           Output time described by the specified format. This string is passed
           directly to strftime(3). The default format is %c.

       -v, --verbose
           Output  more.  ‘Package  not  found' errors will be shown, and empty
           field values will display as 'None'.

       -V, --version
           Display version information and quit.

       -h, --help
           Display the help message and quit.

FORMATTING
       The format argument allows the following interpreted sequences:

         %a    architecture

         %B    backup files

         %b    build date

         %C    conflicts with (no version strings)

         %D    depends on

         %d    description

         %E    depends on (no version strings)

         %e    package base

         %f    filename (only with -S)

         %F    files (only with -Q)

         %g    base64 encoded PGP signature (only with -S)

         %G    groups

         %H    conflicts with

         %h    sha256sum

         %i    has install scriptlet (only with -Q)

         %k    download size (only with -S)

         %l    install date (only with -Q)

         %L    licenses

         %m    install size

         %M    modified backup files (only with -Q)

         %n    package name

         %N    required by

         %O    optional deps

         %o    optional deps (no descriptions)

         %p    packager name

         %P    provides

         %R    replaces (no version strings)

         %r    repo

         %s    md5sum

         %S    provides (no version strings)

         %T    replaces

         %u    project URL

         %V    package validation method

         %v    version

         %w    install reason (only with -Q)

         %!    result number (auto-incremented counter, starts at 0)

         %%    literal %

       Note that for any lowercase tokens aside from %m  and  %k,  full  printf
       support  is  allowed, e.g. %-20n. This does not apply to any list based,
       date, or numerical output.

       Standard backslash escape sequences are supported, as per printf(1).

EXAMPLES
       Emulate pacman's search function:

         $ expac -Ss '%r/%n %v\n    %d' <search terms>

       List the oldest 10 installed packages (by build date):

         $ expac --timefmt=%s '%b\t%n' | sort -n | head -10

```

简短的用法

```tldr
  List the dependencies of a package:

      expac [-S|--sync] '%D' package

  List the optional dependencies of a package:

      expac [-S|--sync] "%o" package

  List the download size of packages in MiB:

      expac [-S|--sync] [-H|--humansize] M '%k\t%n' package1 package2 ...

  List packages marked for upgrade with their download size:

      expac [-S|--sync] [-H|--humansize] M '%k\t%n' $(pacman -Qqu) | sort [-sh|--sort --human-numeric-sort]

  List explicitly-installed packages with their optional dependencies:

      expac [-d|--delim] '\n\n' [-l|listdelim] '\n\t' [-Q|--query] '%n\n\t%O' $(pacman -Qeq)
```

```
expac -Q "%n %w %M" | grep explicit | awk '{print $3}' | grep -v ^$
```

### pacman-contrib

## PKGBUILD

### tips

pacman 进行升级时会将修改后的软件包升级到仓库中的最新版本，可以通过下面方式避免这个行为：

在 PKGBUILD 中将软件包加入 modified 组：

```PKGBUILD
groups=('modified')
```

然后将此组加入 `/etc/pacman.conf` 的 `IgnoreGroup`：

```/etc/pacman.conf
IgnoreGroup = modified
```

当系统升级发现官方仓库中有新版本时，pacman 会显示软件包因为在 IgnoreGroup 中而被忽略的提示，这时需要从 ABS 编译更新的软件包以防止部分升级。

