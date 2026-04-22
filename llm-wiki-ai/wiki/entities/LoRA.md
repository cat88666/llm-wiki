---
type: entity
status: active
entity_type: technique
name: "LoRA"
aliases: ["Low-Rank Adaptation", "低秩适配", "QLoRA", "LoRA微调"]
domain: "LLM微调 / 参数高效微调"
related: ["正则化", "梯度下降与优化", "分布式训练"]
sources:
  - ../raw/LLM工程/07-工程-生产化.md
  - ../raw/LLM工程/11-LLM工程.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# LoRA

> LoRA（Low-Rank Adaptation）是一种参数高效微调技术，通过将权重更新矩阵分解为两个低秩矩阵（ΔW = A·B，A 和 B 远小于原矩阵），在保持预训练参数冻结的前提下用极少的可训练参数完成任务适配，是目前最主流的 LLM 微调方案。

## 基本信息
- **类型**：参数高效微调（PEFT）技术
- **领域**：LLM 微调、领域适配、风格定制
- **关联时间**：2021 年 Hu et al. 提出，2023 年后随 LLaMA 开源在社区广泛应用

## 核心特点
- **低秩分解**：对目标权重矩阵 W（d×k），不直接更新 W，而是添加 ΔW = A·B（d×r 和 r×k，r << min(d,k)）；rank r 通常取 4-64，可训练参数量仅为原矩阵的 r/min(d,k)，如 r=8 对 d=k=4096 的矩阵只有原参数量的 0.4%
- **原参数冻结**：预训练权重 W 完全冻结不更新，只有 A/B 矩阵参与梯度计算；推理时 ΔW = A·B 与 W 合并（W' = W + αΔW），无额外推理延迟
- **隐式正则化**：低秩约束限制了参数更新的表达空间（只能在秩 r 的子空间内更新），防止小数据集微调过拟合，是一种结构化正则化
- **QLoRA（Quantized LoRA）**：将基座模型量化为 4bit（NF4 量化），LoRA 适配器保持 bf16；单张 80GB A100 可微调 65B 参数模型，将大模型微调的硬件门槛从多机多卡降低到单卡
- **适配层选择**：通常对 Attention 层的 Q/K/V/O 投影矩阵应用 LoRA，有时也对 FFN 层应用；不同层对微调效果的贡献不同，需要实验

## 在知识库中的出现
- [[LLM生产化与评估]] summary：LoRA 微调与 RAG 是 LLM 能力增强的两种互补路径，LoRA 适合需要风格/格式定制的场景
- [[正则化]] concept：LoRA 的低秩约束是隐式正则化，与 L2 正则的作用类似（限制参数更新自由度）
- [[RAG-vs-微调-vs-Agent 对比]] synthesis：LoRA 微调是"注入特定知识或行为"的技术选型之一

## 相关实体
- QLoRA：LoRA 的量化版本，进一步降低显存要求，使消费级 GPU 也能微调中型模型
- Prefix Tuning：另一种 PEFT 方法，在输入前添加可训练的"前缀 token"，效果通常弱于 LoRA
- Adapter：早期 PEFT 方法，在 Transformer 层间插入小型 MLP 模块，推理时有额外延迟（LoRA 合并后无延迟）
- [[vLLM]]：LoRA 微调后的模型可以用 vLLM 部署服务，vLLM 支持 LoRA 适配器的动态加载

## 来源
- [07-工程-生产化.md](../raw/LLM工程/07-工程-生产化.md)
- [11-LLM工程.md](../raw/LLM工程/11-LLM工程.md)
