---
title: 用btrfs做linux的快照备份
tags:
  - archlinux
  - geek
categories: handbook
---

# linux下的 btrfs

## btrfs的功能

写时复制 `copy on right`功能使得非常时候做快照。

子卷功能可以更方便的分区。以及添加新设备。

## 官方手册

```man

BTRFS(8)                             BTRFS                             BTRFS(8)

NAME
       btrfs - a toolbox to manage btrfs filesystems

SYNOPSIS
       btrfs [global options] <group> [<group>...] <command> [options] [<args>]

DESCRIPTION
       The  btrfs  utility  is a toolbox for managing btrfs filesystems.  There
       are command groups to work with subvolumes, devices, for whole  filesys‐
       tem or other specific actions. See section COMMANDS.

       There  are also standalone tools for some tasks like btrfs-convert(8) or
       btrfstune(8) that were separate historically and/or haven't been  merged
       to the main utility. See section STANDALONE TOOLS for more details.

       For  other topics (mount options, etc) please refer to the separate man‐
       ual page btrfs(5).

COMMAND SYNTAX
       Any command name can be shortened so long as the shortened form is unam‐
       biguous, however, it  is  recommended  to  use  full  command  names  in
       scripts.  All command groups have their manual page named btrfs-<group>.

       For example: it is possible to run btrfs sub snaps instead of btrfs sub‐
       volume snapshot.  But btrfs file s is not allowed, because file s may be
       interpreted both as filesystem show and as filesystem sync.

       If  the  command  name  is ambiguous, the list of conflicting options is
       printed.

       Sizes, both upon input and output, can be  expressed  in  either  SI  or
       IEC-I  units  (see  numfmt(1))  with the suffix B appended.  All numbers
       will be formatted according to the rules of the C locale  (ignoring  the
       shell locale, see locale(7)).

       For  an  overview  of  a given command use btrfs command --help or btrfs
       [command...] help --full to print all available  options  for  all  com‐
       mands.

       There  are  global  options  that are passed between btrfs and the group
       name and affect behaviour not specific to the command, e.g. verbosity or
       the type of the output:

          btrfs -q subvolume create ...
          btrfs --dry-run subvolume create ...

       --format <format>
              if supported by the command, print subcommand output in that for‐
              mat (text, json)

       -v|--verbose
              increase verbosity of the subcommand

       -q|--quiet
              print only errors

       --log <level>
              set log level (default, info, verbose, debug, quiet)

       The remaining options are relevant only for the main tool:

       --help print condensed help for all subcommands

       --version
              print version string

COMMANDS
       balance
              Balance btrfs filesystem chunks across single or several devices.
              See btrfs-balance(8) for details.

       check  Do off-line check on a btrfs filesystem.  See btrfs-check(8)  for
              details.

       device Manage devices managed by btrfs, including add/delete/scan and so
              on.  See btrfs-device(8) for details.

       filesystem
              Manage  a  btrfs  filesystem, including label setting/sync and so
              on.  See btrfs-filesystem(8) for details.

       inspect-internal
              Debug       tools       for       developers/hackers.         See
              btrfs-inspect-internal(8) for details.

       property
              Get/set a property from/to a btrfs object.  See btrfs-property(8)
              for details.

       qgroup Manage   quota   group(qgroup)   for   btrfs   filesystem.    See
              btrfs-qgroup(8) for details.

       quota  Manage quota on btrfs filesystem like  enabling/rescan  and  etc.
              See btrfs-quota(8) and btrfs-qgroup(8) for details.

       receive
              Receive  subvolume data from stdin/file for restore and etc.  See
              btrfs-receive(8) for details.

       replace
              Replace btrfs devices.  See btrfs-replace(8) for details.

       rescue Try to rescue damaged btrfs filesystem.  See btrfs-rescue(8)  for
              details.

       restore
              Try  to  restore  files  from  a  damaged  btrfs filesystem.  See
              btrfs-restore(8) for details.

       scrub  Scrub a btrfs filesystem.  See btrfs-scrub(8) for details.

       send   Send subvolume data to  stdout/file  for  backup  and  etc.   See
              btrfs-send(8) for details.

       subvolume
              Create/delete/list/manage       btrfs       subvolume.        See
              btrfs-subvolume(8) for details.

STANDALONE TOOLS
       New functionality could be provided using  a  standalone  tool.  If  the
       functionality  proves to be useful, then the standalone tool is declared
       obsolete and its functionality is copied  to  the  main  tool.  Obsolete
       tools are removed after a long (years) depreciation period.

       Tools that are still in active use without an equivalent in btrfs:

       btrfs-convert
              in-place conversion from ext2/3/4 filesystems to btrfs

       btrfstune
              tweak some filesystem properties on a unmounted filesystem

       btrfs-select-super
              rescue tool to overwrite primary superblock from a spare copy

       btrfs-find-root
              rescue helper to find tree roots in a filesystem

       For  space-constrained environments, it's possible to build a single bi‐
       nary with functionality of several standalone tools. This  is  following
       the  concept  of  busybox where the file name selects the functionality.
       This works for symlinks or hardlinks. The full list can be  obtained  by
       btrfs help --box.

EXIT STATUS
       btrfs returns a zero exit status if it succeeds. Non zero is returned in
       case of failure.

AVAILABILITY
       btrfs  is  part  of btrfs-progs.  Please refer to the documentation at ‐
       https://btrfs.readthedocs.io.

SEE ALSO
       btrfs(5),    btrfs-balance(8),     btrfs-check(8),     btrfs-convert(8),
       btrfs-device(8),     btrfs-filesystem(8),     btrfs-inspect-internal(8),
       btrfs-property(8),  btrfs-qgroup(8),  btrfs-quota(8),  btrfs-receive(8),
       btrfs-replace(8),   btrfs-rescue(8),  btrfs-restore(8),  btrfs-scrub(8),
       btrfs-send(8), btrfs-subvolume(8), btrfstune(8), mkfs.btrfs(8)

6.14                              Mar 27, 2025                         BTRFS(8)
```

## 快照

用timeshift和snapper做快照。子卷布局

```
ID 257 gen 96108 top level 5 path @home
ID 318 gen 105959 top level 257 path @home/username
ID 268 gen 96108 top level 318 path @home/username/.snapshots
ID 269 gen 96108 top level 268 path @home/username/.snapshots/1/snapshot
ID 258 gen 105883 top level 5 path @root
ID 259 gen 105850 top level 5 path @cache
ID 260 gen 105959 top level 5 path @log
ID 261 gen 105942 top level 5 path @tmp
ID 262 gen 98722 top level 5 path @opt
ID 273 gen 96108 top level 262 path @opt/.snapshots
ID 274 gen 96108 top level 258 path @root/.snapshots
ID 285 gen 96433 top level 274 path @root/.snapshots/1/snapshot
ID 290 gen 105959 top level 5 path @
ID 519 gen 96445 top level 268 path @home/username/.snapshots/2/snapshot
ID 536 gen 103272 top level 5 path timeshift-btrfs/snapshots/2025-04-21_12-54-20/@
ID 537 gen 104133 top level 5 path timeshift-btrfs/snapshots/2025-04-21_20-12-43/@
```

即根目录用`timeshift`备份，并且回滚过一次（从@的ID可以看出来，不是安装时创建的256），而`/home/username`,`/home/root`,`/home/opt`用`snapper`做了几个快照

### timeshift 备份根目录

注意`btrfs`的子卷布局，根目录必须是`@`，例如

```
ID 257 gen 96108 top level 5 path @home
ID 258 gen 105883 top level 5 path @root
ID 259 gen 105850 top level 5 path @cache
ID 260 gen 105930 top level 5 path @log
ID 261 gen 105928 top level 5 path @tmp
ID 262 gen 98722 top level 5 path @opt
ID 290 gen 105929 top level 5 path @
```

### snapper 备份子卷

参考：

- [Snapper](https://wiki.archlinux.org/title/Snapper)
- [我的ArchLinux下Btrfs方案](https://www.hhhil.com/posts/archlinux-btrfs-scheme/#%e5%bf%ab%e7%85%a7%e4%b8%8e%e5%a4%87%e4%bb%bd)

snapper更加灵活。首先创建一个snapper的配置文件

```sh
sudo snapper -c [name of config] create-config [path-to-subvol]
```

例如对根目录的备份配置，`snapper`默认配置名是`root`

```sh
sudo snapper -c root create-config /
```

`snapper`默认会创建子卷`/.snapshots`，似乎是`opensuse`风格的，把它调整成一般的布局

```
sudo umount /.snapshots
sudo rm -r /.snapshots
sudo btrfs subvolume delete /.snapshots
sudo mkdir /.snapshots
sudo mount -o subvol=/ /dev/nvme0n1p1 /mnt
sudo btrfs subvolume create /mnt/@snapshots
```

可以对其他子卷做同样的操作，比如对`/opt (@opt)`,`/root (@root)`。

对用户目录，首先有`@home`子卷挂载`/home`，然后用

```sh
useradd

Usage: useradd [options] LOGIN
       useradd -D
       useradd -D [options]
Options:
      --btrfs-subvolume-home    use BTRFS subvolume for home directory
```

选项创建`@home/user_name`子卷挂载用户家目录`/home/user_name`。这样再用上述方法可以对特定用户的家目录做快照。子卷会像

### 从快照中恢复

对snapper创建的`/`的快照，复杂的手动方法，要从live
USB开始，就是安装`arch`时的live系统

```sh
sudo mount /dev/nvme0n1p6 /mnt
cd /mnt
sudo mv /mnt/@ /mnt/@.broken
sudo btrfs subvolume snapshot /mnt/@snapshots/{number}/snapshot /mnt/@
sudo btrfs subvolume delete /mnt/@.broken
```

不进入live系统的方法，看上去很危险，最好回滚后立刻重启

```sh
sudo mount -o subvol=/ /dev/nvme0n1p1 /mnt
sudo btrfs subvolume snapshot /mnt/@ /mnt/@bad
sudo btrfs subvolume delete /mnt/@
sudo btrfs subvolume snapshot /mnt/@snapshots/要恢复的快照号/snapshot /mnt/@
```

可以用脚本完成回滚（来自[这里](https://blog.kaaass.net/archives/1748)）

```sh
#!/bin/sh
set -e
if [[ x"$1" == x ]]; then
  echo "No snapshot number given." 1>&2
  echo "Usage: rollback [snapshot to rollback]" 1>&2
  exit 1
fi
root_dev=`findmnt -n -o SOURCE / | sed 's/\[.*\]//g'`
root_subvol=`findmnt -n -o SOURCE / | sed 's/.*\[\(.*\)\].*/\1/'`
echo ">= Rollback to #$1 on device $root_dev"
# create snapshot before
sudo snapper create --read-only --type single -d "Before rollback to #$1" --userdata important=yes
sudo mount -o subvol=/ $root_dev /mnt
# check enviornment
if [[ x"$root_subvol" == x/@ ]]; then
  echo "Warning: Not run in a snapshot, a subvolume @old will be created. You should consider remove it after reboot." 1>&2
  if [[ -d /mnt/@old ]]; then
    echo "Found last @old, remove it."
    sudo btrfs subvolume delete /mnt/@old >/dev/null
  fi
  sudo mv /mnt/@ /mnt/@old
else
  sudo btrfs subvolume delete /mnt/@ >/dev/null
fi
sudo btrfs subvolume snapshot /mnt/@snapshots/$1/snapshot /mnt/@ >/dev/null
sudo umount /mnt
```

其他子卷的`snapper`快照，直接用

```sh
snapper -c [config] rollback [number]
```

对于`timeshift`，直接gui操作就行。

回滚后最好立刻重启。

## 技巧

```
btrfs filesystem du -s /path/to/subvolume
```

## Reference

1.  [ArchWiki-Btrfs](https://wiki.archlinux.org/title/Btrfs)
2.  [Working with Btrfs – General Concepts](https://fedoramagazine.org/working-with-btrfs-general-concepts/)
3.  [Working with Btrfs – Subvolumes](https://fedoramagazine.org/working-with-btrfs-subvolumes/)
4.  [Working with Btrfs – Snapshots](https://fedoramagazine.org/working-with-btrfs-snapshots/)
5.  [ArchWiki-Snapper](https://wiki.archlinux.org/title/snapper)
6.  [BTRFS snapshots and system rollbacks on Arch Linux](https://www.dwarmstrong.org/btrfs-snapshots-rollbacks/)
