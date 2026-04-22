---
type: atomic
layer: L2 能力模块
topic: chunking
tags: [atomic, chunking]
aliases: [切分]
status: evergreen
source: [doc/00原理/05-工程-RAG, doc/00原理/10-LLM原理]
hub: [LLM体系, AI工程体系]
upstream: [RAG]
downstream: [知识库问答]
related: []
---

# Chunking

## 定义
Chunking 是将文档切成适合检索和拼接上下文的文本片段。

## 解决的问题
- 长文档无法直接高质量检索
- 语义上下文容易被切断
- 上下文长度有限

## 核心机制
- 固定长度切分
- 递归切分
- 基于标题结构切分
- Parent-Child 切分

## 关键判断
- Chunk 策略通常比单纯换模型更先决定 RAG 上限

## 上游
- [[RAG]]

## 下游
- [[知识库问答]]

## 相关
