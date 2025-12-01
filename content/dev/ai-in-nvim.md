---
title: AI in neovim
date: 2025-07-20
summary: using nvim to vibe coding
tags:
  - nvim
  - ai
  - geek
draft: true
---

# AI in neovim

## overview

| plugin        | agent功能      | chat功能 | completion | model   | vscode        |
| ------------- | -------------- | -------- | ---------- | ------- | ------------- |
| CodeCanpanion |                | 支持     |            |         | 类似`cline`   |
| avante        |                | 支持     |            |         | 类似`cline`   |
| Copilot chat  |                |          |            | copilot | 类似`copilot` |
| opencode.nvim | 和opencode一致 |          |            |         |               |

some util plugin:

- blink.cmp
- copilot.lua
- mcp-hub

## utils

这些是在`nvim`中使用大模型或者agent功能的基础组件。

### copilot.lua

Github提供的`copilot`的官方插件，以及它的纯lua版本

通过这个插件和Github的`copilot`服务器通讯，也包括授权等。

它本身还提供了`copilot pane`功能，但是不好用

### blink.cmp

nvim的补全框架，ai的补全通过`blink.cmp`呈现并选择

### mcp-hub

`mcp-hub`可以将多个mcp服务集中在一处，供其他`agent`调用

```sh
npm install mcp-hub -g
```

配置文件在`$HOME/.config/mcphub/global.json`等

nvim通过[mcphub.nvim](https://github.com/ravitemer/mcphub.nvim)与`mcp-hub`通讯

## agent

## completion

### copilot.lua

本身就提供补全，可以被`blink.cmp`使用

### minuet-ai

补全插件，可以用其他框架，也可以用nvim-0.11后的virtual text 和 原生snippet使用

### agent

## ref
