# My Knowledge Wiki — Standard Schema（母版）

> **本文件是派生新实例的母版**，内容需自包含。日常操作请勿修改本文件。
> 派生新实例时：复制本文件到新实例目录，按实际路径和特性定制。
>
> 本文件是所有 `llm-wiki-*` 知识库的统一执行规范。每次执行操作前应先读取本文件。
> 角色定位：不是问答机器人，而是这个 wiki 的主动维护者和协作者。

---

## 一、目录结构约定

```text
llm-wiki-xxx/
├── raw/                 ← 只读原始资料（永不修改）
│   ├── tech/
│   ├── books/
│   ├── journal/
│   └── ...
├── wiki/                ← AI 维护的结构化知识库
│   ├── index.md         ← 总索引，每次操作后更新
│   ├── log.md           ← 操作日志，append-only
│   ├── concepts/        ← 概念、主题、方法、模型
│   ├── entities/        ← 人物、项目、工具、机构、书籍
│   ├── summaries/       ← 按知识主题聚合的总结页
│   ├── synthesis/       ← 面向问题的综合分析、判断与方案页
│   └── templates/       ← 标准模板文件
├── Inbox/               ← 待处理新资料临时存放（处理后移入 raw/）
└── CLAUDE.md
```

**核心原则：**
- `raw/` 只读，AI 只读取，永不写入或修改
- `wiki/` 是 AI 的写作空间，对其中内容拥有完整增删改权限
- 新知识必须优先链接已有概念页和实体页，避免重复造页
- 每次 Ingest、Query 回写、Lint 结束后，必须更新 `wiki/index.md` 和 `wiki/log.md`
- 所有结论应尽量保留来源路径，优先指向 `raw/`

**Inbox 处理规则：**
- `Inbox/` 是临时暂存区，不是 raw 的一部分
- 执行 Ingest 前，先将 Inbox 中的文件移动到 `raw/` 对应分类目录，再执行摄入
- Inbox 中的文件不直接作为 Ingest 来源

---

## 二、模板文件与演示文件约定

模板文件路径：
- `wiki/templates/concept.md`
- `wiki/templates/entity.md`
- `wiki/templates/summary.md`
- `wiki/templates/synthesis.md`

模板文件必须带有：
- `status: template`
- `template_file: true`

演示文件必须带有：
- `status: demo`
- `demo_file: true`

演示文件命名约定：所有 demo 文件（原始资料、概念页、实体页、摘要页、综合页）统一使用 `demo-` 前缀。

执行规则：
- Query 时忽略 `template_file: true` 文件
- Lint 时忽略模板文件，但检查 demo 文件和正式知识文件
- 更新 `wiki/index.md` 时，模板文件进入"Template Files"区，不进入实际知识索引区
- 当从母版复制出正式实例库时，可选择删除所有 `demo_file: true` 文件

---

## 三、三种核心操作

### 3.1 Ingest（摄入）

**触发指令示例：**
```
请摄入 raw/tech/xxx.md
```

**执行步骤：**
1. 读取指定 `raw/` 文件，完整理解内容
2. 提取以下信息：主题、概念、实体、观点、方法、事实、案例
3. 判断该资料是否属于已有知识主题簇：
   - 若属于已有主题 → 更新对应 `wiki/summaries/` 页面
   - 若形成新主题 → 创建新的 `wiki/summaries/[抽象主题名].md`
4. 在 `wiki/concepts/` 中创建或更新相关概念页
5. 在 `wiki/entities/` 中创建或更新相关实体页
6. 如有跨来源综合价值，在 `wiki/synthesis/` 中创建或更新综合页
7. 更新 `wiki/index.md`
8. 追加 `wiki/log.md`

**规则补充：**
- `summaries/` 不是"每篇原文一页摘要"，而是"每个知识主题一页总结"
- 多个源文件若讨论同一知识主题，应合并到同一 summary 页面
- summary 标题必须是抽象后的知识标题，不直接复用原文标题
- `synthesis/` 用于回答更高层的问题、比较、方案和判断，不重复 summary 内容
- 一次 Ingest 通常会触及多个页面，而不是只生成一篇新 summary

---

### 3.2 Query（查询）

**执行步骤：**
1. 读取 `wiki/index.md` 定位相关页面
2. 读取相关概念页、实体页、摘要页、综合页
3. 综合回答，并尽量为关键论点标注来源
4. 判断回答是否有回写价值：
   - 属于某个知识主题的稳定总结 → 更新 `wiki/summaries/`
   - 属于跨主题判断、比较、方案 → 回写到 `wiki/synthesis/`
   - 仅为一次性解答 → 不回写
5. 追加 `wiki/log.md`

---

### 3.3 Lint（健康检查）

**触发指令示例：**
```
请对 wiki 做一次健康检查
```

**执行步骤：**
1. 扫描所有 wiki 页面（跳过 `template_file: true` 文件），检查：
   - 矛盾：不同页面对同一概念/事实的描述不一致
   - 过时：某页面的观点已被新摄入资料推翻
   - 孤立：没有被任何其他页面引用的页面
   - 重复：两个概念页实质上描述同一概念
   - 失效链接：`[[页面名]]` 引用但对应文件不存在
   - 缺少来源：有结论但没有指向 `raw/` 的来源引用
2. 生成 Lint 报告，经用户确认后修复
3. 将发现的矛盾或待解决问题写入相关页面的 `lint_notes` 字段
4. 更新 `wiki/index.md`（如有结构变化）
5. 追加 `wiki/log.md`

---

## 四、页面格式规范

使用 `wiki/templates/` 中对应模板新建页面。

**页面通用要求：**
- 优先结构化表达
- 内部链接统一使用 Obsidian 风格 `[[页面名]]`
- 原始资料统一使用相对路径，例如 `../../raw/tech/xxx.md`
- 页面名与文件名保持一致

**summary 与 synthesis 的边界：**
- `summary`：围绕一个知识主题做聚合总结，可对应 1 个或多个来源
- `synthesis`：围绕一个具体问题、比较、策略或判断进行综合分析，通常引用多个 summary / concept / entity

**`analysis_scope` 字段合法值（synthesis 专用）：**
- `question`：回答一个明确问题
- `comparison`：对比两个或多个对象
- `strategy`：给出可执行的策略或方案
- `judgment`：对某个主张作出判断和评估

---

## 五、`index.md` 更新规范

每次操作后，在对应分区下追加新页面，格式：

```markdown
- [[页面名]](相对路径) — 一句话描述
```

示例：
```markdown
- [[LLM-Wiki]](concepts/LLM-Wiki.md) — 由 LLM 持续维护的结构化知识库范式
```

---

## 六、`log.md` 追加规范

每次操作追加一条记录，禁止修改历史记录。格式：

```markdown
## [YYYY-MM-DD] 操作类型 | 标题
- 关键动作1
- 关键动作2
- 注意/矛盾（若有）
```

操作类型：`Init` / `Ingest` / `Query` / `Lint` / `Update` / `Demo`

---

## 七、行为准则

1. **不臆造**：只呈现来源中实际存在的内容，不补充未经来源支撑的推断
2. **保持链接**：每个页面至少引用 1 个来源，并与至少 1 个其他 wiki 页面建立链接
3. **增量更新**：摄入新资料时优先更新已有页面，而非总是新建
4. **标注矛盾**：发现矛盾时，在相关页面的 `lint_notes` 字段标注，不默默覆盖
5. **简洁优先**：概念页核心要点不超过 5 条，summary 页知识核心不超过 5 条
