---
type: concept
status: active
name: "FlashAttention"
aliases: ["Flash Attention", "IO-aware Attention", "FlashAttention-2", "FlashAttention-3"]
related: ["Self-Attention", "多头注意力", "KV-Cache", "分布式训练"]
sources:
  - ../raw/llm-engineering/03-理论-LLM核心机制.md
  - ../raw/llm-engineering/09-Transformer.md
  - ../raw/llm-engineering/10-LLM原理.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# FlashAttention

> 一句话定义：FlashAttention 是一种 IO 感知的注意力实现，通过分块计算和在线 softmax 避免将完整的 n×n 注意力矩阵写入 HBM，在数学上等价于标准 Self-Attention 但速度快 2-4x、显存节省显著。

## 核心要点
- **本质是访存优化，不是数学改进**：FlashAttention 的计算结果与标准 Self-Attention 完全相同，改变的是计算顺序和中间结果的存储位置
- **GPU 内存层次**：HBM（High Bandwidth Memory，显存，容量大但带宽相对低）vs SRAM（片上共享内存，容量小但极高速）；标准 Attention 需要将 n×n 矩阵写到 HBM，FlashAttention 的中间结果保留在 SRAM
- **速度提升来源于减少 HBM 读写**：标准 Attention 需要 O(n²) 次 HBM 读写（写出注意力矩阵、读入做 softmax、再读入加权 V）；FlashAttention 通过分块（Tiling）将 HBM 读写降为 O(n²/M)，其中 M 是 SRAM 大小
- **在线 softmax（Online Softmax）**：softmax 需要全行数据才能归一化，FlashAttention 用增量更新技巧：每看到新的一块数据，更新当前最大值和累计分母，实现分块计算 softmax 而不需要全量数据
- **反向传播的重计算策略**：FlashAttention 不保存完整注意力矩阵（节省 O(n²) 显存），反向传播时重新计算注意力权重（用已保存的 Q/K/V 块重算），以额外计算换显存

## 详细说明

标准 Self-Attention 的 IO 瓶颈分析：计算 QK^T（O(n²d) FLOP）→ 写到 HBM（O(n²) 读写）→ 读回做 softmax（O(n²) 读写）→ 再写到 HBM → 读回乘 V（O(n²) 读写）。对于 n=4096, d=128 的配置，中间矩阵约 128MB，读写开销超过实际计算时间，Attention 成为 memory-bound 操作。

FlashAttention 将 Q/K/V 划分为块（Block），每次将一块 Q、一块 K、一块 V 加载到 SRAM（约 20MB），在 SRAM 内完成局部注意力计算，用在线 softmax 累积全局结果，最后只将 O（输出）写到 HBM。整个过程 HBM 读写量从 O(n²) 降到 O(nd + n)，瓶颈从 IO 变为计算，GPU 利用率大幅提升。

FlashAttention-2 相比 FlashAttention-1 的改进：减少非矩阵乘法操作（non-GEMM ops），更好的工作分配（减少 warp 间通信），支持 causal mask 的优化（跳过下三角无效块的计算）。FlashAttention-3 进一步针对 Hopper（H100）架构优化，利用 wgmma 指令和异步流水线。

## 与其他概念的关系
- 直接优化 [[Self-Attention]]：FlashAttention 是 Self-Attention 的等价但更高效的实现，可以直接替换所有使用标准 Attention 的场景
- 关联 [[KV-Cache]]：两者都是推理优化，方向正交——FlashAttention 优化单次 Attention 计算的 IO，KV Cache 避免历史 token 的重复计算
- 关联 [[多头注意力]]：FlashAttention 对每个 Head 独立分块计算，可以与 GQA/MQA 结合使用
- 关联 [[分布式训练]]：Ring-Flash-Attention 将 FlashAttention 扩展到序列并行（Sequence Parallelism），支持超长序列的分布式训练

## 应用场景
- 所有 Transformer 训练和推理场景：PyTorch 2.0+ 的 scaled_dot_product_attention 默认启用 FlashAttention（如果支持的话）
- 长上下文场景（8K-128K token）：标准 Attention 的显存是 O(n²)，FlashAttention 将中间结果保持在 SRAM，使长上下文成为可能
- 训练加速：特别是在序列较长时效果更明显，短序列（< 512 token）FlashAttention 的优势相对小

## 来源
- [03-理论-LLM核心机制.md](../raw/llm-engineering/03-理论-LLM核心机制.md)
- [09-Transformer.md](../raw/llm-engineering/09-Transformer.md)
- [10-LLM原理.md](../raw/llm-engineering/10-LLM原理.md)
