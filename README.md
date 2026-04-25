# LLM-WIKI

本仓库是一个受约束的 `Karpathy LLM Wiki` / `Obsidian` 工程。目标是把分散的原始资料编译为结构稳定、类型清晰、可持续维护的知识页。

---

## 一、仓库架构

### 1.1 工程目录

```text
llm-wiki/
├── assets/              ← 静态资源文件
├── llm-wiki-ai/         ← AI 大模型技术体系知识库实例
├── llm-wiki-template/   ← 标准模板（派生新实例用）
└── README.md
```

### 1.2 实例目录

```text
llm-wiki-ai/
├── CLAUDE.md      ← 实例路径配置
├── index.md       ← 全局索引（每次操作后更新）
├── log.md         ← 操作日志（append-only）
├── raw/           ← 只读原始资料
└── wiki/
    ├── concepts/  ← 抽象知识对象（概念/机制/方法/框架/体系）
    ├── entities/  ← 具体对象（人物/组织/工具/项目/产品/模型）
    ├── summaries/ ← 每个知识主题一页聚合总结
    └── synthesis/ ← 问题驱动的比较、策略、判断
```

### 1.3 目录职责

- `raw/`：原始资料层，只读，禁止任何修改
- `wiki/concepts/`：脱离具体工具/公司后仍成立的抽象知识对象
- `wiki/entities/`：有明确身份边界的具体对象
- `wiki/summaries/`：一个知识主题对应一页，聚合多个来源（前缀 `主题-`）
- `wiki/synthesis/`：回答一个明确问题，核心是比较/判断/策略（前缀 `问题-/对比-/策略-/判断-`）

---

## 二、核心指令

| 指令 | 说明 |
|------|------|
| `/ingest [路径]` | 把 raw/ 资料编译进 wiki 知识结构 |
| `/query <问题>` | 基于本地 wiki 检索并回答问题 |
| `/lint` | 检查知识库结构健康状态 |

规则与执行细节见根目录 `CLAUDE.md` 和 `.claude/skills/`。
