---
title: "PLAN — LLM lab, from scratch"
date: 2026-10-08
summary: "分阶段计划：环境、预训练、后训练与 RL"
weight: 1
---

> 本节由 AI 整理生成，仅供参考。

# PLAN — LLM lab, from scratch

> **代码状态(2026-09-28)。** 旧仓库的 `src/pretrain/`(3,477 行、22 项测试通过、
> 真跑过 2000 万 token)已被删除,完整归档在
> `archive/llm-pre-restructure.tar.gz`。本仓库现在的 `src/` **只有注释**,
> 代码由用户写。下文出现的 `src/*.py` 是**目标位置**,文件可能还不存在。
> **所有实测数字仍然有效** —— 它们描述的是语料、硬件和参考实现,不是那份被删的实现。


Supersedes the "Learning path" section of `README.md`. Status of that section:
its *content* is still valid as reference reading, but its *staging* is wrong —
it interleaves "run someone else's repo" with "write your own code", which
makes it impossible to tell whether you learned the algorithm or the repo.

**Scope of this plan: write the code yourself.** The `~/Public` repos are
reading material and oracles to diff against, never the deliverable.

---

## The loop every phase uses

This is the core method. Four steps, in this order, always:

1. **Implement minimal** — the smallest version that can be wrong, in one file,
   in `src/`.
2. **Verify against an oracle** — a reference implementation (`~/Public/*`), a
   brute-force version, or hand-computed numbers. Does it produce the same
   *numbers*, not just run?
3. **Swap in the library** — replace your version with `torch`/`peft`/`trl`.
4. **Diff** — same loss? same memory? same speed? Where they differ *is* the
   lesson.

Skipping step 2 produces code that runs and is silently wrong. Skipping step 4
produces code that's right but pointless (you already had the library).

---

## Phase 0 — Environment: verified, frozen

**Status: complete.** Nothing blocking. See the gap table for what was
deliberately dropped.

**Why it matters that the data disk is ext4 and `/` is btrfs:** every large
artifact must land on ext4. btrfs is unsuited to the write patterns here
(checkpoint churn, memmapped shards, thousands of kernel-cache files). All
caches are redirected; see `USAGE.md` → *每个 session*.

### The venv borrows almost everything

`venv-arch` at the repo root, from
`uv venv --python 3.14 --system-site-packages venv-arch`. System Python 3.14.7,
and it must stay 3.14 — Arch's torch is a compiled extension built against it.

**pacman already provides most of the ML stack** (119 `python-*` packages).
`torch`, `transformers`, `accelerate`, `datasets`, `tokenizers`, `safetensors`,
`numpy`, `huggingface-hub`, `triton`, `pandas` and more are **borrowed**, not
duplicated. Only the genuine gaps are installed, each with `--no-deps` so
nothing shadows the system copy.

**16 packages, 191 MB.** torch resolves to
`site-packages/torch` (hip 7.2.53211), all 10 GPU checks
pass, and `requirements.lock` is the frozen record.

Verified end to end, not assumed: `bitsandbytes` 4-bit NF4 works on ROCm —
`Qwen3-0.6B` bf16 **1136.9 MiB peak** vs 4-bit **959.1 MiB peak**, NF4
round-trip rel err 0.092. A peft LoRA on `Qwen3-0.6B` (4.59 M trainable /
600.6 M, 0.76 %) runs **3 real training steps in 0.83 s at 1314.8 MiB peak**.
`load_dataset("openai/gsm8k")` goes through the system `datasets`. Phases 3.3
and 3.5 are unblocked and measured.

### 0.1 What verification said

| Check | Result |
|---|---|
| `scripts/check_gpu.py` | **all 10 pass** |
| torch | `2.14.0`, hip `7.2.53211`, `site-packages/torch` — not shadowed ✅ |
| venv | 3.14.7, `include-system-site-packages = true`, `sys.path` order correct |
| `HSA_OVERRIDE_GFX_VERSION` | unset ✅ |
| ROCm | 7.2.4; 30 pacman-owned `rocm-*`/`hip-*` packages |
| groups / devices | `revo` in `video render`; `/dev/kfd`, `/dev/dri/renderD128` present |
| kernel | `amdgpu` loaded, `amdxdna` loaded (unused, out of scope) |
| rocminfo | `gfx1150`, "AMD Ryzen AI 9 H 365 w/ Radeon 880M" |
| 数据盘 | ext4, `rw,noatime`, 503 G, 23 G used, **455 G free** |
| boot disk | 238 G, 93 G used, 144 G free (40 %) |
| env vars | `HF_HOME=<数据盘>/hf`, `HF_HUB_DISABLE_XET=1`, `HF_ENDPOINT=https://hf-mirror.com`, `TORCH_HOME`, `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL`, `PYTORCH_HIP_ALLOC_CONF`, `ROCM_PATH` |
| packages | borrowed from system: torch 2.14.0, transformers 5.17.0, accelerate 1.15.0, datasets 5.0.1, tokenizers 0.23.2, safetensors 0.8.0, numpy 2.5.3, triton 3.5.1. In the venv: peft 0.21.0, trl 1.14.0, bitsandbytes 0.50.2, tensorboard 2.21.0 |

**The GPU path works. Do not touch the ROCm install.**

### 0.2 Measured baselines — the numbers to beat

Recorded so later phases have something to compare against. Regenerate with
`scripts/bench_gpu.py` (to be written in 0.4).

```
bf16 matmul (peak)     1024³   0.314 ms    6.85 TFLOP/s
                       2048³   1.817 ms    9.46 TFLOP/s   ← peak
                       4096³  18.101 ms    7.59 TFLOP/s

SDPA, bf16, causal, B=4 H=8 D=64
  T= 512   0.80 ms
  T=1024   2.39 ms
  T=2048   9.38 ms      ← ~4× per doubling past 1024

SDPA backend @ B4 T2048
  FLASH (AOTriton)   8.90 ms
  EFFICIENT          8.71 ms
  MATH             114.52 ms   ← 13× worse; confirmed NOT the default

memory: pool total 24.00 GiB GTT, single alloc 22.5 GiB OK
        desktop GTT in use 200 MiB
```

**Reading the memory numbers:** `mem_info_vram_total` is 512 MiB — the BIOS
carve-out, and **never** the ceiling. `mem_info_gtt_total` is 24576 MiB and a
single 22.5 GiB allocation is verified. Measure **one size per fresh process**:
allocating 4→8→12→16 GiB in sequence inside one process reports 16 GiB failing,
suggesting a ~15.7 GiB ceiling that does not exist. That is accumulated
allocations, not a real limit. Measure the allocator in isolation — not via
sysfs, and not from a dirty process.

### 0.3 Implications for the rest of the plan

- **Attention, not matmul, is the binding constraint.** Peak matmul is
  9.5 TFLOP/s but attention costs ~4× per doubling of `T` past 1024. Pretrain at
  **`T=512–1024`**. Long context is a separate experiment, not a default.
- **The fused backend works.** `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL=1` is
  doing its job — FLASH is 13× faster than MATH and *is* the default. Good news:
  no kernel surgery needed.
- **Effective model FLOPs ≈ 6ND.** At a realistic ~4 TFLOP/s through a real
  transformer (not the 9.5 peak):

  | params | tokens/batch | s/step | tok/s | 1 B tokens |
  |---|---|---|---|---|
  | 15 M | 16×1024 | 0.37 | ~44 k | **6 h** |
  | 50 M | 16×1024 | 1.3 | ~13 k | 21 h |
  | 100 M | 16×1024 | 2.5 | ~6.5 k | 43 h |

  So Phase 2 targets **10–20 M params first** (hours-scale, full loop
  verifiable), then 50–125 M only with an explicit token budget.
- **Memory is not the constraint; patience is.** 24 GiB fits a 1 B-param
  *inference* comfortably and 100–300 M training with room to spare. The APU
  shares RAM with the OS, so keep the browser closed during runs.

### 0.4 Gaps to close before Phase 1

| # | Gap | Fix | Status |
|---|---|---|---|
| 0.4.1 | `uv python find` → **3.13**, not 3.14. A bare `uv venv`/`uv run` would build a 3.13 venv with no system torch. | `.python-version` = `3.14` written. `uv venv --python 3.14` is explicit in every command in this plan. | ✅ done |
| 0.4.2 | venv has **no `pip`** (uv default). `python -m pip` fails. | Correct, not a bug — keep it. Use `uv pip … --python venv-arch`. Documented in `AGENTS.md` §1. | ✅ done |
| 0.4.2b | `uv` re-installs packages pacman already provides, shadowing them (it wanted its own `numpy`, `pandas`, `huggingface-hub`). | Install `--no-deps` and rely on system. venv is **16 pkgs / 191 MB**, all heavy libs borrowed. Documented in `AGENTS.md` §3. | ✅ done |
| 0.4.3 | `triton`, `torchinductor`, `modelscope` caches land on the **boot disk (btrfs)**. | `TRITON_CACHE_DIR`, `TORCHINDUCTOR_CACHE_DIR`, `MODELSCOPE_CACHE` → `<数据盘>/cache/*`, added to `dev.zsh` and verified in a login shell. `MIOPEN_USER_DB_PATH` still unset. | ◐ mostly done |
| 0.4.4 | MIOpen's default find mode does an exhaustive search on first op shapes (minutes, once). | `MIOPEN_FIND_MODE=FAST` set in `dev.zsh`, verified. Note: MIOpen implements **convolutions** — a transformer decoder has none, so this is cheap insurance rather than a fix for anything measured. | ✅ done |
| 0.4.5 | No frozen record of the working package set. | `uv pip freeze --python venv-arch > requirements.lock` — **16 packages**. | ✅ done |
| 0.4.6 | `scripts/bench_gpu.py` doesn't exist, so §0.2 isn't reproducible by one command. | **Dropped.** The numbers are recorded in `GUIDE.md` §6 and `LOG.md`. A script would make them re-runnable on demand; it would not produce new information. Revisit only if a measurement is actually doubted. | ✖ dropped |
| 0.4.7 | ~~`trainllm/` GSM8K data on the NTFS mount~~ | **Dropped** — `trainllm` is out of scope. | ✖ dropped |

### 0.5 Done when

- `scripts/check_gpu.py` passes (already true).
- `scripts/bench_gpu.py` reproduces the 0.2 table.
- `requirements.lock` committed.
- ⚠️ `uv pip install --python venv-arch --dry-run <anything>` never lists `torch` or `nvidia-*`.

**Confirmed live:** on uv 0.12.18, a dry-run of `accelerate peft trl bitsandbytes`
*without* `--no-deps` still plans **17** packages — `torch==2.14.0` (CUDA build),
`triton==3.8.0`, and 15 `nvidia-*`. With `--no-deps` it plans exactly 4. The trap
has not been fixed upstream; the rule stands.
- The torch assertion in `AGENTS.md` §2 prints `7.2.53211 site-packages/torch`.

---

## Phase 1 — Foundations, your own code

Small, CPU-first, numerically verified. **This is where correctness is won** —
everything later inherits these bugs.

**Decision (2026-09-25): use minimind's tokenizer. No BPE written from scratch.**

Vocab 6400, loaded from the released model repo. Reasons, in order of weight:

1. **It is the confound.** The Phase 2 target is to reproduce minimind's
   `63.9 M` / `198.4 M` param counts and diff logits against their released
   weights. A different tokenizer makes that comparison meaningless.
2. **Vocab size silently dominates a small model.** At `vocab × d_model`, 6400 ×
   768 = **4.9 M params** in the embedding (7.7 % of the dense 64 M model).
   Qwen2's 151643 vocab would be 116 M — larger than the entire model.
3. **Compression is compute.** Chinese runs ~1.5–1.7 chars/token, English 4–5.
   Better compression means shorter sequences, and attention is O(T²) with a
   measured ~4× cost per doubling past T=1024.

Writing BPE stays available as an optional exercise, but it is **not on the
critical path** and must not be a dependency of Phase 2.

| # | Build | Oracle to diff against | Done when |
|---|---|---|---|
| 1.1 | Load minimind's tokenizer; encode/decode wrapper | the released tokenizer files | round-trips a corpus byte-exactly; `decode(encode(x)) == x` |
| 1.2 | Single-head causal self-attention, explicit `QK^T/√d` + mask | hand-computed 4×4 | matches `F.scaled_dot_product_attention` to ~1e-6 fp32 |
| 1.3 | Multi-head + causal mask cache | `bllm/src/ch06` | same |
| 1.4 | RMSNorm, SwiGLU MLP | closed form | gradient check vs `torch.autograd.gradcheck` |
| 1.5 | Transformer block, then full GPT | `bllm/src/ch09-10` | param count matches analytically; logits match oracle |
| 1.6 | Sampling: greedy, temperature, top-k, top-p | `bllm/src/ch14-15` | deterministic under seed; top-p matches oracle |
| 1.7 | Tokenize → train → generate, end to end | — | coherent Shakespeare from a tiny model |
| 1.8 | *(optional, off critical path)* BPE from scratch | `bllm/src/ch03_tokenizer.py` | only if curiosity demands it — must not gate Phase 2 |

**Exit criterion:** a from-scratch GPT whose forward output matches the `bllm`
oracle within fp32 tolerance, verified by a test file — not by eyeballing text.

---

## Phase 2 — Pretraining from scratch, on GPU

### Decisions (2026-09-25)

- **Reference architecture is `minimind/`'s `minimind-3` / `minimind-3-moe`.**
  8 layers, `d_model=768`, vocab 6400, GQA 8/4, QK-norm, RoPE θ=1e6, SwiGLU,
  tied embeddings. MoE adds **4 experts, top-1 routing, no shared expert**.
  (`minimind/` is a vendored copy at the repo root, sibling of `src/`. Upstream
  is the NTFS mount at `~/Public/minimind`; read the vendored one.)
- **Dense first.** Same repo, same code path, `use_moe` as the only difference.
  Gives a control run and a working loop before routing complexity lands.
- **Token budget: ~20 M tokens.** Designed so an epoch fits in about
  **1 h dense / 1.4 h MoE**.

  | model | params | ms/step | tok/s | 2 h buys |
  |---|---|---|---|---|
  | dense | 63.9 M | 1955 | 5564 | ~40 M tokens |
  | MoE 4×top-1 | 198.4 M (63.9 M active) | 2718 | 4002 | ~29 M tokens |

  Measured on this machine with minimind's own model code, batch 32 × seq 340,
  synthetic tokens — real runs carry dataloader overhead, hence the margin.
  ⚠️ A full 2-epoch run of the 1.2 GB corpus is **17–28 h here** (1.69 h/epoch
  on a 3090). Don't.
- **Both model sizes are known exactly and are the primary correctness oracle:**
  dense **63.9 M**, MoE **198.4 M total / 63.9 M activated**. minimind's released
  `pretrain_768.pth` and `pretrain_768_moe.pth` can be loaded into this code to
  diff logits — an exact check, not "the text looks fine".
- **Batch 32 × seq 340 → 8.2 GiB peak.** Stay under bs32/T512; the GTT pool is
  shared with the desktop.
- **Log `logits_loss` and `aux_loss` separately.** Expert collapse is invisible
  in the total loss.

| # | Build | Done when |
|---|---|---|
| 2.1 | Data pipeline: corpus → tokens → `uint16` shards on `<数据盘>/data`, memmapped | shards reproduce byte-exactly from source; loader yields (x,y) with correct causal offset |
| 2.2 | Training loop: AdamW, cosine schedule + warmup, grad clip, bf16 autocast, grad accumulation | loss curve matches a known-good nanoGPT-style run at the same seed/size |
| 2.3 | Checkpointing: resumable, atomic writes to `ckpt/`, step/model/config recorded | kill -9 mid-run, resume, loss continues on the same curve |
| 2.4 | Eval harness: val loss + a fixed-prompt sample every N steps, reproducible | two runs at the same seed give identical numbers |
| 2.5 | **Dense 63.9 M**, ~20 M tokens, batch 32 × seq 340 | loss curve with no divergence; measured tok/s + peak MiB reported |
| 2.6 | **MoE 198.4 M / 63.9 M active**, same data/seed/steps | same curve quality; MoE measured against dense (expect ~1.39× slower) |
| 2.7 | Ablations that actually teach: top-1 vs top-2 · 2/4/8 experts · aux coef 0 vs 5e-4 (watch collapse) · no-warmup · lr×10 | one measured number per change |
| 2.8 | Replace the Python expert loop with a grouped/fused implementation | measured speedup over 2.6, and bs16-vs-bs32 inverted behaviour explained |

**Exit criterion:** a model you can sample from, a loss curve you can defend, a
measured throughput number, and — for the MoE — a routing distribution with no
collapsed experts. Not "it trained".

**Open question (2.8):** smaller batches measured *faster* per token (bs16/T340
= 5002 tok/s vs bs32/T340 = 4094). That inverts the usual advice and suggests
the expert loop is launch/occupancy-bound, not GEMM-bound. Confirm, then fix.

**Tokenization is fixed for all of Phase 2:** minimind's 6400-vocab tokenizer.
No BPE from scratch — see Phase 1's decision note.

---

## Phase 3 — Post-training: SFT, then LoRA from scratch

**`trainllm/` is out of scope.** It was the original oracle here — a finished CPU
run of Qwen3-0.6B LoRA on GSM8K-125 with published numbers (loss 1.82 → 1.07,
token acc 0.67 → 0.77). Dropped as a dependency, so **the oracle is now `peft`
itself**: write it yourself, then diff against the library on the same batch.
That is the same "implement → verify → swap → diff" loop, and it does not depend
on a repo on an NTFS mount that Windows can delete.

Self-contained target: `Qwen/Qwen3-0.6B` (already in the HF cache), `openai/gsm8k`
(loads through the system `datasets`), a few hundred examples, verified on a
held-out slice.

| # | Build | Done when |
|---|---|---|
| 3.1 | SFT loss **from scratch**: shift + completion-only masking, ignore-index | masked loss equals TRL's on the same batch within tolerance |
| 3.2 | LoRA **from scratch**: `B·A` low-rank update, init (A kaiming / B zero), scaling `α/r` | trainable-param count matches `peft` **exactly** for the same config |
| 3.3 | Train with **your own** LoRA on Qwen3-0.6B + GSM8K, on GPU | loss falls monotonically; measured ms/step and peak MiB reported |
| 3.4 | Re-run the same config with `peft`+`trl` | **step 4 of the loop** — diff loss / memory / speed against yours, explain every gap |
| 3.5 | QLoRA: 4-bit NF4 + double quant + paged AdamW (`bitsandbytes`) | peak MiB measured against 3.3; the accuracy/memory tradeoff quantified |
| 3.6 | Eval that's honest: held-out GSM8K **exact-match**, not train loss | base vs LoRA vs QLoRA accuracy, plus a base-vs-tuned sample comparison |

**Exit criterion:** your LoRA matches `peft`'s numbers on the same data, and you
can say *why* QLoRA traded accuracy for memory, with numbers. Not "loss went
down".

⚠️ `bitsandbytes` 4-bit is already verified working on this ROCm build —
`Qwen3-0.6B` at **959.1 MiB** peak (4-bit) vs **1136.9 MiB** (bf16).

---

## Phase 4 — RL from scratch

| # | Build | Done when |
|---|---|---|
| 4.1 | Policy gradient on a toy bandit / gridworld | reproduces `~/Public/rl/reasoning_rl.py`'s learning curve |
| 4.2 | Add advantage/baseline, then KL penalty to a reference policy | KL term measurably prevents drift in a run where it's disabled |
| 4.3 | **GRPO** from scratch: group sampling, group-relative advantage | beats the 4.1 baseline on the same task, measured |
| 4.4 | Verifiable task: GSM8K exact-match reward on the Phase 3 model | reward climbs; inspect samples for reward hacking |
| 4.5 | Diff against `trl`'s `GRPOTrainer` | same reward trajectory, gaps explained |

**Exit criterion:** a model that measurably improved on a task with a *verifiable*
reward, and can tell you what it did *not* improve.

---

## Phase 5 — Optional tails

Only after Phase 4. Each is a separate experiment, not a default.

- 量化到 GGUF,在 llama.cpp 下跑,比吞吐和质量。
- 长上下文:`T` 扩到 1024 以上,测 attention 的墙。

> ⚠️ **2026-09-28 更正:** 这里原本写"35 B 的 `qwen36` MoE 已在盘上,当天花板对照物"。
> 那个模型(**22 G**)已在 2026-09-28 删掉。要重下:
> `hf download Kreuzzelg/qwen36-35b-a3b-colibri-i4-gs64`
> 另外 `dev.zsh` 里的 `export COLI_MODEL=<数据盘>/models/qwen36` 现在是**悬空的**,
> 那个目录已不存在。

---

## Standing rules

Carried from `AGENTS.md`, restated because they're load-bearing here:

1. Never `uv pip install` a torch-dependent package without `--no-deps`
   (dry-run grep first). Never `uv pip install torch`. Never `pip install` into
   system Python.
2. Never set `HSA_OVERRIDE_GFX_VERSION`.
3. Artifacts on the data disk; `ckpt/` and `data/` are already symlinked there.
4. `~/Public` is an **NTFS mount Windows can modify** — `lfs/` already
   disappeared once. Read from it; never depend on it. Copy anything needed.
5. Report measurements, not adjectives. ms/step, MiB peak, tokens/s, loss.
6. Single-file, explicit, commented *why*. No trainer wrappers.
7. Re-run `scripts/check_gpu.py` after **any** environment change.
