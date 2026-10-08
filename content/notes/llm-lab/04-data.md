---
title: "数据管线 / Data"
date: 2026-10-08
summary: "从 JSONL 语料到 (input_ids, labels) 张量，含实测数字"
weight: 4
---

> 本节由 AI 整理生成，仅供参考。

# 数据管线 / Data

> **代码状态(2026-09-28)。** 旧仓库的 `src/pretrain/`(3,477 行、22 项测试通过、
> 真跑过 2000 万 token)已被删除,完整归档在
> `archive/llm-pre-restructure.tar.gz`。本仓库现在的 `src/` **只有注释**,
> 代码由用户写。下文出现的 `src/*.py` 是**目标位置**,文件可能还不存在。
> **所有实测数字仍然有效** —— 它们描述的是语料、硬件和参考实现,不是那份被删的实现。


从 JSONL 语料到 `(input_ids, labels)` 张量。含实测数字与两处估算更正。

---

## 1. 语料清单

托管在 [ModelScope `gongjy/minimind_dataset`](https://www.modelscope.cn/datasets/gongjy/minimind_dataset/files) /
[HF 镜像 `jingyaogong/minimind_dataset`](https://huggingface.co/datasets/jingyaogong/minimind_dataset)。

| 文件 | 大小 | 阶段 |
|---|---|---|
| `pretrain_t2t_mini.jsonl` | 1.2 GB | 预训练 ← **已在 `data/raw/`(实测 1,183.6 MB)** |
| `pretrain_t2t.jsonl` | 10 GB | 完整预训练,别下 |
| `sft_t2t_mini.jsonl` | 1.6 GB | SFT |
| `sft_t2t.jsonl` | 14 GB | 完整 SFT |
| `rlaif.jsonl` | 24 MB | PPO / GRPO |
| `dpo.jsonl` | 53 MB | DPO |
| `agent_rl.jsonl` | 86 MB | Agent RL |
| `agent_rl_math.jsonl` | 18 MB | Agent RL(纯数学) |

下载:

```bash
cd ~/Projects/llm
# `--repo-type dataset` 是必需的 —— 不加会去同名的 *模型* 仓库找,404
hf download jingyaogong/minimind_dataset pretrain_t2t_mini.jsonl \
    --repo-type dataset --local-dir data/raw
```

**数据一律放数据盘(`data/` 是它的软链),绝不放 `~/Public`(NTFS,Windows 会删)。**

---

## 2. 格式:一行 = 一个样本,没有别的

```jsonl
{"text": "如何才能摆脱拖延症？治愈拖延症并不容易，但以下建议可能有所帮助。"}
{"text": "Transformer 通过自注意力机制建模上下文关系，是现代大语言模型的重要基础结构。"}
```

预训练数据是**纯续写**:没有 `role`、没有指令、没有答案字段。
数据已经清洗、去重、控长度、统一格式 —— 不需要写任何清洗代码。

来源:通用文本 + 对话整理 + 蒸馏补充 + 宽松协议开源集。

---

## 3. 实测:语料到底有多大

`data/prepared/minimind3-6400/`(由**已归档**的 `prepare_data.py` 生成,输出仍在 `<数据盘>`):

| 文件 | 实测 |
|---|---|
| `tokens.bin` | uint16 扁平流,**665 MB / 332,495,324 个 token** |
| `doc_lengths.npy` | int32,**1,270,238 篇**,最短 4 / 最长 11,802 / 平均 262 token |
| `meta.json` | vocab 6400、val_frac 0.01、`source_sha256` |

三个数字自洽:`doc_lengths.sum() == tokens.bin / 2 == 332,495,324`。
这条等式是 `TokenStream.__init__` 现在会强制的检查 —— 不匹配说明两者不来自同一次运行,
之后切出来的每个窗口都会静默错位。

### 两处估算更正(教训:先估再测)

| | 我的估算 | 实测 | 偏差 |
|---|---|---|---|
| 文档数 | ~2,150,000 行 | **1,270,238 篇** | 少 1.7× |
| 平均长度 | ~100 token | **262 token** | 少 2.6× |
| 总 token | ~2.15 亿 | **3.32 亿** | 少 1.5× |

估算建立在"平均行长 200 字符"上,而真实文档明显更长。
**估算可以用来排期,不可以用来定 `max_seq_len` 或 token 预算。**

### uint16 为什么够

词表 6400 < 65536,所以每个 token 2 字节即可。对比:

```
JSONL  1,183.6 MB   ← 原始文本
uint16   665.0 MB   ← 分词后,且不必每 epoch 重新分词
```

---

## 4. 一行文本 → 一个训练 batch(4 步)

```python
def __getitem__(self, index):
    text = str(self.samples[index]["text"])

    # 步 1:编码,给 BOS/EOS 留 2 个位置
    ids = self.tokenizer.encode(text, add_special_tokens=False).ids
    ids = ids[: self.max_length - 2]            # 338,不是 340

    # 步 2:手动加 BOS(1) 和 EOS(2)
    tokens = [BOS_TOKEN_ID] + ids + [EOS_TOKEN_ID]

    # 步 3:右侧用 PAD(0) 补到定长
    input_ids = torch.tensor(tokens + [PAD_TOKEN_ID] * (self.max_length - len(tokens)))

    # 步 4:labels = 副本,pad 位置改成 -100
    labels = input_ids.clone()
    labels[input_ids == PAD_TOKEN_ID] = -100

    return input_ids, labels
```

### 每一步为什么这么写

**步 1 —— `max_length - 2`,不是 340。** 步 2 还要塞 2 个;先截到 340 会正好超出
序列长度 2 个 token。`max_length - 2` 不是凑数。

`add_special_tokens=False` 是必须的:配置里 `add_bos_token=false, add_eos_token=false`,
分词器自己不会加,得你来。

**步 2 —— 手动加 BOS/EOS。** 这一句就是告诉模型"这是一段完整文本"。没有它,
模型学不到什么时候该停。

**步 3 —— 补到定长**,因为 GPU 要方形张量。

**步 4 —— labels 不是"答案",而是同一个序列。** 这是最容易搞混的地方:

```
input_ids : [BOS] 白 日 依 山 尽 [EOS] [PAD] [PAD]
labels    : 白 日 依 山 尽 [EOS] [-100] [-100] [-100]
             ↑ 每个位置要预测的 = 下一个位置的内容
```

因果语言模型的目标就是输入本身,不需要单独的答案字段。
**错位一格发生在模型里**,见下一节。

### 为什么 pad 要掩成 -100

`cross_entropy(..., ignore_index=-100)` 会跳过它们:既不产生梯度,也不消耗损失。

**如果掩成 0**,模型就会花力气去学"预测 pad"。语料里短句占多数,
pad 占比很高,模型会学出一个"只会输出 0"的退化解。

**`PAD(0) != EOS(2)` 就是在这里救命的** —— 掩 pad 不会顺手把真正的句尾目标掩掉。

---

## 5. 错位一格发生在哪

**不在数据里,在 `CausalLM.forward` 里:**

```python
shift_logits = logits[:, :-1, :]     # 丢掉位置 0(没有东西预测它)
shift_labels = labels[:, 1:]         # 丢掉位置 T-1(没有东西由它预测)
loss = F.cross_entropy(shift_logits, shift_labels, ignore_index=-100)
```

**两边切反是一个静默、看起来很合理的 bug**:损失照样下降,只是朝着
"预测上一个 token"的目标下降。`test_model.py:test_shift_direction` 专门测这个。

所以数据层让 `labels == input_ids`(对打包批而言),是刻意的:
**批次对象对"损失怎么算"不持观点,那是模型的事。**

---

## 6. padding vs packing

minimind 的做法是"一篇文档一行,截断到 T,右侧补 PAD"。本机实测的代价:

| | pack(默认) | pad(等同 minimind) |
|---|---|---|
| 吞吐 | **5,501–5,750 tok/s** | 3,109–3,569 tok/s |
| 差距 | — | **1.71×** |

原因,在本语料上是实测得出的:

```
平均文档长度 262 token
最长 11,802 token            ← 在 T=340 下尾巴永久丢失
→ 每次前向 38.4% 的算力花在 PAD 上,20% 的语料被丢弃
```

**packing 的做法:窗口长度恰好 T,取自语料任意位置,可以跨文档边界。**
跨边界是**有意的** —— 流里已在文档之间带着 `[BOS]...[EOS]`,
所以模型看得到边界并学会使用它。而且它零浪费、零截断。

---

## 7. 磁盘上的 token 流长什么样

```
[BOS] 文档1 [EOS] [BOS] 文档2 [EOS] [BOS] 文档3 [EOS] ...
  ↑ 1            ↑ 2   ↑ 1            ↑ 2
```

每个文档由 BOS(1) 和 EOS(2) 包起来。**流里没有任何 PAD** ——
填充是**分批的策略**,不是**存储的格式**。

`doc_lengths.npy` 记每篇文档的长度(**含那 2 个特殊符**),
`doc_starts` 是它的累积和,所以文档 d 占据 `[doc_starts[d], doc_starts[d+1])`。

### 为什么值得做二进制中间产物

| 好处 | 说明 |
|---|---|
| 一次性 | 第一次代价与每 epoch 重分词相同,之后是零 |
| 可复现 | token 序列由源文件 sha256 唯一确定(`meta.json` 记录) |
| 加载快 | memmap,665 MB 不必进 Python 内存 |
| 切批快 | 一次 numpy 花式索引就取出 `[B, T]`,不必 Python 循环 |

> ⚠️ 这是**对 minimind 的一处刻意偏离**。他们 README 明说为了文本完整性
> 放弃了 `.bin` 预分词(轻微牺牲速度)。两条路都合理 —— 但要**有意识地选**。

---

## 8. 时间预算

```
实测稳态吞吐 3,700 tok/s   (2,940 ms/step, bs32 × T340, 打包)
3.32 亿 token 跑满一遍 = ~25 小时        ← 不是 1 小时,也不是 16.5 小时
计划预算 2,000 万 token      = ~93 分钟 (1,838 步)
```

**所以一次运行必须显式设上限**(`--max_tokens` 或 `--max_steps`)。
默认不限步数等于默认跑 25 小时。

⚠️ **这个数字曾经是错的。** 第一次测出 5,736 tok/s / 1,897 ms/step,
只跑了 28 步——全部落在前 ~300 步的快相里。完整 20M token 跑完实测 93.2 分钟,
比预测慢 1.55 倍。详见 `05-measurement.md`。
先估再测可以,但**估完必须用完整长度的运行去验**。

---

## 9. 相关文件

| 文件 | 作用 |
|---|---|
| `prepare_data.py`(**已归档**,只跑一次) | JSONL → uint16 流(已完成,输出在 `<数据盘>`,不需要再跑) |
| `src/data.py`(**待写**) | `TokenStream` / `PackedBatcher` / `PaddedBatcher` |
| `minimind/dataset/lm_dataset.py` | 上游的 `SFTDataset` / `DPODataset` / `RLAIFDataset` / `AgentRLDataset`(已 vendored,原样) |
| `minimind/trainer/train_pretrain.py` | 上游预训练循环,当对照物 |
