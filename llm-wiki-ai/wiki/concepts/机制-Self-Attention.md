---
type: concept
status: active
name: "Self-Attention"
aliases: ["自注意力", "Scaled Dot-Product Attention", "注意力机制"]
related: ["多头注意力", "KV-Cache", "FlashAttention", "Tokenizer"]
sources:
  - ../raw/engineering/02-理论-Transformer.md
  - ../raw/engineering/09-Transformer.md
  - ../raw/engineering/03-理论-LLM核心机制.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# Self-Attention

> 一句话定义：Self-Attention 的本质，是让序列中每个位置按“与我当前最相关”的程度，从全序列中有选择地读取信息。

## 第一性原理
- 序列建模的根问题，不是“把 token 排成一列”，而是“当前 token 需要依赖哪些其他 token 才能决定自己的表示”。
- RNN 用固定的时间路径传播信息，长距离依赖会衰减；Self-Attention 改成显式两两匹配，让每个位置都能直接访问整个上下文。
- 所以 Q/K/V 不是技巧堆砌，而是把“找谁”和“拿什么”拆开：Q/K 负责相关性判断，V 负责真正被聚合的内容。

## 核心机制
- 对每个位置生成 Q、K、V 三个向量。
- 用 `Q · K^T` 计算“我该关注谁”，再经 `softmax` 变成权重分布。
- 用该分布对所有 V 做加权求和，得到当前位置的新表示。
- `÷ √d_k` 的作用不是装饰，而是控制点积尺度，避免高维下 softmax 过早饱和、梯度失效。
- Decoder 中加入 causal mask，本质是在机制层面强制“只能读过去，不能偷看未来”。

## 关键权衡
- 优势：全局依赖建模强、并行性好、表达能力高。
- 代价：任意两位置都要交互，时间和显存复杂度接近 `O(n^2)`，长上下文会迅速变贵。
- 这也是后续 [[机制-FlashAttention]]、稀疏注意力、窗口注意力和 [[机制-KV-Cache]] 出现的根本原因。

## 与其他概念的关系
- [[机制-多头注意力]]：把同一个注意力问题拆到多个子空间并行求解。
- [[机制-KV-Cache]]：把历史 token 的 K/V 缓存下来，避免自回归推理时重复计算。
- [[机制-FlashAttention]]：不改变注意力结果，只重排计算顺序以减少显存读写。
- [[概念-Tokenizer]]：决定输入被切成什么粒度，进而影响注意力看到的基本单位。

## 应用边界
- Encoder-only 模型中，多为双向 Self-Attention，适合理解任务。
- Decoder-only 模型中，多为因果 Self-Attention，适合生成任务。
- 当上下文极长时，瓶颈不在“会不会注意”，而在“算不算得起”。

## 来源
- [02-理论-Transformer.md](../raw/engineering/02-理论-Transformer.md)
- [09-Transformer.md](../raw/engineering/09-Transformer.md)
- [03-理论-LLM核心机制.md](../raw/engineering/03-理论-LLM核心机制.md)
