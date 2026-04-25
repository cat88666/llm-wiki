---
type: concept
status: active
name: "Tokenizer"
aliases: ["分词器", "BPE", "SentencePiece", "WordPiece", "tokenization"]
related: ["Self-Attention", "KV-Cache", "Prompt工程", "LLM体系"]
sources:
  - ../raw/engineering/03-理论-LLM核心机制.md
  - ../raw/engineering/10-LLM原理.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# Tokenizer

> 一句话定义：Tokenizer 的本质，是把连续文本离散化为模型可处理的基本符号单位，并决定模型“看到世界”的粒度。

## 第一性原理
- 神经网络不能直接处理字符串，只能处理有限词表中的离散 ID。
- 所以语言建模的第一步不是理解语义，而是先把文本切成某种可枚举的最小单位。
- 这个切分不是中性的：切得太粗会丢泛化能力，切得太细会拉长序列、放大计算成本。

## 核心机制
- 训练阶段从语料中学习高频子词合并规则或词表。
- 推理阶段把新文本映射成 token 序列，再交给 embedding 和 [[机制-Self-Attention]]。
- 子词方案的核心目标，是在“词表大小”和“平均序列长度”之间取得平衡。
- 特殊 token 进一步承载边界、角色、分隔和控制信号。

## 关键权衡
- 词表越大，单个 token 表达越完整，但嵌入矩阵更大、冷门词泛化更差。
- 词表越小，覆盖越稳，但同一段文本会拆成更多 token，直接推高计算和 [[机制-KV-Cache]] 成本。
- 对中文、代码、数字、公式这类分布差异大的场景，Tokenizer 设计会直接影响模型体验。

## 与其他概念的关系
- [[机制-Self-Attention]]：Tokenizer 输出的是注意力机制处理的输入单位。
- [[机制-KV-Cache]]：token 越多，缓存越大。
- [[方法-Prompt工程]]：同样一句话的 token 化方式不同，会改变成本和上下文利用率。
- [[体系-LLM体系]]：Tokenizer 是从文本进入 Transformer 的入口层，而不是附属细节。

## 应用边界
- Tokenizer 不直接提供语义理解，但它定义了模型能以什么粒度学习语义。
- 很多“模型不擅长某类字符串”的问题，根因其实在 token 切分而不只在参数规模。

## 来源
- [03-理论-LLM核心机制.md](../raw/engineering/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../raw/engineering/10-LLM原理.md)
