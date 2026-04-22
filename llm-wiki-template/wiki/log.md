# Wiki Log

本文件记录所有操作，append-only。

格式：
`## [YYYY-MM-DD] 操作类型 | 标题`

## [2026-04-22] Init | Create standard template
- 创建标准目录结构
- 创建 `CLAUDE.md`
- 创建 `wiki/index.md`
- 创建 `wiki/log.md`
- 创建 `wiki/_templates/`

## [2026-04-22] Update | Migrate previous template content
- 迁移旧版 `CLAUDE.md` 的执行规则
- 补充 `wiki/summaries/`
- 补充 `summary.md` 模板
- 扩充概念页、实体页、综合页模板字段

## [2026-04-22] Update | Align template paths
- 保持模板文件位于 `wiki/concepts/`、`wiki/entities/`、`wiki/summaries/`、`wiki/synthesis/`
- 不再使用 `wiki/_templates/`
- 修正 `CLAUDE.md` 与 `README.md` 中的模板路径说明

## [2026-04-22] Demo | Ingest demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较
- 规范化原文路径到 `raw/tech/demo-001-Karpathy的LLM Wiki个人知识管理方案和本体模型驱动AI写作比较.md`
- 创建主题总结页 `wiki/summaries/demo-知识编译式个人知识库：LLM Wiki的核心机制与本体模型增强.md`
- 创建概念页 `LLM-Wiki`、`RAG`、`三层架构`、`Ingest-Query-Lint`、`本体模型-M1-M4`、`article-cards`
- 创建实体页 `Karpathy`、`Obsidian`、`Vannevar-Bush`、`人月聊IT`
- 创建综合页 `wiki/synthesis/demo-AI时代知识管理的双层引擎：Wiki架构与本体模型驱动写作.md`
- 更新 `wiki/index.md`

## [2026-04-22] Optimize | Separate template files from demo knowledge
- 为四个模板文件增加 `status: template` 与 `template_file: true`
- 为实战示例原文、摘要、概念、实体、综合页增加 `status: demo` 与 `demo_file: true`
- 在 `README.md`、`CLAUDE.md` 中明确模板文件与 demo 文件的执行规则
- 将 `wiki/index.md` 分为 Template Files 与 Demo Knowledge 两层

## [2026-04-22] Optimize | Change summaries to topic-level compilation
- 将 `summaries/` 定义为按知识主题聚合，而不是按源文件逐篇生成
- 将 `synthesis/` 定义为问题驱动的综合分析页
- 重写 summary/synthesis 模板字段与页面边界
- 将 demo 页面改为抽象标题，而不是沿用原文标题

## [2026-04-22] Optimize | Rename demo files explicitly
- 将 demo 原始资料改为 `raw/tech/demo-001-...`
- 将 demo summary 改为 `wiki/summaries/demo-...`
- 将 demo synthesis 改为 `wiki/synthesis/demo-...`
- 修正所有 concept / entity / summary / synthesis 的 demo 来源引用

## [2026-04-22] Optimize | Move template files out of knowledge directories
- 将模板文件统一移动到 `wiki/templates/`
- `wiki/concepts/`、`wiki/entities/`、`wiki/summaries/`、`wiki/synthesis/` 只保留 demo 或正式知识文件
- 修正 `CLAUDE.md`、`wiki/index.md` 中的模板路径说明

## [2026-04-22] Audit | 模板规范审查与修复
- **CLAUDE.md**：统一章节编号（一~七）；补充 Inbox 目录及处理规则；新增 log.md 格式规范（含合法操作类型）；新增 index.md 条目格式规范；新增 `analysis_scope` 合法值说明；Lint 步骤补充 `lint_notes` 写入规则
- **wiki/index.md**：所有条目补全文件路径和一句话描述；分区增加说明注释
- **wiki/templates/concept.md**：frontmatter 新增 `lint_notes` 字段
- **wiki/templates/entity.md**：frontmatter 新增 `domain`、`lint_notes` 字段
- **wiki/templates/summary.md**：frontmatter 新增 `updated`、`lint_notes` 字段
- **wiki/templates/synthesis.md**：frontmatter 新增 `analysis_scope` 合法值注释、`lint_notes` 字段
- **wiki/concepts/demo-Memex.md**：补充缺失的 demo 概念页，完整 demo 集合（7个概念页）
