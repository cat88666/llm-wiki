# dx-LLM

基于 Karpathy LLM Wiki 思路构建的个人知识管理项目。  
每个 `llm-wiki-*` 目录是一套独立的知识库实例，统一遵循 `raw/ + wiki/ + CLAUDE.md` 三层结构，由 Claude Code 主动维护。

---

## 目录结构

```
dx-LLM/
├── llm-wiki-template/    标准母版（含规范、模板、演示样例）
├── llm-wiki-ai/          AI 大模型技术知识库（进行中）
├── llm-wiki-go/          Go 语言知识库（待建）
├── llm-wiki-java/        Java 知识库（待建）
├── llm-wiki-python/      Python 知识库（待建）
└── assets/               图片素材
```

---

## 核心架构

每个知识库实例内部结构：

```
llm-wiki-xxx/
├── raw/          原始资料（只读，永不修改）
├── wiki/
│   ├── index.md          所有页面目录
│   ├── log.md            操作历史（append-only）
│   ├── concepts/         概念页
│   ├── entities/         实体页（人物、工具、项目）
│   ├── summaries/        主题总结页（多源聚合，非逐篇摘要）
│   ├── synthesis/        综合分析页（问题驱动）
│   └── templates/        页面模板
└── CLAUDE.md     本实例的执行规范（Schema）
```

---

## 三条核心指令

```
请摄入 raw/xxx/文章.md
```
→ AI 读取原文，提取概念与实体，更新主题总结页，记录操作日志

```
[任意问题]
```
→ AI 检索 wiki，综合回答，有价值的回答自动回写到 synthesis/

```
请对 wiki 做一次健康检查
```
→ AI 扫描矛盾、孤立页、失效链接，生成报告并修复

---

## 使用流程

**新建知识库实例：**
1. 复制 `llm-wiki-template/` 为新目录（如 `llm-wiki-go/`）
2. 删除 `demo-*` 文件，保留模板文件和 CLAUDE.md
3. 将原始资料放入 `raw/`

**日常使用：**
1. 新文章放入 `raw/` 对应分类
2. 对话框输入摄入指令，Claude Code 自动处理
3. 直接提问即可查询，高价值回答自动沉淀到 wiki

**定期维护：**
- 执行 Lint 健康检查，保持知识库一致性

---

## 核心规则

- `raw/` 只读，AI 只读取，永不修改
- `summaries/` 按知识主题聚合，不按源文件逐篇生成
- `synthesis/` 用于跨主题的判断、对比、方案，不重复 summary
- 每次操作后必须更新 `index.md` 和 `log.md`
- 规范详见各实例的 `CLAUDE.md`，母版规范见 `llm-wiki-template/CLAUDE.md`

---

## 当前状态

| 实例 | 状态 | 源文件 | Wiki 页面 |
|------|------|--------|-----------|
| llm-wiki-ai | 进行中 | 12 篇 | 29 页（7 summary / 14 concept / 6 entity / 2 synthesis） |
| llm-wiki-go | 待建 | — | — |
| llm-wiki-java | 待建 | — | — |
| llm-wiki-python | 待建 | — | — |
