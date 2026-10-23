---
title: 模型训练基础
date: 2026-10-14
summary:
tags:
categories:
topics:
series:
genres:
status:
---

# 模型训练基础

基于 `nano_llm_from_scratch` 的学习记录。
see more about [llm](/notes/model/llm-101)
and a mini [example](/notes/model/train-mini) of pseudo codes.

## prepare

### Tokenizer

BPE 分词，在预分词后合并高频组合。通过原始字节而非字符的起始分词，可以保证不会失效，代价是低频词可能需要很多token

设置参数

```py
if use_bpe:
    vocab_size = vocab_size_bpe
else:
    vocab_size = len(chars) # token词表的长度，which is the number of unique characters in this case
```

### 训练参数

```py
batch_size = 128 # 训练的batch size
block_size = 16 # 输入模型的token的最大数量
learning_rate = 1e-4 # 学习率
print_interval = 200 # 打印训练信息的间隔，每间隔print_interval个步数打印训练信息
num_epoch = 10 # 训练的epoch数
```

### Data

```py
class MyDataset(Dataset):
    def __init__(self, textdata, block_size):
        self.textdata = textdata
        self.block_size = block_size

    def __len__(self):
        return len(self.textdata) - self.block_size  # 需保证 idx+1+block_size <= len，故最后一个合法 idx 为 len-block_size-1，样本数为 len-block_size

    def __getitem__(self, idx):
        return {
            "input": torch.tensor(self.textdata[idx:idx+self.block_size], dtype=torch.long),
            "target": torch.tensor(self.textdata[idx+1:idx+1+self.block_size], dtype=torch.long),
        }

train_dataset = MyDataset(train_set, block_size)
train_datasetloader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True, num_workers = 4)

test_dataset = MyDataset(test_set, block_size)
test_datasetloader = DataLoader(test_dataset, batch_size=batch_size, shuffle=True, num_workers = 4)
```

## 训练

### LOSS

模型每个位置输出的，是“下一个 token 在整个词表上的概率分布” $p_\theta(\cdot \mid \text{context}) = \mathrm{softmax}(\text{logits})$。训练的目标就是：让模型给**真实出现的那个下一个 token** 尽量高的概率。交叉熵正是把这句话写成一个可优化的损失。

**它本质上就是最大似然（MLE）。** 把某个位置真实的下一个 token 记作 $y$，该位置的交叉熵损失是

$$
\ell = -\log p_\theta(y \mid \text{context})
$$

在全部 $B\times T$ 个位置上平均：

$$
\mathcal{L} = -\frac{1}{BT}\sum_{\text{所有位置}} \log p_\theta(y \mid \text{context})
$$

最小化 $\mathcal{L}$ $\Longleftrightarrow$ 最大化模型赋予整份训练数据的对数似然 $\Longleftrightarrow$ **极大似然估计**。这正是前文 $p(t_k \mid t_1,\dots,t_{k-1}, \theta)$ 的训练版：把概率质量尽量压到真实出现的 token 上。

**信息论视角：这里的交叉熵就等于 KL 散度。** 记真实分布 $q$ 为 one-hot（只有 $y$ 处为 1），模型分布为 $p$，则

$$
H(q, p) = -\sum_i q_i \log p_i = -\log p_y
$$

又有恒等式 $H(q,p) = H(q) + D_{\mathrm{KL}}(q \parallel p)$，而 one-hot 分布的熵 $H(q)=0$，于是在“目标是 one-hot”的语言建模里，**交叉熵恰好等于 KL 散度** $D_{\mathrm{KL}}(q \parallel p)$——最小化它，就是把模型分布 $p$ 往数据分布 $q$ 拉近。而 $-\log p_y$ 本身是“真实 token 在模型眼里的**惊讶度 / 信息量**”，训练就是最小化平均惊讶。

**直觉：**

- 答对且自信（$p_y \to 1$）→ loss $\to 0$；而**答错却自信（$p_y \to 0$）→ loss $\to +\infty$**——对“自信地犯错”重罚，逼模型诚实地给出概率。
- 它只盯着“正确 token 拿到多少概率”，不关心其余 token 之间怎么瓜分——正好契合 next-token 任务。

**为什么不用 MSE 或准确率？**

- 准确率不可导，没法反向传播。
- 对分类用 MSE，梯度在 softmax 饱和区会趋近 0、学得很慢；而 **softmax + 交叉熵的梯度特别干净**：$\partial \mathcal{L} / \partial\, \text{logit}_i = p_i - q_i$（预测概率 − one-hot 目标），数值稳定、好优化。
