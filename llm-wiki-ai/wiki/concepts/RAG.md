---
type: concept
status: active
name: "RAG"
aliases: ["检索增强生成", "Retrieval-Augmented Generation", "检索增强"]
related: ["混合检索", "Reranker", "Prompt工程", "Function-Calling"]
sources:
  - ../raw/tech/原理/05-工程-RAG.md
  - ../raw/tech/原理/11-LLM应用.md
  - ../raw/tech/原理/12-项目-实战.md
  - ../raw/tech/原理/07-工程-生产化.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# RAG

> 一句话定义：RAG（检索增强生成）在 LLM 生成答案前，先从外部知识库检索相关文档并注入上下文，解决 LLM 知识截止问题和幻觉问题，使答案有据可查。

## 核心要点
- **RAG 的五个关键阶段**：文档解析 → 分块（Chunking）→ 向量化（Embedding）→ 检索（向量 + 关键词混合）→ 精排（Reranker）→ 上下文组装 → 生成；每个阶段都可能成为瓶颈
- **分块是最关键的基础决策**：FAQ 类 512-800 tokens，技术文档 200-400 tokens，带 10-20% overlap（防止关键信息在块边界被切断）；切片策略需针对文档类型实验调优，没有通用最优方案
- **文档解析质量决定 RAG 上限**：PDF 表格、多栏、公式的解析错误在后续阶段无法弥补；复杂 PDF 用 MinerU 等专业工具，而非简单的文本抽取
- **Faithfulness 是 RAG 的核心评估指标**：答案必须基于检索到的上下文，而不是 LLM 的"记忆"；低 Faithfulness 直接意味着幻觉，是 RAG 系统的致命问题
- **失效诊断顺序**：文档解析 → Embedding 质量 → 检索策略 → Reranker → Prompt 组装 → LLM；调试时从最上游开始排查，不要先怀疑模型

## 详细说明

RAG 的标准技术栈：文档解析（MinerU/pdfplumber）→ 分块（LangChain TextSplitter）→ Embedding 模型（text-embedding-3-small/bge-m3）→ 向量数据库（Qdrant/Chroma/Weaviate）→ 检索（混合检索，向量 + BM25）→ Reranker（cross-encoder）→ LLM（GPT-4/Claude/Qwen）。

上下文组装的最佳实践：最相关文档放 Prompt 首位（利用注意力首端效应），3-10 个 chunk 是最佳区间（过多引入噪声），组装时保留文档来源信息（支持引用标注），设置合理的 chunk 数量上限防止超出 context 窗口。

权限过滤的正确位置：用户 A 不能访问用户 B 的文档，这个过滤必须在向量检索时通过 metadata filter 实现（如 `filter: user_id = current_user`），不能依赖 Prompt 层约束（攻击者可以通过提示注入绕过）。

RAGAS 四指标评估框架：
- Faithfulness（0-1）：答案中的陈述是否都能在检索内容中找到支撑
- Answer Relevancy（0-1）：答案是否真正回答了用户的问题
- Context Precision（0-1）：检索到的内容有多少比例是真正相关的
- Context Recall（0-1）：所有相关内容是否都被检索到

## 与其他概念的关系
- 依赖 [[混合检索]]：混合检索（向量 + BM25 + RRF）是 RAG 检索层的最佳实践，优于纯向量检索
- 依赖 [[Reranker]]：Reranker 是 RAG 精度的最大杠杆，Cross-Encoder 精排效果远优于 Bi-Encoder 召回
- 关联 [[Prompt工程]]：RAG 的上下文组装是 Prompt 工程的核心应用，文档位置和数量直接影响生成质量
- 区别于 [[Function-Calling]]：RAG 通过上下文注入增强知识，Function Calling 通过工具调用获取实时数据；两者可组合使用

## 应用场景
- 企业知识库问答：内部文档、产品手册、规章制度
- 代码仓库搜索和问答：技术文档、API 说明
- 客服系统：产品 FAQ、售后处理流程
- 医疗/法律专业问答：需要引用准确来源的高风险场景（Faithfulness 要求极高）

## 来源
- [05-工程-RAG.md](../raw/tech/原理/05-工程-RAG.md)
- [11-LLM应用.md](../raw/tech/原理/11-LLM应用.md)
- [12-项目-实战.md](../raw/tech/原理/12-项目-实战.md)
- [07-工程-生产化.md](../raw/tech/原理/07-工程-生产化.md)
