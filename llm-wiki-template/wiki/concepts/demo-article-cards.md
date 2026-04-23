---
type: concept
status: demo
demo_file: true
name: "article-cards"
aliases: ["文章卡片层"]
related:
  - "[[demo-本体模型-M1-M4]]"
  - "[[demo-LLM-Wiki]]"
sources:
  - ../../raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md
created: 2026-04-22
updated: 2026-04-22
---

# article-cards

> 位于 raw 与 wiki 之间的结构化中间层，用标准化卡片压缩每篇文章的核心信息。

## 核心要点
- 每篇文章先被压缩成标准化 YAML 卡片。
- 后续问答与写作可以先检索卡片，再按需回原文取证。
- 能显著降低大规模历史文章场景下的重复读取成本。

## 详细说明
作者指出 Karpathy 的原始两层结构在面对 1000 篇历史文章时会有成本问题。  
引入 article-cards 后，模型无需每次重读原文，而是优先读取卡片层，再决定是否回看原文。  
这是作者方案相比 Karpathy 方案一个非常关键的工程化增强。

## 与其他概念的关系
- 是 [[demo-本体模型-M1-M4]] 落地后的中间层表现。
- 相比 [[demo-LLM-Wiki]] 的 raw → wiki，两层方案更适合高规模历史文章处理。

## 应用场景
大规模知识资产冷启动、批量文章摄入、模型驱动写作前置检索。

## 来源
- [Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较](../../raw/articles/demo-001-Karpathy的LLM%20Wiki个人知识管理方案和本体模型驱动AI写作比较.md)
