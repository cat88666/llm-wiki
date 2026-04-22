---
type: atomic
layer: L2 能力模块
topic: rag
tags: [atomic, rag]
aliases: [检索增强生成]
status: evergreen
source: [doc/00原理/05-工程-RAG, doc/22期/02-RAG检索增强]
hub: [LLM体系, AI工程体系]
upstream: [LLM机制, Prompt工程]
downstream: [检索策略, Chunking, Embedding, 重排序, 评估体系, 安全与权限控制, 可观测性, 知识库问答]
related: []
---

# RAG

## 定义
RAG 是将外部知识检索结果注入模型上下文再生成答案的方法。

## 解决的问题
- 模型知识过期
- 私有知识不可直接使用
- 回答缺乏可追溯依据

## 核心机制
- 文档解析
- 分块与向量化
- 检索、重排、上下文组装

## 关键判断
- 当知识经常变化且要求引用依据时，优先考虑 RAG

## 上游
- [[LLM机制]]
- [[Prompt工程]]

## 下游
- [[检索策略]]
- [[Chunking]]
- [[Embedding]]
- [[重排序]]
- [[评估体系]]
- [[安全与权限控制]]
- [[可观测性]]
- [[知识库问答]]

## 相关
