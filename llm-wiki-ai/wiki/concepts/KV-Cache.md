---
type: concept
status: active
name: "KV-Cache"
aliases: ["KV Cache", "Key-Value Cache", "键值缓存"]
related: ["Self-Attention", "多头注意力", "FlashAttention", "分布式训练"]
sources:
  - ../raw/engineering/03-理论-LLM核心机制.md
  - ../raw/engineering/10-LLM原理.md
  - ../raw/engineering/02-理论-Transformer.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# KV-Cache

> 一句话定义：KV Cache 在自回归推理时缓存所有历史 token 的 Key 和 Value 矩阵，避免每次生成新 token 时重复计算历史序列的注意力，是 LLM 推理加速的基础机制。

## 核心要点
- **为什么需要 KV Cache**：自回归生成每步生成一个 token，新 token 的注意力计算需要与所有历史 token 的 K/V 做点积；如果不缓存，每步都要重新计算全部历史 token 的 K/V 投影（重复计算量随序列长度线性增长）
- **KV Cache 的大小**：2 × num_layers × num_heads × head_dim × seq_len × dtype_bytes；70B 模型（80层，64头，128 head_dim，fp16）下，4096 token 的 KV Cache 约 16GB，与模型权重（140GB in fp16）相比不可忽视
- **KV Cache 是推理显存的主要瓶颈**：随序列长度线性增长；长上下文场景（32K-128K token）下 KV Cache 可超过模型权重本身
- **vLLM 的 PagedAttention**：借鉴操作系统虚拟内存的页表机制，将 KV Cache 划分为固定大小的"物理块"（page），按需分配，允许多请求共享物理块；将显存利用率从约 60% 提升到 90%+，大幅提升服务吞吐量
- **GQA/MQA 从根本上减小 KV Cache**：GQA 让多个 Query Head 共享一组 K/V，KV Cache 大小降为 MHA 的 1/group_ratio，是从模型架构层面优化推理显存的主要手段

## 详细说明

KV Cache 的工作机制：第一次 prefill 阶段，将整个输入 prompt 的 K/V 计算并存储；之后每次 decoding 步骤，只需计算新 token 的 Q，与缓存的全部历史 K/V 做注意力计算，输出新 token 的 K/V 并追加到缓存中。这使得 decoding 阶段每步的计算量从 O(n) 变为接近 O(1)（只计算一个新 token 的投影）。

KV Cache 的显存碎片化问题：传统实现为每个请求预先分配连续的最大序列长度显存（如 4096 token），但实际请求长度分布差异很大，导致大量显存浪费（填充未使用的位置）。PagedAttention 的核心创新是不预分配连续显存，而是类似操作系统的页表，用逻辑地址到物理块的映射管理 KV Cache，消除内部碎片和外部碎片。

KV Cache 的量化：将 KV Cache 从 fp16 量化为 int8（或 int4）可以减少一半（或四分之三）的显存占用，但会引入量化误差。实践中 KV Cache int8 量化（KV8）的精度损失通常可接受（< 0.5% 准确率下降），是长上下文场景的常用优化。

## 与其他概念的关系
- 直接依赖 [[Self-Attention]]：KV Cache 缓存的是 Self-Attention 中每层的 K/V 投影结果
- 关联 [[多头注意力]]：MHA/GQA/MQA 的选择直接影响 KV Cache 大小；GQA 是减小 KV Cache 的主要架构手段
- 关联 [[FlashAttention]]：FlashAttention 和 KV Cache 都是推理优化，但方向不同——FlashAttention 优化计算 IO，KV Cache 避免重复计算

## 应用场景
- 所有自回归 LLM 推理场景（GPT/LLaMA/Qwen 等）：KV Cache 是标配，不启用时推理延迟会急剧增加
- 长上下文服务（RAG、文档分析）：KV Cache 显存是系统容量瓶颈，需要 GQA + PagedAttention + 可能的量化组合
- 批量推理：PagedAttention 通过共享物理块支持高效 batching，同一 prefix 的多个请求可以共享 prefix 的 KV Cache（Prefix Caching）

## 来源
- [03-理论-LLM核心机制.md](../raw/engineering/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../raw/engineering/10-LLM原理.md)
- [02-理论-Transformer.md](../raw/engineering/02-理论-Transformer.md)
