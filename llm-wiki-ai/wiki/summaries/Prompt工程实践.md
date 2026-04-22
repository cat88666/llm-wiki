---
type: summary
status: active
summary_scope: topic
title: "Prompt工程实践"
source_count: 1
sources:
  - ../raw/tech/原理/04-工程-Prompt.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# Prompt工程实践

> 这个页面总结的是 Prompt 工程的实用技术：参数调节策略、Few-shot 设计、上下文管理、安全防护分层，以及在实际工程中如何让 LLM 输出更可控。

## 知识核心
- **示例优于规则**：给几个 Few-shot 示例比大量文字约束更有效，LLM 更善于从示例中推断格式和风格，而不是解析复杂规则描述
- **temperature 参数分化明显**：分类/工具调用用 0-0.3（确定性输出），创作/头脑风暴用 0.7-1.0（随机多样），0.3-0.7 是通用对话区间；直接用默认值（通常 0.7）往往不是最优的
- **关键约束要放置在 Prompt 的首尾**：LLM 的注意力在序列首尾更强（始端 + 末尾效应），中间段内容容易被忽视，尤其在长 Prompt 中更明显
- **多轮对话必须主动管理历史**：超过 N 轮后用 LLM 压缩历史为摘要，保留最近 2-3 轮原文；不压缩会导致 context 窗口溢出，且早期信息被注意力稀释
- **安全防护需要分层纵深设计**：System Prompt 声明权限范围 → 输入检测过滤 → 输出脱敏/审查 → 权限隔离（代码层），任何单层防护都可被绕过

## 共识内容
- Chain-of-Thought（CoT）提示在推理类任务中显著提升准确率：让模型先"思考"再给答案，中间步骤不只是输出，也是模型内部的推理引导
- System Prompt 负责角色设定和行为约束，User Prompt 负责具体任务，两者分离是良好工程实践
- 结构化输出（要求 JSON/Markdown 格式）配合 Few-shot 示例效果远好于纯文字描述格式要求
- 自动化评估要配合人工评估：自动指标（BLEU/ROUGE/准确率）无法替代人对"有没有用"的判断

## 关键分歧
- **Few-shot 示例数量**：3-5 个示例是常见甜点区，但对于非常复杂的任务（如多步推理），示例过少反而会误导；示例质量远比数量重要
- **CoT vs 直接答案**：CoT 显著提升推理准确率，但增加 token 消耗（2-5x），生产系统需要权衡延迟与质量

## 适用边界
- 适用场景：LLM 任务设计、输出格式控制、多轮对话系统、RAG 的 Prompt 组装
- 不适用场景：纯粹的参数优化（fine-tuning 能学到的东西不必靠 Prompt 约束），极低延迟场景（CoT 会显著增加推理时间）

## 关键概念
- [[Prompt工程]]：Prompt 设计的系统方法，涵盖 temperature 调节、Few-shot、CoT、对话历史管理等
- [[RAG]]：RAG 系统的上下文组装本质是 Prompt 工程的一部分，文档如何放入 Prompt 直接影响效果
- [[Function-Calling]]：工具调用需要严格的 Prompt 格式，temperature 应设为 0-0.1

## 值得注意的点
- 注意力的"中间段遗失"现象：在包含大量文档的 RAG Prompt 中，放在中间的文档被引用概率显著低于首尾文档，这是"Lost in the Middle"现象的根本原因
- temperature=0 时并非完全确定性：贪婪解码仍受浮点计算精度影响，不同框架/硬件的输出可能略有差异
- 安全 Prompt 注入攻击无法靠 System Prompt 完全防住：攻击者可在 User 输入中嵌入"忽略之前所有指令"类型的攻击，必须配合输入检测

## 延伸方向
- 可拆分为 [[Prompt工程]] concept 页（temperature/top-p 等参数详解）
- 可延伸到 RAG Prompt 组装的最佳实践（连接到 [[RAG系统设计]] summary）
- 可连接到 [[LLM生产化与评估]] 的安全防护部分

## 来源
- [04-工程-Prompt.md](../raw/tech/原理/04-工程-Prompt.md)
