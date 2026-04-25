# Wiki Index — llm-wiki-ai

> 知识域：AI 大模型技术体系（机器学习 → Transformer → LLM → 工程应用）
> 格式：`- [[页面名]](相对路径) — 一句话描述`
> 每次 Ingest、Query 回写、Lint 后必须更新。

---

## Summaries（主题总结页）

> 每页对应一个知识主题，聚合多个源文件，不按源文件逐篇生成。

- [[主题-机器学习基础理论]](wiki/summaries/主题-机器学习基础理论.md) — 从任务范式、核心模型到评估方法的经典机器学习完整体系
- [[主题-Transformer架构原理]](wiki/summaries/主题-Transformer架构原理.md) — Self-Attention 如何工作、多头机制、位置编码及工程优化
- [[主题-LLM核心机制与推理优化]](wiki/summaries/主题-LLM核心机制与推理优化.md) — Tokenizer、分布式训练、KV Cache、FlashAttention 等底层机制
- [[主题-Prompt工程实践]](wiki/summaries/主题-Prompt工程实践.md) — 参数调优、模板设计、安全防护的可用工程指南
- [[主题-5C框架知识笔记撰写]](wiki/summaries/主题-5C框架知识笔记撰写.md) — 将 5C 提示契约落到知识笔记生成模板的轻量化实践
- [[主题-RAG系统设计]](wiki/summaries/主题-RAG系统设计.md) — 从文档解析到检索、重排、上下文组装的全链路工程
- [[主题-Agent与工具调用]](wiki/summaries/主题-Agent与工具调用.md) — Function Calling、工具设计、多 Agent 编排及状态管理
- [[主题-LLM生产化与评估]](wiki/summaries/主题-LLM生产化与评估.md) — 评估框架、可观测性、成本控制、安全防护的生产就绪清单

---

## Concepts（概念页）

### 机器学习基础
- [[概念-监督学习与无监督学习]](wiki/concepts/概念-监督学习与无监督学习.md) — 机器学习最基础的任务划分，决定建模目标和方法选择
- [[概念-集成学习]](wiki/concepts/概念-集成学习.md) — 随机森林（Bagging）、GBDT、XGBoost 的原理与工程差异
- [[方法-正则化]](wiki/concepts/方法-正则化.md) — L1/L2/Elastic Net 的数学直觉、稀疏性与特征选择
- [[方法-梯度下降与优化]](wiki/concepts/方法-梯度下降与优化.md) — SGD、Adam、学习率调度及梯度消失/爆炸的解决方案

### Transformer / LLM 机制
- [[机制-Self-Attention]](wiki/concepts/机制-Self-Attention.md) — Q/K/V 三矩阵的匹配+聚合机制，以及为何除以√d_k
- [[机制-多头注意力]](wiki/concepts/机制-多头注意力.md) — 多个子空间并行捕获不同关系，参数量不增加
- [[机制-KV-Cache]](wiki/concepts/机制-KV-Cache.md) — 推理阶段缓存 K/V 矩阵以避免重复计算，显存瓶颈所在
- [[机制-FlashAttention]](wiki/concepts/机制-FlashAttention.md) — 访存优化而非数学改进，通过分块计算减少 HBM 读写 2-4x
- [[方法-分布式训练]](wiki/concepts/方法-分布式训练.md) — DP/TP/PP/ZeRO 四种策略及 3D 并行的组合逻辑
- [[概念-Tokenizer]](wiki/concepts/概念-Tokenizer.md) — BPE/WordPiece/SentencePiece 及中文过度切分问题

### 知识体系
- [[体系-LLM体系]](wiki/concepts/体系-LLM体系.md) — AI 大模型技术体系的全景分层：底层机制 → 能力模块 → 工程系统
- [[体系-AI工程体系]](wiki/concepts/体系-AI工程体系.md) — 把模型能力转化为稳定生产系统的完整工程框架

### 工程应用
- [[方法-Prompt工程]](wiki/concepts/方法-Prompt工程.md) — temperature、Few-shot、CoT、约束设计的核心方法，兼收工程与理论视角
- [[框架-5C框架]](wiki/concepts/框架-5C框架.md) — Character / Cause / Constraint / Contingency / Calibration 五段式提示契约
- [[方法-Few-Shot-Prompting]](wiki/concepts/方法-Few-Shot-Prompting.md) — 用少量高质量示例稳定输出格式与风格
- [[方法-Chain-of-Thought]](wiki/concepts/方法-Chain-of-Thought.md) — 通过显式推理链提升复杂任务准确率及其使用边界
- [[框架-Context-Engineering]](wiki/concepts/框架-Context-Engineering.md) — 把系统提示、工具、记忆和外部数据视作统一上下文编排问题
- [[概念-RAG]](wiki/concepts/概念-RAG.md) — 检索增强生成的分块、Embedding、检索、Reranker 完整链路
- [[方法-混合检索]](wiki/concepts/方法-混合检索.md) — 向量检索 + BM25 + RRF 融合，解决精确名词匹配失败问题
- [[概念-Function-Calling]](wiki/concepts/概念-Function-Calling.md) — 工具描述设计、参数校验、失败处理及安全约束
- [[概念-Agent]](wiki/concepts/概念-Agent.md) — 决策-行动-观察闭环，把语言理解转成持续执行的核心机制
- [[方法-LoRA]](wiki/concepts/方法-LoRA.md) — 低秩分解微调，冻结原参数只训练低秩矩阵，显存节省 70%+

---

## Entities（实体页）

### 工具
- [[工具-XGBoost]](wiki/entities/工具-XGBoost.md) — 梯度提升框架，二阶泰勒展开 + 正则化，工业界集成学习标配
- [[工具-vLLM]](wiki/entities/工具-vLLM.md) — LLM 推理框架，PagedAttention 使显存利用率从 60% 提升到 90%+
- [[工具-LangFuse]](wiki/entities/工具-LangFuse.md) — LLM 可观测性平台，Trace/Span 追踪、成本分析、评估集管理
- [[工具-Qdrant]](wiki/entities/工具-Qdrant.md) — 向量数据库，支持元数据过滤、稀疏向量混合检索、权限隔离
- [[工具-Reranker]](wiki/entities/工具-Reranker.md) — Cross-Encoder 精排模型，联合编码 query+doc，RAG 质量提升最大杠杆

### 产品 / 模型
- [[产品-Claude]](wiki/entities/产品-Claude.md) — Anthropic 的模型系列，强调字面执行、长上下文和 Agent 工作流
- [[产品-Gemini]](wiki/entities/产品-Gemini.md) — Google 的多模态模型系列，提示更趋向简洁与工具化

### 组织
- [[组织-Anthropic]](wiki/entities/组织-Anthropic.md) — Claude 背后的模型提供方，强调清晰指令、XML 结构和思考预算
- [[组织-Google]](wiki/entities/组织-Google.md) — Gemini 相关提示策略与白皮书的主要来源方

---

## Synthesis（综合分析页）

### 学习路径
- [[策略-LLM核心知识掌握路径]](wiki/synthesis/策略-LLM核心知识掌握路径.md) — 5个阶段的LLM知识学习顺序与判断力自检清单
- [[策略-AI高效使用技巧全景]](wiki/synthesis/策略-AI高效使用技巧全景.md) — 从基础技巧到工程化边界的AI使用实战体系
- [[策略-LLM应用开发入门路线]](wiki/synthesis/策略-LLM应用开发入门路线.md) — 从使用者转型为开发者的3阶段路径与工具选型

### 知识全景
- [[对比-RAG-vs-微调-vs-Agent]](wiki/synthesis/对比-RAG-vs-微调-vs-Agent.md) — 三种增强路径的场景、成本、效果对比及选型决策树
- [[判断-LLM工程全栈知识体系]](wiki/synthesis/判断-LLM工程全栈知识体系.md) — 四层知识体系全景及关键判断力清单

---

## Template Files

> 模板文件，不参与 Query 和知识索引。

- `wiki/templates/concept.md`
- `wiki/templates/entity.md`
- `wiki/templates/summary.md`
- `wiki/templates/synthesis.md`
