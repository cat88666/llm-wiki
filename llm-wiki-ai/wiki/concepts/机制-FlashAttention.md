---
type: concept
status: active
name: "FlashAttention"
aliases: ["Flash Attention", "IO-aware Attention", "FlashAttention-2", "FlashAttention-3"]
related: ["Self-Attention", "多头注意力", "KV-Cache", "分布式训练"]
sources:
  - ../raw/engineering/03-理论-LLM核心机制.md
  - ../raw/engineering/09-Transformer.md
  - ../raw/engineering/10-LLM原理.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# FlashAttention

> 一句话定义：FlashAttention 的本质，不是改变注意力公式，而是把“注意力该怎么算”改成“在 GPU 内存层级上最省搬运地算”。

## 第一性原理
- 标准 [[机制-Self-Attention]] 的数学并不复杂，真正慢的是中间巨大矩阵在 HBM 和片上缓存之间反复搬运。
- GPU 上很多算子不是算力受限，而是访存受限。注意力正是典型的 memory-bound 问题。
- 所以 FlashAttention 抓住的第一性矛盾不是“减少 FLOPs”，而是“减少显存 IO”。

## 核心机制
- 将 Q、K、V 按块切分，在片上 SRAM 中完成局部注意力计算。
- 不把完整 `n × n` 注意力矩阵落到 HBM，而是边算边累计输出。
- 通过 online softmax 在分块条件下保持与标准 softmax 数学等价。
- 训练反向传播时，用“重算部分中间量”换显存占用下降。

## 关键权衡
- 数学结果与标准 attention 等价，收益主要体现在速度和显存。
- 优势在长序列和大 batch 下更明显，因为此时中间矩阵最昂贵。
- 代价是实现更依赖底层硬件、kernel 和框架版本，不是所有环境都能无缝吃满收益。

## 与其他概念的关系
- [[机制-Self-Attention]]：FlashAttention 是其高效实现，而不是替代机制。
- [[机制-多头注意力]]：对每个头分别应用同样的分块思想。
- [[机制-KV-Cache]]：两者都服务推理优化，但一个减少重复计算，一个减少单次 IO。
- [[方法-分布式训练]]：可与序列并行、张量并行结合，处理更长序列训练。

## 应用边界
- 如果序列不长、实现不支持或硬件不匹配，收益可能有限。
- 它解决的是“算 attention 太贵”，不解决“上下文太长导致缓存太大”的所有问题。

## 来源
- [03-理论-LLM核心机制.md](../raw/engineering/03-理论-LLM核心机制.md)
- [09-Transformer.md](../raw/engineering/09-Transformer.md)
- [10-LLM原理.md](../raw/engineering/10-LLM原理.md)
