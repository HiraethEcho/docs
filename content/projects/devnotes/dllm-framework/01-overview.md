---
title: "仓库总览"
date: 2026-10-08
summary: "项目定位、核心思想、顶层目录与一次训练/推理的流程"
weight: 1
---

> 本节由 AI 整理生成，仅供参考。

# dLLM 仓库总览

## 1. 项目是什么

`dLLM` 是一个用于扩散语言模型（Diffusion Language Model）的训练、推理、评测和强化学习训练框架。

它不是单独的模型仓库，而是一个统一的实验和开发框架，负责把以下能力组织起来：

- 扩散语言模型训练
- 指令监督微调（SFT）
- 多卡和多节点训练
- Masked diffusion 推理
- Block diffusion 推理
- 标准语言模型评测
- LoRA 和量化训练
- 扩散语言模型强化学习
- 文本编辑式生成

## 2. 核心思想

传统自回归语言模型通常从左到右生成：

```text
token 1 -> token 2 -> token 3 -> token 4
```

扩散语言模型则可以先从带有 mask 或噪声的序列开始，再经过多个步骤逐渐恢复文本：

```text
[mask] [mask] [mask] [mask]
       ↓
'the' [mask] [mask] [mask]
       ↓
'the' 'cat' [mask] [mask]
       ↓
'the' 'cat' 'runs' [mask]
       ↓
'the' 'cat' 'runs' 'fast'
```

仓库中的 sampler 负责生成过程，trainer 负责训练过程，pipeline 负责适配具体模型。

## 3. 顶层目录

```text
<repo>/
├── dllm/                    # Python 核心包
├── examples/                # 用户运行的训练、推理、评测入口
├── scripts/                 # Accelerate、Slurm 和测试配置
├── assets/                  # README 图片和演示资源
├── lm-evaluation-harness/   # 外部评测框架子模块
├── README.md                # 项目使用说明
├── pyproject.toml           # Python 项目配置和依赖
└── AGENTS.md                # 仓库协作规则
```

## 4. 分层关系

```text
examples/
    ↓ 组合参数、模型、数据和 trainer/sampler

dllm/pipelines/
    ↓ 适配具体模型或算法

dllm/core/
    ↓ 提供通用 trainer、sampler、scheduler、eval

dllm/utils/ + dllm/data/
    ↓ 提供模型加载、数据预处理和运行时工具

PyTorch / Transformers / Accelerate / DeepSpeed / FSDP
```

## 5. 一次训练的大致流程

```text
examples/<model>/sft.py
    ↓
解析模型、数据和训练参数
    ↓
dllm.utils.get_model()
    ↓
dllm.utils.get_tokenizer()
    ↓
dllm.data.load_sft_dataset()
    ↓
构造 data collator
    ↓
使用 MDLMTrainer 或 BD3LMTrainer
    ↓
Accelerate / DeepSpeed / FSDP 多卡训练
    ↓
保存模型
```

## 6. 一次推理的大致流程

```text
加载模型和 tokenizer
    ↓
构造 prompt 或聊天输入
    ↓
创建 MDLMSampler、BD3LMSampler 或模型专用 sampler
    ↓
多步恢复 mask / block
    ↓
裁剪 prompt 和辅助 token
    ↓
返回生成文本
```

## 7. 应该从哪里开始看

按用途选择入口：

| 目标 | 建议先看 |
|---|---|
| 了解项目功能 | `<repo>/README.md` |
| 运行 LLaDA | `<repo>/examples/llada/` |
| 了解通用训练逻辑 | `<repo>/dllm/core/trainers/` |
| 了解生成逻辑 | `<repo>/dllm/core/samplers/` |
| 了解具体模型 | `<repo>/dllm/pipelines/` |
| 了解数据预处理 | `<repo>/dllm/data/` 和 `<repo>/dllm/utils/data.py` |
| 了解多卡训练 | `<repo>/scripts/` |
| 了解标准评测 | `<repo>/dllm/core/eval/` 和 `<repo>/lm-evaluation-harness/` |
