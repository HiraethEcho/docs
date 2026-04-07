---
title: 基于typst的SSG
date: 2026-03-17
summary: 用markdown写作，自由使用typst语法，生成静态网站。
tags:
categories:
topics:
series:
status: idea
---

# 基于typst的SSG

目前是一个计划，希望今年（2026）能完成。上一次写了一个hugo主题，现在写SSG。

## 计划

### 动机

想要发布数学学习笔记，所以需要支持数学公式。通常用latex来写，但是网页端用的katex和mathjax不喜欢。而且想顺便用一用typst。

理想的工作流是：

- zotero读论文，
- 导出划线和其他笔记到markdown，
- 优化markdown笔记（可以反向同步到zotero），把unicode字符之类的转换成typst或latex
- 用obsidian管理笔记
- 用SSG生成网站并发布

### 功能

兼容markdown typst latex html语法。

### 实现

用rust。但bin文件。调用typst，可选依赖。  
学习git的开发，先做mvp。

### 细节

- mathml显示公式
- cite label ref的实现
- 边栏实现footnote（吗？）
- wikilink？

## Log

- 2026-03-17: 先调查一下，创建这个文档。
- 2026-03-18: 发现typst生成html暂时是用svg，还在开发mathml的部分。先学其他的。

## ref

- other typst ssg
  - tufted
  - typstify
  - tylant
  - tola-ssg (svg for math equation)
- mkws A C software, using sh as template language
- ar5iv: latexml
- typst tools
  - typlite transfer typst into markdown
  - mitex latex support for typst
  - wypst
  - mathyml typst package
  - texmath
