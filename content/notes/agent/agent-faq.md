---
title: AGENT 工具 FAQ
date: 2026-10-13
summary:
tags:
categories:
topics:
series:
genres:
status:
---

# AGENT 工具 FAQ

## ReAct

Reason - Act - Observation 推理 - 行动 - 观察 范式。

一条 prompt 触发多个 turn，合并为一个 run
一个session 通常为 run 的序列

## context

模型的 context 包括

- system prompt
- messages
  - user message
  - assistant message
    - tool call and results

## SWE

 SWE = Software Engineering（软件工程）。在这个领域里特指 SWE-bench。

 SWE-bench 是什么：一个评测基准。它把真实 Python 仓库的 GitHub issue 拿出来交给 agent。agent 要自己读代码
 、定位 bug、改文件。改完后跑该仓库原有的测试，测试通过就算解决。任务来自真实项目，不是人造练习题。

 谁做的：Princeton 和 Stanford 团队。mini-swe-agent 出自同一批人。README 第 19 行写着这件事。
