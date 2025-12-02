---
title: obsidian使用与插件
date: 2023-11-11
tags:
  - obsidian
  - pkm
  - tools
categories: handbook
---

# obsidian

## plugins

常用：

- remotely save (using webdav to synchronize files)
- obsidian web clipper 浏览器插件

好用：

- git
- digital garden
- share as gist

没必要：

- gistr: 和`share as gist`比不能添加description
- find orphaned files and broken links

about vim:

- vim toggle
- vim input method switch
- vimrc
- edit in neovim

展示：

- [导入和展示豆瓣](/software/obsidian_douban)
- dataview (display files by metadata)
- kanban (plan things, especially something with many stages)不习惯

## theme

- nord
- minimal

## sync

因为[用obsidian管理blogs](/dev/blogs)，所以重点是同步obsidian仓库。云端是github和webdav（koofr），在两台电脑（都是win/linux双系统）和手机、ipad，总计6个仓库同步。要注意的是

- 手机和ipad很难用git
- 设置需要忽略的文件
- 考虑用插件还是外部命令
- 各种方式下的冲突文件处理

## editor

虽然用obsidian管理markdown文件，但编辑器可以用其他的。比如[neovim](/software/nvim-config).
