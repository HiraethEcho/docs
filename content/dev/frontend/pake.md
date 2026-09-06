---
title: pake打包web应用
date: 2025-06-08
tags:
  - linux
  - archlinux
  - tech
  - tools
  - geek
---

# pake打包web应用

[pake](https://github.com/tw93/Pake)可以将web转化为桌面应用，比`electron`更轻量。

## 安装pake

用户级安装：

```
npm -g install pake-cli
```

需要`rust`等工具链

## 打包web应用

例如

```
pake chat.deepseek.com --name deepseek --width 1080 --height 1920 --show-system-tray --icon 512x512-RGBA.png
```

要注意的是分辨率设置，以及icon格式为512x512的png，似乎必须是`RGBA`

会得到`deepseek.deb`，内部结构为

```deb
control.tar.gz
data.tar.gz
debian-binary
```

主要程序在`data.tar.gz`里面，用`tar -xvf data.tar.gz`得到

```usr
 usr
├──  bin
│   └──  pake
├──  lib
│   └──  deepseek
│       └──  png
│           └──  deepseek_512.png
└──  share
    ├──  applications
    │   └──  deepseek.desktop
    └──  icons
        └──  hicolor
            └──  512x512
                └──  apps
                    └──  pake.png
```

其中`usr/bin/pake`是单二进制文件，可以直接运行`./usr/bin/pake`。其他的是快捷方式`usr/share/application/deepseek.desktop`和图标等。

## On Arch

在`archlinux`上可以将其做成`.zst`格式的包，用`pacman`管理。

把下面内容写入`PKGBUILD`文件

```
# Maintainer: hiraethecho <wyz2016zxc at outlook dot com>
pkgname="${_appname}-pake"
pkgver=1.0.0
pkgrel=1
pkgdesc="${_appname} pake"
arch=('x86_64')
license=('MIT')
conflicts=("${_appname}-pake")
depends=(
    'gtk3'
    'webkit2gtk-4.1'
)
source=(
  ${_appname}.deb
)
sha256sums=('SKIP')
prepare() {
    bsdtar -xf "${srcdir}/data."*
    sed -e "
        s/Exec=pake/Exec=${_appname}/g
        s|Icon=pake|Icon=${_appname}|g
    " -i "${srcdir}/usr/share/applications/${_appname}.desktop"
    mv ${srcdir}/usr/share/icons/hicolor/*/apps/pake.png ${srcdir}/usr/share/icons/pake.png
}
package() {
    install -Dm755 "${srcdir}/usr/bin/pake" "${pkgdir}/usr/bin/${_appname}"
    install -Dm644 "${srcdir}/usr/share/applications/${_appname}.desktop" "${pkgdir}/usr/share/applications/${_appname}.desktop"
    install -Dm644 "${srcdir}/usr/share/icons/pake.png" "${pkgdir}/usr/share/pixmaps/${_appname}.png"
}
```

然后执行这个shell脚本，注意`-i` `-w` `-n`选项

```pake2zst
#!/usr/bin/bash
# generate a zst from pake
url=""
icon=""
name=""
while getopts ":n:i:w:" opt; do
  case $opt in
  n) name="$OPTARG" ;;
  i) icon="$OPTARG" ;;
  w) url="$OPTARG" ;;
  \?)
    echo "无效选项: -$OPTARG" >&2
    ;;
  esac
done

echo -e "$url $icon $name"
# pake $url --icon $icon --height=1080 --width=1920 --show-system-tray --name $appname
sed -i "/pkgdesc/a\\url=${url}" PKGBUILD
sed -i "/pkgname/i\\_appname=${name}" PKGBUILD
makepkg
```

这样就会生成一个`.zst`包，用`pacman -U name-pake-1.0.0-1-x86_64.pkg.tar.zst`即可。
