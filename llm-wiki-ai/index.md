# Wiki Index — llm-wiki-ai

> 知识域：AI 大模型技术体系（机器学习 → Transformer → LLM → 工程应用）
> 格式：`- [[页面名]](相对路径) — 一句话描述`
> 每次 Ingest、Query 回写、Lint 后必须更新。

---

## Summaries（主题总结页）

> 每页对应一个知识主题，聚合多个源文件，不按源文件逐篇生成。

- [[机器学习基础理论]](wiki/summaries/机器学习基础理论.md) — 从任务范式、核心模型到评估方法的经典机器学习完整体系
- [[Transformer架构原理]](wiki/summaries/Transformer架构原理.md) — Self-Attention 如何工作、多头机制、位置编码及工程优化
- [[LLM核心机制与推理优化]](wiki/summaries/LLM核心机制与推理优化.md) — Tokenizer、分布式训练、KV Cache、FlashAttention 等底层机制
- [[Prompt工程实践]](wiki/summaries/Prompt工程实践.md) — 参数调优、模板设计、安全防护的可用工程指南
- [[5C框架-Markdown知识笔记撰写]](wiki/summaries/5C框架-Markdown知识笔记撰写.md) — 将 5C 提示契约落到知识笔记生成模板的轻量化实践
- [[RAG系统设计]](wiki/summaries/RAG系统设计.md) — 从文档解析到检索、重排、上下文组装的全链路工程
- [[Agent与工具调用]](wiki/summaries/Agent与工具调用.md) — Function Calling、工具设计、多 Agent 编排及状态管理
- [[LLM生产化与评估]](wiki/summaries/LLM生产化与评估.md) — 评估框架、可观测性、成本控制、安全防护的生产就绪清单

---

## Concepts（概念页）

### 机器学习基础
- [[监督学习与无监督学习]](wiki/concepts/监督学习与无监督学习.md) — 机器学习最基础的任务划分，决定建模目标和方法选择
- [[集成学习]](wiki/concepts/集成学习.md) — 随机森林（Bagging）、GBDT、XGBoost 的原理与工程差异
- [[正则化]](wiki/concepts/正则化.md) — L1/L2/Elastic Net 的数学直觉、稀疏性与特征选择
- [[梯度下降与优化]](wiki/concepts/梯度下降与优化.md) — SGD、Adam、学习率调度及梯度消失/爆炸的解决方案

### Transformer / LLM
- [[Self-Attention]](wiki/concepts/Self-Attention.md) — Q/K/V 三矩阵的匹配+聚合机制，以及为何除以√d_k
- [[多头注意力]](wiki/concepts/多头注意力.md) — 多个子空间并行捕获不同关系，参数量不增加
- [[KV-Cache]](wiki/concepts/KV-Cache.md) — 推理阶段缓存 K/V 矩阵以避免重复计算，显存瓶颈所在
- [[FlashAttention]](wiki/concepts/FlashAttention.md) — 访存优化而非数学改进，通过分块计算减少 HBM 读写 2-4x
- [[分布式训练]](wiki/concepts/分布式训练.md) — DP/TP/PP/ZeRO 四种策略及 3D 并行的组合逻辑
- [[Tokenizer]](wiki/concepts/Tokenizer.md) — BPE/WordPiece/SentencePiece 及中文过度切分问题

### 工程应用
- [[Prompt工程]](wiki/concepts/Prompt工程.md) — temperature 参数、Few-shot、CoT、多轮压缩的核心技巧
- [[Prompt_Engineering]](wiki/concepts/Prompt_Engineering.md) — 从提示词工程到上下文工程的现代共识与模型差异
- [[5C_Framework]](wiki/concepts/5C_Framework.md) — Character / Cause / Constraint / Contingency / Calibration 五段式提示契约
- [[Few_Shot_Prompting]](wiki/concepts/Few_Shot_Prompting.md) — 用少量高质量示例稳定输出格式与风格
- [[Chain_of_Thought]](wiki/concepts/Chain_of_Thought.md) — 通过显式推理链提升复杂任务准确率及其使用边界
- [[Context_Engineering]](wiki/concepts/Context_Engineering.md) — 把系统提示、工具、记忆和外部数据视作统一上下文编排问题
- [[RAG]](wiki/concepts/RAG.md) — 检索增强生成的分块、Embedding、检索、Reranker 完整链路
- [[混合检索]](wiki/concepts/混合检索.md) — 向量检索 + BM25 + RRF 融合，解决精确名词匹配失败问题
- [[Function-Calling]](wiki/concepts/Function-Calling.md) — 工具描述设计、参数校验、失败处理及安全约束

---

## Entities（实体页）

### 工具
- [[XGBoost]](wiki/entities/XGBoost.md) — 梯度提升框架，二阶泰勒展开 + 正则化，工业界集成学习标配
- [[vLLM]](wiki/entities/vLLM.md) — LLM 推理框架，PagedAttention 使显存利用率从 60% 提升到 90%+
- [[LangFuse]](wiki/entities/LangFuse.md) — LLM 可观测性平台，Trace/Span 追踪、成本分析、评估集管理
- [[Qdrant]](wiki/entities/Qdrant.md) — 向量数据库，支持元数据过滤、稀疏向量混合检索、权限隔离
- [[Claude]](wiki/entities/Claude.md) — Anthropic 的模型系列，强调字面执行、长上下文和 Agent 工作流
- [[Gemini]](wiki/entities/Gemini.md) — Google 的多模态模型系列，提示更趋向简洁与工具化

### 技术
- [[LoRA]](wiki/entities/LoRA.md) — 低秩分解微调，冻结原参数只训练低秩矩阵，显存节省 70%+
- [[Reranker]](wiki/entities/Reranker.md) — Cross-Encoder 精排模型，联合编码 query+doc，RAG 质量提升最大杠杆
- [[Anthropic]](wiki/entities/Anthropic.md) — Claude 背后的模型提供方，强调清晰指令、XML 结构和思考预算
- [[Google]](wiki/entities/Google.md) — Gemini 相关提示策略与白皮书的主要来源方

---

## Synthesis（综合分析页）

- [[RAG-vs-微调-vs-Agent：LLM能力增强三种路径对比]](wiki/synthesis/RAG-vs-微调-vs-Agent：LLM能力增强三种路径对比.md) — 三种增强路径的场景、成本、效果对比及选型决策树
- [[LLM工程全栈：从理论到生产的完整知识体系]](wiki/synthesis/LLM工程全栈：从理论到生产的完整知识体系.md) — 四层知识体系全景及关键判断力清单
- [[摘要-17期课程-ai大模型实战]](wiki/synthesis/摘要-17期课程-ai大模型实战.md) — 17 期课程材料的来源摘要
- [[摘要-22期课程-提示工程到rag]](wiki/synthesis/摘要-22期课程-提示工程到rag.md) — 22 期 Prompt 到 RAG 课程摘要
- [[摘要-22期课程-rag检索增强]](wiki/synthesis/摘要-22期课程-rag检索增强.md) — 22 期 RAG 检索增强课程摘要
- [[摘要-22期课程-agent智能体]](wiki/synthesis/摘要-22期课程-agent智能体.md) — 22 期 Agent 课程摘要
- [[摘要-22期课程-多模态技术]](wiki/synthesis/摘要-22期课程-多模态技术.md) — 22 期多模态课程摘要
- [[摘要-5c-prompt-contracts-paper]](wiki/synthesis/摘要-5c-prompt-contracts-paper.md) — 5C Prompt Contract 论文摘要
- [[摘要-complete-prompt-engineering-guide-2025]](wiki/synthesis/摘要-complete-prompt-engineering-guide-2025.md) — 通用提示工程指南的来源摘要
- [[摘要-ai-prompt-engineering-2025-2026-espo]](wiki/synthesis/摘要-ai-prompt-engineering-2025-2026-espo.md) — 2025-2026 提示工程趋势与模型差异摘要
- [[摘要-anthropic-prompting-best-practices]](wiki/synthesis/摘要-anthropic-prompting-best-practices.md) — Claude 官方提示最佳实践摘要
- [[摘要-gemini-api-prompting-strategies]](wiki/synthesis/摘要-gemini-api-prompting-strategies.md) — Gemini API 提示设计策略摘要
- [[摘要-google-prompt-engineering-whitepaper]](wiki/synthesis/摘要-google-prompt-engineering-whitepaper.md) — Google 提示工程白皮书摘要
- [[摘要-prompt-engineering-2025-guide-promptbuilder]](wiki/synthesis/摘要-prompt-engineering-2025-guide-promptbuilder.md) — PromptBuilder 2025 指南摘要
- [[摘要-tech-with-tim-提示词工程课程]](wiki/synthesis/摘要-tech-with-tim-提示词工程课程.md) — Tech With Tim 长视频课程的来源摘要

---

## Template Files

> 模板文件，不参与 Query 和知识索引。

- `wiki/templates/concept.md`
- `wiki/templates/entity.md`
- `wiki/templates/summary.md`
- `wiki/templates/synthesis.md`
