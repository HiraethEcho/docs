---
title: linux手册
date: 2025-10-25
tags:
  - linux
  - shell
  - tools
categories: handbook
topics: 使用linux
---

# Linux Handbook

https://github.com/SuperManito/LinuxMirrors

```
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
```

```
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

```
bash <(curl -sSL https://linuxmirrors.cn/docker.sh) --only-registry
```

## file

`filedialog` 是其他应用需要选择文件时的弹窗

## boot

## display manager

lightdm 有一堆莫名其妙的问题

gtk display还不知道怎么搞

## keyring

[https://wiki.archlinuxcn.org/zh-sg/GNOME/Keyring](https://wiki.archlinuxcn.org/zh-sg/GNOME/Keyring)

use `gnome-keyring` and `thunar` to auto connect a webdav.
[https://forum.mxlinux.org/viewtoasset.php?t=66523](https://forum.mxlinux.org/viewtoasset.php?t=66523)

## Reference

[https://linuxhandbook.com/](https://linuxhandbook.com/)

## commands

```sh
type something
```

example

```sh
type cd

cd is a shell builtin
```

## 技巧

```tldr
  Change attributes of files or directories.
  More information: <https://manned.org/chattr>.

  Make a file or directory [i]mmutable to changes and deletion, even by superuser:

      chattr +i path/to/file_or_directory

  Make a file or directory mutable:

      chattr -i path/to/file_or_directory

  [R]ecursively make an entire directory and contents immutable:

      chattr -R +i path/to/directory

  Mark a directory and its files to be interpreted in a case-insensitive manner (case-[F]olding):

      chattr +F path/to/directory

  Set a file to only allow [a]ppending:

      chattr +a path/to/file
```
