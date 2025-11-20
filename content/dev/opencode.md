---
title: 探索ai时遇到的混乱
date: 2025-07-30
summary: 玩llm，agent，mcp，感到一些混乱
tags:
  - ai
categories: log
---

# 一团乱麻的ai

## cli

在搞一些cli下的agent，发现一个开源的opencode，以及它在nvim的插件。结果

- 两个不同的插件都叫`opencode.nvim`，于是lazyvim对同名插件支持有问题。用`name`修改其中一个为`opencode-nvim`后，似乎`require("opencode.nvim")`会出问题。disable掉其中一个都不行，必须去掉一个插件，并且手动删除插件，重新下载才行。
- 发现其中一个`opencode.nvim`是用`goose.nvim`改的，`goose`是另一个cli的agent，但是aur里`goose`包是个`Database migration tool written in Go`，我要的`goose`包名是`codename-goose`
- 再之后发现还有另一个`opencode-ai/opencode`，一开始找的有插件的是`sst/opencode`，以及7/30突然前者`archive`停止开发了。
- 吃了个瓜发现一开始是`opencode-ai/opencode`，后来其中一批人想卖给公司，虽然承诺继续开源；另一批人不愿意，跑出去做了`sst/opencode`。再之后前者又搞了个新的`crush`

## llm

我的llm来源，有个本地拉跨的ollama，然后deepseek充值了巨资五元人民币。还有`openroute`一堆免费模型，但是似乎额度在变少。`通义灵码`似乎免费开放了某个版本的`qwen3-coder`，刚出来的时候有人在吹，过两天又有人喷。我最好用的是github-copilot，教育优惠爽啊

## ide

nvim上好几个，都用起来有点费劲。

vscode上有官方的copilot，然后有好用的cline。还有个ali的通义灵码，在我家目录拉屎，有点烦。

## mcp

配置有两三种格式，而且还不太方便放在一个位置，有点烦

uvx和npx都产生好多cache类的东西，赛博洁癖感到痛苦

## app

chatbox 和 cherrystudio，都是home到处拉屎，烦人

provider也不好配，mcp也不好配
