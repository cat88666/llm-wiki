---
type: summary
status: active
summary_scope: topic
title: "LLM生产化与评估"
source_count: 3
sources:
  - ../raw/LLM工程/07-工程-生产化.md
  - ../raw/LLM工程/11-LLM工程.md
  - ../raw/LLM工程/12-项目-实战.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# LLM生产化与评估

> 这个页面总结的是将 LLM 系统从实验推向生产所需的工程实践：评估指标体系、可观测性建设、成本控制策略、安全合规要求，以及上线前的检查清单。

## 知识核心
- **Faithfulness 是 RAG 系统最重要的评估指标**：低 Faithfulness = 幻觉 = 系统不可信，是 RAG 的致命伤；其他指标（相关性、答案准确率）都建立在 Faithfulness 合格的基础上
- **可观测性的最小必要集**：每个请求必须记录 trace_id → model_version → prompt_version → tokens → latency → chunks → user_feedback，缺少任何一项都会导致问题难以复现和定位
- **模型分级路由是最直接的成本优化手段**：简单任务（意图分类、信息提取）用 haiku/mini，中等任务（问答、摘要）用 sonnet，复杂任务（多步推理、代码生成）用 opus；合理路由可降低成本 50%+
- **语义缓存是 FAQ 场景的高 ROI 优化**：相似度 > 0.92 时命中缓存直接返回，FAQ 场景命中率约 60%，成本可降 40%；语义缓存需要向量数据库支持，与 Embedding 层共用基础设施
- **上线前检查清单是工程纪律而非可选项**：超时机制、参数校验、降级策略、LangFuse 接入、安全过滤、权限隔离、100+ 评估集验证、告警配置——缺任何一项都是生产风险

## 共识内容
- 评估集必须在上线前建立：至少 100 条有人工标注的测试用例，覆盖正常路径、边界条件、恶意输入，模型/Prompt 变更前必须跑完评估集
- 降级策略是可用性保障：当 LLM 超时或报错时，返回预设的保守回复（而不是抛错给用户），保持用户体验
- 权限隔离必须在代码层实现，不能依赖 Prompt：用户 A 的数据不能被用户 B 通过 Prompt 注入访问，权限过滤必须在检索层（metadata filter）而不是在 Prompt 层约束
- 动态 Schema 注入优于全量注入：Text-to-SQL 场景中，用 Embedding 找最相关的 top-5 数据表注入 Schema，而不是全量数据库 Schema，大幅削减 token 消耗

## 关键分歧
- **评估自动化 vs 人工评估**：自动化评估（LLM-as-judge、RAGAS 框架）可以大规模运行，但存在评估模型的偏见；人工评估准确但成本高；实践中应自动化为主、人工抽样校准
- **主动监控 vs 被动告警**：主动监控（定期跑评估集检测退化）能提前发现问题，但成本高；被动告警（线上指标超阈值触发）响应快，但已经影响用户；最佳实践是两者结合

## 适用边界
- 适用场景：RAG 系统上线、LLM 应用的 API 服务、多模型路由的成本优化、有合规要求的企业部署
- 不适用场景：纯实验性/研究性 Notebook 不需要全套生产化工程；极低流量（< 1000 QPS）的内部工具可以简化可观测性方案

## 关键概念
- [[RAG]]：Faithfulness/相关性/准确率是 RAG 专项评估指标，RAGAS 框架提供自动化评估
- [[Prompt工程]]：Prompt 版本管理是可观测性的重要组成部分，每次 Prompt 变更需要记录版本并在日志中体现
- [[Function-Calling]]：Agent 工具调用的成功率、失败类型是生产化监控的重要指标

## 值得注意的点
- RAGAS 评估框架的四个核心指标：Faithfulness（答案是否基于检索内容）、Answer Relevancy（答案是否回答了问题）、Context Precision（检索内容是否相关）、Context Recall（相关内容是否被检索到），四者联合评估才能定位问题
- 模型版本锁定的重要性：LLM API 提供商可能悄悄更新模型（同一个 model name 背后不同版本），锁定具体版本（如 gpt-4o-2024-08-06）并记录在日志中，是评估结果可复现的前提
- 成本优化的反直觉点：更长的 Prompt（加更多上下文）有时比短 Prompt 更便宜——因为减少了 LLM 重新推理的次数和错误率，降低了重试成本

## 延伸方向
- 可拆分为 [[LangFuse]] 实体页（可观测性配置、Dashboard 使用）
- 可延伸到 [[vLLM]] 实体页（推理服务部署、PagedAttention 生产配置）
- 可连接到 [[LoRA]] 实体页（微调 vs RAG 的选择边界）
- 可连接综合分析 [[RAG-vs-微调-vs-Agent：LLM能力增强三种路径对比]]

## 来源
- [07-工程-生产化.md](../raw/LLM工程/07-工程-生产化.md)
- [11-LLM工程.md](../raw/LLM工程/11-LLM工程.md)
- [12-项目-实战.md](../raw/LLM工程/12-项目-实战.md)
