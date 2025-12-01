---
title: 用mkdocs、obsidian、zotero搭建在线文献学习笔记
date: 2024-10-15
tags:
  - pkm
  - zotero
  - obsidian
  - hack
categories: handbook
---

# 使用mkdocs、obsidian、zotero搭建在线文献学习笔记

工作流是，在zotero中阅读和标记。用zotero-better-notes插件把标记变成文本格式，并且导出到obsidian的库文件中，进一步编辑（或者用nvim编辑）。在obsidian中更方便笔记链接跳转，zotero只是文献管理和阅读。然后把笔记同步到mkdocs的git仓库（实际上是把mkdocs的文件夹软链接到obsidian库），用github pages搭建在线wiki。

## 设置

### Zotero

关键是`zotero-better-note`插件的[模板设置](/doc/software/zotero)：从注释创建笔记；将笔记插入到其他笔记；将笔记导出为markdown。

> [!important]
> 导出路径和模板中链接路径有硬编码部分，需要根据具体情况手动修改。

### obsidian

需要在设置中选择使用`markdown`风格链接而不是`wiki`风格。根目录`/`是obsidian的库文件夹。

### mkdocs

基本上就是直接使用模板。

## 路径匹配

主要需要注意的，是文件名和链接格式，即`[markdown格式的链接](这里应该写什么路径)`，使得zotero、obsidian、在线网页的跳转都正常。实际上有两个跳转，一个是到md文件的跳转，另一个是到pdf的跳转。前者是obsidian或网页内的跳转，相对简单；后者是跳转到zotero中的pdf，这个实际上只有我能用，因为涉及到zotero的数据库。有多种选择：

### `docs`文件夹作为obsidian仓库

创建仓库`username.github.io`（必须是这个格式），在线网页会是`https://<username>.github.io`，那么链接应该写成`[title](/filename)`，在网页端是`https://<username>.github.io/filename`。把`docs/`文件夹作为obsidian repo，用`.gitignore`忽略掉`docs/.obsidian`。

优点是简单，缺点文件直接在obsidian的根目录，不好管理其他内容了。

### `docs`文件夹作为obsidian仓库子文件夹

使用二级域名的github pages，即创建仓库（例如`wiki`），在线网页会是`https://<username>.github.io/wiki`，那么链接应该写成`[title](/wiki/filename)`，在网页端是`https://<username>.github.io/wiki/filename`

把`docs`变成obsidian的子文件夹，可以用软链接：

```sh
ln -s /path/to/git/repo/docs /path/to/obsidian/repo/wiki
```

或者用[一个obsidian仓库管理多个博客仓库](/doc/dev/blogs)

## 总结

> [!note]
> 似乎需要一个markdown和latex转换的脚本。其中markdown应该混写html比较好。似乎lua就可以，那么就可以做nvim插件。  
> 2025-05-21: 似乎latex转markdown+html比较好，用`<ul>` `<ol>` `<detail>`标签来处理latex环境`itemize` `enumerate` `Theorem`
