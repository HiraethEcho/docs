---
title: "dllm/core 通用组件"
date: 2026-10-08
summary: "samplers、schedulers、trainers、eval 四个通用模块"
weight: 2
---

> 本节由 AI 整理生成，仅供参考。

# `dllm/core`：通用扩散语言模型组件

目录：

```text
<repo>/dllm/core/
├── samplers/
├── schedulers/
├── trainers/
└── eval/
```

`core` 不针对单个模型，而是提供多个 pipeline 可以复用的通用算法组件。

## 1. `core/samplers/`

负责推理阶段的逐步采样。

公开组件包括：

```python
BaseSampler
BaseSamplerConfig
BaseSamplerOutput
MDLMSampler
MDLMSamplerConfig
BD3LMSampler
BD3LMSamplerConfig
```

### BaseSampler

定义采样器的通用接口、配置和输出格式。

### MDLMSampler

用于 Masked Diffusion Language Model。它通常会：

1. 接收包含 mask 的输入序列
2. 调用模型预测每个位置的 token 分布
3. 根据置信度或采样策略选择要恢复的 token
4. 更新序列中的 mask
5. 重复多个 diffusion steps
6. 返回最终序列

### BD3LMSampler

用于 Block Diffusion。它按 block 组织生成过程，而不是只对整条序列进行统一的 mask 恢复。

### 采样工具

`add_gumbel_noise` 用于增加 Gumbel 噪声，帮助进行随机采样或基于 logits 的排序。

`get_num_transfer_tokens` 用于决定每一步应该恢复多少 token。

## 2. `core/schedulers/`

负责控制扩散过程中的时间变化。

### Alpha scheduler

```python
BaseAlphaScheduler
LinearAlphaScheduler
CosineAlphaScheduler
```

用于提供 alpha 随扩散时间变化的函数。线性和 cosine 调度分别表示不同的变化速度。

### Kappa scheduler

```python
BaseKappaScheduler
LinearKappaScheduler
CosineKappaScheduler
CubicKappaScheduler
```

用于控制另一个与扩散强度、token 恢复或噪声变化有关的时间函数。

### 工厂方法

```python
get_alpha_scheduler_class
make_alpha_scheduler
get_kappa_scheduler_class
make_kappa_scheduler
```

这些方法允许通过配置名称选择调度器，而不是在业务代码中手动判断具体类。

## 3. `core/trainers/`

负责训练扩散语言模型。

公开组件包括：

```python
MDLMConfig
MDLMTrainer
BD3LMConfig
BD3LMTrainer
```

### MDLMTrainer

用于 masked diffusion 训练。典型过程是：

1. 对输入 token 进行 mask 或噪声处理
2. 把处理后的序列送入模型
3. 让模型预测原始 token
4. 根据预测和原始标签计算 loss
5. 使用 Transformers Trainer 体系进行反向传播和保存

### BD3LMTrainer

用于 block diffusion 训练。它将扩散训练过程组织成 block 级别的预测和恢复。

### Trainer 的定位

Trainer 负责训练流程，但不负责具体模型结构。模型结构由 `dllm/pipelines/` 中的模型实现提供。

## 4. `core/eval/`

负责将扩散模型接入标准评测框架。

公开组件包括：

```python
BaseEvalConfig
BaseEvalHarness
MDLMEvalConfig
MDLMEvalHarness
MDLMEvalSamplerConfig
BD3LMEvalConfig
BD3LMEvalHarness
BD3LMEvalSamplerConfig
```

扩散模型不能简单照搬自回归模型的 `generate()` 逻辑，因此评测层需要：

- 调用正确的 diffusion sampler
- 设置 diffusion steps
- 处理最大生成长度
- 支持 chat template
- 支持 few-shot 输入
- 将结果转换为评测框架需要的格式

## 5. `core` 的依赖关系

```text
schedulers  → 控制扩散时间函数
samplers   → 控制推理时如何逐步恢复文本
trainers   → 控制训练时如何构造扩散任务和计算 loss
eval       → 将 sampler 和模型包装给 benchmark 使用
```

简单说：

- `trainers` 解决“如何学”
- `samplers` 解决“如何生成”
- `schedulers` 解决“每一步怎么变化”
- `eval` 解决“如何公平评测”
