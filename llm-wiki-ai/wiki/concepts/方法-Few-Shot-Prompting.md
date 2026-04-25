---
title: "少样本提示（Few-Shot Prompting）"
type: concept
tags: [提示工程, 少样本, 示例, 基础技术]
sources:
  - raw/prompt/提示策略Gemini.md
  - raw/prompt/提示词Anthropic.md
  - raw/prompt/提示词白皮书Goolge.md
last_updated: 2026-04-23
---

# 少样本提示（Few-Shot Prompting）

> 一句话定义：Few-shot 的本质，是用少量高代表性的示例直接告诉模型“这个任务应该怎么做、结果长什么样”。

## 第一性原理
- 纯自然语言指令常常不够精确，因为模型对任务边界和输出格式的理解仍可能有歧义。
- 示例比抽象规则更接近任务分布本身。
- 所以 few-shot 不是额外装饰，而是在上下文中临时提供“任务样本”，让模型按样例归纳。

## 核心机制
- 在正式输入前给出少量输入-输出示例。
- 示例同时传递任务目标、格式规范、风格边界和异常处理方式。
- 模型根据示例模式，对新输入进行类比生成。

## 关键权衡
- 示例过少，模式不稳定；过多，又会增加成本并挤压真正任务上下文。
- 示例质量比数量更重要，错误示例会稳定放大错误。
- Few-shot 更适合格式和风格控制，不一定能弥补深层知识缺失。

## 与其他概念的关系
- [[方法-Prompt工程]]：Few-shot 是最常用的提示增强手段之一。
- [[方法-Chain-of-Thought]]：可以用带推理步骤的示例教模型如何分步思考。
- [[概念-Tokenizer]]：示例越长，上下文成本越高。

## 应用边界
- 对结构化抽取、分类、改写、风格迁移很有效。
- 当任务本身需要外部知识时，仅靠 few-shot 仍不如接入 [[概念-RAG]]。

## 来源
- [提示策略Gemini.md](../raw/prompt/提示策略Gemini.md)
- [提示词Anthropic.md](../raw/prompt/提示词Anthropic.md)
- [提示词白皮书Goolge.md](../raw/prompt/提示词白皮书Goolge.md)
