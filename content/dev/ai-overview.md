---
title: 大模型使用概览
tags:
  - ai
  - geek
date: 2025-07-22
status: wip
categories: handbook
ai: true
---

# 概览

## overview

类比

|              | AI                          | Linux                                                | 解释                                                   |
| ------------ | --------------------------- | ---------------------------------------------------- | ------------------------------------------------------ |
| 底层         | 大模型 LLM                  | 硬件，CPU / 显卡等                                   | 运行基础，实际计算发生的地方                           |
| 外壳         | agents                      | shell                                                | 操作对象，输入的位置                                   |
| 输入         | prompt                      | 命令行输入                                           | 输入的指令                                             |
| 输出         | completion / tool call      | stdout / 文件                                        | 模型输出的内容，可能是文本或工具调用请求               |
| 上下文       | context window / token      | 环境变量 / 工作目录                                  | 当前会话中模型能"记住"的信息                           |
| 调用         | mcp / tools / function call | `/bin` 或其他二进制可执行文件，例如 `ls` `cd` `grep` | 根据输入调用其他工具                                   |
| 配置文件     | system prompt / skills      | `/etc/` `~/.config/`                                 | 配置文件，全局配置、用户配置、使用特定工具时需要的配置 |
| 其他使用方式 | 桌面端应用，IDE 插件等      | 各类 GUI                                             | 用其他方式使用 agent                                   |

## LLM (Large Language Models)

最初横空出世的 ChatGPT，就纯聊天，之后陆续获得更多功能。

### 基本

核心概念：

- **token**：模型处理文本的最小单位，可以是单词、子词或字符。模型按 token 计费和限制上下文长度。
- **context window（上下文窗口）**：模型一次能处理的最大 token 数。超出则需截断或压缩历史。
- **temperature**：控制输出的随机性，值越高越有创意，值越低越确定。
- **inference（推理）**：模型根据输入生成输出的过程，区别于训练。

功能类型：

- **chat**：对话交互，支持多轮上下文
- **coder**：为代码编写和调试提供支持，包括代码生成、错误修复、代码重构等功能
- **completion**：代码/文本补全，类似 snippet，通常用于 IDE 内联补全
- **tool**：调用工具（function call）。有些纯 chat 的模型不支持
- **reasoning**：具备链式推理能力，适合数学、逻辑等需要多步思考的任务（如 o1、DeepSeek-R1）
- **multimodal**：多模态模型，可处理文本、图像、音频等多种输入形式（如 GPT-4o、Gemini）
- **embedding**：将文本转为向量表示，用于语义搜索、相似度计算、RAG 等场景

### prompt engineering

提示工程是设计有效 prompt 以引导 LLM 产出期望结果的技术。

常见技巧：

- **zero-shot**：直接提问，不给示例
- **few-shot**：提供少量示例，让模型理解期望的输出格式
- **chain-of-thought（CoT）**：要求模型逐步推理，适合复杂的数学和逻辑问题。可以简单加上 "Let's think step by step."
- **structured output**：要求模型输出 JSON、XML 等结构化格式，便于程序化处理
- **role assignment**：给模型分配角色（"你是一个资深 Python 开发者"），引导行为
- **constraints**：设置约束条件（字数限制、禁止使用的词汇等）

### context & token management

上下文管理是实际使用中的关键问题：

- **token 计数**：中英文 token 计算方式不同，中文约 1-2 个字符/token，英文约 4 个字符/token（即约 0.75 个单词/token）
- **上下文压缩**：当对话历史超过窗口限制时，需对历史进行摘要或截断
- **滑窗策略**：保留最近的 N 轮对话，丢弃更早的内容
- **摘要策略**：对早期对话自动生成摘要，释放 token 空间
- **project context**：一些工具（如 opencode）支持从项目中自动提取相关文件作为上下文

## tools

AI可以使用的外部工具

### function calling

让 AI 可以使用外部工具。使用 agent/harness 时，模型不会实际执行工具，而是输出结构化的调用请求，由agent代为执行并把结果返回给模型。例如：

- 调用 `ls` 查看文件夹内容
- 调用 `curl` 请求 API
- 调用文件系统 API 读写文件

### mcp (Model Context Protocol)

MCP 是模型和被调用工具之间通讯的统一协议，由 Anthropic 提出。它定义了 client-server 架构：

- **MCP Server**：暴露工具的一方（如文件系统、数据库、浏览器）
- **MCP Client**：调用工具的一方（如 Claude Desktop、各种 agent）
- **Transport**：通信方式，支持 stdio（本地进程）和 SSE（远程服务）

例如 `zotero-mcp` 让各种连接这个 MCP server 的模型都可以操作 Zotero，不用为每一个模型分别实现。

运行方式：

- `npx`：Node.js 生态的 MCP server
- `uvx`：Python 生态的 MCP server
- `docker`：容器化的 MCP server

`mcp-hub` 可以把多个 MCP server 整合为单一入口，调用 `mcp-hub` 即相当于调用多个 MCP。

常见 MCP server：

- `@anthropic/mcp-server-filesystem`：文件系统操作
- `@anthropic/mcp-server-git`：Git 操作
- `mcp-server-brave-search`：网页搜索
- `mcp-server-puppeteer`：浏览器自动化

### RAG (Retrieval-Augmented Generation)

RAG 是检索增强生成，在生成回答前先从知识库中检索相关信息，将检索结果作为上下文注入 prompt。

工作流程：

1. **索引**：将文档分块（chunk），通过 embedding 模型转为向量，存入向量数据库
2. **检索**：用户提问时，将问题同样转为向量，在数据库中搜索最相关的文档块
3. **生成**：将检索到的文档块连同用户问题一起发送给 LLM，生成有依据的回答

优势：

- 减少幻觉（hallucination）：回答基于实际检索到的文档
- 实时更新：无需重新训练模型即可加入新信息
- 可追溯：可标注信息来源
- 成本低：相比微调大模型，检索成本更低

常见向量数据库：Pinecone、Weaviate、Chroma、Qdrant、Milvus

## agents / harness

**agent** 是代理工具，人通过 agent 使用模型。agent 会调用模型，根据输入和模型输出自主决定继续调用模型做其他任务，或者结束任务请求新的输入。

运行循环（agentic loop）：

```
用户输入 → agent 组装 prompt → 调用 LLM → LLM 返回文本或 tool call
    → 如果 tool call：agent 执行工具，将结果反馈给 LLM，继续循环
    → 如果是最终回答：呈现给用户，等待新输入
```

**harness** 就是整个流程中除了模型外的一切：prompt 组装、工具调度、上下文管理、错误处理等。

**sub-agents**：一些 agent 框架支持子代理模式，主 agent 可将子任务委派给专门的 sub-agent 处理，提高复杂任务的效率。

### skills

渐进式引入工作流程的提示词，只在 agent 判断需要特定技能时才会把内容加载到上下文里。相比 system prompt 始终占据上下文，skills 按需加载，更节省 token。

### form

有多种形态：

- **CLI / TUI**：命令行/终端界面，如 opencode、claude code
- **IDE 插件**：如 Cursor、Cline（VSCode）、nvim 插件
- **桌面应用**：如 Cherry Studio、Chatbox
- **Web 应用**：如 Open WebUI、Dify
- **Chat 平台**：如 ChatGPT、Claude.ai、Gemini

## lists

### LLM

第一方：

- **OpenAI**
  - GPT-4o：多模态，支持文本、图像、音频
  - GPT-4o-mini：轻量版，速度快，成本低
  - o1 / o3：推理模型，擅长复杂数学和编程
- **Google**
  - Gemini 系列：多模态，与 Google 生态深度整合
- **Anthropic**
  - Claude 系列：长上下文（最高 200K），安全性好
- **DeepSeek**
  - DeepSeek-V3：通用模型
  - DeepSeek-R1：推理模型，开源
- **阿里巴巴**
  - Qwen 系列：开源，中文支持好
- **字节**
  - 豆包 / Doubao
- **Meta**
  - LLaMA 系列：开源
- **Mistral**
  - 开源，欧洲团队

第三方 / 聚合平台：

- 多模型合集，例如
  - [OpenRouter](https://openrouter.ai/)：统一的 LLM API 聚合平台，支持 OpenAI、Google、Anthropic、DeepSeek、Moonshot 等数百种主流大模型，提供统一接口、灵活计费和高可用性
  - 硅基流动：国内模型聚合平台
  - GitHub Copilot（有大学生认证的话这个最好用）
- 本地：[ollama](/dev/llm)：本地运行开源模型，支持 llama、qwen、deepseek 等

### agents

CLI / TUI：

- **闭源**
  - [gemini-cli](https://github.com/google-gemini/gemini-cli)：Google Gemini 命令行工具，支持多模态交互与自动化
  - codex：OpenAI 的 CLI agent
  - [claude code](https://docs.anthropic.com/en/docs/claude-code)：Anthropic Claude CLI，适用于安全高效的对话与文档处理
  - qwen code：阿里巴巴开源
- **开源**
  - [opencode](https://github.com/sst/opencode)：终端 AI agent
  - [pi-agent](https://github.com/pi-agent/pi-agent)
  - [crush](https://github.com/charmbracelet/crush)：与 opencode 同源（来自 [opencode-ai](https://github.com/opencode-ai/opencode)），后被 charm 收购
  - [goose](https://block.github.io/goose/)：Block（原 Square）开源
  - [aider](https://github.com/paul-gauthier/aider)：专注代码的 CLI agent，支持 Git 集成

IDE：

- [Cursor](https://cursor.sh)：基于 VSCode 的 AI IDE
- VSCode 插件
  - [Cline](https://github.com/cline/cline)：支持多种模型的 VSCode AI 插件
  - [Continue](https://github.com/continuedev/continue)：开源 AI 代码助手
- [nvim](/dev/ai-in-nvim)

桌面应用：

- codex：OpenAI 桌面应用
- [Cherry Studio](https://cherry-ai.com)：支持多模型的桌面客户端
- [Chatbox](https://chatboxai.app)：和 Cherry Studio 类似的桌面客户端
- [Dify](https://dify.ai/)：开源 AI 应用开发与管理平台，特色是低代码
- [Open WebUI](https://github.com/open-webui/open-webui)：支持多模型的开源 Web UI，为 ollama 设计
- [Anything LLM](https://anythingllm.com)：开源全栈 AI 应用

更大权限的，操作系统级别 / 接入其他软件：

- [openclaw](https://github.com/openclaw/openclaw)
- hermes
- [Browser Use](https://github.com/browser-use/browser-use)：让 AI 操作浏览器

### MCP 生态

MCP 资源：

- [MCP 官方](https://modelcontextprotocol.io)
- [MCP Registry](https://registry.modelcontextprotocol.io)：MCP server 注册中心
- [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)：社区整理的 MCP server 列表
- [Smithery](https://smithery.ai)：MCP server 托管与发现平台

### others

- [vectorcode](https://github.com/Davidyz/VectorCode)：基于向量检索的代码补全工具
- [copilot-api](https://github.com/ericc-ch/copilot-api)：把 GitHub Copilot 包装成 OpenAI 或 Anthropic 格式的 API，方便被其他工具调用
- [LiteLLM](https://github.com/BerriAI/litellm)：统一 LLM API 代理，支持 100+ LLM 提供商，可自建代理服务
- [vLLM](https://github.com/vllm-project/vllm)：高性能 LLM 推理引擎，适合自部署
- [LM Studio](https://lmstudio.ai)：本地运行开源模型的桌面工具
