---
title: 好用的单行命令
date: 2026-03-27
summary: awesome one-liner commands
tags:
categories:
topics:
series:
status: wip
---

# 好用的单行命令

## git

```sh
git log --graph --oneline --decorate --all
```

## nvim

use another config

```sh
NVIM_APPNAME=lazyvim nvim
```

use nvim as pager

```sh
ls --help | nvim -R -
```

## fzf

```sh
fzf --preview 'bat {}' --bind 'enter:become(nvim {})'
```

## curl something

```sh
sudo sh -c 'sed -i \"/# GitHub520 Host Start/Q\" /etc/hosts && curl https://raw.hellogithub.com/hosts >> /etc/hosts'
```

```sh
curl 'v2d.wttr.in/Beijing'
```

## archlinux

```sh
pacman -Qqe | fzf --preview-window=50%:bottom: --preview 'pacman -Qiil {}' --layout=reverse --bind 'enter:execute(pacman -Qiil {} | less)'
```

```sh
pacman -Qq | fzf --preview-window=50%:bottom: --preview 'pacman -Qiil {}' --layout=reverse --bind 'enter:execute(pacman -Qiil {} | less)'
```

```sh
pacman -Qq | fzf --bind=ctrl-d:preview-down,ctrl-u:preview-up --prompt='depends on ' --preview-window=70%:bottom: --preview 'pactree -rd2 {}'
```

```sh
pacman -Qq | fzf --bind=ctrl-d:preview-down,ctrl-u:preview-up --prompt='required by ' --preview-window=70%:bottom: --preview 'pactree -d2 {}'
```

```sh
pacman -Slq | fzf --preview 'pacman -Si {}' --layout=reverse --bind 'enter:execute(pacman -Si {} | less)'
```

## misc

shut the monitor after 1 min afk in tty

```sh
setterm --blank=1
```

du for btrfs

```sh
sudo btrfs filesystem du -s /home
```
