---
type: atomic
layer: L2 能力模块
topic: prompt
tags: [atomic, prompt]
aliases: [提示工程]
status: evergreen
source: [raw/llm-engineering/04-工程-Prompt, raw/course/01-提示工程到RAG]
hub: [LLM体系, AI工程体系]
upstream: [LLM机制, 微调与对齐]
downstream: [RAG, Text2SQL]
related: []
---

# Prompt工程

## 定义
Prompt 工程是通过设计输入结构、约束和示例控制模型输出的实践。

## 解决的问题
- 输出不稳定
- 格式不一致
- 回答偏航

## 核心机制
- 角色定义
- 约束说明
- Few-shot 示例
- 输出结构约束

## 关键判断
- 能用 Prompt 解决的行为问题，不要过早升级到微调

## 上游
- [[LLM机制]]
- [[微调与对齐]]

## 下游
- [[RAG]]
- [[Text2SQL]]

## 相关
