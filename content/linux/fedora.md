---
title: Fedora
date: 2026-09-12
summary:
tags:
categories:
topics:
series:
genres:
draft: true
status:
---

# Fedora

被 archlinux 滚累了，选择试一试 fedora

## minimal

用 everything 版本的 iso （有点抽象，恰恰是这个镜像里什么都没有），实际上是从网络安装

装完发现却 AX210 的网卡驱动，还需要手动下载、复制进 /lib/firmware/

## workstation

没力气折腾了，还是当个正常人，选择了 gnome 版的

### net

网络还是费劲，似乎默认用了虚拟网卡地址。但是我的校园网是记录网卡物理地址开通的，所以要手动修改

```
nmcli connection modify "CampusNet" wifi.cloned-mac-address permanent
```
