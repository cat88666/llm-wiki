---
type: summary
status: active
summary_scope: topic
title: "LLM核心机制与推理优化"
source_count: 2
sources:
  - ../raw/tech/原理/03-理论-LLM核心机制.md
  - ../raw/tech/原理/10-LLM原理.md
ingested: 2026-04-22
updated: 2026-04-22
topics: []
lint_notes: ""
---

# LLM核心机制与推理优化

> 这个页面总结的是 LLM 在训练和推理阶段的核心工程机制：Tokenizer 设计、显存分配、分布式并行策略、KV Cache 优化，以及 GQA/MQA 的权衡。

## 知识核心
- **KV Cache 是推理的核心瓶颈**：随序列长度线性增长，对于 70B 模型，单序列 KV Cache 可达数 GB；[[vLLM]] 的 PagedAttention 借鉴操作系统虚拟内存思路，将显存利用率从约 60% 提升到 90%+
- **三种并行策略解决不同问题**：DP（数据并行）扩充吞吐量，TP（张量并行）切分单层矩阵，PP（流水线并行）切分层数；TP 通信量最大，**必须使用 NVLink 节点内通信才有意义**；PP 通信量最小但有 bubble 问题
- **ZeRO 和 3D 并行解决不同层次的问题**：ZeRO 消除数据并行中参数/梯度/优化器状态的冗余存储，3D 并行解决单张卡放不下一个层（TP）或放不下所有层（PP）的问题，两者正交互补
- **GQA 是 MHA 和 MQA 的工程最优折中**：MHA 每个 Head 有独立 K/V，KV Cache 大；MQA 所有 Head 共享 K/V，节省显存但表达能力损失明显；GQA 分组共享，LLaMA2/ChatGLM2 已将其作为标配
- **Tokenizer 选择影响全链路效率**：中文 BPE 容易过度切分（一个汉字可能占多个 token），直接影响上下文利用率、推理 token 成本和 OOV 处理能力

## 共识内容
- FlashAttention 不改变 Attention 的数学结果，本质是 IO 感知的实现优化：通过分块计算避免将完整 n×n Attention 矩阵写到 HBM，速度提升 2-4x，显存节省同等比例
- 显存分配有"三角平衡"：DP 靠堆卡，TP 靠高速互联，PP 靠流水调度；超大模型（百亿以上）需要 3D 并行同时使用三种策略
- BPE（Byte-Pair Encoding）是目前最主流的 Tokenizer 方案，通过合并频率最高的字符对迭代构建词表；WordPiece 用于 BERT，SentencePiece 实现语言无关

## 关键分歧
- **MQA vs GQA vs MHA**：MQA 推理最快但训练效果有损失，MHA 表达最强但 KV Cache 最大，GQA 是目前工业界共识的折中方案；具体分组数（如 8 组）是超参，需要实验确定
- **TP 节点内 vs 节点间**：TP 通信量是 All-Reduce，每层前向+反向各一次，跨节点带宽（100Gb/s InfiniBand）远低于节点内 NVLink（600GB/s），所以 TP 通常限于 8 卡单机，跨节点用 PP

## 适用边界
- 适用场景：大规模 LLM 训练基础设施设计、推理服务性能优化、多卡/多机部署方案选型
- 不适用场景：百亿参数以下的小模型通常单卡可以训练，不需要复杂并行策略；TP/PP 引入的工程复杂度在小模型上得不偿失

## 关键概念
- [[KV-Cache]]：推理时缓存历史 token 的 K/V 矩阵，是自回归生成可用的核心加速手段
- [[FlashAttention]]：IO 感知的注意力实现，通过减少 HBM 访问实现速度和显存双重优化
- [[分布式训练]]：DP/TP/PP/ZeRO 四种策略的组合，解决大模型无法在单卡上训练的问题
- [[Tokenizer]]：BPE/WordPiece/SentencePiece，决定文本到 token 的映射效率

## 值得注意的点
- PagedAttention 的核心思路：不预分配连续显存给 KV Cache，而是用类似页表的方式按需分配物理块，允许多请求的 KV Cache 交错存储，消除碎片化，这是 vLLM 高吞吐的核心机制
- GQA 的"分组数"设计：如果分组数等于 head 数，退化为 MHA；如果分组数为 1，退化为 MQA；分组数为 4-8 是实践中常见的平衡点
- 中文模型的 Tokenizer 效率问题：GPT 系列词表对中文覆盖不足，一个中文词可能需要 3-4 个 token，而专门为中文优化的模型（如 ChatGLM）可以将中文 token 效率提升 2-3 倍

## 延伸方向
- 可拆分为 [[vLLM]] 实体页（PagedAttention 详细机制、部署配置）
- 可延伸到 [[分布式训练]] concept 页（ZeRO-1/2/3 的差异）
- 可连接到 [[LLM生产化与评估]] summary 页（推理延迟优化与成本控制）

## 来源
- [03-理论-LLM核心机制.md](../raw/tech/原理/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../raw/tech/原理/10-LLM原理.md)
