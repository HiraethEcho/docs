---
title: "MiniMind — 项目解剖"
date: 2026-10-08
summary: "上游项目的结构：trainer 阶梯、model 与 dataset 的取舍"
weight: 6
---

> 本节由 AI 整理生成，仅供参考。

# MiniMind — 项目解剖

参考实现已 **vendored 到仓库根的 `minimind/`**(和 `src/` 同级)。
上游在 `~/Public/minimind`
读仓库里这份,不要读那份。
本文回答三个问题:它是什么、`trainer/` 九个脚本各训什么、`model/` 和 `dataset/` 该不该复制。

> **行数的写法。** 出现两个数字的地方一律是 **上游原始 / 本仓库**。
> 本仓库那份是 vendored 后**逐行加过注释**的,所以第二列更大。
> **差值就是注释量**,不是代码差异 —— 2026-09-28 实测:用 `ast.unparse` 剥离注释和
> docstring 后逐文件比对,**22/22 逐字节相同**。

---

## 1. 它是什么

一个**教学用**的从零 LLM 项目。两个主线模型,结构刻意对齐 Qwen3 生态
(方便转 `transformers` / `llama.cpp` / `ollama` / `vllm`)。

| Model | params | d_model | n_layers | 说明 |
|---|---|---|---|---|
| **minimind-3** | 64M | 768 | 8 | Dense ← 我们复现的目标 |
| **minimind-3-moe** | 198M-A64M | 768 | 8 | 4 experts / top-1 |
| minimind2 | 104M | 768 | 16 | 历史版本 |
| minimind2-moe | 145M | 640 | 8 | 历史版本 |
| minimind2-small | 26M | 512 | 8 | 历史版本 |

架构要点:Pre-Norm + RMSNorm · SwiGLU · RoPE θ=1e6(支持 YaRN 外推)·
GQA 8/4 · QK-norm · tied embeddings · `max_position_embeddings=32768` · vocab 6400。

**参数量精确值**(可解析验算,见 `src/config.py:param_breakdown`):

```
dense  63,912,192          # 实测 == 解析值
MoE   198,416,640          # 63.9M active
```

为什么 `d_model=768 / 8 层`:README 引 MobileLLM —— 参数固定时**深度比宽度重要**。
但 `d_model < 512` 时词嵌入太窄会明显吃亏;`> 1536` 则加层比加宽更划算。

**框架:纯 PyTorch。** 全仓库 import 计数:`67 × torch` / `14 × transformers` /
**`0 × trl`** / **`0 × peft`** / **`0 × HF Trainer`**。

`transformers` 只当接口层用 4 个地方,不是训练框架:

| 用处 | 为什么 |
|---|---|
| `PreTrainedModel` / `GenerationMixin` / `PretrainedConfig` | 让 `from_pretrained()` / `.generate()` 能用 |
| `ACT2FN` | 一个 `{'silu': nn.SiLU()}` 查表 |
| `MoeCausalLMOutputWithPast` | 一个返回值容器 |

LoRA 也是自己写的(`model/model_lora.py`,65 行),不用 `peft`。

> `requirements.txt` 是一份**混杂四类东西**的清单(训练 + 网页 demo + 数据清洗 + 蒸馏),
> 而且过期(`transformers==4.57.6` 与代码里的 5.x 兼容代码矛盾,`torch` 被注释掉)。
> `AGENTS.md` 里"忽略 `~/Public` 下所有 requirements.txt"就是这个原因。

---

## 2. `trainer/` — 一棵阶梯

**九个脚本的循环骨架逐字相同,变的只有两件事:数据长什么样,和 loss 怎么算。**

```python
for step, batch in enumerate(loader):
    lr = get_lr(epoch*iters + step, args.epochs*iters, 5e-4)   # 同一套余弦,无预热
    with autocast:
        loss = <─── 唯一的区别 ───>
    scaler.scale(loss).backward()
    if step % accum == 0:
        unscale / clip_grad_norm_(1.0) / step / zero_grad      # 逐字相同
    if step % log_interval == 0:  log
    if step % save_interval == 0: lm_checkpoint(...)
```

| # | 文件 | 上游 | 仓库 | 输入数据 | 优化的目标 | 产出 |
|---|---|---|---|---|---|---|
| 0 | `train_tokenizer` | 189 | 211 | sft jsonl | 无(统计 BPE merge) | 词表 |
| 1 | `train_pretrain` | 164 | 216 | `{"text"}` | 预测下一个词 | `pretrain_768.pth` |
| 2 | `train_full_sft` | 165 | 185 | `{"conversations"}` | 预测**回答**的下一个词 | `full_sft_768.pth` |
| 3 | `train_lora` | 177 | 195 | 同上 | **同上**,只训 0.76% 参数 | `full_sft_lora_*.pth` |
| 4 | `train_distillation` | 239 | 260 | 同上 | 学生模仿教师整个概率分布 | `full_sft_768.pth` |
| 5 | `train_dpo` | 219 | 242 | `{"chosen","rejected"}` | 好回答比坏回答更可能 | `dpo_768.pth` |
| 6 | `train_ppo` | 448 | 473 | `rlaif`(只有题) | 规则分 + reward model,配 critic | `ppo_768.pth` |
| 7 | `train_grpo` | 327 | 347 | 同上 | 同上,**不用 critic** | `grpo_768.pth` |
| 8 | `train_agent` | 485 | 504 | `agent_rl` | 同上 + 多轮工具调用 | `agent_768.pth` |
| — | `rollout_engine` | 224 | 244 | — | **不训练**,生成回答的推理引擎 | — |
| — | `trainer_utils` | 209 | 258 | — | 共享层 | — |

`trainer/` 合计 **2,846 → 3,135 行**;其中 9 个 `train_*.py` 是 **2,224 → 2,422**。

### 谁 import 谁

2026-09-28 用 `ast` 从本仓库抽的真实 import 图(不是推测):

```
零本地依赖 ──┬── model/model_minimind.py      509L   模型 + MiniMindConfig 全在这一个文件里
             ├── model/model_lora.py          103L
             ├── dataset/lm_dataset.py        330L   只要 tokenizer.json
             ├── trainer/rollout_engine.py    244L   只有 RL 组用
             └── trainer/train_tokenizer.py   211L   我们不用
                          │
第 1 层 ──────────────────┴── trainer/trainer_utils.py  258L  → import model_minimind
                          │      get_lr / lm_checkpoint / init_model / Logger / …
                          ▼
第 2 层 ────────────────────── 8 个 train_*.py,每个只导出一个 train_epoch()
                          │     全都 import: lm_dataset + model + trainer_utils
                          ▼
第 3 层 ────────────────────── scripts/ 5 个 + eval_llm.py
```

**两个反直觉的地方:**

1. **`trainer_utils.py` 依赖模型** —— 它里面有 `init_model()` 和 `LMForRewardModel`。
   所以顺序是 **模型 → utils → 训练脚本**,不是反过来。
2. **`dataset/lm_dataset.py` 零依赖** —— 它只吃 `tokenizer.json`。
   所以它和模型**可以并行写,先后随便**。

### 分三组理解

**第一组(1 个)—— 学语言。** `train_pretrain` 从随机初始化开始,
`loss = cross_entropy(logits[:, :-1], labels[:, 1:])`,没有别的。

**第二组(3 个)—— 学听指令。这三个的 loss 完全一样,都是同一个 cross-entropy。**
区别只在两处:

1. **数据格式**:`SFTDataset` 套 chat 模板
   ```
   <|im_start|>user\n1+1等于几?<|im_end|>\n<|im_start|>assistant\n2<|im_end|>
   ```
2. **掩码**:`labels[:prompt_len] = -100`,**只在 assistant 回答上算损失**。
   这是关键 —— 否则模型会去学"怎么提问"。

| | `full_sft` | `lora` | `distillation` |
|---|---|---|---|
| 训哪些参数 | 全部 63.9M | 只 `B·A` | 全部(学生) |
| `optimizer` 传什么 | `model.parameters()` | **`lora_params`** | `model.parameters()` |
| 额外 | — | 基础权重冻结 | 另有教师模型(MoE) |

`lora` 的差别只有一行:`optim.AdamW(lora_params, ...)`。

`distillation` 是这组唯一 loss 不同的:

```python
loss = alpha * ce_loss + (1 - alpha) * distill_loss
#                 ↑硬标签          ↑软标签: T² · KL(学生/T ‖ 教师/T)
```

教师默认 `--teacher_use_moe 1 --from_teacher_weight full_sft` —— 即
**把 MoE 的容量蒸馏进稠密模型**。软标签带的信息比硬标签多:教师对错答案的
概率分布本身就是知识。

**第三组(4 个)—— 学"哪个回答更好"。从这里开始没有标准答案,目标变成优化一个分数。**

`train_dpo`(不需要 reward model、不需要采样,所以和 SFT 一样快):

```python
pi_logratios  = logπ(chosen)     - logπ(rejected)          # 当前模型
ref_logratios = logπ_ref(chosen) - logπ_ref(rejected)      # 冻结参考模型
loss = -logsigmoid(beta * (pi_logratios - ref_logratios))  # beta 默认 0.15
```

直觉:直接拉大"好答案相对坏答案"的概率差距,用冻结参考模型当锚防跑偏。
一个 batch 里 chosen/rejected 各占一半,靠 `[:B//2]` / `[B//2:]` 切开。

`train_grpo` 的关键只有一句 —— 用**组内相对**替掉整个 critic:

```python
grouped_rewards = rewards.view(-1, num_generations)          # [B, num_gen]
advantages = (rewards - grouped.mean(dim=1)) / (grouped.std(dim=1) + 1e-4)
```

同一道题生成 N 个回答,组内均值和标准差当 baseline。比 PPO 少 121 行,
少的基本就是 critic。之后仍是 PPO 那套 clip + KL。

`train_ppo` 有 critic,所以要两套优化器(actor + critic)、两套 lr,
外加 GAE:`delta = r + γ·V(t+1) - V(t)`。这就是 447 行花在哪。

`train_agent` = GRPO 的算法 + 多轮循环(吐 `<tool_call>` → 执行 →
结果作 `<tool_response>` 拼回 → 再生成)。reward 是多项加减后 clip 到 ±3:

```
-0.5 × 未闭合的 <tool_call> 标签数      标签扣分
+0.5 / -0.5      回答长度 5..800
+1.0 / -0.5      think 长度 20..300
+0.25 / -0.25    </think> 恰好出现 1 次
+ reward_model.get_score(...)           RM 分
- rep_penalty(answer)                   重复惩罚
+0.5 / -0.5×tool_gap                    工具对齐分
+2.5 × 命中标准答案的比例                 有 GT 时
-0.5             未完成
```

### 数据怎么流动

```
tokenizer.json ──────────── 已有,不训(官方劝退重训)
      │
      ▼
train_pretrain ──► pretrain_768.pth
      ▼
train_full_sft ──► full_sft_768.pth      ← 变成会对话
      ├──► train_lora
      ├──► train_distillation  (MoE 教师 → 稠密学生)
      ▼
train_dpo      ──► dpo_768.pth
      ├──► train_ppo    (有 critic)  ┐
      ├──► train_grpo   (无 critic)  ├── 需要 reward model
      └──► train_agent  (多轮工具)   ┘
```

**每一级吃上一级的产出。不能跳过 `full_sft` 直接跑 DPO** —— DPO 的参考模型
需要一个已经会对话的模型。

---

## 3. `model/` — 三个文件,只该要一个

| 文件 | 上游 | 仓库 | 是什么 | 复制? |
|---|---|---|---|---|
| `model_minimind.py` | 293 | 509 | 模型本体 | ❌ **不要**(见下) |
| `model_lora.py` | 65 | 103 | 手写 LoRA | ✅ 当参照物 |
| `tokenizer.json` + `tokenizer_config.json` | — | 453 KB | 词表 6400 | ✅ 已在 `tokenizer/`(仓库根) |

**为什么不要 `model_minimind.py`:** 我们实测两者**逐位相同**
(91 个 state_dict key 一致,`max |logit diff| = 0.000e+00`)。
再复制一份 = 仓库里两份模型定义 = `PLAN.md` 警告的那种混乱。
真身留在 `minimind/model/` 当参考读物,永远不当依赖。

**为什么 `model_lora.py` 值得复制:** 它是 Phase 3 的对照物
(`PLAN.md` 3.2 要求可训练参数量与 `peft` 完全一致)。三处偏离标准做法:

1. **只贴在方阵上** —— `in_features == out_features`,所以只有 `q_proj`
   和 `o_proj` 被加上;`k_proj`/`v_proj`(768→384)和 MLP 都不会。
   `peft` 默认 target 是 `q,k,v,o` → **参数量对不上**。
2. **没有 `alpha/r` 缩放** —— 就是 `B(A(x))`,等价于 `alpha = rank`。
3. **猴补丁改 `module.forward`** —— 所以到处是 `getattr(model, '_orig_mod', model)`
   来兼容 `torch.compile`。读得懂但不是惯用写法。

---

## 4. `dataset/` — 不是选择题,是硬依赖

| 文件 | 上游 | 仓库 | 是什么 | 复制? |
|---|---|---|---|---|
| `lm_dataset.py` | 260 | 330 | 5 个 Dataset 类 + 2 个对话处理函数 | ✅ **被迫** |
| `dataset.md` + `__init__.py` | — | — | 写着"把数据文件放这里" | ❌ 没价值 |

**为什么是被迫的:** `train_full_sft` / `train_lora` / `train_dpo` /
`train_grpo` / `train_ppo` / `train_agent` **六个文件**都
`from dataset.lm_dataset import ...`。不复制它们连 import 都过不去。

五个类各是一个训练阶段的"拼数据"逻辑:

| 类 | 输入格式 | 干什么 |
|---|---|---|
| `PretrainDataset` | `{"text": ...}` | 纯续写。已被 `src/data.py` 取代 |
| `SFTDataset` | `{"conversations":[{role,content}...]}` | 套 chat 模板,**把 prompt 掩成 -100** |
| `DPODataset` | chosen / rejected 成对 | 两边都套模板 |
| `RLAIFDataset` | 只有 prompt | 给 on-policy 生成用 |
| `AgentRLDataset` | 多轮 tool use | Phase 4/5 |

两个辅助函数:`pre_processing_chat`(20% 概率注入随机 system prompt)、
`post_processing_chat`(80% 概率删掉空的 `<think>\n\n</think>`)。

> **顺带发现:`SFTDataset` 里的 prompt 掩码,正是 `PLAN.md` 3.1 要我们
> 从零实现的那个 completion-only masking。** 所以 `dataset.py` 不只是脚手架,
> 它同时是 Phase 3 的对照物。

---

## 5. 上游已 vendored 进仓库

`minimind/` — **22 个 py 文件,4,667 → 5,352 行**(+685,全部是注释;
整个目录含 json/md 共 48 个文件)。上游自己的目录布局
(`trainer/` `dataset/` `model/` `scripts/`),所以与原版的 diff 只要一条命令:

```bash
diff ~/Public/minimind/trainer/train_dpo.py minimind/trainer/train_dpo.py
```

**为什么要 vendored:** `~/Public` 是 NTFS,Windows 那边已经删过一个仓库。
这份在 git 里,不会消失,而且能与我们的代码并排看。

| | |
|---|---|
| 模型 | `model/model_minimind.py` —— **上游的,不是我们的**。仍然加载 `MiniMindForCausalLM` |
| 路径 | **未适配** —— 仍是 `../dataset/`、`../out`、`../checkpoints` |
| 第三方 | `streamlit` / `vllm` / `ollama` / `lm-eval` 都没装 |

⚠️ **这是用来 diff 的参照物,不是拿来跑的。** 要跑用 `src/train.py`。

> 曾经有一份扁平化的适配移植版在 `src/minimind/`(17 文件、4,333 行、
> 每个文件距上游 5 处机械改动)。**已删除** —— vendored 副本让它变得多余。
> 要恢复得从归档取:`archive/llm-pre-restructure.tar.gz`。
> 旧仓库在 2026-09-28 被完整重建过(用户选择抹掉 `.git`),所以历史里的
> 提交号和 `git checkout HEAD~` 这类命令**已经全部失效**。

---

## 6. 对我们的意义

| `PLAN.md` 阶段 | 对应 trainer |
|---|---|
| Phase 2 预训练 | `train_pretrain` → 我们写 `src/train.py` |
| Phase 3 SFT | `train_full_sft` |
| Phase 3 LoRA | `train_lora`(自己写一个再和它对照) |
| Phase 4 RL | `train_grpo`(PLAN 4.3 点名) |
| Phase 5 尾巴 | `train_distillation` / `train_agent` |

**最省心的一点:九个里面有 3 个是同一个 cross-entropy,只是喂不同数据。**
我们自己实现的 pretrain 循环已经跑通过其中一个(那个实现已随重构归档,
现在的对应位置是 `src/train.py`)。Phase 3 的 `train_full_sft`
对我们来说其实是"换个 Dataset + 加个掩码",不是新东西。

真正新的只有第三组那 4 个 —— 因为那时优化目标第一次不再是"预测下一个词"。

---

## 7. 一处实测更正

我最初说"把 `dataset.py` 的 15 个分词器调用点从 HF API 改成原生 API"。**错了。**
实测 `tokenizers` 0.23.2 上 `Tokenizer.apply_chat_template` **不存在**,
所以那不是改方法名,而是要自己重写 3895 字符的 Jinja chat 模板。

正确做法是两个加载器各司其职,详见 `02-tokenizer.md` §5。
