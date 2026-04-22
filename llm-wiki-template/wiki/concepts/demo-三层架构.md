---
type: concept
status: demo
demo_file: true
name: "三层架构"
aliases: ["Raw-Wiki-Schema"]
related:
  - "[[demo-LLM-Wiki]]"
  - "[[demo-Ingest-Query-Lint]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# 三层架构

> LLM Wiki 的核心组织方式，由 Raw Sources、Wiki 和 Schema 三层组成。

## 核心要点
- Raw Sources 只读，保存原始资料。
- Wiki 层负责结构化知识沉淀。
- Schema 层负责定义组织方式与工作流。

## 详细说明
Raw Sources 是所有知识的原始依据，强调原文不可随意修改。  
Wiki 层是由 LLM 生成和维护的结构化 Markdown 文件集合，承载摘要、概念、实体和综合洞见。  
Schema 则像操作宪法，约束 LLM 如何 ingest、如何 query、如何 lint。

## 与其他概念的关系
- 支撑了 [[demo-LLM-Wiki]]：三层架构是方案基础。
- 通过 [[demo-Ingest-Query-Lint]] 运转：三种核心操作在这三层之间流动。

## 应用场景
所有需要将原始资料与 AI 生成知识严格分层的知识库系统。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
