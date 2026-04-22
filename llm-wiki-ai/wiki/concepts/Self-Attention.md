---
type: concept
status: active
name: "Self-Attention"
aliases: ["自注意力", "Scaled Dot-Product Attention", "注意力机制"]
related: ["多头注意力", "KV-Cache", "FlashAttention", "Tokenizer"]
sources:
  - ../raw/LLM工程/02-理论-Transformer.md
  - ../raw/LLM工程/09-Transformer.md
  - ../raw/LLM工程/03-理论-LLM核心机制.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Self-Attention

> 一句话定义：Self-Attention 通过 Q/K/V 三个矩阵实现序列内每个位置对其他所有位置的动态加权聚合，使模型能捕获全局依赖关系。

## 核心要点
- 计算公式：Attention(Q, K, V) = softmax(QK^T / √d_k) · V，其中 Q/K/V 分别由输入经线性投影得到
- **Q/K 负责匹配，V 负责承载内容**：Q（查询）和 K（键）的点积衡量两个位置的相关性，V（值）的加权和是实际聚合的信息；匹配和聚合解耦，使注意力模式可以灵活学习
- **除以 √d_k 是数值稳定性的必要操作**：d_k 维向量的点积方差为 d_k，高维时点积值很大，softmax 在极端值处梯度趋近于零（饱和）；除以 √d_k 将方差归一化到 1，保持梯度有效传播
- **计算复杂度是 O(n²d)**：n 个位置两两计算注意力权重，序列翻倍计算量四倍增长，这是长文本的根本瓶颈
- Decoder 中的 Causal Mask（因果掩码）：将上三角设为 -∞（softmax 后为 0），强制每个位置只能看到自身及之前的 token，保证自回归生成的因果性

## 详细说明

Self-Attention 的核心设计哲学是"动态权重"：传统 RNN 对序列的处理是固定路径的顺序传递，Self-Attention 允许每个位置直接与序列中任意位置交互，权重由当前输入动态决定（而不是固定的连接权重）。这使得 Transformer 能更有效地捕获长距离依赖，同时支持并行计算。

注意力矩阵 QK^T / √d_k 的维度是 [n, n]，每一行是一个 query 对所有 key 的相关性分数，softmax 归一化后得到注意力权重（概率分布）。V 矩阵乘以注意力权重，相当于对所有位置的 Value 做加权平均，权重越高的位置贡献越多。

推理阶段的 Self-Attention 与训练阶段有本质差异：训练时所有位置并行计算（Teacher Forcing），推理时必须逐步生成（自回归），每次生成新 token 需要重新计算它与所有历史 token 的注意力——这就是为什么需要 [[KV-Cache]]：将历史 token 的 K/V 矩阵缓存下来，避免重复计算。

## 与其他概念的关系
- 延伸自 Self-Attention → [[多头注意力]]：多头注意力是 Self-Attention 的并行扩展，在多个子空间独立学习不同的注意力模式
- 关联 [[KV-Cache]]：KV Cache 缓存的正是 Self-Attention 中的 K/V 矩阵，是推理加速的基础
- 关联 [[FlashAttention]]：FlashAttention 通过 IO 优化重新实现 Self-Attention 的计算，结果相同但速度和显存效率更高
- 区别于 Cross-Attention：Self-Attention 的 Q/K/V 来自同一序列，Cross-Attention 的 Q 来自 Decoder，K/V 来自 Encoder 输出

## 应用场景
- Encoder-only 模型（BERT）：双向 Self-Attention，每个位置可以看到整个序列，适合分类/NER/QA
- Decoder-only 模型（GPT/LLaMA）：因果 Self-Attention，每个位置只看之前的 token，适合生成任务
- 长文本处理：Sliding Window Attention（稀疏注意力）、FlashAttention（IO 优化）是处理万 token 以上序列的实用方案

## 来源
- [02-理论-Transformer.md](../raw/LLM工程/02-理论-Transformer.md)
- [09-Transformer.md](../raw/LLM工程/09-Transformer.md)
- [03-理论-LLM核心机制.md](../raw/LLM工程/03-理论-LLM核心机制.md)
