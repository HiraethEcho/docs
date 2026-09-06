---
title: caldav协议
date: 2025-06-30
summary: caldav是一个基于webdav的日历协议，可以用来同步日历数据。它是一个开放的标准，允许不同的日历应用程序和服务之间进行互操作。
tags:
  - caldav
  - geek
categories: 规范协议技术
status: wip
---

# Caldav

## caldav

caldav是一个基于webdav的日历协议，可以用来同步日历数据。它是一个开放的标准，允许不同的日历应用程序和服务之间进行互操作。

包括calendar、todo、event等。

## manage

### todoman

### khal

### apple

原生日历和提醒事项应用程序可以通过CalDAV协议与服务器进行同步。

## host a caldav

### radicale

#### Install and configurations

ref: [radicale](https://radicale.org/v3.html):

Install radicale: 建议用包管理器安装，可以帮助解决依赖，以及创建`radicale`用户、创建文件夹之类的杂事。比如

```sh
sudo dnf install radicale
```

configurations:

```
# /etc/radicale/config

[server]
hosts = [::]:5242, 0.0.0.0:5242 # ipv6, ipv4的host，从任何ip地址都可以访问
# hosts = localhost:5242 # 只允许本地地址访问
[auth]
type = htpasswd
htpasswd_encryption = autodetect
htpasswd_filename = /etc/radicale/users
[storage]
filesystem_folder = /var/lib/radicale/collections

```

set a user and passwd:

```shell
htpasswd -5 -c /path/to/users user1
```

Start as a systemd service:

```
# /etc/systemd/system/radicale.service
[Unit]
Description=A simple CalDAV (calendar) and CardDAV (contact) server
After=network.target
Requires=network.target

[Service]
ExecStart=/usr/bin/radicale --config /etc/radicale/confif
Restart=on-failure
User=radicale
# Deny other users access to the calendar data
UMask=0027
# Optional security settings
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
PrivateDevices=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
NoNewPrivileges=true
ReadWritePaths=/var/lib/radicale/ /var/cache/radicale/

[Install]
WantedBy=multi-user.target
```

这样在`http://localhost:5242/`就可以访问web页面，可以创建collection等。

#### host

用`cloudflare`的zero trust - network - tunnel来将radicale服务暴露到公网。记得选https。

### baikal

## connect a caldav

### from linux

vdirsync

### from apple to caldav server

### To apple caldav

ref:

- https://cyp0633.icu/post/icloud-calendar-sync/
- https://www.aurinko.io/blog/caldav-apple-calendar-integration/

云上贵州服务器

```
https://<server-id>-caldav.icloud.com.cn/<user-id>/calendars/<calendar-id>/
```

在网页版icloud的日历界面打开开发者工具，然后找`Network`里的`Get`请求，过滤`XHR`，然后找类似 `https://<server-id>-calendarws.icloud.com.cn/ca/eventdetail/<calendar-id>/<event-id>?clientBuildNumber=<some-random-text>&clientId=<some-random-text>&clientMasteringNumber=<some-random-text>&clientVersion=<some-random-text>&dsid=<user-id>&lang=<some-random-text>&requestID=<some-random-text>&usertz=<some-random-text>`的内容。

用 [vdirsync](https://vdirsyncer.readthedocs.io/) 将`icloud`的日历同步到本地，以`.ics`格式存储。

```conf
[storage cal]
type = "caldav"
url = "https://caldav.icloud.com/"
username = "..."
password = "..."

[storage card]
type = "carddav"
url = "https://contacts.icloud.com/"
username = "..."
password = "..."
```

这里`username`就是用户名，而`password`是双重认证后再生成的app密码。服务器的话中国应该要`url = "https://caldav.icloud.com.cn/"`

但是有两个问题：

- 不支持`VTODO`格式，apple的提醒事项似乎不是这个协议。
- `icloud`如果有多个日历的话，标识id很长，难以记忆或使用，例如`87BBBA05-934B-4D5A-BB27-771A9CD413A2`。或者其中一个会被记为`home`，但是不知道什么规则。例如同步到`.local/share/calendars`后
  ```
  ~/.local/share/calendars
  ├── 87BBBA05-934B-4D5A-BB27-771A9CD413A2
  └── home
  ```
