---
title: "模型结构 / Model"
date: 2026-10-08
summary: "MiniMind-3 的规格、一次前向的顺序与五个静默出错的坑"
weight: 3
---

> 本节由 AI 整理生成，仅供参考。

# 模型结构 / Model

> **代码状态(2026-09-28)。** 旧仓库的 `src/pretrain/`(3,477 行、22 项测试通过、
> 真跑过 2000 万 token)已被删除,完整归档在
> `archive/llm-pre-restructure.tar.gz`。本仓库现在的 `src/` **只有注释**,
> 代码由用户写。下文出现的 `src/*.py` 是**目标位置**,文件可能还不存在。
> **所有实测数字仍然有效** —— 它们描述的是语料、硬件和参考实现,不是那份被删的实现。


稠密 MiniMind-3 的规格。参照实现是 `minimind/model/model_minimind.py`(已加注释);
我们自己的 `src/model.py` 是**待写**目标(现在只有注释)。
已归档的那份实现**实测与 minimind 逐位相同**(91 个 state_dict key 一致,
`max |logit diff| = 0.000e+00`)—— 拿这个当重写时的验收标准。

---

## 1. 这些数字就是契约

| | 值 | 备注 |
|---|---|---|
| layers | 8 | |
| `hidden_size` | 768 | |
| heads / kv_heads | 8 / 4 | GQA |
| **`head_dim`** | **96** | = 768/8。**不是 64** |
| **`intermediate_size`** | **2432** | = `ceil(768·π/64)·64` |
| vocab | 6400 | tied embeddings |
| RoPE θ | 1e6 | split-half 约定 |
| norm | RMSNorm, eps 1e-6 | **内部走 fp32** |
| experts | — | 稠密版没有 |
| **params** | **63,912,192** | |

`2432` 一行里有两个想法:Llama 用 `8/3·768 = 2048`(SwiGLU 有三个矩阵,
要按 2/3 折算宽度),minimind 向上取到 π(约 3.14);`/64*64` 是向上取整到 64 的倍数,
为了 GEMM 命中对齐 tile —— 是 kernel 效率取整,不是数学要求。

### 参数量逐项验算

```
attention   1,769,472      (q 589,824 + k 294,912 + v 294,912 + o 589,824)
qk_norm           192      (2 × 96)
mlp         5,603,328      (3 × 768 × 2432)
norms           1,536      (2 × 768)
─────────────────────
per layer   7,374,528
× 8       = 58,996,224
final norm        768
embedding     4,915,200    (6400 × 768,tied —— 只算一次)
═════════════════════
TOTAL      63,912,192      ✓ 实测与解析值相等
```

MoE 版(未实现):把 SwiGLU 换成 4 个专家 + 路由(768×4),
每层 24,187,584 → 总计 **198,416,640**,其中每 token 只有 63.9M 参与计算。

**任何一处对不上,问题在配置,不在数学。**

---

## 2. 一次前向的顺序

```
input_ids [B, T]
  └─ Embedding(6400, 768)                    → [B, T, 768]
     └─ × 8 个 Block:
         h = x + Attention( RMSNorm(x) )        ← pre-norm
         h = h + SwiGLU( RMSNorm(h) )
     └─ RMSNorm                                → [B, T, 768]
        └─ lm_head (tied)                      → [B, T, 6400] logits
           └─ shift + cross_entropy            → scalar
```

---

## 3. 每一块在干什么

### RMSNorm —— fp32 那个转换是承重的

```
x / sqrt(mean(x²) + eps) × weight
```

关键是 `self.norm(x.float())`。**bf16 只有 8 位尾数**,均方要在 768 个数上求和,
在 bf16 里精度会悄悄丢失 —— 结果不报错,只是模型稍微差一点。
**这是手写 transformer 里最常见的静默 bug。**

只对最后一维做,这也让它之后能逐头使用。没有减均值(这是与 LayerNorm 的区别),
也没有 bias。

### RoPE —— 48 个频率,前后半交换

`dim = head_dim = 96` → 48 个频率对。预计算一张 `[max_pos, 96]` 的表:

```
inv_freq[i] = 1 / θ^(2i/96)              i = 0..47
angle = outer(positions, inv_freq)        [T, 48]
cos = cat([cos(angle), cos(angle)], -1)   [T, 96]      ← 复制是关键技巧
```

复制之后旋转就是纯粹的元素逐个相乘,不需要 gather 也不需要索引算术:

```
rotate_half(x) = cat([-x[..., 48:], x[..., :48]], -1)
q' = q*cos + rotate_half(q)*sin
```

**配对方式是 (i, i+48),不是 (2i, 2i+1)。** HuggingFace 叫这个
`interleaved=False`(GPT-NeoX 风格)。**两种都是有效的 RoPE,但不可互换** ——
换错了照样能训练、照样收敛,只是输出与 minimind 完全对不上,
没有参考实现根本发现不了。

只旋转 q 和 k,**永不旋转 v**。顺序是 **qk-norm 之后、attention 之前**。

θ=1e6 越大旋转越慢,相距很远的位置区分度越好,这是长上下文的来源。
YaRN 只用于推理,训练时不要开。

### Attention —— GQA 8/4 + QK-norm

```
q: [B,T,768] → [B,T,8,96]      k,v: [B,T,768] → [B,T,4,96]
q_norm, k_norm: 在 head_dim=96 上逐头做 RMSNorm
RoPE
repeat_kv: 4 → 8(用 expand,不是 repeat)
→ SDPA(is_causal=True)
→ [B,T,768] → o_proj
```

三件事重要:

1. **`repeat_kv` 必须用 `expand`。** `expand` 建视图(零拷贝),
   `repeat` 会真的复制整份张量。这段在热路径上。
2. **QK-norm 是 lr 5e-4 不用预热也能稳的原因** —— 它限制了注意力分数 q·k 的量级。
3. **SDPA 的快速路径条件。** minimind 只在 `seq_len > 1 且 无 KV cache
   且 mask 全为 1` 时走融合 kernel。**条件写错了不会报错,只是掉进慢路径:
   本机实测 B4/T2048 下 8.90 ms vs 114.52 ms,差 13 倍。**

### SwiGLU —— 三个矩阵,没有 bias

```
down( silu(gate(x)) × up(x) )
```

`gate` 是过激活函数的那一条。**把 gate 和 up 互换模型就变了** ——
测试套件发现不了,只有与参考实现做 diff 才能发现。

### Block —— pre-norm,两条残差,两个独立的 norm

```
h = x + attn(input_layernorm(x))
h = h + mlp(post_attention_layernorm(h))
```

**norm 在分支上,永远不在残差通路上。** 这保持了从 loss 到 embedding 的
干净梯度高速路,也是 8 层堆叠无需预热的原因。

两个 layernorm 是独立的模块、独立的权重 —— 不是同一个被调用两次。

### 输出头 —— 权重共享,位移在头之后

```python
self.lm_head.weight = self.model.embed_tokens.weight   # 同一个对象
```

**tied 是一个参数用两次。** 之所以在 768 这个尺度成立,是因为词表只有 6400:
`6400 × 768 = 4.9M`,占整个模型的 7.7%。Qwen2 的 151k 词表会变成 116M
——比整个模型还大,那时就该解除绑定。

位移发生在 `lm_head` **之后**,所以投射了 340 个位置只用 339 个。
省下的是 0.3% FLOPs,换来的可读性更值。

---

## 4. MoE 的特殊之处(未实现,但要知道)

从 `minimind/model/model_minimind.py:MOEFeedForward` 读出来的:

```python
top1 = topk(softmax(gate(x.detach())), k=1)[0]
topk_weight = top1 - top1.detach() + 1.0        # 恒等于 1.0
```

**在前向里它精确等于 1.0;而 `+ top1 - top1.detach()` 让梯度为 0,
`x` 又被 `.detach()` 过,所以没有任何路径回到 gate。**

**结论:默认配置(top-1、`norm_topk_prob=True`)下,路由网络从主损失拿不到任何梯度,
它完全由 `aux_loss` 训练。** 没有人能猜到这个。这也解释了为什么 5e-4 这个
系数这么要紧,以及为什么"`aux_coef=0` 会塌缩专家"。

其他两点:

- **Python 专家循环是性能天花板。** 四个专家逐个跑是启动/占用受限,不是 GEMM 受限。
  这正是计划里"bs16 反而比 bs32 快"的原因(实测 5002 vs 4094 tok/s)。
- **死专家需要 `y[0,0] += 0 * sum(p.sum() ...)`。** 一个专家没拿到 token,
  它的参数就没有梯度,DDP 会报"unused parameters"。乘 0 保持它在计算图里、
  贡献为零。单卡不需要,但留着无害。

`aux_loss = sum(各专家被路由比例 × 平均路由概率) × num_experts × coef`。
路由均匀时最小值为 1.0,所以它永远是个惩罚项。**必须与 CE 损失分开记录**
—— 专家塌缩在总损失里看不出来。

---

## 5. 五个会静默出错的坑

| # | 坑 | 症状 |
|---|---|---|
| 1 | RoPE 配对方式用 (2i, 2i+1) | 能训能收敛,与参考实现完全对不上 |
| 2 | RMSNorm 不做 fp32 内部计算 | 无报错,收敛更差 |
| 3 | RoPE 也旋转了 v | 无报错,无意义 |
| 4 | SDPA 快速路径条件写错 | 无报错,**慢 13 倍** |
| 5 | `q_norm`/`k_norm` 用在 768 而不是 96 | 通常会报错,但报错信息不会告诉你为什么 |

前四个的共同点:**没有错误信息**。这就是"与参考实现做 diff"不可替代的原因。

---

## 6. 验证阶梯

1. **参数量 == 63,912,192。** 解析计算,不需要任何参考物。抓所有配置错误。
2. **手算 4×4 attention。** CPU、fp32、容差 1e-6。用纸笔算出来的数字对照。
3. **`torch.autograd.gradcheck`** 跑 RMSNorm 和 SwiGLU。抓反向写错。
4. **模块命名与 minimind 完全一致** → `load_state_dict(strict=True)` 加载官方权重,
   同一输入对比 logits。**这是唯一能抓上表前四个坑的一步。**
5. 然后才训练。

第 4 步是杀手锏。前面全是记账,第 4 步才是证明。

`test_model.py`(**已归档**)覆盖 1–3 和第 4 步的接口部分。
最初记的是 **19/19、约 6 秒** —— 测试后来又长了,**最终是 22 项、约 7 秒**。
两个数字都留着,因为差值就是那次补充覆盖的内容。取回:

```bash
cd /tmp && tar -xzf archive/llm-pre-restructure.tar.gz \
    llm/src/pretrain/test_model.py
```

**测试抓不到什么**(必须在文档里写明,否则"22/22 通过"会被误当成"正确"):
上表前四个坑、SwigLU 的 gate/up 互换、错误的权重初始化分布、
任何与分词器和数据管线有关的问题。
**测试证明代码没坏;第 4 步证明它是同一个模型。**
