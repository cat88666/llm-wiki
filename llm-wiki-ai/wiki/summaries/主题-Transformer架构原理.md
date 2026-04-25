---
type: summary
status: active
summary_scope: topic
title: "Transformer架构原理"
source_count: 2
sources:
  - ../raw/engineering/02-理论-Transformer.md
  - ../raw/engineering/09-Transformer.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# Transformer架构原理

> 这个页面总结的是 Transformer 的架构设计原理：Self-Attention 机制、多头注意力、Decoder 结构、训练与推理的差异，以及工程稳定性设计（LayerNorm、残差连接）。

## 知识核心
- **Self-Attention 是匹配与聚合的解耦设计**：Q/K 计算 token 之间的相关性（匹配），V 承载实际内容（聚合），三者分工明确，这个设计使得注意力可以全局动态计算
- **除以 √d_k 是数值稳定性的必要条件**：高维点积因为累加项多，方差正比于 d_k，不缩放会导致 softmax 饱和在极端值，梯度消失
- **多头注意力的核心价值是子空间多样性**：不同 Head 分别学习指代关系、句法关系、语义关系，且投影后总参数量与单头相同，没有额外代价
- **Attention 复杂度是 O(n²)**：序列翻倍，计算量四倍增长，这是长文本处理的根本瓶颈，需要稀疏/线性/低秩近似或 [[机制-FlashAttention]] 优化
- **Pre-LN 比 Post-LN 更稳定**：LayerNorm 放在子层输入前，残差路径梯度传播更平顺，现代大模型（GPT-3 之后）几乎全部采用 Pre-LN

## 共识内容
- 残差连接是解决深层网络梯度消失的关键结构，使得梯度可以绕过子层直接传播
- Decoder 有两层 Attention：Masked Self-Attention（保证自回归因果性）+ Cross-Attention（对齐 Encoder 表征），两者缺一不可
- 训练时 Teacher Forcing 并行计算所有位置，推理时必须逐步自回归生成，存在 Exposure Bias（训练时看真实 token，推理时看自己预测的 token）
- Batch 大小影响训练稳定性：大 batch 梯度估计更稳定，但可能陷入尖锐极小值，学习率需相应调整

## 关键分歧
- **Pre-LN vs Post-LN**：Post-LN 是原始论文设计，理论上归一化效果更好，但训练初期梯度不稳定，需要 warmup；Pre-LN 更容易训练，但表达能力略有折损，工程上 Pre-LN 是主流选择
- **绝对位置编码 vs RoPE**：原始 Transformer 用正弦绝对编码，RoPE（旋转位置编码）在 LLaMA 系列中成为标准，支持长度外推且保持相对位置关系

## 适用边界
- 适用场景：序列建模、语言模型、机器翻译、文本分类，任何需要全局依赖建模的任务
- 不适用场景：极长序列（数万 token 以上）的直接全注意力计算（O(n²) 代价过高），需要低延迟流式推理时 Decoder 架构需要 KV Cache 支持

## 关键概念
- [[机制-Self-Attention]]：Transformer 的核心计算单元，通过 Q/K/V 实现动态全局依赖建模
- [[机制-多头注意力]]：Self-Attention 的扩展，多个子空间并行学习不同类型的依赖关系
- [[机制-KV-Cache]]：推理阶段缓存 K/V 矩阵，避免重复计算历史 token 的注意力，是推理加速的基础
- [[机制-FlashAttention]]：IO 感知的注意力实现，通过减少 HBM 读写实现 2-4x 加速，不改变数学结果

## 值得注意的点
- Attention 的 O(n²) 复杂度是"时间+空间"双重瓶颈：计算量 O(n²d)，但内存中存储完整 Attention 矩阵是 O(n²)，FlashAttention 主要解决的是后者
- Exposure Bias 在短文本影响有限，但在长文本生成（摘要、代码）中累积误差会显著影响质量，是强化学习微调（RLHF）的动机之一
- 多头注意力中每个 Head 的维度是 d_model / num_heads，并行计算后 concat 再投影，这保证了总参数量不变

## 延伸方向
- 可拆分为 [[机制-Self-Attention]] 和 [[机制-多头注意力]] 的详细 concept 页
- 可延伸到 [[机制-KV-Cache]] 和 [[机制-FlashAttention]] 的工程优化 concept 页
- 可连接到 [[主题-LLM核心机制与推理优化]] summary 页（GQA、分布式训练）

## 来源
- [02-理论-Transformer.md](../raw/engineering/02-理论-Transformer.md)
- [09-Transformer.md](../raw/engineering/09-Transformer.md)
