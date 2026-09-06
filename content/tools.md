---
title: 我的工具
date: 2025-12-02
summary: 我使用的工具
menus: main
HideInFileTree: true
---

# 我的工具

## On My Archlinux

I use arch, btw

### DE

X11:

- dwm
- dwmblocks
- st
- dmenu
- tabbed

I have those intalled as a bundle called `suckmore` (because it sucks a little bit more than origin suckless) though a PKGBUILD. it pull my repo [suckless](https://github.com/hiraethecho/suckless), complie them, and install into `/usr/bin`.  
My dwm inits `~/.config/dwm/autostart.sh` at startup, in which i start `dwmblock`, `xbindkeys`, `dunst`, `fcitx5`, `feh` for wallpaper, `xhidecusor` so hide cursor while typing. [^1]

[^1]: it works on my mechine

Wayland:

- niri
- noctalia
- foot
- niri-ocr, niri-pimg

I use following softwares as componets of both DE:

- display manager: ly use sx-starx for X11
- terminal: kitty
- network: iwd, impala (tui) also i use systemd to solve dns directly
- sound: pipewire, wiremix (tui) and pavucontrol-gtk3 (gui)
- light: brightnessctl, ddcutil for external monitor
- bluetooth: bluez, bluetui
- app lanucher: rofi
- file manger: yazi (tui), pcmanfm (gui)
- editor: neovim
- browser: zen (for now)
- notifier: dunst

### softwares

- rss: [markerss](https://github.com/hiraethecho/markerss) vibe coding app
- document: only office
- pdf: sioyek, zathura
- obsidian
- btop
- piclist
- zotero
- marktext
- fcitx
- flameshot
- gparted
- rclone, koofr

## Dev

- herdr
- pi
- opencode
- neovim
