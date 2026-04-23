---
name: ingest
description: 将 `llm-wiki-ai/raw/` 下的原始资料编译到 `llm-wiki-ai/wiki/` 中。支持 `/ingest` 或 `/ingest <path>`。当用户提到“摄入”“导入”“纳入知识库”资料时，也应触发此技能。不得修改 `raw/` 中的原始文件。
user-invocable: true
---

# ingest 技能

## 核心工作流

你正在维护 `llm-wiki-ai/` 这个 LLM Wiki 实例。

**当前目录约定：**
- `llm-wiki-ai/raw/`：原始资料，只读
- `llm-wiki-ai/wiki/concepts/`：概念页
- `llm-wiki-ai/wiki/entities/`：实体页
- `llm-wiki-ai/wiki/summaries/`：主题总结页
- `llm-wiki-ai/wiki/synthesis/`：综合分析页
- `llm-wiki-ai/index.md`：总索引
- `llm-wiki-ai/log.md`：操作日志
- `llm-wiki-ai/CLAUDE.md`：实例规范

不要假设存在 `wiki/sources/`、`wiki/syntheses/`、`raw/09-archive/` 或 Inbox 工作流，除非仓库里实际存在并被用户明确要求使用。

## 触发逻辑

1. 用户执行 `/ingest`：扫描 `llm-wiki-ai/raw/` 下待处理文件
2. 用户执行 `/ingest <path>`：仅处理指定文件
3. 用户自然语言要求把资料加入知识库时，执行 ingest

## 编译流水线

### 步骤 1：读取源文件

- 优先读取用户指定的 `raw/` 文件
- 支持 Markdown 等文本资料
- 若遇到无法直接解析的文件类型，先向用户说明限制，不要伪造内容

### 步骤 2：提取知识元素

从源文件提取：
- 主题
- 概念
- 实体
- 观点
- 方法
- 事实
- 案例

如果资料属于已有主题簇，应优先更新已有页面，而不是机械新建。

### 步骤 3：更新主题页

根据 `llm-wiki-ai/CLAUDE.md`：

- `wiki/summaries/` 是按知识主题聚合的总结页，不是“一篇原文一篇摘要”
- 若资料属于已有主题，更新对应 summary
- 若形成新主题，再创建新的 summary

### 步骤 4：更新概念页和实体页

- 概念写入 `wiki/concepts/`
- 实体写入 `wiki/entities/`
- 页面不存在则新建，已存在则增量合并
- 若发现明显冲突，优先标注冲突来源和差异，不要静默覆盖
- 在创建任何新页面前，必须先调用 `naming` 规则确定文件名，禁止自由命名

其中 `wiki/concepts/` 必须优先写成“概念 / 机制 / 方法 / 抽象一线原理”页面，而不是术语堆砌、资料摘抄或目录导航。

概念页默认结构：

```markdown
# 概念名

> 一句话定义：说明这个概念在解决什么根问题

## 第一性原理
- 这个概念为什么会存在
- 它解决的底层矛盾是什么

## 核心机制
- 它是如何工作的
- 关键组成与运行逻辑是什么

## 关键权衡
- 带来什么收益
- 代价、边界和常见误区是什么

## 与其他概念的关系
- 与已有概念页建立链接

## 应用边界
- 适用场景与不适用场景
```

如果原始资料里有大量实现细节、参数经验或案例，这些内容应服务于解释原理，不应压过“第一性原理”主线。

### 步骤 5：必要时更新综合页

当资料带来跨主题比较、判断、策略或方案价值时，更新或创建 `wiki/synthesis/` 页面。

### 步骤 5.5：统一命名

所有新建页面在落盘前，必须先用统一命名规则生成文件名：

- `concepts/`：`概念-` / `机制-` / `方法-` / `框架-` / `体系-`
- `entities/`：`人物-` / `组织-` / `工具-` / `项目-` / `产品-` / `模型-`
- `summaries/`：`主题-`
- `synthesis/`：`问题-` / `对比-` / `策略-` / `判断-`

例如：

- `多头注意力.md` 不合格
- `机制-多头注意力.md` 合格

### 步骤 6：更新索引与日志

完成后必须：

- 更新 `llm-wiki-ai/index.md`
- 追加 `llm-wiki-ai/log.md`

日志格式遵循实例中的现有写法，例如：

```markdown
## [YYYY-MM-DD] Ingest | 标题
- 来源：`raw/...`
- 更新 summary / concept / entity / synthesis 页面
- 更新 `index.md`
```

## 强制约束

- `raw/` 只读，绝对不要移动、归档、改写源文件
- 不要生成 `wiki/sources/` 这类当前仓库不存在的目录
- `summary` 与 `synthesis` 必须区分清楚
- `concepts` 页面应优先采用“第一性原理 -> 核心机制 -> 关键权衡 -> 概念关系 -> 应用边界”结构
- 所有新建页面必须先经过统一命名规则，不得自由命名
- 每次 ingest 后都要同步更新 `index.md` 和 `log.md`
- 优先沿用现有页面命名与分层，不要另起一套新规范
