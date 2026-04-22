---
type: entity
status: active
entity_type: tool
name: "Qdrant"
aliases: ["qdrant", "向量数据库"]
domain: "向量检索 / RAG基础设施"
related: ["RAG", "混合检索", "Prompt工程"]
sources:
  - ../raw/tech/原理/05-工程-RAG.md
  - ../raw/tech/原理/11-LLM应用.md
  - ../raw/tech/原理/12-项目-实战.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Qdrant

> Qdrant 是一个高性能开源向量数据库，支持向量相似性搜索、payload 过滤和原生混合检索（dense + sparse 向量），是 RAG 系统中向量存储和检索的主流开源选择。

## 基本信息
- **类型**：开源向量数据库（Rust 实现，Apache 2.0 许可，支持自托管和云服务）
- **领域**：语义搜索、RAG 向量存储、推荐系统特征检索
- **关联时间**：2021 年发布，在 RAG 应用爆发（2023年）后迅速成为主流选择

## 核心特点
- **高效向量索引**：基于 HNSW（Hierarchical Navigable Small World）图索引，支持十亿级向量的高速近似最近邻搜索；支持 float32/float16/int8 量化降低内存占用
- **Payload 过滤（Pre-filtering）**：在向量搜索时附加 metadata 条件过滤（如 `filter: user_id = X AND doc_type = "manual"`），过滤在 HNSW 索引内部实现，不是搜索后过滤，性能高且权限安全
- **原生混合检索支持**：同时存储 dense vector（Embedding）和 sparse vector（BM25/SPLADE），单次查询可同时检索两路并在 Qdrant 内部 RRF 融合，简化应用层实现
- **Collections 和 Named Vectors**：支持在同一个 Collection 中存储多种不同维度/模型的向量（如 Embedding 模型 A 和 B），方便 A/B 测试
- **滚动更新（Soft Delete + Rebuild）**：向量数据库的增量更新需要特别处理，Qdrant 支持 soft delete + 定期重建索引的模式，适合文档库定期更新的场景

## 在知识库中的出现
- [[RAG]] concept：Qdrant 是 RAG 系统向量存储层的典型工具，提供向量存储、检索和过滤能力
- [[混合检索]] concept：Qdrant 的 sparse vector 支持使其成为混合检索的一体化解决方案，无需单独维护 BM25 引擎
- [[RAG系统设计]] summary：作为开源方案中工程成熟度较高的向量数据库，是 RAG 基础设施选型的首选

## 相关实体
- Chroma：轻量级开源向量数据库，适合原型开发和本地测试，生产规模不如 Qdrant
- Weaviate：功能丰富的开源向量数据库，内置模块化（内置 BM25/向量模型），但配置复杂
- Pinecone：云原生向量数据库，无需自托管，但成本高，不支持私有化部署
- Elasticsearch：成熟的搜索引擎，7.x 版本后支持 KNN 向量搜索，适合已有 ES 基础设施的团队，混合检索支持好但向量性能不如专用向量数据库

## 来源
- [05-工程-RAG.md](../raw/tech/原理/05-工程-RAG.md)
- [11-LLM应用.md](../raw/tech/原理/11-LLM应用.md)
- [12-项目-实战.md](../raw/tech/原理/12-项目-实战.md)
