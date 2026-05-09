---
type: concept
status: active
name: "KV-Cache"
aliases: ["KV Cache", "Key-Value Cache", "键值缓存"]
related: ["Self-Attention", "多头注意力", "FlashAttention", "分布式训练"]
sources:
  - raw/engineering/03-理论-LLM核心机制.md
  - raw/engineering/10-LLM原理.md
  - raw/engineering/02-理论-Transformer.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# KV-Cache

> 一句话定义：KV Cache 的本质，是把“已经算过的历史上下文表示”保留下来，避免自回归生成时一遍遍重复计算过去。

## 第一性原理
- Decoder 生成第 `t` 个 token 时，新的 Query 需要读取前 `1..t-1` 个位置的信息。
- 过去 token 的 Key/Value 一旦确定，就不会因为后续生成而改变。
- 因此最自然的优化不是重新计算历史，而是缓存历史的 K/V，只增量计算新 token 的部分。

## 核心机制
- Prefill 阶段：对整个输入 prompt 一次性计算各层 K/V 并存入缓存。
- Decode 阶段：每生成一个新 token，只计算该 token 的 Q/K/V，并与历史缓存交互。
- 这样每一步避免了对全部历史 token 重新投影，显著降低重复算力。
- 缓存通常按 layer、head、sequence 维度组织，因此模型结构会直接决定缓存大小。

## 关键权衡
- 收益：把推理从“重复做历史工作”转成“只做增量工作”。
- 代价：显存占用会随序列长度线性增长，长上下文下缓存本身会成为主瓶颈。
- 所以推理优化很快会从“算得快不快”转成“缓存放不放得下、调度得好不好”。
- [[机制-多头注意力]] 中 MHA、GQA、MQA 的差异，本质上也是在改 KV 缓存成本。

## 与其他概念的关系
- [[机制-Self-Attention]]：缓存的是注意力层中的历史 K/V，而不是最终 hidden state。
- [[机制-多头注意力]]：K/V 是否共享直接决定缓存体积。
- [[机制-FlashAttention]]：FlashAttention 优化单次 attention 的 IO，KV Cache 避免历史重算，两者正交。
- [[方法-分布式训练]]：推理并行时缓存也需要按张量切分策略同步布局。

## 应用边界
- 对 Encoder-only 模型价值较小，对 Decoder-only 生成模型价值极大。
- 在极长上下文服务中，缓存管理策略往往比单纯模型算力更关键。

## 来源
- [03-理论-LLM核心机制.md](../../raw/engineering/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../../raw/engineering/10-LLM原理.md)
- [02-理论-Transformer.md](../../raw/engineering/02-理论-Transformer.md)
