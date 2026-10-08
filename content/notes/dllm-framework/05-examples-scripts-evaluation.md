---
title: "examples、scripts 和评测"
date: 2026-10-08
summary: "运行入口、多卡配置、测试与评测组件"
weight: 5
---

> 本节由 AI 整理生成，仅供参考。

# examples、scripts 和评测组件

## 1. `examples/`

目录：

```text
<repo>/examples/
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

这是面向使用者的入口层。它们负责把模型、数据、训练器和 sampler 组合起来。

底层实现主要位于：

```text
<repo>/dllm/
```

## 2. `examples/llada/`

典型文件包括：

```text
chat.py
sample.py
pt.py
sft.py
eval.sh
README.md
```

职责：

- `chat.py`：交互式多轮聊天
- `sample.py`：普通推理
- `pt.py`：预训练
- `sft.py`：监督微调
- `eval.sh`：批量评测
- `README.md`：LLaDA 专用说明

## 3. 其他 examples 目录

| 目录 | 用途 |
|---|---|
| `<repo>/examples/a2d/` | 将自回归模型改造成扩散模型 |
| `<repo>/examples/bert/` | BERT-Chat 训练和推理 |
| `<repo>/examples/dream/` | Dream 训练、推理和评测 |
| `<repo>/examples/editflow/` | Edit Flow 训练和采样 |
| `<repo>/examples/fastdllm/` | Fast-dLLM 加速推理和评测 |
| `<repo>/examples/llada2/` | LLaDA2.0 推理 |
| `<repo>/examples/llada21/` | LLaDA2.1 推理 |
| `<repo>/examples/rl/` | diffu-GRPO 强化学习训练 |

## 4. `scripts/`

目录：

```text
<repo>/scripts/
├── accelerate_configs/
├── tests/
└── train.slurm.sh
```

## 5. `scripts/accelerate_configs/`

放置 Hugging Face Accelerate 配置，用于控制分布式训练方式。

README 中提到的训练方式包括：

```text
ddp
zero-1
zero-2
zero-3
fsdp
```

这些配置决定：

- GPU 进程数量
- 多机数量
- 混合精度
- 参数如何切分
- 梯度如何同步
- optimizer state 如何分布
- 是否启用 FSDP
- 是否启用 DeepSpeed ZeRO

例如 FSDP 配置中可以看到：

- FULL_SHARD
- bf16 混合精度
- Transformer 自动 wrap
- 多进程训练
- 参数同步
- CPU RAM 高效加载

## 6. `scripts/train.slurm.sh`

这是 Slurm 集群启动脚本。

它负责：

1. 申请节点和 GPU
2. 读取 Slurm 分配的节点信息
3. 计算 GPU 总数
4. 设置 master address 和 port
5. 选择 Accelerate 配置
6. 调用 `accelerate launch`
7. 将其他参数转发给具体训练脚本
8. 保存任务日志

因此它主要解决“如何在集群上启动训练”，而不是“模型如何学习”。

## 7. `scripts/tests/`

`<repo>/pyproject.toml` 中将其配置为 pytest 测试目录：

```toml
testpaths = ["scripts/tests"]
python_files = ["test_*.py"]
```

因此测试文件通常命名为：

```text
test_*.py
```

## 8. `lm-evaluation-harness/`

这是一个外部评测框架子模块。

dLLM 自己负责扩散模型的推理适配，`lm-evaluation-harness` 负责标准 benchmark 的任务定义和评测执行。

README 提到的使用场景包括：

- MMLU-Pro
- IFEval
- 数学任务
- 其他标准语言模型 benchmark

两者的关系是：

```text
扩散模型专用 Eval Harness
    ↓
将模型包装成标准评测接口
    ↓
lm-evaluation-harness
    ↓
执行 benchmark 并统计结果
```

## 9. `assets/`

用于项目展示和文档资源，例如：

- logo
- README 动图
- 聊天效果演示
- Edit Flow 采样可视化

这些资源通常不参与训练和推理。

## 10. `pyproject.toml`

这是 Python 项目配置文件，负责：

- 项目名和版本
- Python 版本要求
- 依赖包
- 可选依赖
- Black 格式化配置
- pytest 配置

核心依赖涉及：

- PyTorch
- Transformers
- Accelerate
- DeepSpeed
- PEFT
- Datasets
- SentencePiece
- WandB
- Tyro
- OmegaConf

可选依赖包括：

- bitsandbytes：量化
- vLLM：推理加速相关能力
- flash-attn：Flash Attention

## 11. 推荐阅读顺序

如果想快速理解代码，建议按以下顺序阅读：

1. `<repo>/README.md`
2. `<repo>/examples/llada/sft.py`
3. `<repo>/dllm/utils/configs.py`
4. `<repo>/dllm/utils/models.py`
5. `<repo>/dllm/data/`
6. `<repo>/dllm/core/trainers/`
7. `<repo>/dllm/core/samplers/`
8. `<repo>/dllm/pipelines/llada/`
9. `<repo>/scripts/accelerate_configs/`

这个顺序是从“用户入口”逐步进入“公共框架”和“模型细节”。
