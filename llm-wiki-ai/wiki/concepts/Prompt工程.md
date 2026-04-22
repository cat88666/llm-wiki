---
type: concept
status: active
name: "Prompt工程"
aliases: ["Prompt Engineering", "提示工程", "few-shot", "CoT", "Chain-of-Thought", "temperature"]
related: ["RAG", "Function-Calling", "Self-Attention", "Tokenizer"]
sources:
  - ../raw/tech/原理/04-工程-Prompt.md
  - ../raw/tech/原理/11-LLM应用.md
  - ../raw/tech/原理/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Prompt工程

> 一句话定义：Prompt 工程是通过设计输入文本（指令、示例、上下文）来控制 LLM 输出行为的系统方法，是 LLM 应用开发中的核心工程技能。

## 核心要点
- **temperature 参数决定输出随机性**：0-0.3 用于需要确定性的任务（分类、工具调用、结构化提取），0.7-1.0 用于创作和多样性场景，0.3-0.7 是通用对话区间；不同任务不应使用默认值
- **示例优于规则**：3-5 个高质量 Few-shot 示例远比大量约束描述文字有效；LLM 更善于从示例推断格式和风格，规则描述过多反而会产生干扰
- **CoT（Chain-of-Thought）提升推理准确率**：要求模型"先思考再回答"，通过中间推理步骤引导内部计算；代价是 token 消耗增加 2-5x，需要权衡
- **注意力位置效应**：Prompt 首尾位置注意力最强，关键约束和最重要的上下文应放在首部或尾部，中间段容易被忽视（Lost in the Middle 现象）
- **多轮历史必须主动压缩**：超过约 20 轮后用轻量 LLM 压缩历史为摘要，保留最近 2-3 轮原文；不压缩会导致 context 溢出和早期信息注意力稀释

## 详细说明

Prompt 的结构设计有工程最佳实践：System Prompt（角色定义 + 行为约束 + 权限边界）分离于 User Prompt（具体任务），这两者的分离是工程上的重要约定，不同模型对 System Prompt 的权重处理有差异，Claude 和 GPT-4 对 System Prompt 有较强遵循能力。

Few-shot 示例的设计要点：示例格式要与目标任务完全一致（包括 JSON 字段名、缩进、顺序），示例应覆盖目标任务的典型情况（包括边界情况），示例数量 3-5 个是甜点区（过多增加 token 成本且边际效益递减，过少不足以建立模式）。

安全防护必须分层设计：
1. System Prompt 声明权限边界（"你只能回答关于XX的问题"）
2. 输入检测（过滤明显恶意内容，如提示注入攻击关键词）
3. 输出审查/脱敏（检测并屏蔽敏感信息，如身份证号）
4. 权限隔离（代码层验证用户权限，不依赖 Prompt 约束）

任何单层防护都可被绕过，"忽略之前所有指令"类的提示注入攻击对 System Prompt 的约束有穿透能力，必须配合输入检测。

## 与其他概念的关系
- 关联 [[RAG]]：RAG 的上下文组装本质是 Prompt 工程——文档如何放置在 Prompt 中（顺序、数量）直接影响 Faithfulness
- 关联 [[Function-Calling]]：工具调用需要在 System Prompt 中描述工具列表和调用格式，temperature 应设为 0-0.1 保证格式正确性
- 依赖 [[Tokenizer]]：了解 Tokenizer 才能估计 Prompt 的 token 成本，优化长 Prompt 的 token 效率

## 应用场景
- RAG 系统：上下文组装、答案生成指令、引用要求
- Text-to-SQL：动态 Schema 注入（用 Embedding 选 top-5 相关表，而非注入整个数据库）
- 多轮对话系统：历史压缩策略、角色一致性维护
- 内容审核和安全过滤：System Prompt 约束 + 输入/输出检测的分层架构

## 来源
- [04-工程-Prompt.md](../raw/tech/原理/04-工程-Prompt.md)
- [11-LLM应用.md](../raw/tech/原理/11-LLM应用.md)
- [12-项目-实战.md](../raw/tech/原理/12-项目-实战.md)
