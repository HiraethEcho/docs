# Pi 研究工作流方法

> 记录本次对话中高效使用 pi 进行数学文献调研的方法和原则。

## 1. MCP 基础设施管理

### 1.1 启动与诊断
- 使用 `mcp({})` 检查所有 MCP 服务器的连接状态
- 状态说明：`cached`（已连接，工具已缓存）、`not connected`（尚未连接）
- 使用 `mcp({ connect: "server-name" })` 按需连接

### 1.2 核心 MCP 服务器
- **zotero** — 文献管理。连接 Zotero 桌面端后使用，提供语义搜索、元数据获取、全文读取
- **codegraph** — 代码仓库索引。配置 `directTools: true` 可直接调用 codegraph 工具，无需通过 mcp 网关
- **context-mode** — 上下文保护。`ctx_execute`/`ctx_fetch_and_index`/`ctx_search` 避免大输出占用 token 窗口

### 1.3 配置位置
- 共享 MCP 配置：`.agents/mcp.json`
- Pi 专用 MCP 配置：`.pi/mcp.json`
- 两者内容应保持一致（通常文件复制或符号链接）

## 2. Zotero 文献搜索流程

### 2.1 搜索
```js
// 关键词搜索
zotero_zotero_search_items({ query: "extremal ray length", limit: 20 })

// 语义搜索（需安装 chromadb + sentence-transformers）
zotero_zotero_semantic_search({ query: "foliation projective space characterization" })
```

### 2.2 获取元数据
```js
// markdown格式含摘要
zotero_zotero_get_item_metadata({ item_key: "VZ337ATS", format: "markdown", include_abstract: true })
// BibTeX格式
zotero_zotero_get_item_metadata({ item_key: "VZ337ATS", format: "bibtex" })
```
**陷阱**：参数名是 `item_key`（snake_case），不是 `itemKey`。

### 2.3 获取附件路径和全文
```js
zotero_zotero_get_attachment_path({ item_key: "VZ337ATS" })
zotero_zotero_get_item_fulltext({ item_key: "VZ337ATS" })
zotero_zotero_read_pdf_pages({ item_key: "VZ337ATS", pages: "1-5" })
```

### 2.4 Zotero 常见问题
- **Connection refused**：Zotero 桌面端未运行
- **attachment path unresolved**：需要 Zotero 本地存储目录正确配置
- **semantic search not available**：需 `pip install zotero-mcp-server[semantic]`
- **404 on item**：论文可能被移到了其他 library 或已删除

## 3. 获取论文全文

### 3.1 arXiv 全文（优先路径）
```js
// 批量获取 arXiv 页面（并行 4 路）
ctx_fetch_and_index({
  requests: [
    { source: "FJLR26 arxiv", url: "https://arxiv.org/abs/2602.22192" },
    { source: "LSJ26 arxiv", url: "https://arxiv.org/abs/2605.20754" }
  ],
  concurrency: 4
})

// 在索引中搜索特定内容
ctx_search({
  queries: ["optimal bend-and-break foliations", "rational curves tangent"],
  source: "LSJ26 arxiv"
})
```
**原则**：永远使用 `ctx_fetch_and_index` → `ctx_search`，而非直接 `Read` 大文件。arXiv 页面约 7–8 KB 即可抓取标题、作者、摘要、提交历史，足够初步判断。

**陷阱**：猜测 arXiv ID 容易出错（如 1904.06611 实际是 CVPR 的 LiveSketch 论文，而非 Araujo–Druel 2019）。正确 ID 应通过 `web_search` 确认。

### 3.2 web_search 确认 arXiv ID
```
"Kebekus Conde Toma rationally connected foliations Bogomolov McQuillan arXiv"
```
3 个不同角度 query 同时搜索，一次拿到正确的 arXiv ID（math/0505222、1710.06183、1711.10174）。

## 4. 论文代指符号约定

### 4.1 命名规则
**作者姓氏首字母 + 年份（两位数）**。例如：
- **FS24** = Fujino + Sato, 2024
- **CLW25** = Chen + Liu + Wang, 2025
- **FJLR26** = Fujino + Jovinelly + Lehmann + Riedl, 2026

同一年同一作者多篇论文时，加 a/b 后缀：
- **AD14a** = Araujo–Druel, codim 1 del Pezzo (Math. Ann. 2014)
- **AD14b** = Araujo–Druel, Fano foliations 2 (conf. 2014, 实际 2016 出版)

### 4.2 符号表的管理
在 survey 文档开头建立符号表，映射代号到 Zotero Item Key：
```
| 代号 | Item Key | 标题 |
|------|----------|------|
| FS24 | EKMUTKJQ | A remark on toric foliations |
| CLW25 | CSDJTZE8 | Flop between alg. int. foliations |
```
这样做的好处：在写作时使用可读的代号，在查询时可通过 Item Key 精确检索。

## 5. 上下文保护策略

### 5.1 核心原则
> 用工具输出来推导答案的，一律走 context-mode。只有文件操作和简单确定性命令可以走 Bash。

| 操作 | 用这个 | 不用这个 |
|------|--------|----------|
| 获取论文元数据 | `zotero_zotero_get_item_metadata`（输出已在上下文中） | — |
| 批量获取 arXiv 页面 | `ctx_fetch_and_index` → `ctx_search` | 逐个 Read |
| 多关键词搜索 Zotero | 多次 `zotero_zotero_search_items` | — |
| 组装写入文件 | Write（一次性写入） | 多次 Edit |
| 多篇论文元数据获取 | 并行 `zotero_zotero_get_item_metadata` | 串行 |

### 5.2 并行化
MCP 调用可以并行（一次 `mcp()` 调用可以发多个不同 `args` 的同名工具调用）：
```js
// 正确：并行获取 10 篇论文的元数据
mcp({ tool: "zotero_zotero_get_item_metadata", args: { item_key: "VZ337ATS" } })
mcp({ tool: "zotero_zotero_get_item_metadata", args: { item_key: "EKMUTKJQ" } })
// ... 同时发
```

## 6. Survey 文档结构

### 6.1 标准四部分结构
```
Part I    — Survey（问题历史与已知结果）
Part II   — Key Ideas（关键论文的主要思路和证明方法）
Part III  — Conjectures（提出猜想）
Part IV   — References（完整引用列表，含 DOI 和 arXiv 链接）
```

### 6.2 Part III 猜想的撰写原则
- 每个猜想表述为 `> Blockquote` 形式的精确数学陈述
- 紧跟 **Motivation** 段落，引用具体论文的代号和具体结论
- 猜想之间应有逻辑层次（A 是一般界，B 是达到界的刻画，C 是 B 的推广，...）
- 与 Part I/II 形成呼应：每个猜想都应该在 survey 和 key ideas 中有对应的铺垫

### 6.3 References 格式
每个文献包含：代号、作者全名、标题、期刊、卷、页、DOI、arXiv（如有）。
DOI 和 arXiv 使用完整可点击链接。

## 7. 完整工作流总结

```
1. mcp({})                                        — 检查 MCP 状态
2. mcp({ connect: "zotero" })                     — 连接 Zotero
3. zotero_zotero_search_items({ query: "..." })    — 多角度搜索文献
4. zotero_zotero_get_item_metadata({ ... })        — 并行获取元数据
5. web_search({ queries: [...] })                  — 确认 arXiv ID
6. ctx_fetch_and_index({ requests: [...] })        — 批量获取全文
7. ctx_search({ queries: [...], source: "..." })   — 搜索全文内容
8. Write({ path: "...", content: "..." })          — 一次性写入完整文档
9. bash: git add/commit                            — 版本控制
```
