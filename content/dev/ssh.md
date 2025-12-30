---
title: ssh的使用
date: 2025-04-13
summary: ssh的简单使用
tags:
  - tools
  - linux
categories: handbook
status: draft
---

# ssh

## 内网ssh连接

一个ipv6地址的电脑，作为中间机，一个内网的电脑，记作服务机，期望其他的客户机（比如iphone）连接到内网的服务机。

### 思路

将服务机的ssh端口转发到中间机的端口，即数据从中间机的端口传送到服务机；客户机连接中间机的端口即可。中间机有ipv6地址，就可以ssh直连。

### 操作

在服务机上

```shell
ssh -R 1234:localhost:22 midle_username@ipv6
```

将服务机的22端口和中间机的1234端口连接。

在客户机上用

```shell
ssh -p 1234 server_user@ipv6
```

连接，那么数据就从客户机到中间机的1234端口，再转发到服务机的22端口

ssh 有一些选项：

- -f: Fork into background 让 SSH 在认证成功后进入后台运行，不占用当前终端。
- -N: No remote command 不执行远程命令，仅用于端口转发。
- -T: Disable pseudo-terminal allocation 不分配伪终端（PTY），适用于纯端口转发。
- -R: Remote port forwarding 建立反向隧道（远程端口转发）。

### 注意

中间机上在`/etc/ssh/sshd_config`（作为服务端的配置）设置

```conf
AllowTcpForwarding yes
GatewayPorts yes
```

在服务机 `/etc/ssh/sshd_config`（作为服务端的配置）设置监听端口，默认为22

根据系统设置防火墙

## X through ssh

to share X11 through ssh, need `Xorg` on server and client.  
on client using

```shell
ssh -X -C remote_user@remote_host
```

where `X` for X forwarding, and `-C` for compression. In this ssh session, GUI
like `gvim` should show on client.

### systemd 自动启动

create `/etc/systemd/system/ssh-tunnel.service`

```
[Unit]
Description=Autossh Persistent Reverse SSH Tunnel
After=network.target

[Service]
User=username
ExecStart=/usr/bin/autossh -M 20000 -NR 1234:localhost:22 midle_username@ipv6
Restart=always
RestartSec=10
KillMode=process
Environment="AUTOSSH_GATETIME=0"
Environment="AUTOSSH_POLL=30"

[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now ssh-tunnel
```

## debug

在debian上，防火墙

```sh
sudo ufw allow 22/tcp 1234/tcp

sudo ufw status

1234/tcp                   ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
1234/tcp (v6)              ALLOW       Anywhere (v6)
22/tcp (v6)                ALLOW       Anywhere (v6)
```

在`/etc/ssh/sshd_config`中

```config
Port 22
# Port 2222 # custom port
AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
PubkeyAuthentication yes
AuthorizedKeysFile  .ssh/authorized_keys

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#AllowAgentForwarding yes
AllowTcpForwarding yes
GatewayPorts yes
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no
```

检查网络，包含这样的

```sh
ss -tnlp

State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
LISTEN 0      128          0.0.0.0:22         0.0.0.0:*
LISTEN 0      1          127.0.0.1:20001      0.0.0.0:*    users:(("autossh",pid=15104,fd=3))
LISTEN 0      128        127.0.0.1:20000      0.0.0.0:*    users:(("ssh",pid=21175,fd=5))
LISTEN 0      128             [::]:22            [::]:*
LISTEN 0      128            [::1]:20000         [::]:*    users:(("ssh",pid=21175,fd=4))
```

检查端口占用：

```
lsof -i :22
lsof -i :1234
```

修改配置后记得重启各种服务，例如

```sh
sudo systemctl restart sshd
sudo systemctl daemon-reload
sudo systemctl enable --now ssh-tunnel
sudo systemctl restart ssh-tunnel
sudo ufw reload
```
