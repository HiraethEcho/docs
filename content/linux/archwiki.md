---
title: ArchWiki摘抄
date: 2025-01-03
summary: 从archwiki找的一些有用的东西
tags:
  - archlinux
categories: handbook
---

# ArchWiki摘抄

## font

find the font that contains chinese

```sh
fc-list -f '%{file}\n' :lang=zh
```

check monospace

```sh
fc-match monospace
```

set fallback fonts, edit `$XDG_CONFIG_HOME/fontconfig/fonts.conf`:

```text
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<alias>
   <family>serif</family>
   <prefer>
     <family>LXGW Wenkai mono</family>
   </prefer>
 </alias>
<alias>
   <family>monospace</family>
   <prefer>
     <family>CodeNewRoman Nerd Font</family>
     <family>LXGW Wenkai mono</family>
   </prefer>
 </alias>
</fontconfig>

```

## xorg

> [!note]
> 注意：Arch 提供了位于 `/usr/share/X11/xorg.conf.d` 的默认配置文件。通常情况下，用户无需进行额外的配置与修改即可正常使用。

## others tricks

### short cuts

[kernel sysrq](<https://wiki.archlinux.org/title/Keyboard_shortcuts#Kernel_(SysRq)>)
