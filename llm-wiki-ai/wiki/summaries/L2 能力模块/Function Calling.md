---
type: atomic
layer: L2 能力模块
topic: function-calling
tags: [atomic, tool]
aliases: [工具调用]
status: evergreen
source: [raw/LLM工程/06-工程-Agent]
hub: [LLM体系, AI工程体系]
upstream: [Agent]
downstream: [Text2SQL]
related: []
---

# Function Calling

## 定义
Function Calling 是让模型输出结构化工具参数并由系统执行的机制。

## 解决的问题
- 让模型访问真实系统
- 将自然语言转为结构化调用
- 提高工具执行可控性

## 核心机制
- 工具描述
- 参数 schema
- 执行回填

## 关键判断
- Function Calling 是 Agent 的基础能力，但不等于 Agent 本身

## 上游
- [[Agent]]

## 下游
- [[Text2SQL]]

## 相关
## 相关
