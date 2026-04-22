# Wiki Index

> 本文件是当前知识库的总索引。每次 Ingest、Query 回写、Lint 后必须更新。
> 格式：`- [[页面名]](相对路径) — 一句话描述`

---

## Template Files

> 模板文件，不参与 Query 和知识索引，仅供新建页面时参考。

- `wiki/templates/concept.md` — 概念页标准模板
- `wiki/templates/entity.md` — 实体页标准模板
- `wiki/templates/summary.md` — 主题总结页标准模板
- `wiki/templates/synthesis.md` — 综合分析页标准模板

---

## Concepts（概念页）

### Demo
- [[demo-LLM-Wiki]](concepts/demo-LLM-Wiki.md) — 由 LLM 持续维护的结构化知识库范式，核心是知识"编译"而非"检索"
- [[demo-RAG]](concepts/demo-RAG.md) — 传统检索增强生成，每次查询时从原始文档实时检索，无积累
- [[demo-三层架构]](concepts/demo-三层架构.md) — LLM Wiki 的组织结构：raw（只读）/ wiki（AI维护）/ schema（配置）
- [[demo-Ingest-Query-Lint]](concepts/demo-Ingest-Query-Lint.md) — LLM Wiki 的三种核心操作：摄入、查询回写、健康检查
- [[demo-本体模型-M1-M4]](concepts/demo-本体模型-M1-M4.md) — 四层知识提取框架，填补 Karpathy 方案中"提取什么"的关键空白
- [[demo-article-cards]](concepts/demo-article-cards.md) — raw 与 wiki 之间的标准化卡片中间层，解决大规模文章的检索效率问题
- [[demo-Memex]](concepts/demo-Memex.md) — Vannevar Bush 1945年提出的个人知识存储设想，LLM Wiki 的精神先祖

---

## Entities（实体页）

### Demo
- [[demo-Karpathy]](entities/demo-Karpathy.md) — LLM Wiki 方案提出者，AI 研究者
- [[demo-Obsidian]](entities/demo-Obsidian.md) — 本地 Markdown 知识管理工具，LLM Wiki 工作流中扮演"IDE"角色
- [[demo-Vannevar-Bush]](entities/demo-Vannevar-Bush.md) — 1945年 Memex 概念提出者，个人知识管理思想先驱
- [[demo-人月聊IT]](entities/demo-人月聊IT.md) — 本体模型驱动 AI 写作方案提出者

---

## Summaries（主题总结页）

> 每页对应一个知识主题，可聚合多个来源，不按源文件逐篇生成。

### Demo
- [[demo-知识编译式个人知识库：LLM Wiki的核心机制与本体模型增强]](summaries/demo-知识编译式个人知识库：LLM Wiki的核心机制与本体模型增强.md) — LLM Wiki 如何从检索式问答演进为知识编译系统，及本体模型的增强作用

---

## Synthesis（综合分析页）

> 围绕一个问题、比较或判断展开，不重复 summary 内容。

### Demo
- [[demo-AI时代知识管理的双层引擎：Wiki架构与本体模型驱动写作]](synthesis/demo-AI时代知识管理的双层引擎：Wiki架构与本体模型驱动写作.md) — LLM Wiki 架构与本体模型驱动写作如何互补，构成 AI 时代知识管理的完整方案
