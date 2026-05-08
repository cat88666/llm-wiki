# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库用途

受约束的 Karpathy LLM Wiki / Obsidian 工程。把 `raw/` 中的分散原始资料**编译**为 `wiki/` 中结构稳定、类型清晰的知识页。这是硬约束——不允许简单导入，必须先判断类型再写入。

活跃实例：`llm-wiki-ai/`（AI 大模型技术体系）
模板母版：`llm-wiki-template/`（派生新实例用，不参与日常操作）

## 技能指令

| 指令 | 触发条件 |
|------|---------|
| `/ingest [路径]` | 用户要摄入/导入/纳入资料到知识库 |
| `/query <问题>` | 用户要基于 wiki 提问 |
| `/lint` | 用户要检查知识库健康状态 |
| `/transcribe` | 用户要将视频课程转录为笔记并写入 raw/course/ |
| `/bug <描述>` | 用户要排查线上 bug，执行 Kibana + Yearning + 代码全链路排障 |

详细执行流水线见 `.claude/skills/` 对应 SKILL.md。
命名规则的唯一权威：`.claude/skills/naming/SKILL.md`。

## 铁律（任何操作均不可违反）

1. **`raw/` 只读**：绝对不写入、移动或修改 `raw/` 下任何文件
2. **操作后必须收尾**：每次 Ingest / Query 回写 / Lint 后，必须更新 `llm-wiki-ai/index.md` 并追加 `llm-wiki-ai/log.md`
3. **新建页面前必须命名**：任何新文件落盘前，先按 `naming/SKILL.md` 规则确定文件名，禁止自由命名
4. **优先更新，不轻易新建**：新资料优先合并进已有页面，确实需要新主题时才新建
5. **`concepts/` 页面结构**：必须包含以下 6 个部分，缺一不可：
   > 一句话定义 → 第一性原理 → 核心机制 → 关键权衡 → 与其他概念的关系 → 应用边界
