---
type: concept
status: active
name: "Function-Calling"
aliases: ["工具调用", "Tool Use", "Tool Calling", "函数调用"]
related: ["RAG", "Prompt工程", "Agent与工具调用"]
sources:
  - ../raw/llm-engineering/06-工程-Agent.md
  - ../raw/llm-engineering/11-LLM工程.md
  - ../raw/llm-engineering/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Function-Calling

> 一句话定义：Function Calling（工具调用）是 LLM 根据用户意图选择并调用预定义外部函数/工具的能力，通过结构化 JSON 格式传递参数，使 LLM 能与外部系统交互。

## 核心要点
- **工具描述是调用准确率的第一决定因素**：一个完整的工具描述应包含"做什么（功能说明）""何时调用（触发条件）""参数格式（类型、约束、示例）""返回内容（结构和含义）"，模糊的描述是调用错误的首要原因
- **temperature 应设为 0-0.1**：工具调用需要确定性的 JSON 输出，高 temperature 会引入随机性导致格式错误或参数值异常
- **工具返回值必须是结构化信息，包括错误**：工具不应该抛出 exception（LLM 无法处理），应返回包含 success、error_code、error_message 的 JSON；90% 的 Agent 问题源于工具失败处理不当
- **幂等性设计**：网络抖动可能导致工具被重复调用，工具应支持幂等（同样参数多次调用结果一致，或有去重机制），防止重复扣款/发邮件等副作用
- **高风险操作必须有确认工具**：发邮件、支付、删除数据等操作，在工具列表中加入 `request_human_confirmation(action, details)` 工具，Agent 必须先调用确认工具，等待确认信号后再执行

## 详细说明

Function Calling 的技术实现：用户发送消息 → LLM 决定调用哪个工具（输出 JSON：tool_name + arguments）→ 应用层执行工具 → 将结果返回给 LLM → LLM 根据工具结果生成最终回答。这个过程在 API 层面通过 `tools` 参数传递工具描述，LLM 输出 `tool_calls` 字段包含调用信息。

工具描述的标准格式（OpenAI 风格）：
```json
{
  "name": "search_product",
  "description": "搜索产品目录。当用户询问特定产品信息时调用。不适用于订单查询。",
  "parameters": {
    "type": "object",
    "properties": {
      "query": {"type": "string", "description": "搜索关键词，支持产品名称或型号"},
      "category": {"type": "string", "enum": ["electronics", "clothing"], "description": "产品类别，不确定时不传"}
    },
    "required": ["query"]
  }
}
```

多工具选择的设计原则：工具数量超过 10 个时，LLM 的工具选择准确率开始下降（注意力分散）；应将相关工具合并（如 search_by_id 和 search_by_name 合并为 search，内部路由），保持工具列表简洁；优先通过工具描述的"何时调用"明确区分工具的适用场景，减少 LLM 的选择困难。

## 与其他概念的关系
- 关联 [[RAG]]：在 Agent 系统中，RAG 通常以工具形式存在（knowledge_search 工具），而不是独立系统；Function Calling 是 Agent 调用 RAG 的接口
- 关联 [[Prompt工程]]：工具描述是 System Prompt 的核心组成部分，描述质量直接影响工具调用准确率；工具调用结果如何组织进 Prompt 也是 Prompt 工程的一部分
- 区别于 [[RAG]]：RAG 直接将知识注入上下文，Function Calling 通过结构化接口调用外部能力；两者适用场景不同，RAG 适合静态知识库，Function Calling 适合实时数据和操作型任务

## 应用场景
- 查询类工具：数据库查询、API 数据获取、搜索引擎调用
- 操作类工具：发送消息、创建记录、触发工作流（需要幂等设计和确认机制）
- 计算类工具：代码执行、数学计算、数据处理（LLM 不擅长的精确计算交给工具）
- 多步任务 Agent：工具的组合调用实现复杂任务（查询 → 计算 → 汇总 → 发送报告）

## 来源
- [06-工程-Agent.md](../raw/llm-engineering/06-工程-Agent.md)
- [11-LLM工程.md](../raw/llm-engineering/11-LLM工程.md)
- [12-项目-实战.md](../raw/llm-engineering/12-项目-实战.md)
