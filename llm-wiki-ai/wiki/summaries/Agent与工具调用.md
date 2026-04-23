---
type: summary
status: active
summary_scope: topic
title: "Agent与工具调用"
source_count: 2
sources:
  - ../raw/llm-engineering/06-工程-Agent.md
  - ../raw/llm-engineering/11-LLM工程.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# Agent与工具调用

> 这个页面总结的是 LLM Agent 系统的工程设计：工具描述规范、失败处理策略、单/多 Agent 选择边界、危险操作的安全机制，以及状态可追踪性的重要性。

## 知识核心
- **工具描述是 Agent 效果的第一杠杆**：一个好的工具描述需要包含"做什么""何时调用""参数格式""返回内容"四要素，模糊的描述是 Agent 错误调用的首要原因
- **失败处理比规划逻辑更重要**：90% 的 Agent 线上问题源于工具调用失败后的处理不当；工具必须返回结构化错误信息（而不是 exception），Agent 需要能识别失败并采取备选策略
- **单 Agent 优先，多 Agent 代价高**：工具数 ≤ 10、任务步骤 ≤ 5 时，单 Agent 足够；多 Agent 引入协调开销（消息传递、状态同步、调试复杂度），应先证明单 Agent 确实不够再拆分
- **危险操作必须有代码层独立校验**：发邮件、扣款、删除数据等高风险操作，仅靠 Prompt 约束不够；必须有代码层的参数校验 + 强制人工确认节点（第二个"确认"工具），不能绕过
- **状态不可追踪 = 不可调试 = 不可上线**：每个 Agent 的工具调用、中间状态、决策路径必须记录，没有可观测性的 Agent 系统无法在生产环境中维护

## 共识内容
- Function Calling（工具调用）的 temperature 应设为 0-0.1：工具调用需要确定性，高 temperature 会导致参数格式错误或随机选错工具
- 工具的幂等性设计：同一工具被多次调用结果应一致（或有去重机制），避免网络抖动导致的重复执行副作用
- 长运行 Agent 需要 checkpoint 机制：超过 10 步的任务应定期保存中间状态，允许从断点恢复而不是重新开始
- ReAct 框架（Reasoning + Acting）是目前主流的 Agent 推理范式：交替进行思考步骤（Thought）和行动步骤（Action），使推理链可观察

## 关键分歧
- **多 Agent 拓扑：中心 vs 对等**：Orchestrator + Sub-Agent 中心化架构更易控制和调试；对等多 Agent 协作更灵活但状态同步复杂度高；工业实践中中心化方案更主流
- **工具粒度：细粒度 vs 粗粒度**：细粒度工具（如 search_by_id、search_by_name 分开）给 LLM 更多控制粒度；粗粒度工具（如 search 统一入口）减少工具数量但需要更复杂的内部路由；权衡点是工具数和描述复杂度

## 适用边界
- 适用场景：需要多步推理的任务（查询+计算+写入）、需要使用外部工具/API 的任务、流程自动化（但不涉及高风险操作或需人工确认节点）
- 不适用场景：单次查询即可完成的任务（用 RAG 更合适），强实时性要求（Agent 多步推理延迟高），完全无人监督的高风险自动化（邮件/财务操作必须有人工节点）

## 关键概念
- [[Function-Calling]]：LLM 结构化工具调用的基础机制，工具描述的设计直接决定调用准确率
- [[Prompt工程]]：Agent 的系统提示设计是 Prompt 工程的高级应用，需要描述角色、工具列表、推理格式
- [[RAG]]：Agent 系统中 RAG 通常作为一个工具（知识检索工具）被调用，而不是独立系统

## 值得注意的点
- "工具返回的错误信息"设计反常识：工具不应该 throw exception，而应该返回包含 error_code 和 error_message 的正常 JSON；LLM 无法处理 exception，但可以理解结构化错误并调整策略
- 人工确认节点的实现方式：在工具列表中加入一个 request_human_confirmation(action, details) 工具，Agent 在危险操作前必须调用此工具并等待确认信号，这比在 Prompt 中说"危险操作要确认"更可靠
- 多 Agent 的调试困难：当 Sub-Agent 的输出作为 Orchestrator 的输入时，错误传播链很长，需要每个 Agent 的 trace 可以独立查看，[[LangFuse]] 等观测工具是必须的

## 延伸方向
- 可拆分为 [[Function-Calling]] concept 页（工具描述规范、结构化输出）
- 可连接到 [[LLM生产化与评估]] summary（Agent 的 trace_id 追踪、LangFuse 集成）
- 可延伸到多 Agent 系统的架构模式（Orchestrator/Worker 模式详解）

## 来源
- [06-工程-Agent.md](../raw/llm-engineering/06-工程-Agent.md)
- [11-LLM工程.md](../raw/llm-engineering/11-LLM工程.md)
