---
type: concept
status: demo
demo_file: true
name: "本体模型-M1-M4"
aliases: ["M1-M4 四层模型"]
related:
  - "[[demo-LLM-Wiki]]"
  - "[[demo-article-cards]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# 本体模型-M1-M4

> 作者提出的四层建模框架，用来定义文章中提取什么、如何提取、按什么规则提取、服务什么场景。

## 核心要点
- M1 对象模型回答“有什么”。
- M2 行为模型回答“做了什么”。
- M3 规则模型回答“按什么规则抽取”。
- M4 场景模型回答“为什么这样写、服务什么场景”。

## 详细说明
作者认为 Karpathy 的 LLM Wiki 虽然给出了知识库架构，但没有明确知识抽取对象和规则。  
M1-M4 模型正是对这个空白的补全。  
它将文章解析从模糊的“提取关键信息”变成明确的模型驱动过程，因此更适合稳定写作与大规模历史资料改造。

## 与其他概念的关系
- 补足了 [[demo-LLM-Wiki]] 的关键留白。
- 与 [[demo-article-cards]] 结合后，形成更高效的写作与检索中间层。
- 相比 [[demo-RAG]] 更强调先有结构再找内容。

## 应用场景
AI 辅助写作、历史文章逆向建模、知识元抽取、结构化写作系统设计。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/articles/demo-001-Karpathy的LLM%20Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
