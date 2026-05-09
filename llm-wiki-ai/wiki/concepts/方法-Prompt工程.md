---
type: concept
subtype: 方法
status: active
name: "Prompt工程"
aliases: ["Prompt Engineering", "提示工程", "提示词工程"]
related: ["概念-RAG", "概念-Function-Calling", "机制-Self-Attention", "概念-Tokenizer", "框架-Context-Engineering"]
sources:
  - raw/engineering/04-工程-Prompt.md
  - raw/engineering/11-LLM工程.md
  - raw/engineering/12-项目-实战.md
  - raw/prompt/提示工程指南.md
  - raw/prompt/提示策略Gemini.md
  - raw/prompt/提示词Anthropic.md
  - raw/prompt/提示词白皮书Goolge.md
created: 2026-04-22
updated: 2026-04-25
---

# Prompt工程

> 一句话定义：Prompt 工程的本质，是把任务目标、约束和必要上下文组织成模型最容易正确执行的输入条件，通过收缩输出空间让模型稳定落到目标任务上。

## 第一性原理
- LLM 并不是"知道你真正想要什么"，它只在当前上下文条件下预测最可能的下一个输出。
- Prompt 的作用本质上是在推理时临时塑造任务分布——减少歧义、补齐条件、限制输出空间。
- 因此提示词工作的核心不是"写漂亮句子"，而是让模型在统计意义上更难产生你不想要的输出。

## 核心机制
- **角色与任务定义**：告诉模型当前在扮演什么功能，明确任务目标和完成标准。
- **上下文补充**：提供模型完成任务所缺少的背景信息。
- **示例与格式**：通过少量样本直接展示期望输出模式，减少模型误解。
- **约束与边界**：限制格式、风格、角色、权限和失败策略。
- 参数控制（temperature、top_p）与上下文编排共同决定结果稳定性，单靠措辞无法保证。

## 关键权衡
- 约束过少输出容易发散；约束过多会压缩模型有效发挥空间。
- 示例越多不一定越好，关键在代表性和信噪比。
- 越详细不一定越好，信息密度与相关性比绝对长度更重要。
- 长 prompt 提供更多信息，但也提高成本，并引入"中间信息被稀释"的风险。
- 现代 Prompt 工程越来越接近 [[框架-Context-Engineering]]，重点从措辞转向上下文系统设计。

## 与其他概念的关系
- [[概念-Tokenizer]]：决定 prompt 的成本和上下文占用，长 prompt 必须关注 token 消耗。
- [[概念-RAG]]：RAG 的最后一步本质就是如何把检索结果组织进 prompt。
- [[概念-Function-Calling]]：工具描述、参数格式和失败处理都属于 prompt 约束设计的一部分。
- [[方法-Few-Shot-Prompting]]：通过示例增强任务模式学习，是 Prompt 工程的核心技法之一。
- [[方法-Chain-of-Thought]]：通过中间步骤提升复杂推理稳定性，适用于逻辑密集任务。

## 应用边界
- Prompt 工程可以显著改善任务执行质量，但不能替代模型能力本身。
- 当问题根源在知识缺失、工具缺失或模型推理上限时，单纯改 prompt 收益有限。
- 适合概括提示工程的通用原理；关注工程落地与系统视角时，优先看 [[框架-Context-Engineering]]。

## 来源
- [04-工程-Prompt.md](../../raw/engineering/04-工程-Prompt.md)
- [11-LLM工程.md](../../raw/engineering/11-LLM工程.md)
- [12-项目-实战.md](../../raw/engineering/12-项目-实战.md)
- [提示工程指南.md](../../raw/prompt/提示工程指南.md)
- [提示策略Gemini.md](../../raw/prompt/提示策略Gemini.md)
- [提示词Anthropic.md](../../raw/prompt/提示词Anthropic.md)
- [提示词白皮书Goolge.md](../../raw/prompt/提示词白皮书Goolge.md)
