---
title: "摘要-22期课程 提示工程到RAG"
type: source
tags: [来源, 课程, 提示工程, RAG]
sources: [raw/course/01-提示工程到RAG.md]
last_updated: 2026-04-21
---

## 核心摘要

这份课程笔记从应用层视角概括了 Prompt 的作用边界：Prompt 是与大语言模型沟通的主接口，但单纯依赖提示词会受到长度和注意力稀释的限制，因此自然过渡到 RAG。

**关键要点：**
- Prompt 的常见组成包括身份、背景、参考资料、示例、指令、限制条件
- Zero-Shot 和 Few-Shot 是最基础的两种提示模式
- 提示词长度并非越长越好，内容过多会导致模型性能下降
- 当知识量超出单条 Prompt 承载能力时，需要引入检索式上下文补充

## 关联连接
- [[Prompt_Engineering]] — 提示工程基础
- [[Context_Engineering]] — 从单提示到系统上下文
- [[RAG]] — 通过检索补足上下文
- [[摘要-22期课程-rag检索增强]] — 后续 RAG 课程摘要
