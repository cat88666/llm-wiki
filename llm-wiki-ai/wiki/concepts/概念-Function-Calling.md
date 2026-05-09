---
type: concept
status: active
name: "Function-Calling"
aliases: ["工具调用", "Tool Use", "Tool Calling", "函数调用"]
related: ["RAG", "Prompt工程", "Agent与工具调用"]
sources:
  - raw/engineering/06-工程-Agent.md
  - raw/engineering/11-LLM工程.md
  - raw/engineering/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-23
lint_notes: ""
---

# Function-Calling

> 一句话定义：Function Calling 的本质，是把“语言上的意图理解”与“外部系统里的确定性动作执行”明确分层。

## 第一性原理
- 语言模型擅长理解与生成，但不擅长直接、安全、可验证地操作外部系统。
- 真正的系统动作需要结构化参数、确定性执行和可追踪结果。
- 所以工具调用不是让模型“自己去做事”，而是让模型负责决策，由程序负责执行。

## 核心机制
- 先向模型声明可用工具、适用场景和参数模式。
- 模型根据用户意图选择是否调用工具，并生成结构化参数。
- 应用层执行工具，将结果或错误回传给模型。
- 模型再基于工具结果生成最终响应或继续决策。

## 关键权衡
- 工具越强，系统能力越强，但错误调用的风险也越大。
- 工具描述越模糊，模型越容易选错工具或填错参数。
- 高风险动作必须把“理解”和“执行”之间再加确认层，否则语言错误会直接变成业务事故。

## 与其他概念的关系
- [[方法-Prompt工程]]：工具描述本身就是结构化 prompt 的一部分。
- [[概念-Agent]]：Agent 往往在多步循环中多次使用 Function Calling。
- [[概念-RAG]]：检索也可以被封装成工具，但 RAG 更强调知识召回链路本身。

## 应用边界
- Function Calling 擅长把模型接入外部能力，不等于自动获得可靠自治。
- 复杂工作流仍需要状态管理、权限控制、重试和幂等等系统设计。

## 来源
- [06-工程-Agent.md](../../raw/engineering/06-工程-Agent.md)
- [11-LLM工程.md](../../raw/engineering/11-LLM工程.md)
- [12-项目-实战.md](../../raw/engineering/12-项目-实战.md)
