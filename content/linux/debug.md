---
title: 修 archlinux 记录
date: 2026-09-08
summary:
tags:
categories:
topics:
series:
genres:
draft: true
status:
---

# 修 archlinux 记录

## wechat

这几天 wechat 又抽风了，和fcitx配合不是很好
在wayland niri上无法切换中文输入法，弹窗飞来飞去
在X11上还正常。但隔了一天直接无法启动了

目测是可能因为切换到原生wayland了，作为QT应用，默认 QT_QPA_PLATFORM=wayland。但是在X11上需要QT_QPA_PLATFORM=xcb. 可以通过新建一个 .local/share/applications/wechat-x11.desktop

Exec=env QT_QPA_PLATFORM=xcb QT_IM_MODULE=fcitx XMODIFIERS=@im=fcitx GTK_IM_MODULE=fcitx /opt/wechat/wechat %U

来处理

输入法这里很复杂，要看这个候选词的框是什么窗口（wayland或X），被niri和wayland怎么处理

需要注意一些变量： DISPLAY=:0, WAYLAND_DISPLAY=:wayland-0 等

要知道应用根据哪些环境变量启动，它们怎么感知环境变量，在哪里设置环境变量，初始化的环境变量是什么，优先关系

我的环境是 ly display manager, sx-startx 作为 X 的启动器, wayland 应该是由 niri 的配置文件管理的
另外还有 /etc/environment, /etc/zsh/zprofile /etc/zsh/zshenv /etc/profile, ~/.config/zsh/{.zshenv,.zprofile}
X 上 dwm 有一个 .config/dwm/autostart.sh, niri 则是 .config/niri/config.kdl

关于 xwayland, niri 用 xwayland-satellite. 还要注意有些要用 xhost

> DISPLAY：xwayland-satellite 会创建并管理一个新的 DISPLAY 编号（如 :1）供 X11 客户端使用。这解释了为什么在 Wayland 下 X11 应用也需要 DISPLAY 变量。

> 默认 DISPLAY 不同：传统 startx 默认分配 :0，而 sx 的第一个 DISPLAY 是 :1，并与 TTY 编号绑定（如 tty1 为 :1）

## fcitx

### x11

X11 下，输入法可使用 GTK 和 Qt 的输入法模块通过 D-Bus 与输入法通信。某些非 GTK 和 Qt 的应用程序（或者框架）也实现了 fcitx 或者 ibus 的 D-Bus 通信协议。其它程序可以通过 X11 的 XIM 协议进行通信。

为了告诉程序使用 fcitx5 输入法，需要设置相应的环境变量。编辑 /etc/environment 并添加以下几行，然后重新登录

GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus

### wayland

为了支持运行于Xwayland的GTK软件，在~/.config/gtk-3.0/settings.ini中添加：

[Settings]
gtk-im-module = fcitx

警告：
请勿设置GTK_IM_MODULE环境变量。

为了支持Qt5和运行于Xwayland的Qt软件，设置以下环境变量：

QT_IM_MODULES=wayland;fcitx
QT_IM_MODULE=fcitx

为了支持运行于Xwayland 的其他软件，设置环境变量XMODIFIERS=@im=fcitx。
