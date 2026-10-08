---
title: "数据、工具和运行时支持"
date: 2026-10-08
summary: "dllm/data、dllm/utils、dllm/tools 的职责"
weight: 4
---

> 本节由 AI 整理生成，仅供参考。

# 数据、工具和运行时支持

相关目录：

```text
<repo>/dllm/data/
<repo>/dllm/utils/
<repo>/dllm/tools/
```

## 1. `dllm/data/`

负责数据集加载，是训练流程的数据入口。

公开函数包括：

```python
load_pt_dataset
load_sft_dataset
```

### `load_pt_dataset`

用于预训练数据。适合大规模无监督文本或语言建模数据。

### `load_sft_dataset`

用于监督微调数据，通常处理包含 `messages` 字段的对话数据。

### 主要能力

- 加载 Hugging Face Dataset
- 解析数据集名称和切片参数
- 支持数据集拼接
- 支持 streaming
- 为 trainer 提供统一数据格式

## 2. `dllm/utils/`

这是跨模型复用的公共工具层。

主要模块：

```text
chat.py
collators.py
configs.py
data.py
models.py
sampling.py
utils.py
visualizers.py
```

## 3. `utils/configs.py`

定义命令行和训练所需的参数对象：

```python
ModelArguments
DataArguments
TrainingArguments
```

它们分别描述：

- 使用什么模型和 tokenizer
- 使用什么数据集
- 使用什么 batch size、学习率和训练策略

## 4. `utils/models.py`

提供统一的模型和 tokenizer 加载函数：

```python
get_model
get_tokenizer
```

它负责根据参数加载正确的模型，也可以处理：

- LoRA
- PEFT
- 4-bit 量化
- 基础模型目录
- 不同 pipeline 的模型类

## 5. `utils/data.py`

负责将原始数据转换成训练样本。

公开函数包括：

```python
clip_row
clip_row_streaming
default_sft_map_fn
post_process_dataset
post_process_dataset_streaming
prepend_bos
tokenize_and_group
```

主要工作：

- 将 messages 转成 token ids
- 构造 labels
- 对 prompt 部分进行 loss masking
- 截断过长样本
- 拼接预训练文本
- 支持 streaming 数据处理
- 处理 BOS token

## 6. `utils/collators.py`

负责将单个样本整理成 batch。

公开组件包括：

```python
CollatorWrapper
NoAttentionMaskWrapper
PrependBOSWrapper
RandomTruncateWrapper
```

主要处理：

- padding
- attention mask
- BOS token
- 随机截断
- 不同模型之间的数据格式差异

## 7. `utils/sampling.py`

负责生成结果的后处理。

公开函数包括：

```python
sample_trim
infill_trim
```

扩散模型输出中可能还包含：

- prompt
- mask token
- 辅助位置
- 填充 token

这些函数用于提取最终有效文本。

## 8. `utils/chat.py`

负责交互式聊天功能。

功能包括：

- 构造聊天输入
- 单轮采样
- 多轮对话
- 渲染终端菜单
- 展示对话历史
- 文本换行

它主要服务于 `examples/llada/chat.py` 等交互式入口。

## 9. `utils/visualizers.py`

负责把采样过程可视化。

公开类包括：

```python
BaseVisualizer
TerminalVisualizer
VideoVisualizer
```

用途：

- 在终端查看 token 如何逐步恢复
- 调试采样过程
- 生成视频或演示效果

## 10. `utils/utils.py`

提供运行时和训练辅助函数，例如：

```python
initial_training_setup
init_device_context_manager
load_peft
parse_spec
print_args
print_args_main
print_main
pprint_main
resolve_with_base_env
```

还包括：

- 数据集缓存开关
- 日志初始化
- 参数打印
- CUDA 内存分配器设置
- 环境变量解析
- 多进程训练初始化

## 11. `dllm/tools/`

这里放独立的辅助脚本。

已知脚本：

```text
<repo>/dllm/tools/preprocess_sft_dataset.py
```

它用于离线预处理 SFT 数据：

```text
原始数据集
    ↓
读取 tokenizer
    ↓
调用指定 map function
    ↓
生成 input_ids、labels 等字段
    ↓
可选删除无关字段
    ↓
save_to_disk()
```

这样训练时可以直接加载已经 tokenized 的数据，避免每次训练重复处理。

该脚本还支持：

- 自定义映射函数
- 多进程处理
- prompt loss masking
- 保存 DatasetDict
- 只保留必要字段
