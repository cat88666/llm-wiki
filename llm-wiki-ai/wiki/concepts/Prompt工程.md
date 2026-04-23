---
type: concept
status: active
name: "Prompt工程"
aliases: ["Prompt Engineering", "提示工程", "few-shot", "CoT", "Chain-of-Thought", "temperature"]
related: ["RAG", "Function-Calling", "Self-Attention", "Tokenizer"]
sources:
  - ../raw/engineering/04-工程-Prompt.md
  - ../raw/engineering/11-LLM工程.md
  - ../raw/engineering/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# Prompt工程

> 一句话定义：Prompt 工程的本质，是把任务目标、约束和必要上下文组织成模型最容易正确执行的输入条件。

## 第一性原理
- LLM 并不是“知道你真正想要什么”，它只会在当前上下文条件下预测最可能的下一个输出。
- 因此提示词工作的核心，不是“写漂亮句子”，而是减少歧义、补齐条件、限制输出空间。
- Prompt 的作用本质上是在推理时临时塑造任务分布，让模型更像在做“你要的那个任务”。

## 核心机制
- 指令：明确任务目标和完成标准。
- 上下文：提供模型完成任务所缺少的信息。
- 约束：限制格式、边界、角色、权限和失败策略。
- 示例：通过少量样本直接展示模式，减少模型误解。
- 参数控制与上下文编排共同决定结果稳定性，而不是单靠某一句 magic prompt。

## 关键权衡
- 约束过少，输出容易发散；约束过多，又会压缩模型有效发挥空间。
- 示例越多不一定越好，关键在代表性和信噪比。
- 长 prompt 提供更多信息，但也提高成本，并引入“中间信息被稀释”的风险。
- 现代 Prompt 工程越来越接近 [[Context_Engineering]]，重点从措辞转向上下文系统设计。

## 与其他概念的关系
- [[Tokenizer]]：决定 prompt 的成本和上下文占用。
- [[RAG]]：RAG 的最后一步，本质就是如何把检索结果组织进 prompt。
- [[Function-Calling]]：工具描述、参数格式和失败处理都属于 prompt 约束设计的一部分。

## 应用边界
- Prompt 工程可以显著改善任务执行质量，但不能替代模型能力本身。
- 当问题根源在知识缺失、工具缺失或模型推理上限时，单纯改 prompt 收益有限。

## 来源
- [04-工程-Prompt.md](../raw/engineering/04-工程-Prompt.md)
- [11-LLM工程.md](../raw/engineering/11-LLM工程.md)
- [12-项目-实战.md](../raw/engineering/12-项目-实战.md)
