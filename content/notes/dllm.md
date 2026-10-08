# Diffusion Language Model (DLM)

Diffusion Language Models (DLMs) are a class of generative models that leverage diffusion processes to generate high-quality data samples, such as images, audio, and text. Unlike traditional autoregressive models, DLMs iteratively refine random noise into coherent outputs through a series of denoising steps.

连续的 词嵌入空间

文本长度问题

KV cache 问题

语言嵌入空间的 _噪声_ 不是高斯分布的，不像图片那样有自然的语义

Scaling low proble

是否允许已生成的内容回退？

## Painter

绘画需要现有骨架草稿，再补充细节。
例如绘画人体，虽然画出的是皮肤或者衣服，但画家仍然要知道骨骼和肌肉。
如果只看到皮肤，能模拟出形体的皮肤，但不能推理出形体的皮肤。

## diffusion

不应该从噪声扩散成结果，而应该从骨架扩散成结果。

## skeleton

产生骨架的能力，是AGI的标准。

AGI 的全称是 Artificial General Intelligence，即人工通用智能。
我认为这里的 General 不是指通常的、一般的、大部分的任务，不是指能做到很多具体的任务。
General 应该指「一个」能力，一个「形式的」而非「具体的」能力。
更进一步地，从这个能力出发，面对一个全新的任务，能够把这个形式能力应用到任务上，得到这个任务的「具体的」能力。
这是「能」做到一般性任务的能力，和对各种各样的任务具有完成它的能力，前者是单数，后者是复数。

或者更简洁通俗地，有先天认知能力，和后天综合实践和学习能力。
后天学习的能力，必然与先天能力有区分，从实现方式和存储方式上。（也许RAG是一个不成熟的尝试）
