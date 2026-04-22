---
type: application
layer: L3 工程系统
topic: text2sql
tags: [application, prompt, tool]
aliases: [ChatBI, SQL生成]
status: evergreen
source: [doc/00原理/04-工程-Prompt, doc/00原理/06-工程-Agent, doc/00原理/11-LLM应用, doc/00原理/12-LLM实战]
hub: [AI工程体系]
upstream: [Prompt工程, Function Calling, 特征工程]
downstream: []
related: [LLM应用]
---

# Text2SQL

## 定义
Text2SQL 是将自然语言问题转换为数据库查询语句并返回结果的应用形态。

## 解决的问题
- 非技术用户无法直接写 SQL
- 结构化数据查询门槛高
- 分析需求响应慢

## 核心机制
- [[Prompt工程]]
- [[Function Calling]]
- Schema 注入
- SQL 校验

## 关键判断
- Text2SQL 不是单纯生成 SQL，而是受控查询系统

## 上游
- [[Prompt工程]]
- [[Function Calling]]
- [[特征工程]]

## 相关
- [[LLM应用]]
