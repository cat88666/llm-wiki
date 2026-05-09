---
type: concept
status: active
name: "LoRA"
aliases: ["Low-Rank Adaptation", "低秩适配", "QLoRA", "PEFT", "参数高效微调"]
related:
  - 方法-分布式训练
  - 概念-RAG
  - 对比-RAG-vs-微调-vs-Agent
  - 机制-KV-Cache
sources:
  - raw/engineering/07-工程-生产化.md
  - raw/engineering/11-LLM工程.md
  - raw/papers/大模型微调指南.md
created: 2026-04-22
updated: 2026-04-25
lint_notes: ""
---

# LoRA

> 一句话定义：LoRA 通过在预训练权重旁并联两个可训练的低秩小矩阵（ΔW = A·B），以原参数量不到 1% 的代价完成任务适配，是显存-效果最优的 LLM 微调主流方案。

## 第一性原理

- 全参数微调的根本矛盾：7B+ 参数模型全量更新需要数十 GB 显存，但下游任务的有效信息量远比预训练数据少，导致"大炮打蚊子"——浪费资源且容易过拟合。
- 核心洞察：权重的**更新矩阵 ΔW 在任务适配时是低秩的**——大模型已经具备通用能力，适配只需在低维子空间内微调，不需要改变整个权重空间。
- 因此 LoRA 的选择是：冻结原权重 W，只在旁边并联两个小矩阵 A（d×r）和 B（r×k），用它们的乘积 A·B 近似完整更新，r 远小于 d 和 k。

## 核心机制

**数学形式：**
```
推理时：W' = W + α · (A · B)
训练时：只有 A、B 参与梯度计算，W 冻结
```
- `r`（rank）：低秩子空间大小，典型取值 4-64；r=8 对 4096×4096 矩阵只产生 0.4% 的可训练参数
- `lora_alpha`：缩放因子，实际缩放比例为 α/r；增大 alpha 相当于放大 LoRA 对原权重的影响权重
- `target_modules`：通常对 Attention 的 Q/K/V/O 投影 + FFN 的 gate/up/down 层应用 LoRA；只选 Q/V 可进一步减少参数
- 推理无延迟：训练完后将 α·A·B 直接合并进 W，推理路径与原模型完全相同

**主要变体对比：**

| 方法 | 核心思路 | 显存需求（7B） | 适用场景 |
|------|---------|-------------|---------|
| LoRA | 低秩矩阵旁路，FP16 加载 | ~15 GB | 中等 GPU，主流选择 |
| QLoRA | LoRA + 基座模型 4-bit NF4 量化 | ~8-11 GB | 单卡 16GB 跑 7B，48GB 跑 65B |
| P-Tuning | 只训练输入端的虚拟 token embedding | 极少 | 简单任务快速尝试，效果有限 |
| 全参数微调 | 所有参数更新 | 30 GB+ | 数据极丰富 + 计算资源充足 |

**QLoRA 关键技术：**
- 基座模型以 4-bit NF4（Norm Float 4）精度加载，大幅压缩显存
- LoRA 适配器本身保持 bf16 精度，保证训练稳定性
- 依赖 `bitsandbytes` 库；需调用 `prepare_model_for_int8_training` 解冻 LayerNorm 等层

## 关键权衡

- **r 的选择**：r=4-8 适合格式/风格定制（简单任务）；r=16-64 适合复杂领域适配（医疗/法律）；r 过大接近全参数微调，失去 LoRA 意义
- **target_modules 范围**：仅 Q/V 参数最少效果够用；Q/K/V/O + FFN 效果更强但显存更高；显存紧张时先精简 target_modules
- **LoRA vs P-Tuning**：P-Tuning 只改输入不改权重，复杂任务效果明显弱于 LoRA，除非显存极度受限
- **合并 vs 不合并权重**：开发测试阶段用 `AutoPeftModelForCausalLM` 加载适配器（灵活切换）；生产部署用 `merge_and_unload()` 合并（摆脱 PEFT 依赖，但文件变大）
- **数据量边界**：< 1000 条优先考虑 Few-Shot Prompt；1000-10 万条 LoRA 最合适；数据极多才值得全参数

## 与其他概念的关系

- [[对比-RAG-vs-微调-vs-Agent]]：LoRA 是三条增强路径之一，解决"模型说话方式不对"的问题，而非注入知识
- [[主题-大模型微调工程实践]]：汇总 LoRA/QLoRA/P-Tuning 的工程流水线、参数选择和常见排障。
- [[方法-分布式训练]]：QLoRA 让单卡可微调原本需要多卡的大模型，从根本上改变了微调的硬件门槛
- [[机制-KV-Cache]]：LoRA 合并后的模型与原始模型推理路径相同，KV Cache 策略不受影响
- [[概念-RAG]]：两者互补，LoRA 改善"说话方式"，RAG 保证"知识准确"；组合使用是大模型落地常见方案

## 应用边界

- **适合**：输出格式规范化（特定 JSON schema）、风格定制（品牌语气）、领域术语理解、角色扮演
- **不适合**：注入最新知识（知识截止问题用 RAG 更合适）、数据极少（< 1000 条，容易过拟合）、知识更新频繁（重新训练成本高）
- **硬件要求**：LoRA 需要 16GB+ 显存跑 7B；QLoRA 可将需求压缩到 10-12GB；全参数 7B 需要 30GB+

## 来源

- [07-工程-生产化.md](../../raw/engineering/07-工程-生产化.md)
- [11-LLM工程.md](../../raw/engineering/11-LLM工程.md)
- [大模型微调指南.md](../../raw/papers/大模型微调指南.md)
