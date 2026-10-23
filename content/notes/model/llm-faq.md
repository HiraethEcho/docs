---
title: FAQ for LLM
date: 2026-10-13
summary:
tags:
categories:
topics:
series:
genres:
status:
---

# FAQ for LLM

prefill decoder

稀疏自编码器 sparse autoencoders

## transformer

### Q/K/V

MHA/MQA/GQA
KVcache

### RoPE

**为什么作用在 Q/K，不在 embedding，也不在 V**

注意力分数来自 $\text{scores} = QK^\top/\sqrt{d_k}$。位置信息要影响「关注谁」，就得直接进入这个匹配打分。

- 加在 **embedding** 上：位置信息要先穿过很多层线性变换，到了某一层算 `QK^T` 时未必还以直接的形式存在。
- 加在 **V** 上：V 是被加权汇总的「内容」，改它只改「取出来什么」，不改「关注谁」。
- 加在 **Q、K** 上：直接改变每一对 query-key 的点积，位置立刻参与匹配。

所以 RoPE 选择改 Q、K。

### Attention

flash attention
page attention

## FFN

### Norm

RMSNorm vs LayerNorm 的区别：RMSNorm 不做均值中心化（没有减去 mean），只做方差归一化。计算量更少，效果相当。

### 激活

Swiglu/Relu

## post train

LoRA
