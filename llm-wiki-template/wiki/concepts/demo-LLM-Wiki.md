---
type: concept
status: demo
demo_file: true
name: "LLM-Wiki"
aliases: ["LLM Wiki"]
related:
  - "[[demo-RAG]]"
  - "[[demo-三层架构]]"
  - "[[demo-Ingest-Query-Lint]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# LLM-Wiki

> 一种由大模型持续维护的结构化个人知识库方案，核心是把知识从“临时检索”变成“持久编译”。

## 核心要点
- 原始资料只读，结构化 wiki 由 LLM 负责持续维护。
- 新资料进入后，不只是回答问题，而是会更新已有知识页面。
- 好的查询结果本身也可以反向回写为新知识。

## 详细说明
LLM-Wiki 是 Karpathy 提出的一种个人知识管理思路。它不是简单地把大量文档交给模型临时检索，而是让模型把知识整理成持久化的 Markdown Wiki。  
它强调知识是可编译、可积累、可维护的，因此回答质量会随着资料输入和使用过程逐步提升。  
在本文语境下，LLM-Wiki 更像一个架构蓝图，真正的难点在于如何定义知识提取粒度和更新规则。

## 与其他概念的关系
- 区别于 [[demo-RAG]]：LLM-Wiki 强调持久化积累，RAG 更偏向临时检索。
- 建立在 [[demo-三层架构]] 之上：raw、wiki、schema 三层分工明确。
- 依赖 [[demo-Ingest-Query-Lint]]：通过三种核心操作实现持续维护。

## 应用场景
个人知识库、团队内部知识库、研究笔记、竞品分析、尽职调查、课程学习与长期写作积累。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/articles/demo-001-Karpathy的LLM%20Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
