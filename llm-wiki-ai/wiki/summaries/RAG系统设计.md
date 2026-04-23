---
type: summary
status: active
summary_scope: topic
title: "RAG系统设计"
source_count: 2
sources:
  - ../raw/llm-engineering/05-工程-RAG.md
  - ../raw/llm-engineering/11-LLM工程.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# RAG系统设计

> 这个页面总结的是检索增强生成（RAG）系统的设计原则：分块策略、混合检索、Reranker 精排、上下文组装，以及系统失效时的诊断优先级。

## 知识核心
- **分块策略是 RAG 系统质量的第一决定因素**：切片方式直接决定后续检索的召回率上限；FAQ 类文档适合 512-800 tokens，技术文档适合 200-400 tokens（保持语义完整性），切片策略必须针对文档类型实验调优
- **纯向量检索有盲区，混合检索补完**：向量检索对精确型号、代码片段、专有名词不敏感（语义相似但字符不同），BM25 精确匹配补充关键词查找，RRF（Reciprocal Rank Fusion）融合两路排名，效果优于单路
- **Reranker 是 ROI 最高的优化点**：Cross-Encoder 联合编码问题和文档，相比 Bi-Encoder 向量召回，Faithfulness 评分从 3.1 提升到 4.3，成本只增加约 10%，效果常优于向量检索 20%+
- **上下文组装有位置效应**：最相关文档放首位（注意力最强），3-10 个 chunk 是最佳区间，超过 10 个无关噪声会降低 Faithfulness
- **文档解析质量决定 RAG 上限**：MinerU 等工具用于复杂 PDF 的高质量解析（表格、公式、多栏），错误解析在后续阶段无法弥补

## 共识内容
- 失效诊断优先级：文档解析质量 → Embedding 质量 → 检索策略 → Reranker → Prompt 组装 → LLM；调试时从上游开始排查，而不是先怀疑模型
- 权限过滤必须在检索层实现（metadata filter），不能依赖 Prompt 层约束（Prompt 层可被绕过或被遗忘）
- 语义缓存可大幅降低重复查询成本：相似度 > 0.92 时直接命中缓存，FAQ 场景命中率约 60%，成本可降 40%
- 向量数据库选型要考虑：过滤能力（metadata）、更新频率、部署方式（本地/云端）；[[Qdrant]] 是开源方案中工程成熟度较高的选择

## 关键分歧
- **Bi-Encoder vs Cross-Encoder**：Bi-Encoder（Embedding 检索）离线计算文档向量、查询时只编码问题，速度快但精度有限；Cross-Encoder（Reranker）实时联合编码问题+文档，精度高但延迟大，两者应分层使用而非二选一
- **固定分块 vs 语义分块**：固定 token 数分块简单但可能切断语义；语义分块（按段落/句子边界切）更自然但实现复杂；实际中常见混合方案（固定大小 + overlap）

## 适用边界
- 适用场景：知识库问答、企业文档检索、代码库搜索、产品手册查询
- 不适用场景：需要推理/计算的任务（RAG 检索的是知识，不能替代推理能力），实时数据（RAG 基于离线索引，不适合秒级更新的数据源）

## 关键概念
- [[RAG]]：检索增强生成的整体架构，解决 LLM 知识截止和幻觉问题
- [[混合检索]]：向量检索 + BM25 + RRF 融合，是 RAG 检索层的最佳实践
- [[Reranker]]：Cross-Encoder 精排，是 RAG 精度的最大杠杆
- [[Prompt工程]]：RAG 的上下文组装是 Prompt 工程的核心应用场景

## 值得注意的点
- RRF（Reciprocal Rank Fusion）的融合公式：score = Σ 1/(k + rank_i)，其中 k 通常取 60，这个简单公式在实践中表现出色，不需要训练任何参数
- chunk overlap 的必要性：切块时保留 10-20% 的重叠（如每块 512 tokens，重叠 64 tokens），防止关键信息恰好被切断在块边界
- Reranker 的延迟代价：Cross-Encoder 需要对每个候选文档单独前向传播，检索 Top-20 再 Rerank 到 Top-5 是常见的两阶段设计，总延迟约增加 100-300ms

## 延伸方向
- 可拆分为 [[RAG]] concept 页（端到端流程详解）
- 可拆分为 [[混合检索]] concept 页（RRF 算法、BM25 参数调优）
- 可拆分为 [[Reranker]] 实体页（模型选型、部署方式）
- 可连接到 [[LLM生产化与评估]] summary（Faithfulness 指标评估）

## 来源
- [05-工程-RAG.md](../raw/llm-engineering/05-工程-RAG.md)
- [11-LLM工程.md](../raw/llm-engineering/11-LLM工程.md)
