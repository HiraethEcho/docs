---
title: 安装后的建议配置
date: 2024-11-18
summary: archlinux安装后的建议配置，新建用户，软件等
tags:
  - archlinux
  - geek
categories: handbook
---

# 安装后的建议配置

> [!important] [官方推荐](https://wiki.archlinux.org/title/General_recommendations)

## add user

添加用户并设置密码

```sh
useradd -m USERNAME
passwd USERNAME
```

可以将用户家目录设置成btrfs的子卷，这样可以更好的管理快照。

```
useradd -m --btrfs-subvolume-home USERNAME
```

添加用户到wheel组：

```
usermod -aG wheel USERNAME
```

或者在创建用户时用

```
useradd -m -G wheel USERNAME
```

让`wheel`组的用户可以使用`sudo`，需要编辑`/etc/sudoers`文件，可以使用`visudo`命令，如果环境变量没有指定默认编辑器，会提示选择，选择一个之后会进入文件编辑界面。或者使用

```
EDITOR=nvim visudo
```

取消下面的注释

```
# %wheel ALL=(ALL:ALL)
```

## Package managerment

换源：

```
# /etc/pacman.d/mirrorlist
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

添加`archlinuxcn`仓库。在 `/etc/pacman.conf`中添加

```text
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

更新keyring

```
pacman -Sy archlinux-keyring
pacman -Sy archlinuxcn-keyring
```

然后建议滚到最新

```
pacman -Syyu
```

在`archlinuxcn`中有aur-helper，比如

```
pacman -S paru
```

clean pacman:

```sh
paccache -r # 清理缓存,仅包含最近的三个版本
paccache -rk1 # 清理缓存,仅包含最近的1个版本
pacman -Sc # 清理未安装软件包
pacman -Scc # 清理缓存中所有内容
sudo pacman -Rcsn $(pacman -Qdtq -)
journalctl --vacuum-size=50M #限制日志
```

## 基础功能

### 网络

启动服务，`iwd`是连接网络的，自带域名解析。需要

```
# /etc/iwd/main.conf
[General]
EnableNetworkConfiguration=true
```

也可以用`dhcpcd`解析

```
systemctl enable dhcpcd
```

tui: `impala`

或者用`networkmanager`

Using iwd as backend of NetworkManager, `/etc/NetworkManager/NetworkManager.conf`

```
[device]
wifi.backend=iwd
```

then

```sh
systemctl mask wpa_supplicant
systemctl enable iwd
```

### sound

- ALSA: is a set of built-in Linux kernel modules.
- PulseAudio: is a general purpose sound server intended to run as a middleware between your applications and your hardware devices, either using ALSA or OSS.
- pamixer: cli mixer of PulseAudio
- pavucontrol: gui of PulseAudio

```
sudo pacman -S alsa-ultis pulseaudio pavucontrol
pulseaudio --check
pulseaudio -D
```

### light

`video`组的用户可以控制亮度

```sh
sudo pacman -S acpilight
sudo gpasswd video -a _username_ # 或者
sudo usermod -aG video _username_
```

### bluetooth

蓝牙耳机需要`pulseaudio-bluetooth`和`bluez-utils`。

```
sudo systemctl enable bluetooth.service --now
```

tui: `bluetui`

### input

在`X11`中需要安装`xf86-input-libinput`等

To find id of touchpad

```
xinput list | grep -i "Touchpad" | awk '{print $6}' | sed 's/[^0-9]//g'
```

keys:

```sh
setxkbmap -option ctrl:nocaps &
xcape -e 'Control_L=Return' &
xcape -e 'Alt_L=Escape' &
```

### battery

```
upower -e

/org/freedesktop/UPower/devices/line_power_ACAD
/org/freedesktop/UPower/devices/battery_BAT1
/org/freedesktop/UPower/devices/DisplayDevice
```

check battery `sudo upower -i /org/freedesktop/UPower/devices/battery_BAT1 `

### driver

for amd integrated, open source driver:

```
sudo pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon
```

## Desktop Environment

[archwiki文档](https://wiki.archlinux.org/title/Desktop_environment)

### 成品桌面环境

- [GNOME](https://wiki.archlinux.org/title/GNOME)
- [KDE](https://wiki.archlinux.org/title/KDE)
- [Xfce](https://wiki.archlinux.org/title/Xfce)
- [LXQt](https://wiki.archlinux.org/title/LXQt)

### components

At least one need

- window manager or [compositor](https://wiki.archlinux.org/title/Wayland#Compositors "Wayland")
- [terminal emulator](https://wiki.archlinux.org/title/Terminal_emulator "Terminal emulator")

Usually also need:

- [Application launcher](https://wiki.archlinux.org/title/List_of_applications/Other#Application_launchers "List of applications/Other")
- [text editor](https://wiki.archlinux.org/title/Text_editor "Text editor")).
- [file manager](https://wiki.archlinux.org/title/List_of_applications/Utilities#File_managers "List of applications/Utilities")
- [taskbar](https://wiki.archlinux.org/title/Taskbar "Taskbar")
- [Polkit authentication agent](https://wiki.archlinux.org/title/Polkit#Authentication_agents "Polkit")
- [Display manager](https://wiki.archlinux.org/title/Display_manager#List_of_display_managers "Display manager")
- [Backlight control](https://wiki.archlinux.org/title/Backlight#Backlight_utilities "Backlight")

Other components usually provided by desktop environments are:

- [Audio control](https://wiki.archlinux.org/title/List_of_applications/Multimedia#Volume_control "List of applications/Multimedia")
- [Compositor](https://wiki.archlinux.org/title/Compositor "Compositor")
- [Default applications](https://wiki.archlinux.org/title/XDG_MIME_Applications#mimeapps.list "XDG MIME Applications")
- [Logout dialogue](https://wiki.archlinux.org/title/List_of_applications/Other#Logout_UI "List of applications/Other")
- [Media control](https://wiki.archlinux.org/title/MPRIS#Control_utilities "MPRIS")
- [Notification daemon](https://wiki.archlinux.org/title/Desktop_notifications#Standalone "Desktop notifications")
- [Power management](https://wiki.archlinux.org/title/Power_management "Power management")
- [Screen capture](https://wiki.archlinux.org/title/Screen_capture "Screen capture")
- [Screen locker](https://wiki.archlinux.org/title/List_of_applications/Security#Screen_lockers "List of applications/Security")
- [Screen temperature](https://wiki.archlinux.org/title/Backlight#Color_correction "Backlight")
- [Wallpaper setter](https://wiki.archlinux.org/title/List_of_applications/Other#Wallpaper_setters "List of applications/Other")

### utils

`filedialog` 是其他应用需要选择文件时的弹窗。通常需要一些`xdg-desktop-portal`，不同的包有不同的[具体功能](https://wiki.archlinux.org/title/XDG_Desktop_Portal)。

### X11

`Xlibre` `wayback`

### wayland

gnome, kde

niri, hyprland, sway, river, wayfire,

## Applications

### shell and terminal

### steam:

```
id -u
id -g
```

mount:

```
sudo mkdir /media/gamedisk
```

```
# /etc/fstab
UUID=38CE9483CE943AD8 /media/gamedisk lowntfs-3g uid=1000,gid=1000,rw,user,exec,umask=000 0 0
```

debug:

```
mkdir -p ~/.steam/steam/steamapps/compatdata
ln -s ~/.steam/steam/steamapps/compatdata /media/gamedisk/Steam/steamapps/
```

### wechat

微信使用fcitx输入法会有一些问题，似乎是不能正常读取环境变量。修改`/usr/share/applications/wechat.desktop`来设定变量。有可能在更新后被覆盖，或许可以放在`~/.local/share/applications/wechat.desktop`中。

```
[Desktop Entry]
Name=wechat
Name[zh_CN]=微信
Exec=env XMODIFIERS="@im=fcitx" GTK_IM_MODULE="fcitx" QT_IM_MODULE="fcitx" /usr/bin/wechat %U
StartupNotify=true
Terminal=false
Icon=/opt/wechat/icons/wechat.png
Type=Application
Categories=Utility;
Comment=Wechat Desktop
Comment[zh_CN]=微信桌面版

```
