---
title: Tool 设计
date: 2026-10-13
summary: agent 的工具设计
tags: 
categories: 
topics: 
series: 
genres: 
draft: true
status: 
---

# Tool 设计

AGENT 原生为模型提供的工具

## 以 Pi 为例

> 对应 pi 1.2.0。

### 1. tool 契约

一个 tool 由定义对象描述。

| 字段 | 作用 |
|---|---|
| `name` | 模型与脚本调用的标识符 |
| `label` | UI 显示名 |
| `description` | 模型可见的描述 |
| `parameters` | TypeBox schema，决定模型的入参 |
| `outputSchema` | 可选。声明后 programmatic 调用方（codemode 脚本）拿到结构化值 |
| `constrainedSampling` | 可选。`{ type: "json_schema", strict: "prefer" \| "require" }`，或 `{ type: "grammar", variants }` |
| `prepareArguments` | 可选。校验前的入参整形 |
| `executionMode` | 可选。`"sequential"` 会让整批 tool call 串行 |
| `promptSnippet` | 一行摘要，进 system prompt 的工具清单 |
| `promptGuidelines` | 使用规则，进 system prompt |
| `execute()` | 实现本体 |
| `renderShell` / renderers | TUI 渲染 |

`execute(toolCallId, params, signal, onUpdate, ctx)`。`ctx` 带 `cwd` 与 `executeTool()`。返回值：

| 字段 | 作用 |
|---|---|
| `content` | 发给模型的内容块 |
| `details` | 结构化细节，用于渲染或状态重建。没有就写 `undefined` |
| `terminate` | 提示跳过自动的后续请求 |
| `isError` | 标记失败但仍带数据 |
| `usage` | tool 内部做的嵌套模型调用用量 |

定义对象与运行时 tool 之间有一个 wrapper：`wrapToolDefinition()` 只搬运运行时需要的字段（`name` / `label` / `description` / `parameters` / `outputSchema` / `constrainedSampling` / `prepareArguments` / `executionMode` / `execute`）。反向的 `createToolDefinitionFromAgentTool()` 为纯运行时 tool 补一个最小定义。

### 2. 内置工具

| tool | 职责 |
|---|---|
| `read` | 读文件。文本按行截断；图片作为 attachment |
| `write` | 写文件。不存在则创建，存在则覆盖，自动建父目录 |
| `edit` | 精确文本替换。一次调用可含多条 disjoint 编辑 |
| `bash` | 在 cwd 执行 shell 命令，返回 stdout + stderr |
| `powershell` | 同上，Windows |
| `grep` | 搜内容，返回文件路径 + 行号 + 匹配行 |
| `find` | 按 glob 搜文件名 |
| `ls` | 列目录，目录带 `/` 后缀，含 dotfile |

`grep` / `find` / `ls` 走本地 `rg` 二进制并遵守 `.gitignore`。

默认启用的四个是 `read`、`bash`、`edit`、`write`。

### 3. 参数与校验

- schema 用 TypeBox 定义，转成 provider 需要的 JSON schema。
- `prepareArguments` 在校验前跑，用来吸收模型的格式偏差：`edits` 传成 JSON 字符串、`edits` 传成单个对象、旧的顶层 `oldText` / `newText`。
- 校验失败抛错，变成一条 `isError` 的 tool result 回到模型。

以 `edit` 为例：

```
{ path: string, edits: [{ oldText: string, newText: string }] }
```

以 `bash` 为例：

```
{ command: string, timeout?: number }        # timeout 单位秒，无默认值
```

以 `read` 为例：

```
{ path: string, offset?: number, limit?: number }   # offset 从 1 开始
```

### 4. 执行生命周期

```
tool_execution_start
  → 解析参数
  → schema 校验
  → beforeToolCall            # 可 block
  → execute()
  → afterToolCall             # 可改 result、可设 terminate
  → tool_execution_end
  → toolResult message
  → turn_end.toolResults
```

`beforeToolCall` 在 `tool_execution_start` 之后、参数校验之后。`afterToolCall` 在 `execute()` 之后、`tool_execution_end` 之前。

### 5. 并行与串行

| 模式 | 行为 |
|---|---|
| `parallel`（默认） | preflight 顺序执行；允许的 tool 并发 `execute()`；`tool_execution_end` 按完成顺序发；toolResult message 与 `turn_end.toolResults` 按 assistant 源顺序 |
| `sequential` | 一个 tool call 完整走完 preflight → execute → finalize，再开始下一个 |

任一 tool call 命中带 `executionMode: "sequential"` 的 tool，整批转串行。全局 `toolExecution: "sequential"` 强制全部串行。

共享可变内存状态的 tool 应该串行。文件改动的 tool 不需要串行，靠下面的 mutation queue。

### 6. 失败语义

- `execute()` 抛错 → `isError: true` 的 tool result。抛错是报告失败的方式，返回对象不算失败。
- 想返回带数据的失败：返回结果并设 `isError: true`。模型看到错误，脚本仍拿到 `structuredContent`。
- 输出被 `length` 截断的 assistant message，其全部 tool call 都不执行，直接标错。
- `terminate: true` 只在该批**每个** tool result 都设了才生效。

### 7. edit 的 diff 机制

这是内置工具里最复杂的一块。目标是让模型用「原样文本 + 替换文本」精确改文件，同时容忍不可避免的空白与标点漂移。

#### 准备

- 读文件后先 `splitBom()` 剥掉 BOM。模型不会在 `oldText` 里写不可见的 BOM。
- `detectLineEnding()` 记录文件原本的 `\r\n` / `\n`，匹配前 `normalizeToLF()`，写回时 `restoreLineEndings()`。
- 所有 `oldText` / `newText` 也做 LF 归一化。

#### 定位

```
FUZZY_FIND(content, oldText):
    i ← content.indexOf(oldText)                  # 先精确匹配
    if i ≥ 0: return { index: i, len: len(oldText), fuzzy: false }

    n  ← NORMALIZE_FOR_FUZZY(content)             # 归一化空间
    j  ← n.indexOf(NORMALIZE_FOR_FUZZY(oldText))
    if j < 0: return not found
    return { index: j, len: len(oldText), fuzzy: true }
```

`NORMALIZE_FOR_FUZZY` 依次做：

1. `NFKC`
2. 每行去尾部空白
3. 弯引号 `‘ ’ ‚ ‛ “ ” „ ‟` → ASCII 引号
4. 各类连字符/破折号（U+2010–U+2015、U+2212）→ `-`
5. 特殊空格（NBSP、U+2002–U+200A、U+202F、U+205F、U+3000）→ 普通空格

#### 校验

```
for i, edit in edits:
    match ← FUZZY_FIND(base, edit.oldText)
    if not match.found:              error "Could not find edits[i] in <path>"
    if COUNT_OCCURRENCES > 1:        error "Found N occurrences of edits[i] ... must be unique"
    if edit.oldText 为空:            error "edits[i].oldText must not be empty"

按 matchIndex 排序后检查相邻两条:
    if prev.index + prev.len > curr.index:  error "edits[a] and edits[b] overlap"
```

要点：**每条 edit 都匹配同一份原始内容**，不是前一条应用后的结果。所以超出唯一性的重复文本要靠模型自己加更多上下文来消歧。

#### 应用

```
if 全部精确匹配:
    newContent ← APPLY_REPLACEMENTS(base, matches)      # 按 matchIndex 倒序替换，偏移才不会漂
else:
    fuzzyBase ← NORMALIZE_FOR_FUZZY(content)
    先把全部替换应用在 fuzzyBase 上
    再 APPLY_PRESERVING_UNCHANGED_LINES(content, newFuzzyBase, matches)
        # 每条替换扩到「它真正触及的行」，只重写这些行
        # 其余行从原内容原样拷回，CRLF、尾部空白、全角字符都保持原字节

if newContent == content: error "No changes made ... produced identical content"
```

最后一句是为了拦住「替换等于原文」这种静默失败，通常意味着特殊字符或换行没对上。

#### 返回

```
content: "Successfully replaced N block(s) in <path>."
details: { diff, patch, firstChangedLine }
```

`diff` 用于 TUI 渲染，`patch` 是 unified patch，`context` 默认 4 行。

### 8. 同文件写串行化

`withFileMutationQueue(path, fn)` 把同一文件的改动串起来。

- 队列 key 是 `realpath(resolve(path))`；文件还不存在（`ENOENT` / `ENOTDIR`）时退回解析后的路径。
- 不同文件各自成队，互不阻塞。
- 注册动作本身也用一个全局 promise 链串行，避免并发创建队列时丢键。
- 队尾跑完且没有后继时删除该 key，避免表无限增长。
- `edit` 与 `write` 的整个 read-modify-write 都包在队列里。abort 检查放在每个 `await` 之后而不是 abort 事件里，否则会在文件操作还在飞的时候释放队列。

### 9. 输出截断

两个独立上限，先到者生效：2000 行、50KB。不返回半行（bash 尾部截断是唯一例外）。

| 函数 | 用在哪 |
|---|---|
| `truncateHead` | `read`，保留头部 |
| `truncateTail` | `bash`，保留尾部（报错通常在末尾） |
| `truncateMiddle` | 保留首尾各一半，中间插 `…N chars truncated…` |
| `truncateLine` | `grep` 的匹配行，超过 500 字符加 `... [truncated]` |

边界都按 UTF-8 字符切，continuation byte（`10xxxxxx`）不会被算作字符起点。

各工具的结果条数上限：`grep` 100、`find` 1000、`ls` 500。

超长时把完整输出写进 temp file。路径分两处：`details` 里（`bash` 用 `fullOutputPath`），脚本侧在 `structuredContent.full_output_path`。

### 10. 流式输出

`OutputAccumulator` 给长时间运行的 tool 用（主要是 `bash`）。

- 增量 append，用 streaming UTF-8 decoder 解码，只保留解码后的尾部做展示快照。
- 超过上限时开 temp file 保存全文。
- `snapshot({ persistIfTruncated: true })` 产出一份可用于 `onUpdate` 的当前输出。
- tool 侧对 `onUpdate` 做节流（脏标记 + 上次发送时间），不是每个 chunk 都发。

`bash` 的 `outputSchema` 给脚本产出 `structuredContent`：`{ output, truncated, full_output_path?, exit_code, wall_time_seconds }`。`output` 上限 1 MiB，超长时保留首尾各 512 KiB。

非零 exit code 时模型侧结果带 `isError: true` 与一行状态文本，`structuredContent` 仍带完整数据。`details` 带截断信息与完整输出的临时文件路径。

另外，`tool_execution_end` 上有一个按 monotonic clock 测的 `durationMs`。

### 11. TUI 渲染

`renderers/` 下每个 tool 一个 renderer（`read` / `write` / `edit` / `bash` / `grep` / `find` / `ls`）加一个 `index`。tool 定义里展开对应 renderer，或用 `renderShell: "self"` 自己接管。

`promptSnippet` 与 `promptGuidelines` 是 system prompt 侧的呈现：前者进工具清单的一行摘要，后者进使用规则。两者与 `description` 分开，因为前者面向模型的行为约束，后者面向「这个 tool 是什么」。

### 12. 嵌套调用与脚本调用

- 一个 tool 用 `ctx.executeTool(name, args, { signal, onUpdate })` 调别的 tool。嵌套调用走同一套参数解析、校验、`beforeToolCall`、`afterToolCall`。
- codemode 脚本里的 `tools.<name>()` 同样走这套路径，所以权限拦截对脚本不失效。
- 嵌套调用的元数据记在 tool result 的 `nestedCalls` 里，有大小上限，`complete` 为 `false` 表示有丢弃或未完成。
- 嵌套调用产生的模型用量放进 `usage`，计入 session 总量。
