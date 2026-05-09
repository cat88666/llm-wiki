---
type: entity
status: active
entity_type: technique
name: "Reranker"
aliases: ["精排", "Cross-Encoder", "重排序", "Reranking"]
domain: "RAG / 信息检索"
related: ["RAG", "混合检索", "Prompt工程"]
sources:
  - raw/engineering/05-工程-RAG.md
  - raw/engineering/11-LLM工程.md
  - raw/engineering/07-工程-生产化.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Reranker

> Reranker 是 RAG 检索管道的精排层，使用 Cross-Encoder 模型联合编码问题和候选文档，对召回候选重新打分排序，相比 Bi-Encoder 向量检索的准确率大幅提升，是 RAG 系统 ROI 最高的优化手段。

## 基本信息
- **类型**：精排技术（模型 + 工程模式）
- **领域**：RAG 检索管道、信息检索、搜索引擎
- **关联时间**：Cross-Encoder 技术由来已久，在 RAG 爆发（2023年）后作为精排层被广泛采用

## 核心特点
- **Cross-Encoder vs Bi-Encoder**：Bi-Encoder（向量检索）离线计算文档向量，查询时只编码问题向量做相似度计算，速度快（O(1) 查询）但无法捕获问题-文档间细粒度交互；Cross-Encoder 实时将问题+文档 concat 输入模型，输出相关性分数，能捕获精确的语义匹配但速度慢（O(n) 查询）
- **效果提升显著**：Faithfulness 评分从 3.1（无 Reranker）提升到 4.3（有 Reranker），效果常优于纯向量检索 20%+；这来自 Cross-Encoder 对问题和文档的深度交互建模，而不是独立编码
- **成本增加有限**：Reranker 只对召回的 Top-20/50 候选运行，不对全量文档推理；额外计算成本约 10-15%，ROI 极高
- **两阶段架构**：召回（Recall）→ 精排（Rerank）是标准设计；召回层（向量+BM25 混合）追求高召回率，精排层追求高精度，两者分工明确
- **常用模型**：bge-reranker-v2-m3（BAAI，中英文，开源）、mxbai-rerank（混合架构，开源）、Cohere Rerank（API，闭源，效果强但有成本）

## 在知识库中的出现
- [[概念-RAG]] concept：Reranker 是 RAG 五阶段流程的核心精排步骤，紧接在检索之后
- [[方法-混合检索]] concept：混合检索（召回）→ Reranker（精排）是 RAG 检索管道的完整最佳实践
- [[主题-RAG系统设计]] summary：Reranker 被明确为 RAG 系统 ROI 最高的单点优化，Faithfulness 从 3.1→4.3 的数据来源

## 相关实体
- BGE-Reranker（BAAI）：中英文双语 Reranker，开源免费，是中文 RAG 场景的首选
- Cohere Rerank：商业 Reranker API，效果强，按调用量计费，适合不想自托管的场景
- [[工具-Qdrant]]：向量数据库，Reranker 在 Qdrant 召回结果之上运行，两者是 RAG 检索链路的组合
- LambdaMART：传统机器学习 Reranker（Gradient Boosting for ranking），在神经 Reranker 出现前的主流方案，现在主要作为轻量级备选

## 来源
- [05-工程-RAG.md](../../raw/engineering/05-工程-RAG.md)
- [11-LLM工程.md](../../raw/engineering/11-LLM工程.md)
- [07-工程-生产化.md](../../raw/engineering/07-工程-生产化.md)
