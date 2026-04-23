---
title: "RAG"
type: concept
tags: [概念, RAG, 检索增强生成, 应用工程]
sources:
  - raw/course/02-RAG检索增强.md
  - raw/llm-engineering/05-工程-RAG.md
  - raw/llm-engineering/11-LLM工程.md
last_updated: 2026-04-21
---

## 定义

RAG（Retrieval-Augmented Generation，检索增强生成）是一种在生成答案前先检索外部知识，再把证据注入上下文的系统模式。它解决的重点不是提升模型推理上限，而是降低知识过期、事实缺失和幻觉风险。

## 核心链路

### 1. 离线知识库构建
- 文档接入与解析
- 文本清洗与标准化
- Chunking 切片
- Embedding 向量化
- 写入向量数据库并附带元数据

### 2. 在线检索与生成
- 用户查询进入系统
- 进行查询改写、扩展或分解
- 执行向量检索，必要时叠加 BM25
- 用 Reranker 精排结果
- 组装上下文并交给 LLM 生成答案

## 工程要点

### 切片策略
- 过小会导致语义不完整
- 过大会引入噪声
- 常见做法是固定长度加重叠，或按语义结构切片

### 检索质量
- 向量检索适合语义召回
- BM25 更适合关键词、型号、代码、否定查询
- 混合检索与重排序通常比单一路线更稳

### 幻觉控制
- System Prompt 要求“仅基于资料回答”
- 检索结果要附来源信息
- 用 Faithfulness 等指标做持续评估

## 适用场景
- 企业知识库问答
- 文档助手与客服系统
- 多文档检索与引用回答
- Text2SQL、ChatBI 等需要外部上下文的系统

## 关联连接
- [[Context_Engineering]] — RAG 是上下文工程的关键组成
- [[Agent]] — Agent 可把 RAG 作为工具链的一部分
- [[AI工程体系]] — RAG 属于核心工程能力模块
- [[摘要-22期课程-rag检索增强]] — 课程级摘要
- [[摘要-22期课程-提示工程到rag]] — 从 Prompt 过渡到 RAG 的课程摘要
