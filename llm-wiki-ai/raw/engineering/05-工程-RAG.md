# RAG 工程实操

> **本文定位**：工程实操层。RAG 全链路的设计、实现和优化，含代码示例。
> 原理见 [03-理论-LLM核心机制.md](03-理论-LLM核心机制.md)，面试题见 [11-LLM工程.md](11-LLM工程.md)

---

## 一、为什么需要 RAG

LLM 三个硬伤：
1. **知识截止**：无法感知训练后发生的事
2. **幻觉**：对不确定的事实倾向于编造而非拒答
3. **知识更新成本高**：重训练代价极大

RAG 的本质：**把检索到的正确上下文注入 Prompt，让模型基于证据生成答案**。RAG 不解决模型能力问题，它解决的是知识来源问题。

**一句话决策树**：
- 知识更新频繁 → RAG
- 行为/风格/格式对齐 → 微调
- 生产环境：两者结合

---

## 二、完整 RAG 链路

```
【离线索引阶段】
文档接入（PDF/Word/HTML）
  ↓ 解析（提取纯文本）
  ↓ 清洗（去噪、去重、格式统一）
  ↓ 分块（Chunking）
  ↓ Embedding 模型（文本 → 向量）
  ↓ 写入向量数据库（附元数据）

【在线查询阶段】
用户提问
  ↓ 查询改写（扩展/分解/假设文档）
  ↓ 向量检索（Top-K 近似最近邻）
  ↓ 关键词检索（BM25，可选）
  ↓ 混合召回融合（RRF 算法）
  ↓ 重排序（Cross-Encoder Reranker）
  ↓ 上下文组装
  ↓ LLM 生成答案
```

---

## 三、文档解析

### 3.1 工具选型

| 文档类型 | 推荐工具 | 注意事项 |
|----------|----------|----------|
| PDF（纯文本） | PDFBox、iText | 直接提取，效果好 |
| PDF（扫描件/图片） | MinerU + OCR | 质量取决于 OCR 准确率 |
| Word/Excel | Apache POI | 注意表格结构保留 |
| HTML/网页 | Jsoup | 需过滤导航、广告等噪声 |
| 复杂排版 PDF | MinerU（首选） | 中文 PDF 解析最好用 |

**关键原则**：文档解析质量决定 RAG 效果上限。解析出来的文本有错误/乱码，后续步骤无法弥补。

### 3.2 文本清洗

```java
public String cleanText(String raw) {
    return raw
        .replaceAll("\\s{3,}", "\n\n")     // 连续空行合并
        .replaceAll("[\\u0000-\\u001F]", "")  // 去除控制字符
        .replaceAll("(?m)^\\s*$\\n", "")   // 去除空行
        .trim();
}
```

---

## 四、分块策略（最关键的一步）

**分块粒度影响**：
- 太小：单块缺乏上下文，召回后语义不完整
- 太大：相关内容被无关内容淹没，噪声增加

### 4.1 策略对比

| 策略 | 适用场景 | 优缺点 |
|------|----------|--------|
| 固定大小（+ 重叠） | 通用默认 | 简单，可能切断语义 |
| 按段落/章节边界 | 结构清晰文档 | 语义完整，长度不均 |
| 滑动窗口 | 边界敏感场景 | 避免边界丢失，存储翻倍 |
| 语义分块 | 高质量要求 | 效果最好，实现复杂 |
| 层次化分块 | 长文档精读 | 大块召回 + 小块精读 |

### 4.2 推荐配置

```
通用文档（FAQ、文章）：
  chunk_size = 512-800 tokens
  chunk_overlap = 50-100 tokens

技术文档（代码、规范）：
  chunk_size = 200-400 tokens（按函数/章节切）
  chunk_overlap = 20-50 tokens

长篇报告：
  父块 = 1500 tokens（召回用）
  子块 = 200 tokens（展示用）
```

**LangChain4j 示例**：
```java
DocumentSplitter splitter = DocumentSplitters.recursive(
    512,   // maxSegmentSizeInTokens
    50     // maxOverlapSizeInTokens
);
```

---

## 五、Embedding 模型选型

| 模型 | 维度 | 特点 |
|------|------|------|
| text-embedding-3-small | 1536 | OpenAI，性价比高 |
| text-embedding-3-large | 3072 | OpenAI，效果最好 |
| BGE-large-zh | 1024 | 中文最强开源，本地部署 |
| E5-large | 1024 | 多语言，效果好 |
| m3e-large | 768 | 中文，轻量 |

**中文推荐**：BGE-large-zh（BAAI 出品，C-MTEB 榜首）或 text-embedding-3-small（成本低）。

---

## 六、向量数据库选型

| 数据库 | 特点 | 适用场景 |
|--------|------|----------|
| Qdrant | Rust 实现，高性能，REST + gRPC | 中小项目首选 |
| pgvector | PostgreSQL 扩展 | 已有 PG 的项目 |
| Milvus | 分布式，支持超大规模 | 企业级大规模 |
| Elasticsearch | 全文 + 向量混合检索 | 已有 ES 的项目 |
| Weaviate | 内置 Embedding | 快速原型 |

**Qdrant 快速启动**：
```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

```java
EmbeddingStore<TextSegment> store = QdrantEmbeddingStore.builder()
    .host("localhost")
    .port(6334)
    .collectionName("knowledge-base")
    .build();
```

---

## 七、检索优化（从基础到高级）

### 7.1 基础：向量检索

```java
ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
    .embeddingStore(embeddingStore)
    .embeddingModel(embeddingModel)
    .maxResults(5)           // Top-K
    .minScore(0.65)          // 相似度阈值（0-1）
    .build();
```

**Score 阈值调优**：
- 阈值太高（>0.8）：召回率低，可能漏掉相关文档
- 阈值太低（<0.5）：噪声多，干扰模型
- 推荐起点：0.65，根据业务测试调整

### 7.2 混合检索（向量 + 关键词）

纯向量检索的问题：对专有名词、型号、代码等精确匹配效果差。

```
混合检索 = 向量检索结果 + BM25 关键词检索结果
         ↓
         RRF 融合（Reciprocal Rank Fusion）
         score = Σ(1 / (rank_i + 60))
```

Elasticsearch 天然支持混合检索（kNN + BM25），是企业最常用的选择。

### 7.3 查询改写（解决表达不匹配）

**HyDE（假设文档嵌入）**：
```
原始问题："XX 产品保修多久？"
↓ 让 LLM 生成假设答案
假设答案："XX 产品提供2年保修服务，在此期间..."
↓ 用假设答案向量检索（比原始问题向量更接近文档风格）
```

**查询分解**：
```
复杂问题："2023年北京和上海的GDP分别是多少，哪个更高？"
↓ 分解为子问题
子问题1："2023年北京GDP"
子问题2："2023年上海GDP"
↓ 分别检索 → 合并结果
```

### 7.4 重排序（Reranker）

召回后的 Top-K 结果，用 Cross-Encoder 精排，效果比只用向量检索提升显著：

```
向量检索（Bi-Encoder）：快，可以大量候选
  ↓ Top-20
Cross-Encoder Reranker：慢，但精度高
  ↓ Top-5
组装 Prompt
```

**Cross-Encoder vs Bi-Encoder**：
- Bi-Encoder：问题和文档分别编码，内积计算相似度，速度快
- Cross-Encoder：问题和文档拼接后联合编码，考虑交互，精度高但慢

推荐模型：`bge-reranker-large`（中文）、`cross-encoder/ms-marco-MiniLM-L-12-v2`（英文）

---

## 八、高级 RAG 技巧

### 8.1 元数据过滤

结合向量相似度 + 元数据精确过滤，大幅减少噪声：

```java
// 只检索特定部门、特定时间范围的文档
Filter filter = metadataKey("department").isEqualTo("finance")
    .and(metadataKey("year").isGreaterThanOrEqualTo(2023));

retriever = EmbeddingStoreContentRetriever.builder()
    .filter(filter)
    .build();
```

### 8.2 层次化分块（Parent-Child）

```
存储：
  父块（1500 tokens）→ 存入向量库
  子块（200 tokens）→ 存入向量库，关联父块 ID

检索：
  1. 用子块检索（精度高）
  2. 找到子块后，取出对应父块（上下文更完整）
  3. 把父块内容拼入 Prompt
```

### 8.3 GraphRAG

在传统 RAG 基础上构建知识图谱，适合多跳推理场景（需要关联多个文档中的信息）：

```
文档 → 实体抽取 → 关系抽取 → 知识图谱
查询 → 图检索（BFS/路径检索）→ 子图 → 拼入 Prompt
```

适用场景：法规关联分析、企业关系图谱、多文档事实链推理。

---

## 九、上下文组装

### 9.1 Prompt 结构

```
你是 [角色]。基于以下参考资料回答用户问题。

参考资料：
---
[文档1标题] (来源：XX.pdf 第X页)
[文档1内容]
---
[文档2标题] (来源：XX.pdf 第X页)
[文档2内容]
---

规则：
1. 只基于参考资料回答
2. 资料不足时说明"根据现有资料暂无相关信息"
3. 在答案末尾注明引用来源

用户问题：{question}
```

### 9.2 上下文排序

LLM 对上下文首尾部分注意力更强，中间部分容易被忽略（"Lost in the Middle" 论文）：

- 最相关的文档放在**最前面**
- 次相关的放最后
- 相关性最低的放中间

### 9.3 上下文长度控制

```
可用 token 预算 = context_window - system_prompt - question - output_buffer
                = 128K - 1K - 0.5K - 2K = ~124K

但注意：context 太长时 LLM 注意力分散，通常 3-10 个 chunk 效果最佳
```

---

## 十、RAG 失效模式及解法

| 失效现象 | 根本原因 | 解法 |
|----------|----------|------|
| 检索不到答案 | 分块太细/太粗，向量语义偏移 | 调整 chunk size，尝试 HyDE，层次化分块 |
| 检索到了但答案错 | 相关文档被噪声淹没 | Reranker 精排，提高 score 阈值 |
| 中间上下文被忽略 | LLM "Lost in Middle" | 关键信息前置，压缩上下文 |
| 语义不匹配 | 问题与文档表达风格差异 | 查询改写，文档侧标题增强 |
| 仍有幻觉 | 模型在证据不足时生成 | System Prompt 约束"不知道就说不知道" |
| 专有名词检索差 | 纯向量检索对精确词不敏感 | 加入 BM25 混合检索 |

---

## 十一、Spring AI 完整示例

```java
@Service
public class RagService {

    private final ChatClient chatClient;
    private final EmbeddingModel embeddingModel;
    private final VectorStore vectorStore;

    // 文档入库
    public void ingest(List<Document> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(512, 50, 5, 10000, true);
        List<Document> splitDocs = splitter.apply(documents);
        vectorStore.add(splitDocs);
    }

    // 检索生成
    public String query(String question) {
        // 检索相关文档
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(question)
                .withTopK(5)
                .withSimilarityThreshold(0.65)
        );

        String context = docs.stream()
            .map(d -> String.format("[%s]\n%s", 
                d.getMetadata().getOrDefault("source", "未知来源"), 
                d.getContent()))
            .collect(Collectors.joining("\n---\n"));

        return chatClient.prompt()
            .system(s -> s.text(SYSTEM_PROMPT))
            .user(u -> u.text(USER_TEMPLATE)
                .param("context", context)
                .param("question", question))
            .call()
            .content();
    }

    private static final String SYSTEM_PROMPT = """
        你是企业知识库助手。只基于参考资料中的内容回答，不要编造。
        如资料不足，明确告知用户。
        """;

    private static final String USER_TEMPLATE = """
        参考资料：
        {context}
        
        问题：{question}
        """;
}
```
