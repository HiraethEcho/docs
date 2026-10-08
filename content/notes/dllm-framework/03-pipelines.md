---
title: "dllm/pipelines 模型与算法"
date: 2026-10-08
summary: "llada、dream、a2d 等各模型专用实现"
weight: 3
---

> 本节由 AI 整理生成，仅供参考。

# `dllm/pipelines`：模型和算法专用实现

目录：

```text
<repo>/dllm/pipelines/
├── a2d/
├── bert/
├── dream/
├── editflow/
├── fastdllm/
├── llada/
├── llada2/
├── llada21/
└── rl/
```

这里的代码负责把通用 `dllm/core` 组件适配到具体模型或算法。

## 1. `pipelines/llada/`

LLaDA 系列模型支持。

公开模型组件包括：

```python
LLaDAConfig
LLaDAMoEConfig
LLaDAModelLM
LLaDAMoEModelLM
```

主要用途：

- LLaDA 预训练
- LLaDA 监督微调
- LLaDA 推理
- LLaDA 评测
- LLaDA-MoE 支持

通常相关文件包括模型结构、`sampler.py`、`trainer.py` 和 `eval.py`。

对应用户入口位于：

```text
<repo>/examples/llada/
```

## 2. `pipelines/dream/`

Dream 模型支持。

公开组件包括：

```python
DreamConfig
DreamModel
DreamTokenizer
DreamSampler
DreamSamplerConfig
DreamTrainer
```

它负责 Dream 的模型配置、模型结构、tokenizer、训练器和采样器。

对应入口：

```text
<repo>/examples/dream/
```

## 3. `pipelines/a2d/`

A2D 用于将传统自回归模型改造成扩散语言模型。

公开模型包括：

```python
A2DLlamaConfig
A2DLlamaLMHeadModel
A2DQwen2Config
A2DQwen2LMHeadModel
A2DQwen3Config
A2DQwen3LMHeadModel
```

支持的基础模型方向包括 LLaMA、Qwen2 和 Qwen3。

典型转换路径是：

```text
已有自回归模型
    ↓
加入扩散训练所需的模型行为
    ↓
使用 masked diffusion 或 block diffusion 训练
    ↓
得到扩散式生成模型
```

对应入口：

```text
<repo>/examples/a2d/
```

## 4. `pipelines/bert/`

BERT-Chat 方向。

目标是将 BERT、RoBERTa 或 ModernBERT 一类 encoder 模型微调成轻量聊天模型。

由于 BERT 不是传统 decoder-only 自回归模型，该 pipeline 主要利用 mask 预测和扩散式恢复实现生成。

对应入口：

```text
<repo>/examples/bert/
```

## 5. `pipelines/editflow/`

Edit Flow 支持编辑式扩散生成。

公开模型适配包括：

```python
EditFlowModernBertModel
EditFlowDreamModel
EditFlowLLaDAModel
EditFlowQwen2Model
EditFlowQwen3Model
```

并提供：

```python
EditFlowSampler
EditFlowSamplerConfig
EditFlowTrainer
```

与普通 masked diffusion 相比，Edit Flow 不只替换 mask token，还可以学习：

- 插入 token
- 删除 token
- 替换 token
- 修改已有文本

对应入口：

```text
<repo>/examples/editflow/
```

## 6. `pipelines/fastdllm/`

Fast-dLLM 推理加速支持。

包含：

```python
dream
llada
```

主要加速思路包括：

- 缓存已经稳定的内容
- 对高置信度 token 提前固定
- 减少每一步需要重新计算的内容

对应入口：

```text
<repo>/examples/fastdllm/
```

## 7. `pipelines/llada2/`

LLaDA2.0 推理支持。

公开组件包括：

```python
LLaDA2MoeConfig
LLaDA2MoeModelLM
LLaDA2Sampler
LLaDA2SamplerConfig
```

这个目录重点是 LLaDA2.0 的模型结构和专用采样逻辑。

对应入口：

```text
<repo>/examples/llada2/
```

## 8. `pipelines/llada21/`

LLaDA2.1 推理支持。

公开组件包括：

```python
LLaDA2MoeConfig
LLaDA2MoeModelLM
LLaDA21Sampler
LLaDA21SamplerConfig
```

它与 `llada2` 分开，表示 LLaDA2.1 需要单独的模型或采样适配。

对应入口：

```text
<repo>/examples/llada21/
```

## 9. `pipelines/rl/`

扩散语言模型的强化学习训练。

公开组件包括：

```python
DiffuGRPOConfig
DiffuGRPOTrainer
get_dataset_and_rewards
SUPPORTED_DATASETS
```

基本过程是：

```text
扩散模型生成多个答案
    ↓
根据任务规则计算 reward
    ↓
使用 GRPO 更新模型
```

README 中提到的任务包括 GSM8K、MATH、Countdown、Sudoku 和 Code。

对应入口：

```text
<repo>/examples/rl/
```

## 10. pipeline 与 core 的关系

```text
pipelines/llada/
    ├── 提供 LLaDA 模型结构
    ├── 选择 MDLM 或其他 sampler
    ├── 适配训练参数
    └── 适配评测接口

core/
    ├── 提供通用 trainer
    ├── 提供通用 sampler
    ├── 提供 scheduler
    └── 提供评测基类
```

因此新增一个模型时，通常需要新增一个 pipeline，而不需要重新实现整个训练框架。
