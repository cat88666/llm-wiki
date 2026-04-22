# Wiki Log

本文件记录所有操作，append-only，禁止修改历史记录。

格式：`## [YYYY-MM-DD] 操作类型 | 标题`
操作类型：`Init` / `Ingest` / `Query` / `Lint` / `Update`

---

## [2026-04-22] Init | 初始化 wiki 目录结构
- 创建 `raw/`、`wiki/`、`CLAUDE.md`
- 创建 `wiki/index.md`、`wiki/log.md`
- 创建 `wiki/concepts/`、`wiki/entities/`、`wiki/synthesis/`、`wiki/summaries/`

## [2026-04-22] Update | 完善初始模板
- 写入 `CLAUDE.md`（系统 Schema，包含三种操作流程、四类页面格式规范、行为准则）
- 更新 `wiki/index.md`（规范格式，增加分类说明）
- 创建各类页面模板：`_template.md`（concepts / entities / summaries / synthesis）

## [2026-04-22] Ingest | Karpathy 的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较
- 来源：`raw/wap/001-Wiki方案.md`，作者：人月聊IT，发布：2026-04-10
- 创建摘要页：`summaries/001-Wiki方案.md`
- 新增概念页（7个）：`LLM-Wiki`、`RAG`、`三层架构`、`Ingest-Query-Lint`、`本体模型-M1-M4`、`article-cards`、`Memex`
- 新增实体页（3个）：`Karpathy`、`Obsidian`、`Vannevar-Bush`
- 更新 `wiki/index.md`（登记所有新增页面）
- 注意：本体模型-M1-M4 中矛盾解决机制尚未显式化，article-cards 具体 YAML schema 待设计

## [2026-04-22] Ingest | 01-理论-机器学习
- 来源：`raw/tech/原理/01-理论-机器学习.md`
- 创建主题总结页：`wiki/summaries/经典机器学习基础：任务范式、核心模型与评估框架.md`
- 新增概念页（9个）：`机器学习`、`监督学习与无监督学习`、`线性模型`、`正则化`、`支持向量机`、`树模型与集成学习`、`聚类与降维`、`梯度下降`、`模型评估`
- 未新增实体页：该资料以理论知识为主，不包含稳定实体对象
- 更新 `index.md`
- 说明：该 summary 为主题聚合页，后续与传统机器学习相关的课程资料应继续合并更新，而不是重复生成新的 summary

## [2026-04-22] Update | 统一 llm-wiki-ai 到主题聚合规则
- 用母版规则替换 `CLAUDE.md`，将 summary 从“单篇摘要”统一为“主题聚合页”
- 新增 `wiki/templates/`，补齐 concept/entity/summary/synthesis 模板
- 统一 `index.md` 路径规则到 `wiki/...`
- 补充缺失概念页：`判别式与生成式模型`、`贝叶斯方法`
- 更新主题总结页 `经典机器学习基础：任务范式、核心模型与评估框架.md`

## [2026-04-22] Update | 清理 llm-wiki-ai 的旧 demo 内容
- 删除与当前课程资料无关的旧 `LLM Wiki` 演示页：`LLM-Wiki`、`RAG`、`三层架构`、`Ingest-Query-Lint`、`本体模型-M1-M4`、`article-cards`、`Memex`
- 删除旧演示实体页：`Karpathy`、`Obsidian`、`Vannevar-Bush`
- 删除旧单篇摘要页：`001-Wiki方案.md`
- 清理 `.DS_Store`
- 重写 `index.md`，将知识域明确限定为”AI 大模型应用课程资料”

## [2026-04-22] Ingest | 全量摄入12个源文件，重建 wiki
- 来源：`raw/tech/原理/` 下全部12个文件（01-12）
- 清理旧概念页（旧体系11个），重建为新体系
- 新增 Summary 页（7个）：
  - `机器学习基础理论`（源：01、08）
  - `Transformer架构原理`（源：02、09）
  - `LLM核心机制与推理优化`（源：03、10）
  - `Prompt工程实践`（源：04）
  - `RAG系统设计`（源：05、11）
  - `Agent与工具调用`（源：06、11）
  - `LLM生产化与评估`（源：07、11、12）
- 新增 Concept 页（14个）：监督学习与无监督学习、集成学习、正则化、梯度下降与优化、Self-Attention、多头注意力、KV-Cache、FlashAttention、分布式训练、Tokenizer、Prompt工程、RAG、混合检索、Function-Calling
- 新增 Entity 页（6个）：XGBoost、vLLM、LangFuse、Qdrant、LoRA、Reranker
- 新增 Synthesis 页（2个）：RAG-vs-微调-vs-Agent 对比、LLM工程全栈知识体系
- 重写 `index.md`（含完整路径和描述）
