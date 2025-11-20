---
title: Display Manager
date: 2025-06-05
summary: 在Linux系统中配置显示管理器（Display Manager）的指南，包括无显示管理器和sddm的自动登录设置。
tags:
  - linux
topics: 使用linux
---

# Display Manager

## No DM

skip username: `/etc/systemd/system/getty@tty1.service.d/skip-username.conf`

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty -o '-p -- hiraeth' --noclear --skip-login - $TERM
```

## sddm

### autologin

默认session `/etc/sddm.conf`

```
[Theme]
Current=chili

[Autologin]
User=hiraeth
Session=dwm.desktop
```

免密码
You must then also be part of the nopasswdlogin group to be able to login interactively without entering your password:

```sh
groupadd -r nopasswdlogin
gpasswd -a username nopasswdlogin
```

SDDM goes through PAM so you must configure the SDDM configuration of PAM: `/etc/pam.d/sddm`

```
#%PAM-1.0
auth        sufficient  pam_succeed_if.so user ingroup nopasswdlogin
auth        include     system-login
-auth       optional    pam_gnome_keyring.so
-auth       optional    pam_kwallet5.so

account     include     system-login

password    include     system-login
-password   optional    pam_gnome_keyring.so    use_authtok

session     optional    pam_keyinit.so          force revoke
session     include     system-login
-session    optional    pam_gnome_keyring.so    auto_start
-session    optional    pam_kwallet5.so         auto_start
```

`/etc/pam.d/sddm-autologin`

```
#%PAM-1.0
auth        required    pam_env.so
auth        required    pam_faillock.so preauth
auth        required    pam_shells.so
auth        required    pam_nologin.so
auth        required    pam_permit.so
-auth       optional    pam_gnome_keyring.so
-auth       optional    pam_kwallet5.so
account     include     system-local-login
password    include     system-local-login
session     include     system-local-login
-session    optional    pam_gnome_keyring.so auto_start
-session    optional    pam_kwallet5.so auto_start
```

## files

```/etc/zsh/zshenv
export ERRFILE="$XDG_CACHE_HOME/X11/xsession-errors"
```
