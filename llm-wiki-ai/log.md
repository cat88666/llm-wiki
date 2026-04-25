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
- 来源：`raw/engineering/01-理论-机器学习.md`
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
- 来源：`raw/engineering/` 下全部12个文件（01-12）
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

## [2026-04-23] Ingest | 增量摄入 prompt 资料并修正索引
- 来源：`raw/prompt/` 下 8 个文件（5C 提示词、完整提示工程指南、提示工程指南、提示策略 Gemini、提示词 Anthropic、提示词工程、提示词工程指南、提示词白皮书 Goolge）
- 将 prompt 资料正式并入 `wiki/summaries/Prompt工程实践.md`，补充现代提示工程的共识、模型差异与轻量框架视角
- 修正已有 prompt 相关页面中的旧来源路径，统一到当前 `raw/prompt/...` 目录
- 新增来源摘要页：`wiki/synthesis/摘要-tech-with-tim-提示词工程课程.md`
- 更新 `index.md`，补登记既有的 prompt 相关 summary / concept / entity / synthesis 页面

## [2026-04-23] Update | 重写 concepts 为第一性原理风格
- 统一 `wiki/concepts/` 主要概念页结构为：一句话定义、第一性原理、核心机制、关键权衡、概念关系、应用边界
- 重写注意力、推理优化、机器学习基础、Prompt/RAG/Agent 等概念页，去掉术语罗列和导航式写法
- 将 `AI工程体系`、`LLM体系`、`5C_Framework` 等框架页改写为框架性概念，而非目录索引页

## [2026-04-25] Lint | 全库结构清理与命名规范化

### 删除内容
- 删除 `wiki/summaries/L1 基础原理/`、`L2 能力模块/`、`L3 工程系统/` 三个子目录（共 30 个薄卡片页，`type: atomic`，内容过薄不符合 summaries 规范）
- 删除 `wiki/synthesis/` 中 13 个 `摘要-xxx.md` 文件（类型混放，单篇摘要不得进入 synthesis）

### 合并内容
- `concepts/Prompt工程.md` + `concepts/Prompt_Engineering.md` → `concepts/方法-Prompt工程.md`（来源合并为 7 个，保留双方视角）

### 迁移内容
- `entities/LoRA.md` → `concepts/方法-LoRA.md`（LoRA 是训练方法，不是实体）

### 文件重命名（全库规范化）
- concepts/（22 个）：统一加 `概念-` / `机制-` / `方法-` / `框架-` / `体系-` 前缀
- entities/（9 个）：统一加 `工具-` / `产品-` / `组织-` 前缀
- summaries/（8 个）：统一加 `主题-` 前缀
- synthesis/（2 个）：统一加 `对比-` / `判断-` 前缀

### 链接更新
- 批量更新所有 wiki 页面中的 `[[旧名]]` → `[[新名]]`（含 concepts、entities、summaries 链接）

### 索引更新
- 重写 `index.md`，登记所有 41 个页面（含新增的 `概念-Agent`、`体系-AI工程体系`、`方法-LoRA`）

## [2026-04-25] 新建 | 三条学习路径 synthesis 页

- 新建 `wiki/synthesis/策略-LLM核心知识掌握路径.md`：5阶段学习顺序（Transformer→机制→Prompt→RAG/Agent→生产化），含判断力自检清单
- 新建 `wiki/synthesis/策略-AI高效使用技巧全景.md`：7个维度的AI使用实战技巧，含Claude/Gemini差异、高频场景套路、5C框架
- 新建 `wiki/synthesis/策略-LLM应用开发入门路线.md`：3阶段转型路径（接入→用好→上线），含工具选型表和常见误区
- 更新 `index.md`，Synthesis 新增"学习路径"分组

## [2026-04-25] Query | LLM 核心机制

- 检索页面：`主题-LLM核心机制与推理优化`、`机制-KV-Cache`、`机制-FlashAttention`、`概念-Tokenizer`
- 回答内容：Tokenizer → KV Cache → FlashAttention → 分布式训练四块机制及相互关系
- 无新增回写（现有页面已覆盖，内容完整）
