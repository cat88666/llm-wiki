---
type: concept
status: demo
demo_file: true
name: "RAG"
aliases: ["检索增强生成"]
related:
  - "[[demo-LLM-Wiki]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# RAG

> 在回答问题时，从外部资料中临时检索相关内容并交给模型生成答案的方法。

## 核心要点
- 每次查询都需要重新检索。
- 不天然保留历史综合结果。
- 适合即时问答，不擅长长期知识累积。

## 详细说明
文中把 NotebookLM、ChatGPT 文件上传等方式归为传统 RAG 思路。  
这类方案的主要特点是“每次重新推导”，即便文档相同、问题相似，也不会天然形成知识沉淀。  
因此，RAG 在本文中是 LLM-Wiki 的对照对象，用来凸显“编译式知识管理”的优势。

## 与其他概念的关系
- 区别于 [[demo-LLM-Wiki]]：RAG 偏临时检索，LLM-Wiki 偏持续编译。
- 被 [[demo-本体模型-M1-M4]] 所超越：作者认为仅靠检索不够，必须先有对象和规则模型。

## 应用场景
临时文件问答、快速信息定位、外部知识即时补充。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
