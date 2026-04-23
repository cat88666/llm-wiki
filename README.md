# LLM-WIKI

本仓库是一个受约束的 `Karpathy LLM Wiki` / `Obsidian` 工程。目标是把分散的原始资料编译为结构稳定、类型清晰、可持续维护的知识页。

---
## 一、仓库架构

### 1.1 工程目录

```text
dx-LLM/
├── assets/  静态资源文件
├── llm-wiki-ai/   AI知识库实例
├── llm-wiki-template/ 标准模板实例
└── README.md
```

### 1.2 实例目录

```text
llm-wiki-ai/
├── CLAUDE.md
├── index.md
├── log.md
├── raw/
└── wiki/
    ├── concepts/
    ├── entities/
    ├── summaries/
    └── synthesis/
```

### 1.3 目录职责

- `raw/`
  定义：原始资料层。
  规则：只读，不修改，不润色，不重写。

- `wiki/concepts/`
  定义：抽象知识对象层。
  规则：只放概念、机制、方法、框架、体系。

- `wiki/entities/`
  定义：具体对象层。
  规则：只放人物、组织、工具、项目、产品、模型系列等具体实体。

- `wiki/summaries/`
  定义：主题总结层。
  规则：只放某个知识主题的聚合总结，不按单篇资料逐篇建页。

- `wiki/synthesis/`
  定义：综合判断层。
  规则：只放问题、比较、策略、判断，不放主题介绍。

- `index.md`
  定义：全局索引。
  规则：结构变化后必须更新。

- `log.md`
  定义：操作日志。
  规则：append-only，只追加，不改历史。

---

## 二、使用方法

### 2.1 `/ingest`

目标：把 `raw/` 资料编译进现有知识结构。


### 2.2 `/query`

目标：基于现有 wiki 回答问题，而不是脱离知识库自由发挥。


### 2.3 `/lint`

目标：检查结构是否仍然满足工程约束。


---
