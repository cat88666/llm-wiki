# My Knowledge Wiki — Standard Schema

> 本文件是 `llm-wiki-ai` 的统一执行规范。每次执行操作前应先读取本文件。
> 角色定位：不是问答机器人，而是这个 wiki 的主动维护者和协作者。

## 一、目录结构约定

```text
llm-wiki-ai/
├── raw/                 ← 只读原始资料（永不修改）
│   ├── tech/
│   ├── books/
│   ├── journal/
│   └── ...
├── wiki/                ← AI 维护的结构化知识库
│   ├── concepts/        ← 概念、主题、方法、模型
│   ├── entities/        ← 人物、项目、工具、机构、书籍
│   ├── summaries/       ← 按知识主题聚合的总结页
│   ├── synthesis/       ← 面向问题的综合分析、判断与方案页
│   └── templates/       ← 标准模板文件
├── index.md             ← 总索引，每次操作后更新
├── log.md               ← 操作日志，append-only
└── CLAUDE.md
```

核心原则：
- `raw/` 只读，AI 只读取，永不写入或修改
- `wiki/` 是 AI 的写作空间，对其中内容拥有完整增删改权限
- 新知识必须优先链接已有概念页和实体页，避免重复造页
- 每次 Ingest、Query 回写、Lint 结束后，必须更新 `index.md` 和 `log.md`
- 所有结论应尽量保留来源路径，优先指向 `raw/`

## 二、三种核心操作

### 2.1 Ingest（摄入）

执行步骤：
1. 读取指定 `raw/` 文件，完整理解内容
2. 提取以下信息：
   - 主题
   - 概念
   - 实体
   - 观点
   - 方法
   - 事实
   - 案例
3. 判断该资料是否属于已有知识主题簇：
   - 若属于已有主题，则更新对应 `wiki/summaries/` 页面
   - 若形成新主题，创建新的 `wiki/summaries/[抽象主题名].md`
4. 在 `wiki/concepts/` 中创建或更新相关概念页
5. 在 `wiki/entities/` 中创建或更新相关实体页
6. 如有跨来源价值，在 `wiki/synthesis/` 中创建或更新综合页
7. 更新 `index.md`
8. 追加 `log.md`

规则补充：
- `summaries/` 不是“每篇原文一页摘要”，而是“每个知识主题一页总结”
- 多个源文件若讨论同一知识主题，应合并到同一 summary 页面
- summary 标题必须是抽象后的知识标题，不直接复用原文标题
- `synthesis/` 用于回答更高层的问题、比较、方案和判断，不重复 summary

### 2.2 Query（查询）

执行步骤：
1. 读取 `index.md` 定位相关页面
2. 读取相关概念页、实体页、摘要页、综合页
3. 综合回答，并尽量为关键论点标注来源
4. 如果回答本身形成稳定知识：
   - 属于某个知识主题的稳定总结，则更新 `wiki/summaries/`
   - 属于跨主题判断、比较、方案，则回写到 `wiki/synthesis/`
5. 追加 `log.md`

### 2.3 Lint（健康检查）

执行步骤：
1. 扫描所有 wiki 页面，检查：
   - 矛盾
   - 过时内容
   - 孤立页面
   - 重复概念
   - 失效链接
   - 缺少来源的结论
2. 生成修复建议，或在风险可控时直接修复
3. 更新 `index.md`（如有结构变化）
4. 追加 `log.md`

## 三、页面格式规范

- 概念页模板使用 `wiki/templates/concept.md`
- 实体页模板使用 `wiki/templates/entity.md`
- 摘要页模板使用 `wiki/templates/summary.md`
- 综合页模板使用 `wiki/templates/synthesis.md`

页面要求：
- 优先结构化表达
- 链接统一使用 Obsidian 风格 `[[页面名]]`
- 原始资料统一使用相对路径指向 `raw/`
- 页面名应与文件名一致

summary 与 synthesis 的边界：
- `summary`：围绕一个知识主题做聚合总结，可对应 1 个或多个来源
- `synthesis`：围绕一个问题、比较、策略或判断进行综合分析，通常引用多个 summary / concept / entity
