---
type: atomic
layer: L1 基础原理
topic: tokenizer
tags: [atomic, tokenizer]
aliases: []
status: evergreen
source: [raw/LLM工程/03-LLM核心机制, raw/LLM工程/10-LLM原理]
hub: [LLM体系]
upstream: [位置编码, LLM机制]
downstream: [Embedding]
related: []
---

# Tokenizer

## 定义
Tokenizer 是把文本切分为 token 序列的机制。

## 解决的问题
- 文本如何进入模型
- 上下文窗口如何被消耗
- 多语言和 OOV 如何处理

## 核心机制
- BPE
- WordPiece
- SentencePiece

## 关键判断
- Tokenizer 直接影响成本、上下文利用率和 RAG 切分表现

## 上游
- [[位置编码]]
- [[LLM机制]]

## 下游
- [[Embedding]]

## 相关
