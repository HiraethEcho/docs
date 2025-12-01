---
title: 各种同步方法
date: 2024-12-21
tags:
  - tools
status: draft
draft: true
categories: handbook
---

# 各种云同步方法

## 云盘

- koofr
- onedrive
- ali

## 文件同步方法

- webdav 协议
- rclone 命令行工具

rclone --filter-from list.txt localdir remote:target

without

```txt
- **.git/**
```

not ignore `.git` in localdir

without

```txt
- *.git
```

not ignore `.git` in subdir in localdir

have absolute no idea why

```txt
- *.git
- **.git/**
```

can ignore all `.git` folders

## 管理

~~alist~~ openlist

在`archlinux`中，`data`等配置文件的位置有些刁钻。注意附带的systemd服务

```
[Unit]
Description=openlist
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=openlist
WorkingDirectory=/var/lib/openlist
ExecStart=/usr/bin/openlist server --data /var/lib/openlist
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

说明要在`/var/lib/openlist`目录下使用`openlist`程序配置和初始化等，或者加上`openlist --data /var/lib/openlist`选项
