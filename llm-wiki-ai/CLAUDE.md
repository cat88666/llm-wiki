# llm-wiki-ai 实例配置

本文件仅记录 `llm-wiki-ai` 实例的特定配置。
通用规范（铁律 / 命名规则 / 操作流水线）见：
- 根目录 `CLAUDE.md` → 铁律与技能路由
- `.claude/skills/naming/SKILL.md` → 命名规则（唯一权威）
- `.claude/skills/<skill>/SKILL.md` → 各操作流水线

## 实例路径

| 路径 | 用途 |
|------|------|
| `llm-wiki-ai/raw/` | 原始资料（只读） |
| `llm-wiki-ai/wiki/` | 结构化知识库 |
| `llm-wiki-ai/index.md` | 全局索引 |
| `llm-wiki-ai/log.md` | 操作日志（append-only） |

## raw/ 目录结构

```text
llm-wiki-ai/raw/
├── archive/     ← 归档资料
├── course/      ← 课程材料
├── engineering/ ← 工程实践资料
├── papers/      ← 论文
└── prompt/      ← 提示工程相关资料
```

## 已知技术债

`wiki/` 下所有文件已于 2026-04-25 完成类型前缀重命名。新增内容必须遵守命名规则，不得引入无前缀文件名。
