---
title: Filesystem Hierarchy Standard
date: 2025-06-09
summary: linux文件结构标准
tags:
  - linux
  - geek
categories: 规范协议技术
---

# Filesystem Hierarchy Standard

[standard ref](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

This standard consists of a set of requirements and guidelines for file and directory placement under UNIX-like operating systems. The guidelines are intended to support interoperability of applications, system administration tools, development tools, and scripts as well as greater uniformity of documentation for these systems.

## General setting

### under `/`

`/bin` : Essential user command binaries (for use by all users). `/sbin` : System binaries. stored in /sbin, /usr/sbin, and /usr/local/sbin.
`/lib` : Essential shared libraries and kernel modules `lib32` and `lib64`: Alternate format essential shared libraries (optional)
`/etc` : Host-specific system configuration. A "configuration file" is a local file used to control the operation of a program; it must be static and cannot be an executable binary.

`/media` : Mount point for removable media
`/mnt` : Mount point for a temporarily mounted filesystem
`/opt` : Add-on application software packages
`/run` : Run-time variable data
`/srv` : Data for services provided by this system
`/tmp` : Temporary files

### `/usr`

```
/
└── usr
    ├── bin
    ├── include
    ├── lib
    ├── sbin
    ├── share
    └── src
```

`/usr` is the second major section of the filesystem. /usr is shareable, read-only data. That means that /usr should be shareable between various FHS-compliant hosts and must not be written to. Any information that is host-specific or varies with time is stored elsewhere.  
Large software packages must not use a direct subdirectory under the /usr hierarchy. (maybe `/opt`)

- `/usr/share` : Architecture-independent data. The /usr/share hierarchy is for all read-only architecture independent data files.  
  This hierarchy is intended to be shareable among all architecture platforms of a given OS; thus, for example, a site with i386, Alpha, and PPC platforms might maintain a single /usr/share directory that is centrally-mounted. Note, however, that /usr/share is generally not intended to be shared by different OSes or by different releases of the same OS.  
  Any program or package which contains or requires data that doesn't need to be modified should store that data in /usr/share (or /usr/local/share, if installed locally). It is recommended that a subdirectory be used in /usr/share for this purpose. Applications using a single file may use /usr/share/misc.  
  Game data stored in /usr/share/games must be purely static data. Any modifiable files, such as score files, game play logs, and so forth, should be placed in /var/games.
- `/usr/lib`: Includes object files and libraries. On some systems, it may also include internal binaries that are not intended to be executed directly by users or shell scripts.  
  Applications may use a single subdirectory under /usr/lib. If an application uses a subdirectory, all architecture-dependent data exclusively used by the application must be placed within that subdirectory.
- `/usr/local` : Local hierarchy (empty after main installation) The /usr/local hierarchy is for use by the system administrator when installing software locally. It needs to be safe from being overwritten when the system software is updated. It may be used for programs and data that are shareable amongst a group of hosts, but not found in /usr.

And others:

- `games`: Games and educational binaries (optional)
- `include`: Header files included by C programs. This is where all of the system's general-use include files for the C programming language should be placed.
- `libexec`: Binaries run by other programs (optional)
- `src`: Source code (optional)

### Summary

|          | shareable                      | unshareable             |
| -------- | ------------------------------ | ----------------------- |
| static   | `/usr`, `/opt`                 | `/etc`, `/boot`         |
| variable | `/var/mail`, `/var/spool/news` | `/var/run`, `/var/lock` |

- May under '/',`/usr`,`/var`

```
├── bin # binary
├── lib # library
├── include # C header
├── games   # games
├── opt # opt
├── share   # data that share with other
└── src     # sources
```

- only under `/`

```
/
├── boot
├── etc # configraturation
├── dev
├── media
├── mnt
├── proc
├── run
├── srv
├── sys
└── tmp
```

## On archlinux,

```
/
├── bin ⇒ usr/bin
├── boot
├── dev
├── etc
├── home
├── lib ⇒ usr/lib
├── lib64 ⇒ usr/lib
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin ⇒ usr/bin
├── srv
├── sys
├── tmp
├── usr
└── var
```

`lib32` and `lib64` for 32-bit and 64-bit. `sbin` binaries for super user. (On debian, only root can `reboot`.)

```
/
└── opt
    ├── waterfox
    ├── wechat-universal
    └── wemeet
```

```
/
└── usr
    └── share
        ├── alsa
        ├── applications
        ├── awk
        ├── blueman
        ├── mime
        ├── nvim
        ├── wayland
        ├── wayland-sessions
        ├── X11
        └── zsh
```

```
/
└── usr
    └── local
        ├── bin
        ├── etc
        ├── games
        ├── include
        ├── lib
        ├── man
        ├── sbin
        ├── share
        └── src
```

```
/
└── var
    ├── cache
    ├── db
    ├── empty
    ├── games
    ├── lib
    ├── local
    ├── lock ⇒ ../run/lock
    ├── log
    ├── mail ⇒ spool/mail
    ├── opt
    ├── run ⇒ ../run
    ├── spool
    └── tmp
```

### special directories

```
/
├── boot
│   ├── efi
│   ├── grub
│   ├── initramfs-linux.img
│   └── vmlinuz-linux
├── dev
│   ├── disk
│   ├── null
│   ├── random
│   └── zero
├── mnt
├── proc
│   ├── 1
│   ├── uptime
│   └── zoneinfo
├── run
│   ├── dhcpcd
│   ├── initramfs
│   ├── media # This is different
│   ├── libvirt
│   ├── systemd
│   └── xtables.lock
├── srv
│   ├── ftp
│   └── http
├── sys
│   ├── kernel
│   └── power
└── tmp
```

## homes

```
/
├── home
│   └── user
│       ├── Desktop
│       ├── Documents
│       ├── Downloads
│       ├── Music
│       ├── Pictures
│       ├── Public
│       ├── Templates
│       └── Videos
└── root
    ├── Desktop
    ├── Documents
    ├── Downloads
    ├── Music
    ├── Pictures
    ├── Public
    ├── Templates
    └── Videos

```
