---
title: "分词器 / Tokenizer"
date: 2026-10-08
summary: "词表 6400 的来源、两套 API 的分工与实测更正"
weight: 2
---

> 本节由 AI 整理生成，仅供参考。

# 分词器 / Tokenizer

> **代码状态(2026-09-28)。** 旧仓库的 `src/pretrain/`(3,477 行、22 项测试通过、
> 真跑过 2000 万 token)已被删除,完整归档在
> `archive/llm-pre-restructure.tar.gz`。本仓库现在的 `src/` **只有注释**,
> 代码由用户写。下文出现的 `src/*.py` 是**目标位置**,文件可能还不存在。
> **所有实测数字仍然有效** —— 它们描述的是语料、硬件和参考实现,不是那份被删的实现。


回答四个问题:分词器是什么、"原生 API"和"HF API"分别是什么、我们的代码哪里用哪个、
以及为什么不能只用一个。

---

## 1. 分词器 = tokenizer

**同一个东西。** 英文 `tokenizer`,中文"分词器"。

它干三件事:

| 职责 | 干什么 | 实测例子 |
|---|---|---|
| **编码** | 文字 → 一串整数 | `'人工智能'` → `[2225, 3593, 2886, 1950, 302]` |
| **解码** | 一串整数 → 文字 | 反向还原,byte-exact |
| **持词表** | 记住全部 6400 个 token 和合并规则 | 就是 440 KB 的 `tokenizer.json` |

为什么必须有它:**模型从头到尾没见过一个汉字,它只见 `[2225, 3593, ...]`。**

### 命名陷阱:它不按"词"切

"分词"听起来像按词切,但它不按词。它按**字节**切,再看哪些字节常一起出现:

```
'人工智能'      → 1 个 token   [2225]
'Transformer'  → 3 个 token   ['Trans', 'form', 'er']
'龘'(生僻字)    → 2 个 token   [2519, 282]
'🧠'            → 3 个 token   [4544, 136, 290]
```

真正的按词切是 jieba 那种,完全另一回事。这个东西的准确名字是
**byte-level BPE 子词编码器**;"分词器"是中文圈的历史习惯叫法。

---

## 2. 词表 6400 是怎么来的

不是"训出来 6400 条",而是一个精确的加法(实测拆解):

```
  36   特殊 token     ← <|im_start|> <|im_end|> <|endoftext|> 等,id 0..35
 256   byte 回退表    ← 全部 256 个字节各占一个位置
6108   学到的 merge   ← BPE 真正训练出来的部分
────
6400   ✓ 精确相等
```

那 256 个字节位是关键:**任何输入都能编码,所以没有 OOV。**

```python
unk token: None          # 根本没有 unk
```

最坏情况 = 1 个 token 表示 1 个字节。这就是 **byte-level** 的含义。

### 特殊 token 的编号是契约,不能改

| id | token | 角色 |
|---|---|---|
| **0** | `<\|endoftext\|>` | **PAD**(也是 unk) |
| **1** | `<\|im_start\|>` | **BOS** |
| **2** | `<\|im_end\|>` | **EOS** |
| 3–20 | vision / audio / tts | 多模态,预训练用不上 |
| 21–26 | `tool_call` / `think` | 后训练用 |
| 27–35 | `<\|buffer1\|>`…`<\|buffer9\|>` | 预留空位 |

**`PAD(0) != EOS(2)` 这一点很关键**:所以在 labels 里把 pad 掩成 `-100`
不会顺手把真正的句尾目标也掩掉。见 `04-data.md`。

### 实测压缩率

```
'Transformer 通过自注意力机制建模上下文关系。'   28 字符 → 14 tokens   2.00 字符/token
'人工智能正在改变世界。'                          11 字符 →  5 tokens   2.20
'The quick brown fox jumps over the lazy dog.'   44 字符 → 18 tokens   2.44
```

README 声称中文 `1.5~1.7 字符/token`。上面是**短样本**,压缩率天然更差
(merge 机会少),所以不矛盾。真数据上要自己测一次。

---

## 3. 两个 API:同一个文件,两套外壳

它们**读同一份 `tokenizer.json`**。区别只是外层包装。

```
你的代码
   │
   ├─ AutoTokenizer.from_pretrained(dir)     ← HF API      (transformers 库)
   │      └─ 内部就持有一个 ↓
   │
   └─ Tokenizer.from_file(path)              ← 原生 API    (tokenizers 库, Rust)
          └─ 读同一份 tokenizer.json
```

**`PreTrainedTokenizerFast` 内部包着一个 `tokenizers.Tokenizer`。**
不是两个竞争的东西,是一层包一层。

### 名字先分清(最容易搞混的一步)

| 名字 | 来自 | 是什么 |
|---|---|---|
| `tokenizers` | 库名,小写复数 | HuggingFace 的**底层**库,Rust 实现,快 |
| `Tokenizer` | 上面的库 | 里面的**类**。`Tokenizer.from_file()` = "原生 API" |
| `transformers` | 另一个库 | HuggingFace 的**上层**库,装模型的那个 |
| `AutoTokenizer` | 上面的库 | 里面的类。`AutoTokenizer.from_pretrained()` = "HF API" |
| `PreTrainedTokenizerFast` | `transformers` | `AutoTokenizer` 实际返回的类型 |

### 同一件事,两种写法

| 想做的事 | 原生 API(`tokenizers`) | HF API(`transformers`) |
|---|---|---|
| 加载 | `Tokenizer.from_file("tokenizer.json")` | `AutoTokenizer.from_pretrained("dir/")` |
| 编码 | `tk.encode(s).ids` | `tk(s).input_ids` |
| 不进特殊符 | `tk.encode(s, add_special_tokens=False).ids` | `tk(s, add_special_tokens=False).input_ids` |
| 解码 | `tk.decode(ids)` | `tk.decode(ids)` |
| 词表大小 | `tk.get_vocab_size()` | `tk.vocab_size` |
| BOS 编号 | `tk.token_to_id("<\|im_start\|>")` | `tk.bos_token_id` |
| **套 chat 模板** | ❌ **没有** | ✅ `tk.apply_chat_template(...)` |
| 直接出张量 | ❌ 自己 `torch.tensor` | ✅ `return_tensors="pt"` |
| 速度 | 快 | 略慢(多一层 Python) |

注意两种"可调用性"的差别:

```python
tk(s)          # 原生:✗ 不可调用。Tokenizer 不是 callable
tk.encode(s)   # 原生:✓ 必须用这个方法
tk(s)          # HF:✓ 可以当函数调
```

---

## 4. 我们的代码哪里用哪个

| 文件 | 用哪个 | 为什么 |
|---|---|---|
| `src/data.py:load_tokenizer`(**待写**) | **原生** | 预训练只要 encode/decode。3.3 亿 token,越快越好 |
| `minimind/dataset/lm_dataset.py`(SFT/DPO/RL) | **HF** | 需要 chat 模板和 `.bos_token_id`。上游原样 vendored |

`minimind/dataset/lm_dataset.py:109` 是典型:

```python
self.bos_id = tokenizer(f'{tokenizer.bos_token}assistant\n',
                        add_special_tokens=False).input_ids
#                 ↑ 把 tokenizer 当函数调(HF 专有)  ↑ .bos_token 属性(HF 专有)
```

```python
# dataset.py:84
return self.tokenizer.apply_chat_template(messages, tokenize=False, ...)
#                     ↑ HF 专有。原生 API 没有
```

**所以不是"哪个更好",是"哪一层需要什么"。**

---

## 5. 实测更正:chat 模板不能靠原生 API

我最初打算"把 `dataset.py` 的 15 个分词器调用点从 HF API 改成原生 API"。**这个方案是错的。**

本机实测(`tokenizers` **0.23.2**):

```
Tokenizer 有 apply_chat_template ?  False     ← 原生 API 没有这个能力
tokenizer_config.json 有 chat_template ?  True   ← 3895 字符的 Jinja 模板
```

`apply_chat_template` 在原生 API 上**不存在**。所以那不是改个方法名,
而是要**自己重新实现 chat 模板** —— 而 chat 模板是 3895 字符的 Jinja,
自己重写等于制造一个会静默出错的第二实现。

### 正确做法:两个加载器,各司其职

| 用途 | 加载器 | 理由 |
|---|---|---|
| 预训练(只要 encode/decode) | `src/data.py:load_tokenizer` ⚠️待写 → 原生 | 快,不模板化 |
| SFT / DPO / RL(要 chat 模板) | `AutoTokenizer.from_pretrained("tokenizer")` → HF | 模板在 config 里,别重写 |

上游的 `minimind/dataset/lm_dataset.py` **需要 HF 接口**。如果以后要给我们的管线
接上 SFT/DPO/RL,就得准备两个 tokenizer:原生给预训练,HF 给这套 chat 数据集。
它本身一行都不用改 —— 只要构造时传对。

> 这不是"一份加载器统一天下",而是**按层选接口**。想省一份加载器而重写
> chat 模板,是拿正确性换整洁。

---

## 6. 分词器该放哪

**`tokenizer/`,进版本控制。** 不在 `<数据盘>`。

理由:440 KB,不是一个会变化的东西,而**项目里每个数字都依赖它**
——63.9M 参数量、与参考实现的 logit 对比、每个 token 计数。
`<数据盘>` 上的东西可以丢、可以删、可以重下;这个词表不行。

```bash
cp minimind/model/tokenizer.json \
   minimind/model/tokenizer_config.json \
   ~/Projects/llm/tokenizer/
```

⚠️ 仓库里现在有**两份**,md5 相同(`2fa841c1d5632b846e05a7fd241a4287`):
`tokenizer/`(我们的,代码用这份)和 `minimind/model/`
(参考副本自带,它的代码按 `model/tokenizer.json` 找词表)。
保留两份是为了让 `minimind/` 自包含、可以直接跑。**要改只改
`tokenizer/`**,另一份跟着上游走。

**不要重训。** minimind 的 `train_tokenizer.py` 开头就写着不建议重训:
不同词表的模型输出完全不统一。而且换词表会让 63.9M 参数量和与官方权重的
对比全部失效。
