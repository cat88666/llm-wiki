---
type: entity
status: active
entity_type: tool
name: "LangFuse"
aliases: ["Langfuse", "LLM可观测性", "LLM Observability"]
domain: "LLM生产化 / 可观测性 / 评估"
related: ["Prompt工程", "RAG", "Function-Calling"]
sources:
  - raw/engineering/07-工程-生产化.md
  - raw/engineering/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# LangFuse

> LangFuse 是面向 LLM 应用的开源可观测性和评估平台，提供 Trace/Span 追踪、Prompt 版本管理、成本统计和评估数据集管理，是将 LLM 应用推向生产的必备工具。

## 基本信息
- **类型**：开源可观测性平台（支持自托管，也有云服务版本，MIT 许可）
- **领域**：LLM 应用监控、Prompt 工程管理、评估数据集管理
- **关联时间**：2023 年推出，迅速成为 LLM 可观测性领域最流行的开源工具

## 核心特点
- **Trace 追踪**：以 trace_id 为核心，记录每个请求的完整调用链（Span）：输入 Prompt → LLM 调用 → 工具调用 → RAG 检索 → 最终输出，支持嵌套 Span（父子关系）
- **必追踪指标集**：trace_id、model_version、prompt_version、input_tokens、output_tokens、latency_ms、retrieved_chunks、user_feedback；这些字段是问题定位和质量分析的基础
- **Prompt 版本管理**：在 LangFuse 中管理 Prompt 模板版本，应用代码通过 API 获取指定版本的 Prompt，实现 Prompt 变更的可追踪和回滚
- **评估数据集管理**：在平台上创建和维护评估数据集，运行自动评估（LLM-as-judge）并记录分数，每次 Prompt/模型变更前跑评估集验证无退化
- **成本分析 Dashboard**：按 model、user、session 维度统计 token 消耗和成本，支持识别成本异常（如某个 Prompt 版本 token 消耗暴涨）

## 在知识库中的出现
- [[主题-LLM生产化与评估]] summary：LangFuse 是生产上线检查清单的必选项，实现可观测性的核心工具
- [[主题-Agent与工具调用]] summary：多 Agent 系统的 trace 复杂度高，LangFuse 的嵌套 Span 支持是 Agent 调试的关键

## 相关实体
- LangSmith：LangChain 官方的可观测性工具，功能类似 LangFuse，与 LangChain 框架深度集成，但仅有云服务版本
- Helicone：另一个 LLM 可观测性工具，以代理模式接入（无需修改代码），但定制化能力弱于 LangFuse
- Arize Phoenix：开源 LLM 评估工具，与 LangFuse 互补，侧重评估管道而非实时追踪
- [[工具-vLLM]]：推理框架，与 LangFuse 组合使用，vLLM 提供推理服务，LangFuse 提供可观测性

## 来源
- [07-工程-生产化.md](../../raw/engineering/07-工程-生产化.md)
- [12-项目-实战.md](../../raw/engineering/12-项目-实战.md)
