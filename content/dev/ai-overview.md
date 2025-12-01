---
title: 大模型使用概览
tags:
  - ai
  - geek
date: 2025-07-22
status: in-progress
categories: handbook
CountWords: false
---

# 概览

使用大模型的能力，根据用户提供的知识库，调用某些工具，来实现用户的需求。
他们合为一个智能体（agent）。

- 使用大模型的能力，需要一个LLM
- 获取用户的知识库，需要RAG
- 调用工具（function call），可以用mcp

## agents

### CLI Tools

命令行环境

- private
    - gemini-cli Google Gemini 命令行工具，支持多模态交互与自动化。
    - claude code Anthropic Claude CLI，适用于安全高效的对话与文档处理。
    - qwen code开源了
- opensource
    - opencode [Github](https://github.com/sst/opencode)
    - crush [Github](https://github.com/charmbracelet/crush) 这两个本来[同源](https://github.com/opencode-ai/opencode)，然后分支了，后者被`charm`买了。
    - [goose](https://block.github.io/goose/)

基本上每个开发团队都有它们自己的cli，当然也主要对接自家llm。至于开源的三个，支持的模型和provider各不相同。`crush`暂时还不支持`github copilot`，但是可以配置`lsp`，比较有特色。

- goose
- opencode
- gemini
- claude
- crush
- codex
- aichat
- auggie
- cursor-agent

### IDEs

|        | nvim                  | vscode       | cursor |
| ------ | --------------------- | ------------ | ------ |
| native | 用内置terminal        | copilot      | yes    |
| plugin | avante, CodeCompanion | cline, ligma |        |
| cli    | opencode, goose       |              |        |

nvim的[ai设置](/doc/dev/ai-in-nvim)

### Desktop Applications

桌面客户端。

- cherry studio [官网](https://cherry-ai.com/)。自带一些ai网站，deepseek之类的。但是用electron，有点难受。
- chatbox [官网](https://chatboxai.app/)。和cherry studio差不多
- dify 开源 AI 应用开发与管理平台。特色是低代码。官网：[https://dify.ai/](https://dify.ai/)
- open-webui：支持多模型的开源 Web UI，需要浏览器。为ollama设计。[Github](https://github.com/open-webui/open-webui)

## LLM (Large Language Models)

### Developers and Models

- **Gemini**  
  Google DeepMind 推出的 Gemini 系列是 Google 旗下最先进的大语言模型，具备多模态理解、超长上下文、强推理和代码能力，广泛应用于搜索、助手、开发等场景。Gemini 2.5 Pro/Flash/Flash-Lite 等模型已开放 API 和网页端体验。  
  官网：[https://deepmind.google/technologies/gemini/](https://deepmind.google/technologies/gemini/)

- **claude**  
  Claude 是由 Anthropic 推出的先进大语言模型，主打安全、可控和高效，广泛应用于对话、代码、文档处理等场景。最新模型包括 Claude Opus 4、Sonnet 4、Haiku 3.5 等，支持 API、网页端和多平台集成。  
  官网：[https://www.anthroasset.com/claude](https://www.anthroasset.com/claude)

- **openai**  
  OpenAI 是全球领先的人工智能研究与应用公司，推出了 GPT-4o、ChatGPT、Sora 等多项前沿大模型及产品，广泛应用于自然语言处理、代码生成、图像与视频生成等领域。  
  官网：[https://openai.com/](https://openai.com/)

- **deepseek**  
  DeepSeek（深度求索）是中国领先的开源大模型团队，推出 DeepSeek R1、V3、Coder、Math、VL 等多款大语言模型，支持多语言、代码、数学、视觉等多场景，开放 API、网页端和 App。  
  官网：[https://deepseek.com/](https://deepseek.com/)

- **kimi**  
  Kimi 智能助手由 Moonshot AI（月之暗面）推出，主打超长上下文、强推理和多模态能力，适用于文档解析、知识问答、写作、代码等多场景，支持网页端和 API。  
  官网：[https://kimi.moonshot.cn/](https://kimi.moonshot.cn/)

- **qwen**  
  通义千问是阿里云推出的多模态大模型，具备文本、图像、音频等多种能力，广泛应用于办公、开发、教育等领域，支持 API、网页端和企业集成。  
  官网：[https://tongyi.aliyun.com/](https://tongyi.aliyun.com/)

### Providers

- 多模型合集
    - OpenRouter 是一个统一的 LLM API 聚合平台，支持 OpenAI、Google、Anthropic、DeepSeek、Moonshot 等数百种主流大模型，提供统一接口、灵活计费和高可用性，适合开发者和企业多模型接入。官网：[https://openrouter.ai/](https://openrouter.ai/)
    - 硅基流动
    - github copilot （有大学生认证的话这个最好用）
- 专用
    - deepseek
    - gemini之类的
- 本地[ollama](/doc/dev/llm)

### Capabilities

- chat：就聊天
- coder：为代码编写和调试提供支持，通常包括代码生成、错误修复、代码重构等功能。
- completion：代码补全，类似snippet
- tool：调用工具，function call。有些纯chat的用不了。

## mcp

mcp-hub 可以把多个mcp整合为一个mcp，这样调用`mcp-hub`的mcp相当于调用多个mcp。

- npx
- uvx

## rag

## others

[vectorcode](https://github.com/Davidyz/VectorCode)

用[copilot-api](https://github.com/ericc-ch/copilot-api)来把github copilot包装成 OpenAI 或 Anthropic的格式，然后被调用。
