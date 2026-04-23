---
title: "提示工程（Prompt Engineering）"
type: concept
tags: [提示工程, LLM, AI, 核心概念]
sources:
  - raw/prompt/提示工程指南.md
  - raw/prompt/提示策略Gemini.md
  - raw/prompt/提示词Anthropic.md
  - raw/prompt/提示词白皮书Goolge.md
last_updated: 2026-04-23
---

# 提示工程（Prompt Engineering）

> 一句话定义：提示工程的本质，是通过设计输入条件来改变模型输出分布，让模型更稳定地落到目标任务上。

## 第一性原理
- 语言模型输出什么，取决于它当前接收到的上下文条件。
- 因此提示并不是“命令模型”，而是在统计意义上收缩它的可选输出空间。
- 好的提示工程，本质是减少歧义、显化约束、补齐缺失信息。

## 核心机制
- 角色与任务定义：告诉模型当前在扮演什么功能。
- 上下文补充：提供完成任务所需的背景。
- 示例与格式：展示期望输出模式。
- 约束与边界：限制回答范围、风格、结构和失败策略。

## 关键权衡
- 越详细不一定越好，关键在信息密度与相关性。
- 提示能引导模型，但不能替代模型本身缺失的知识和能力。
- 当任务进入多轮状态、工具或外部知识编排时，提示工程会自然延伸到 [[Context_Engineering]]。

## 与其他概念的关系
- [[Few_Shot_Prompting]]：通过示例增强任务模式学习。
- [[Chain_of_Thought]]：通过中间步骤提升复杂推理稳定性。
- [[Prompt工程]]：是同一主题的中文主表述，可作为互补视角。

## 应用边界
- 适合概括提示工程的通用原理。
- 若关注工程落地与系统视角，优先看 [[Prompt工程]] 与 [[Context_Engineering]]。

## 来源
- [提示工程指南.md](../raw/prompt/提示工程指南.md)
- [提示策略Gemini.md](../raw/prompt/提示策略Gemini.md)
- [提示词Anthropic.md](../raw/prompt/提示词Anthropic.md)
- [提示词白皮书Goolge.md](../raw/prompt/提示词白皮书Goolge.md)
