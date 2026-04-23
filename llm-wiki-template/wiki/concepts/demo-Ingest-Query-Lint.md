---
type: concept
status: demo
demo_file: true
name: "Ingest-Query-Lint"
aliases: []
related:
  - "[[demo-LLM-Wiki]]"
  - "[[demo-三层架构]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# Ingest-Query-Lint

> LLM Wiki 的三类核心操作：摄入、查询和健康检查。

## 核心要点
- Ingest 负责把新资料变成结构化知识。
- Query 负责基于现有知识页生成回答，并可回写结果。
- Lint 负责检查矛盾、过时内容和孤立页面。

## 详细说明
Ingest 是最关键的入口操作，它决定一篇原始文章会影响哪些概念、实体和综合页。  
Query 不只是问答，而是可能产生新的知识沉淀。  
Lint 是让知识库具备自我修正能力的关键机制，也是作者认为自己方案尚未完全显式化的部分。

## 与其他概念的关系
- 是 [[demo-LLM-Wiki]] 的运行机制。
- 建立在 [[demo-三层架构]] 之上。
- 可与 [[demo-本体模型-M1-M4]] 结合，让知识提取更稳定。

## 应用场景
任何需要长期维护结构化知识库的 AI 工作流。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/articles/demo-001-Karpathy的LLM%20Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
