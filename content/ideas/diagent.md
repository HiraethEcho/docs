---
title: DiAgent
date: 2026-10-08
summary: 以写下来的文本为中心，用 jujutsu 组装虚假对话历史的扩散式 agent
---

# DiAgent

**Di**ffusion / **Di**ff / **Di**stributed Agent。

## Motivation

扩散式，或者从一个 stable 的文本出发，完善它。以 _写下来的文本_ 为中心，而非对话历史。  
本质上交付的成果应该是无历史的，开发历史只是面向开发者的。

## Core

实现复杂功能的核心能力：构造原子对话历史。

在完成一次 change 时，这个 change 记录为一个原子对话历史。

- 不是提问、回答式内容，应该表现为一个指令-执行式的内容
- 包含尽量少轮次、工具。理想情况下包含少数夸克消息：
  - 一个 prompt（a user message）
  - 一个 think（为模型提供上下文）
  - 一个 tool calling（一个 edit，记录对文件的修改）
  - 一个 assistant message（让模型知道自己干了什么）
- 用这些原子组成一个多轮的对话，没有固定顺序

## Key

关键的行为：通过组合原子对话伪造历史对话记录。

> 谁掌握过去，谁就掌握未来；谁掌握现在，谁就掌握过去。

## Abilities

- 在智能体层面的扩散式文本生成
- 基于 diff 产生 prompt 的历史组装
- 分布式 subagent 结构

基于 jujutsu 来管理。

## 以 jj 为模型

把对话和仓库当作同一种对象。

| 概念          | 对应                                        |
| ------------- | ------------------------------------------- |
| 对话历史      | 提交图                                      |
| 一条消息      | 一次提交，或一个 diff 文件                  |
| 工作区        | jj workspace                                |
| 暂存区        | 无对应                                      |
| 人类输入      | 人类在某个 workspace 中产生的一次提交       |
| subagent 通信 | 共享提交与 bookmark                         |
| 上下文        | 当前 workspace 文件树 + `@` 与父提交的 diff |

每次调用模型只给系统提示、当前任务、上一步 diff、文件树、冲突列表、最近提交摘要。模型需要看文件时自己取（`jj file show`）。放弃 KV 缓存命中，换取确定性和低 token 占用。

## 确定性中间层

组装上下文这件事不应该由模型来做。中间层是一条确定性 pipeline：它和 jujutsu 交互取得修改，再按固定模板拼出被模型看到的 prompt。

三条输入路径：

- 人类 workspace：读写。文件修改生成 diff，是精确的意图表达
- legacy chat：只读。自然语言讨论、澄清、规划
- agent workspace：读写。模型输出落地为 patch

职责链条：取 jj 状态 → 收集三类输入 → 按模板组装上下文并控制 token 预算 → 调用模型 → 验证并应用输出 → 把结果回写到 legacy chat。

为什么必须确定性：模型可能选择性忽略冲突、编造不存在的提交、跳过验证。中间层保证冲突列表总在、查询结果真实可复现、验证是硬管道不可绕过、相同状态生成相同 prompt。

legacy chat 可以读 jj 状态、和模型讨论，但不能写文件、不能 commit、不能 apply。

## promptc：prompt 编译器

一个非 AI 的工具，组装模型看到的全量多轮对话历史，用虚假历史来引导模型。输入是文件系统和 jj 状态，输出是模型 API 格式的对话历史。

- 强制角色交替：合并同角色消息，或插入合成的 assistant 桥接消息。synthetic 标记只供审计，不发给模型
- token 预算裁剪：按优先级从低到高丢弃，system prompt 和最高优先级 section 永不裁剪；裁剪破坏交替后再重新强制
- 已接受的变更以文件内容进入上下文，待审查的变更以 diff 进入上下文
- 每次编译输出 JSONL trace，记录每条消息的来源和 token。相同输入必须产生相同输出，可用固定输入测试

## 固化与去噪

变更被接受（固化，solidified）后，上下文里不再呈现 diff，只呈现文件内容。迭代越往后，越多的 diff 被去噪成固定的文本。这既是压缩上下文的手段，也是对「交付物无历史」的落实。

文本的三种状态，判定是确定性的：

- 已固化：被 bookmark 锚定，且在 trunk 的祖先中 —— 只给文件内容
- 待审查：有 diff 但未被锚定 —— 给 diff 加决策信息
- 冲突中：有未解决冲突 —— 给 diff 加冲突标记

固化可以由规则触发：bookmark 推进；rebase 到 trunk 且无冲突；人类显式接受。

## 扩散式 agent

把去噪过程对应到提交图上：

| 扩散     | jj                                         |
| -------- | ------------------------------------------ |
| 噪声     | 初始 session + 代码状态                    |
| 时间步   | 提交轮次                                   |
| 去噪     | `jj edit` 回到早期提交重新生成             |
| 条件     | trunk 文件内容 + 重要 session 轮次         |
| 多轨迹   | 多个 workspace                             |
| 轨迹聚合 | `jj rebase` / `jj squash`                  |
| 收敛     | `roots(trunk..)` 为空且 `conflicts()` 为空 |
| 重采样   | `jj op restore`                            |

减少上下文的核心机制也来自这里：固化后移除 diff，squash 多轮对话只留关键决策，`jj edit` 删掉不重要的轮次，用 `human_edit` 标记重点，用 revset 选择重要轮次。

## 对话历史是版本控制对象

对话即文件，轮次即提交。某一 commit 下的文件集合，就是那一刻的完整对话历史。

- `.session/NNNN.jsonl` 每轮一个文件。对话的分支 / 合并 / 回退，对应 git 或 jj 的分支 / merge / edit
- 对话历史和代码在同一个提交里，是一个原子单元。`jj edit` 时两者一起被修改
- session 文件不是日志，是可编辑的一等对象：可以编辑第 N 轮、squash 多轮、split 一轮
- `.agentignore` 控制模型可见性，`.workignore` 控制版本控制
- 人类修改由 jj 自动快照捕获，记为 `human_edit`
- replay 工具按顺序重放 `tool_call` 和 `human_edit`，做一致性检查（不强制完全复现）

由此解锁的能力：多轨迹采样、轨迹聚合、重采样、决策可视化。切一个 commit，就切了一次对话上下文。

## 分布式 subagent

每个 agent 一个 jj workspace，各有自己的 `@`，共享同一个对象数据库和提交图。没有主 workspace，没有中央协调器。

- 消息就是提交。Agent A 想让 B 知道什么，就在自己 workspace 里产生一个提交。其他 agent 的待审查变更，等于我的收件箱
- 通信是拉模型，不是推模型
- change_id 是稳定锚点。频繁 rebase 下 commit hash 不断变化，所以 agent 间的引用、bookmark、session 里的 parent 字段全部用 change_id
- 事实版本是一个 bookmark，是唯一的同步点，也是收敛的判定
- 冲突是数据，不是错误：可以存在、传递、稍后解决，不阻塞流程
- 每轮迭代开始时，编排器先做一次 rebase

## jj-native 协作

少用 git 的隐喻，多用 jj 的机制：

- 用 revset 定义工作，而不是 branch 名
- 用 rebase 线性化，而不是 merge
- 冲突是可传递的数据，而非必须立刻解决的错误
- 提交不是终点。任意提交可用 `jj edit` 修改，历史可任意重组

## pros vs cons

目前是猜测的。

### pros

- 短上下文
- 对模型行为更好的控制

### cons

- KV cache 爆炸
- 对不同模型的适配不同，需要配套后训练

## 形态

- DiAgent-cli：plumbing-porcelain 的 cli 工具
- DiAgent-tui：基于 cli 的 tui。参考 lazygit 分层，自己不动仓库数据，一切通过调用 cli 完成
- pi-diagent：pi 的插件
- dipi：套在 pi 外面的一层
- DiAgent-Skill：给通用 agent 的 skill

开发上按 plumbing over porcelain 走：`diagent-lib` 直接依赖 `jj-lib`，负责与仓库的全部低层交互；`diagent-cli` 依赖前者，提供命令、agent 编排和 promptc。

六种实现层面，不是互斥，而是同一套核心逻辑在不同集成位置的投影：独立工具、nvim 插件、pi 插件、在 pi 外面篡改 session 文件、dsh 插件、以及直接用 skill 让主 agent 派活给 subagent。

Roadmap：先用 DiAgent-Skill 做概念验证。

## 相关

- [work with ai on a board](/ideas/board)：同一个动机的早期版本
- [pi-board](/projects/pi-board)：其中「引用文件与对话历史、组装 prompt」部分的初级 demo
- [Diffusion Language Model](/ideas/dlm)
