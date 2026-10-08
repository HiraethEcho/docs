---
title: "MiniMind 训练阶梯"
date: 2026-10-08
summary: "训练顺序：每步做什么、要什么、产出什么、多久"
weight: 7
---

> 本节由 AI 整理生成，仅供参考。

# MiniMind 精简 README —— 训练阶梯

> **代码状态(2026-09-28)。** 旧仓库的 `src/pretrain/`(3,477 行、22 项测试通过、
> 真跑过 2000 万 token)已被删除,完整归档在
> `archive/llm-pre-restructure.tar.gz`。本仓库现在的 `src/` **只有注释**,
> 代码由用户写。下文出现的 `src/*.py` 是**目标位置**,文件可能还不存在。
> **所有实测数字仍然有效** —— 它们描述的是语料、硬件和参考实现,不是那份被删的实现。


上游 `README.md` 是 **1,932 行 / 132 KB**。这份是操作视图:先做什么、后做什么、每步要什么、产出什么、多久。
原文: `minimind/README.md`(vendored)。项目解剖与实现细节见 `06-minimind.md`。

**结构与代码全部来自上游,未适配本机** —— `minimind/` 里的路径仍是 `../dataset/`、`../out`。
这份文档给的是**上游的、能跑通的原样流程**;我们自己的 `src/train.py` **待写**
(已归档的那份跑通过 20M token,可以取回来对照)。

---

## 1. 一句话

`minimind` 是一条**从随机初始化到会用工具的完整阶梯**,九个训练脚本、2,406 行。
每个脚本的循环骨架逐字相同,**变的只有两件事:数据长什么样,和 loss 怎么算**。

| Model | params | 说明 |
|---|---|---|
| **minimind-3** | 64M | Dense。主线,先跑这个 |
| **minimind-3-moe** | 198M-A64M | 4 experts / top-1。同结构换 FFN |

架构:Pre-Norm + RMSNorm · SwiGLU · RoPE θ=1e6 · GQA 8/4 · QK-norm · tied embeddings · vocab 6400。

---

## 2. 顺序 —— 这是全文的重点

上游 README 用 `1'` 到 `7'` 编号,**那个编号本身就是执行顺序**。`必须` 是能跑出对话模型的最小路径,`可选` 是在其之上逐层加能力。

```
必须 ┌─────────────────────────────────────────────────────────────┐
     │  1' 预训练 Pretrain        →  pretrain_768.pth              │
     │         ↓                                                   │
     │  2' 指令微调 SFT            →  full_sft_768.pth  ← Zero 模型 │
     └─────────────────────────────────────────────────────────────┘
                        ↓ (以下都吃 2' 的产出)
可选   3' 知识蒸馏 KD      MoE 教师 → 稠密学生      → full_sft_768.pth
       4' LoRA             只训 0.76% 参数           → full_sft_lora_*.pth
       5' 工具调用&自适应思考   无独立脚本,数据已并入 2'
       6' RLHF → 6.1 DPO    成对偏好,无 reward model  → dpo_768.pth
       7' RLAIF → 7.1 PPO   有 critic,需要 RM         → ppo_768.pth
                  7.2 GRPO  无 critic,需要 RM         → grpo_768.pth
                  7.3 CISPO train_agent.py 内的开关     → agent_768.pth
                  7.4 Agentic RL 多轮工具调用           → agent_768.pth
```

### 逐条:做什么、要什么、产出什么、多久

时间为**单卡 3090 的 1 epoch 经验值**(`7￥ ≈ 1 美元`,租卡约 `1.3￥/h`)。

| # | 命令 | 做什么 | 需要的数据 | 产出 | 64M | 198M |
|---|---|---|---|---|---|---|
| 1' | `train_pretrain.py` | 从随机初始化学语言。`loss = CE(logits[:, :-1], labels[:, 1:])` | `pretrain_t2t_mini.jsonl` | `pretrain_768.pth` | 1.21h | 1.69h |
| 2' | `train_full_sft.py` | 学会对话。**同一个 CE,但 prompt 部分掩成 -100** | `sft_t2t_mini.jsonl` | `full_sft_768.pth` | 1.10h | 1.54h |
| 3' | `train_distillation.py` | 学生模仿教师的**软标签**(`T²·KL(学生/T‖教师/T)`) | `sft_t2t_mini.jsonl` + 教师权重 | `full_sft_768.pth` | — | — |
| 4' | `train_lora.py` | 同样的 SFT,只训 `B·A`。CPU 上也能跑 | `sft` / `lora_medical.jsonl` | `full_sft_lora_*.pth` | — | — |
| 5' | *(无脚本)* | 工具调用数据已混入 `sft_t2t_mini`,模板自动展开成 `<tool_call>` | — | 同 2' | 0.9h | 1.26h |
| 6' | `train_dpo.py` | 拉大"好答案…坏答案"概率差,冻结参考模型当锚 | `dpo.jsonl` | `dpo_768.pth` | — | — |
| 7' | `train_ppo.py` | 真 RL。GAE + critic + PPO clip + KL | `rlaif.jsonl` + reward model | `ppo_768.pth` | — | — |
| 7' | `train_grpo.py` | **砍掉 critic**:同题采 N 个,组内均值/标准差当 baseline | `rlaif.jsonl` + reward model | `grpo_768.pth` | 1.1h | 1.54h |
| 7' | `train_agent.py` | GRPO + 多轮工具循环。`--loss_type` 默认 `cispo` | `agent_rl.jsonl` + reward model | `agent_768.pth` | — | — |

> **零成本门槛:** 64M 跑 `1' + 2'` 一遍 ≈ **2.31 h ≈ 3.0 元**(3090),得到 `MiniMind Zero` 对话模型。
> 198M MoE 同样两步 ≈ **3.23 h ≈ 4.2 元**。这就是"Zero"这个词的含义 —— 2 小时量级从零到能对话。

### 三条容易踩的规则

1. **每级吃上一级的产出,不能跳。** 不能跳过 2' 直接跑 6' —— DPO 的参考模型需要一个已经会对话的模型。
2. **7' 全部需要 reward model**,且 `LMForRewardModel` 用 `trust_remote_code=True` 加载 —— 会执行那个目录里的代码。默认路径 `../../internlm2-1_8b-reward`,**要自己下**。
3. **`5'` 没有独立 trainer。** 上游 2026-03 移除了 `train_reason.py`;工具调用数据已并入 `sft_t2t_mini`,由 `chat_template` 把 `tools` / `tool_calls` 字段自动展开。

### 训练开销原始表

| Model | params | pretrain_t2t_mini | sft_t2t_mini | toolcall | RLAIF |
|---|---|---|---|---|---|
| minimind-3 | 64M | ≈1.21h / ≈1.57￥ | ≈1.10h / ≈1.43￥ | ≈0.9h / ≈1.17￥ | ≈1.1h / ≈1.43￥ |
| minimind-3-moe | 198M-A64M | ≈1.69h / ≈2.20￥ | ≈1.54h / ≈2.00￥ | ≈1.26h / ≈1.64￥ | ≈1.54h / ≈2.00￥ |

---

## 3. 数据清单

[ModelScope `gongjy/minimind_dataset`](https://www.modelscope.cn/datasets/gongjy/minimind_dataset/files) ·
[HF 镜像](https://huggingface.co/datasets/jingyaogong/minimind_dataset)

| 文件 | 大小 | 给谁 |
|---|---|---|
| `pretrain_t2t_mini.jsonl` ✨ | 1.2 GB | `1'` |
| `sft_t2t_mini.jsonl` ✨ | 1.6 GB | `2'` `3'` `4'` `5'` |
| `pretrain_t2t.jsonl` | 10 GB | `1'` 完整版 |
| `sft_t2t.jsonl` | 14 GB | `2'` 完整版 |
| `rlaif.jsonl` ✨ | 24 MB | `7.1` `7.2` |
| `dpo.jsonl` | 53 MB | `6.1` |
| `agent_rl.jsonl` | 86 MB | `7.4` |
| `agent_rl_math.jsonl` | 18 MB | 纯数学 Agent RL |

**最小复现只要 ✨ 那两个**(`pretrain_t2t_mini` + `sft_t2t_mini`)。

分词器**不在数据仓库**,在模型仓库(`gongjy/minimind-3`),440 KB。

数据格式:

```jsonl
{"text": "Transformer 通过自注意力机制建模上下文关系。"}                          ← 1' 预训练
{"conversations": [{"role":"user","content":"..."}, {"role":"assistant","content":"..."}]}   ← 2' 及以后
```

---

## 4. 目录结构

```
minimind/
├── model/
│   ├── model_minimind.py     292 行 ← 模型本体。MiniMindConfig + MiniMindForCausalLM
│   ├── model_lora.py          65 行 ← 自己写的 LoRA(不用 peft)。3 个坑见 06-minimind.md
│   └── tokenizer.json / tokenizer_config.json  453 KB ← vocab 6400,每个数字都依赖它
├── dataset/
│   └── lm_dataset.py         260 行 ← 5 个 Dataset 类 + 2 个对话处理函数
├── trainer/                          2,406 行 ← 第 2 节那张表
│   ├── train_pretrain / train_full_sft / train_lora / train_dpo
│   ├── train_distillation / train_ppo / train_grpo / train_agent
│   ├── train_tokenizer.py          ← 训词表。**上游自己写"不建议重训"**
│   ├── rollout_engine.py           ← 给 RL 生成回答的推理引擎(不训练)
│   └── trainer_utils.py            ← 共享层:get_lr / Logger / ckpt / DDP / reward model
├── scripts/
│   ├── web_demo.py                 ← Streamlit 网页
│   ├── serve_openai_api.py         ← OpenAI 兼容 API
│   ├── chat_api.py
│   ├── eval_toolcall.py            ← 工具调用评测
│   └── convert_model.py            ← 转 transformers / GGUF 格式
└── eval_llm.py                     ← CLI 推理 + 评测
```

---

## 5. 完整跑一遍的命令序列

```bash
cd minimind
pip install -r requirements.txt          # ⚠️ 上游钉的版本已过期，见下

# 数据 + 词表放进 ./dataset 和 ./model（见第 3 节）

cd trainer
python train_pretrain.py                 # 1'  → ../out/pretrain_768.pth
python train_full_sft.py                 # 2'  → ../out/full_sft_768.pth
python eval_llm.py --weight full_sft     # 3'  测试（在仓库根目录执行）

# 可选，按需
python train_lora.py                     # 4'
python train_distillation.py             # 3'
python train_dpo.py                      # 6.1
python train_ppo.py                      # 7.1  需要 reward model
python train_grpo.py                     # 7.2  需要 reward model
python train_agent.py                    # 7.4  需要 reward model

# 多卡：把 python 换成 torchrun --nproc_per_node N
# 续训：任何脚本加 --from_resume 1
```

`--from_resume 1` **所有脚本都支持**:检查点写在 `./checkpoints/`(模型 + 优化器 + 进度),
支持跨不同卡数恢复,支持 wandb/swanlab run 连续性。

---

## 6. 推理与评测

```bash
# CLI 推理(仓库根目录)
python eval_llm.py --load_from ./minimind-3              # transformers 格式
python eval_llm.py --weight full_sft                     # ./out/ 下的 .pth
python eval_llm.py --weight full_sft --lora_weight lora_medical
python eval_llm.py --load_from ./minimind-3 --open_thinking 1   # 自适应思考

# 工具调用
python eval_toolcall.py --weight full_sft

# 客观评测(lm-evaluation-harness)
#   ceval-valid / cmmlu / arc_easy / piqa / openbookqa / hellaswag / social_iqa
#   指令模型加 --apply_chat_template，纯基座不加

# 服务
cd scripts && python serve_openai_api.py     # OpenAI 兼容
cd scripts && streamlit run web_demo.py      # 网页
```

**第三方推理框架**(都在 `其他` 一节):`vllm serve` · `SGLang`(`--attention-backend triton`) ·
`llama.cpp`(转 GGUF) · `ollama` · `MNN`(4-bit HQQ 量化)。

> **注意 `--attention-backend triton`** —— attention 后端的选型在这个平台上同样是决定性的。
> 本机 `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL=1` 就是把 SDPA 从 MATH 后端切到融合后端,
> 实测差 **13 倍**(8.90 ms vs 114.52 ms @ B4/T2048)。

---

## 7. 上游 README 的其余章节(本文件未展开)

| 原章节 | 内容 | 值不值得看 |
|---|---|---|
| 📌 项目介绍 | 包含内容、已发布模型、更新日志 | 更新日志有版本决策理由,值得扫 |
| 📌 快速开始 Ⅰ | 模型推理(下载 / CLI / WebUI / 第三方框架) | 第 6 节已覆盖 |
| 📌 数据介绍 Ⅰ | **Tokenizer 一节** | 值得看。已整理进 `02-tokenizer.md` |
| 📌 模型 | 结构 + 模型配置(**引 MobileLLM 讲深度 vs 宽度**) | 值得看。已整理进 `03-model.md` |
| 📌 实验 Ⅳ | **RL 小结** + "PO 算法的统一视角" | 值得看,讲 PPO/GRPO/CISPO 的关系 |
| 📌 评估 Ⅰ Ⅱ | RL 模型对比、与其他模型对比(含逐模型点评) | 主观,看结论即可 |
| 📌 评估 Ⅳ | **RoPE 长度外推**(YaRN) | 做长上下文时再看 |
| 📌 致谢 | 贡献者、引用、协议 | — |

---

## 8. 在本机跑之前必须知道的三件事

**① `requirements.txt` 是混杂清单而且过期。**
一个文件塞了四类东西(训练 + 网页 demo + 数据清洗 + 蒸馏造数据),
钉的 `transformers==4.57.6` 与代码里的 5.x 兼容代码矛盾,`torch` 被注释掉。
**别用它。** 我们的 `venv-arch` 已经够用(见 `USAGE.md`)。

**② `minimind/` 里的路径全部未适配。**
`../dataset/`、`../out`、`../checkpoints` 都是相对 `trainer/` 的。
在 `~/Public/minimind` 那个布局下才对;vendored 进我们仓库后没有 `dataset/`、`out/`、`checkpoints/`。
要跑就得先改路径常量 —— 或者等我们自己的 `src/train.py`(待写)适配好。

**③ SFT/DPO/RL 的 Dataset 需要 HuggingFace 接口的分词器。**
`minimind/dataset/lm_dataset.py` 用 `tokenizer(text, ...).input_ids` 和
`apply_chat_template(...)`,而我们的 `load_tokenizer` 返回原生 `tokenizers.Tokenizer`。
两者不是一回事,详见 `02-tokenizer.md` §3–5。

---

## 9. 这张表和我们仓库的对应

| 上游阶段 | 我们的状态 |
|---|---|
| `1'` 预训练 | ✅ 已跑通 20M token(已归档的 `train.py`,packed,5,604 tok/s,6,485 MiB) |
| `2'` SFT | ⚠️ 卡在 ②③ 两条 + `sft_t2t_mini.jsonl` 未下载 |
| `3'` KD | ⚠️ 同 2' |
| `4'` LoRA | ⚠️ 同 2';另外 `PLAN.md` 3.2 要求自己写一个再和上游对照 |
| `5'` 工具调用 | ⚠️ 无独立脚本,随 2' 一起来 |
| `6'` DPO | ⚠️ 需 `dpo.jsonl` 53 MB |
| `7'` PPO/GRPO/Agent | ⚠️ 需 `rlaif.jsonl` / `agent_rl.jsonl` + reward model |

数据与 reference 都在:`data/prepared/minimind3-6400/`(3.32 亿 token 的 uint16 流)、
`minimind/`(上游 vendored)。**下一个能解锁的阶段是 `2'` SFT。**
