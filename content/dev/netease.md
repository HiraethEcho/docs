---
title: 网页端下载网易云音乐的音乐
date: 2025-04-29
summary: 通过浏览器的审查元素功能下载网易云音乐的音乐
tags:
  - web
  - hack
categories: handbook
---

# 从网页端下载音乐

打开网易云音乐，然后打开`页面审查元素`

- firefox系一般在`setting`-`more tools`-`webdeveloper tools`， 快捷键是<kbd>ctrl</kbd>+<kbd>shift</kbd>+<kbd>I</kbd>
- chrome系忘记了， 快捷键似乎是<kbd>F11</kbd>或<kbd>F12</kbd>

在新页面里选择`Network`，可以在`Filter`里写m4a。然后点击播放，会有新的请求。右键网址后用`open in new tab`，就可以在新页面打开音频。再右键保存音频即可。

> [!note]
> ~~或者直接`Save reponse as`来下载。~~ 这样只下载`1024k`的文件，不是整个。

理论上是网页能播放就能下载。
