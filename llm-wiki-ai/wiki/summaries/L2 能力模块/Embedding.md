---
type: atomic
layer: L2 能力模块
topic: embedding
tags: [atomic, embedding]
aliases: []
status: evergreen
source: [raw/engineering/05-工程-RAG, raw/engineering/10-LLM原理]
hub: [LLM体系, AI工程体系]
upstream: [Tokenizer, RAG]
downstream: [特征工程]
related: []
---

# Embedding

## 定义
Embedding 是把文本表示为向量的语义编码方式。

## 解决的问题
- 支撑语义相似度检索
- 统一文本向量空间
- 提高召回阶段的语义匹配能力

## 核心机制
- 文本编码为稠密向量
- 相似度计算
- 索引与召回

## 关键判断
- Embedding 评估先看召回，再看端到端答案

## 上游
- [[RAG]]
- [[Tokenizer]]

## 下游
- [[特征工程]]

## 相关
