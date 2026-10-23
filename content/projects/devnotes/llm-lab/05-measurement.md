---
title: "测量 / Measurement"
date: 2026-10-08
summary: "四个测量 bug、累计 tok/s 的陷阱与有效算力"
weight: 5
---

> 本节由 AI 整理生成，仅供参考。

# 测量 / Measurement

这个文件不讲模型,讲**怎么知道自己有多快**。

理由:2026-09-26 在自己的代码里抓到 4 个 bug,**其中 3 个是测量 bug**。
一个错误的数字比一次崩溃更贵 —— 崩溃会告诉你去看,错误的数字不会。
它还不会自己暴露:93 分钟的训练和 60 分钟的训练,日志长得一模一样。

---

## 1. 四个 bug

### Bug 1 — 分母里混进了评估时间

**症状:** 训练循环报 `4,400 tok/s`,而隔离 benchmark 报 `5,736 tok/s`。

**原因:** 分母用了墙钟。

```python
tok_s = (tokens_done - tokens_seen) / max(elapsed, 1e-9)   # ✗ elapsed = 墙钟
```

`elapsed` 里包含每 200 步一次的验证(20 个批)+ 生成一个 48 token 的样本,约 11 s。

**修:**

```python
t_step0 = time.perf_counter()
...                       # 整个一次参数更新
train_seconds += time.perf_counter() - t_step0
tok_s = (tokens_done - tokens_seen) / train_seconds   # ✓ 不含评估
```

并且**两个都报**:

```
3,700 tok/s train-only     3,575 tok/s wall
```

前者是做计划用的(下一次要跑多久),后者是你实际会花掉的时间。

---

### Bug 2 — 累计量除以本次耗时

**症状:** 一次续训跑完,总结行写 `11,002 tok/s`。真实值约 5,500。

**原因:** 分子是全局的,分母是本地的。

```python
print(f"{tokens_done / max(train_seconds, 1e-9):,.0f} tok/s")   # ✗
```

`tokens_done` 是**累计**(含检查点里已经看过的 1.19 M token),
`train_seconds` 只是**本次**这 3.6 分钟。

**修:** 分子也取增量。

```python
ran_tokens = tokens_done - tokens_seen
print(f"{ran_tokens/1e6:.2f} M this run ({tokens_done/1e6:.2f} M cumulative)")
```

⚠️ **这个 bug 只在续训时出现。** 第一次跑永远是对的,所以你第一次测试也不会发现它。

---

### Bug 3 — 拿暂态当稳态(最贵的)

**症状:** benchmark 测出 `1,897 ms/step`,据此预测 20M token 需要 **59.8 分钟**。
**实际 93.2 分钟。**

**原因:** benchmark 是 `3 次预热 + 25 步`。而完整运行的真实形状(按区间重建,见 §2)是:

```
steps    1-280    2,340 ms/step    5,500-6,300 tok/s
steps  280-320    7,350 ms/step    1,480 tok/s   ← 停顿
steps  320-1838   2,940 ms/step    3,700 tok/s   ← 新基线,保持 90 分钟
```

**那 28 步全部落在前 280 步里。** 之后那个相就不存在了。
所以 benchmark 没有测错 —— 它测的那个相是真实的,只是不持久。

```
93.2 / 59.8 = 1.56          2,940 / 1,897 = 1.55     完全吻合
```

**修:benchmark 必须跑到稳态,不能短于 400 步**,并且先丢弃预热段再看。

这条的教训比前两条重:**同一个 benchmark,28 步和 400 步可能给出差 1.55 倍的答案。**
凡是打算拿来外推的数字,都必须测到外推区间所对应的那个相。

---

### Bug 4 — `| tail` 吞掉了全部输出

**症状:**

```bash
timeout 900 python -m src.pretrain.train --max_tokens ... 2>&1 | tail -22
# 15 分钟,零输出。看起来像卡死了。
# (当时跑的是 `src.minimind.train`,那个移植版已删除 —— 教训与命令无关。)
```

**原因:** `tail -N` 要等到 EOF 才打印。harness 在 900 s 杀掉进程组,
`tail` 一起被杀,它缓冲的内容全丢。

**程序其实跑得好好的** —— 它甚至已经把 1.2 GB 的 Arrow 缓存建完了,事后才查到:

```
datasets/.../json-train-*.arrow   1.2 GB
```

**修:**

- 长跑**不要**接 `tail`。用 `tee file.log`,或者什么都不接。
- 代码侧一行保险 —— Python 在 stdout 是管道时是**块缓冲**(8 KB):

```python
sys.stdout.reconfigure(line_buffering=True)
```

`Logger()` 里的 `print(..., flush=True)` 是另一条防线,但如果日志分散在多处,
重配一次 stdout 比给每一处加 `flush=True` 更可靠。

---

## 2. 日志里的 tok/s 是**累计平均**,会掩盖真实形状

这是 Bug 3 能被发现的唯一原因。日志里那一列是"到目前为止的平均",
所以一条**前 300 步快、之后恒定**的曲线,和一条**真正逐渐变慢**的曲线,
在这一列里长得几乎一样 —— 都是单调下降然后收敛。

按区间还原真实速度:

```python
# 日志行: step N, 累计 tok/s = c
#   train_seconds(N) = N × T / c
pts = [(n, n * T / c) for n, c in rows]           # rows = [(step, cum_tok_s), ...]
for (n1, s1), (n2, s2) in zip(pts, pts[1:]):
    dt, dn = s2 - s1, n2 - n1
    print(f"{n1}->{n2}   {dn * T / dt:9,.0f} tok/s   {dt / dn * 1000:8.2f} ms/step")
```

**看到一条单调下降的 tok/s 曲线,先怀疑它是累计平均。** 不要先怀疑模型、不要先怀疑热降频。

---

## 3. 有效算力 —— 决定"要不要优化"的那个数

```
FLOPs/step ≈ 6 × N_params × tokens_per_step            (前向 1× + 反向 2×)
           = 6 × 63.9e6 × 10,880 = 4.17e12

实测有效   = 4.17e12 / 1.897 s = 2.20 TFLOP/s          (快相)
           = 4.17e12 / 2.940 s = 1.42 TFLOP/s          (稳态)
实测峰值   = 9.46 TFLOP/s                              (bf16 matmul 2048³)
─────────────────────────────────────────────────────────────────────
利用率       23% (快相)  /  15% (稳态)
```

**这个比例才是判断依据。** 显存占用 `6,485 / 24,576 = 26%` 说明的是
**不在显存上**,23% 说明**在 kernel 效率上**。

分项(benchmark, 快相,共 1,897 ms):

```
gather (组批)        0.20 ms     0.0%
host→device          0.24 ms     0.0%
forward            507.35 ms    26.7%
backward         1,304.17 ms    68.8%    ← 杠杆在这里
optimizer step      84.99 ms     4.5%
```

---

## 4. 验算:为什么"加显存"不会更快

若只是带宽受限,可以这样估:

```
每步最少要读的权重 = 63.9 M × 2 B = 128 MB
前向读 1 次 + 反向读 2 次       = 384 MB
384 MB / 1.897 s                = 202 MB/s     ← 远低于任何内存带宽
```

**所以瓶颈不是带宽,也不是容量。** 推论(全部与"多用显存"相反):

| 做法 | 为什么没用 |
|---|---|
| 把 GTT 填满 | 卷不动 kernel 效率;而且 GTT 和桌面共享,填满 = 桌面 OOM |
| 加 `--num_workers` | 数据组批只占 `0.20 ms / 1,897 ms = 0.01%` |
| 梯度累积 | 显存占用**完全不变**,只是把 batch 拆开算 |
| 更大 batch | ✅ 有用 —— GEMM 更胖,GPU 利用率上升 |
| `torch.compile` | ✅ 有用 —— fusion,直接对着那 23% 去 |

---

## 5. 一个还没解释的现象

step 280–320 有一次 **7.35 s/step** 的停顿,之后基线**永久**从 2,340 掉到 2,940 ms/step
(慢 26%),并保持了 90 分钟。温度当时只有 42–47 °C,所以**不像**热降频。

两个候选:

1. **GTT 页映射成本** —— 分配器到达高水位后,每次访问要走更多页表项。
   APU 的 GTT 就是系统内存,这一点比独显明显得多。
2. **共享内存被别的进程占了** —— GTT 池和桌面共享(总内存 30.6 GiB)。

**判别实验:** 跑 **500 步** `--batch_size 16`(显存减半)。

- 如果它也停在 ~2,940 ms/step → 和 GTT 有关
- 如果回到 ~2,000 ms/step → 和分配器 / 池大小有关

⚠️ **别用 28 步的 benchmark 去判断。** 见 Bug 3。

---

## 6. checklist

| 规则 | 出处 |
|---|---|
| 长跑不接 `tail`,用 `tee` | Bug 4 |
| 分母只放**这一段**的时间,别放墙钟 | Bug 1 |
| 分子分母**范围一致**(都增量或都累计) | Bug 2 |
| benchmark ≥ 400 步,且先丢弃预热 | Bug 3 |
| 看到单调下降,先怀疑累计平均 | §2 |
| 说"训练快"要给出**有效算力占峰值的比例** | §3 |
| 报数字时说明**测的是哪个相** | Bug 3 |
| `Predicted` 和 `Measured` 分两列写,不要合并 | 全篇 |

最后一条是这份文件存在的原因:上面那张
`predicted 59.8 min | measured 93.2 min` 的表,如果写成 `~60 min`,
它就会一直错下去,而且**下一次没人会去查**。
