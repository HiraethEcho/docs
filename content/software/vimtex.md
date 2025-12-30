---
title: Write latex with neovim
date: 2025-04-27
tags:
  - latex
  - nvim
  - geek
  - tools
---

# Write latex with neovim

\(\latex\)

## vimtex

`vimtex`基本上是提供了全套的功能，包括

- 目录，标签等
- 各种环境，`insert`模式下的各种snippets，`normal`模式下的文本对象和命令
- 编译，报错
- 正向搜索（从tex定位到pdf）和反向搜索（从pdf定位到tex）
- conceal （有点影响性能）

而且帮助文档非常详尽，维护积极，而且`vim`上也能用

缺点几乎没有，除了对与某些信奉unix哲学*做一件事并做到最好*的人（比如说我）来说有些臃肿。另一个是预设置的快捷键很多，但不一定符合所有人习惯（又是我）。

## latex

首先需要一个`latex`发行版，比如`texlive`或者`miktex`。实际编译过程很复杂，为了生成目录和参考文献需要多次编译。但是`latexmk`可以帮助完成复杂的顺序。最重要的是，`latexmk`可以监听文本变化，自动重新编译，以此实现“实时”预览。

> [!tip] 实际上不是实时。会在每次保存文件时编译，所以`insert`模式下敲字时并不会更新。另一方面还取决于编译速度。

在`archlinux`上，可以安装整个`texlive`，如果觉得太臃肿也可以安装其中部分包，例如我只安装了

```
texlive-bibtexextra
texlive-binextra
texlive-langchinese
texlive-latexextra
texlive-mathscience
texlive-pstricks
texlive-xetex
```

除此之外还有`tectonic`等可用，但是难以实现实时预览。

> [!NOTE] 可以用texpresso

## lsp

`texlab`

`tex-fmt`

## compile

原则上可以直接用命令行编译。但是既然在`neovim`里，可以用插件封装的命令。

用`vimtex`或`texlab`

## inverse and reverse search

## snippets

### mathzone

为了在文本和数学公式环境中使用不同的片段，例如`CC`只在数学公式中自动展开为`\mathbb{C}`，需要对能检测数学环境。

最简单的方法是调用`vimtex`的功能。

用`python`检测。

用treesitter检测。

### engine

UltiSnip

LuaSnip
