---
title: niri
date: 2025-12-02
summary: wayland下paper scroll 的 wm
tags:
categories: handbook
topics: 使用linux
---

# 滚动式窗口管理器niri

长期使用dwm作为X11下的窗口管理器，一直想找一个舒服wayland下窗口管理器试试。hyprland太重了，dwl没力气再写了，river太像了，sway不习惯。  
最后发现了niri，水平滚动式WM，非常有趣好用。

## 特点

某种意义上niri是*反*窗口管理器：不管理窗口，就一排铺开。

- 最大的优点应该是窗口不会变化大小和位置。
- 我用dwm时主要用monocle布局，并且尽量放在不同tag下（dwm的tag类似于工作区），通过切换tag来切换窗口。少数需要同时查看多个窗口时才会用grid或tile布局。  
  在niri下用named workspace可以实现相同效果，并且更符合直觉，就像在一个大桌子上摊开文件一样。
- 另一个优势是niri的逻辑很适合触控板，在外出无鼠标使用笔记本的场景非常舒服。

## set-up

各种配置文档和实例已经很多了，所以只列出我选择的搭配的组合：

- niri
- dms/noctalia
- rofi
- foot

然后是我倾向的设定：

- 虽然niri推荐动态工作区，但我还是习惯named space，所以用dms和noctalia这种quick-shell系的组件作为bar（waybar用不习惯）
- 我习惯少一点快捷键，不介意稍微多按几下，所以只使用
  ```
      Super+H {
          consume-or-expel-window-left
      }
      Super+L {
          consume-or-expel-window-right
      }
      Super+J {
          move-window-down-or-to-workspace-down
      }
      Super+K {
          move-window-up-or-to-workspace-up
      }
  ```
  而且基本不用移动workspace或column的操作
- 使用rofi是因为X11下也可以用，保持一点一致性。

## dot

我的具体配置可以参考我的[dot仓库](https://github.com/hiraethecho/dotfiles)，使用stow管理。  
niri相关的文件在`wm/.config/niri`，rofi相关的配置（没有整理）在`tools/.config/rofi`和`bin/.local/bin/`
