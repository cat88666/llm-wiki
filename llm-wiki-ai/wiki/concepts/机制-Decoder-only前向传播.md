---
type: concept
subtype: 机制
status: active
name: "Decoder-only前向传播"
aliases: ["GPT前向传播", "自回归前向传播", "Decoder前向传播"]
related:
  - 机制-Self-Attention
  - 机制-多头注意力
  - 机制-KV-Cache
  - 机制-FlashAttention
  - 概念-Tokenizer
  - 体系-LLM体系
sources:
  - assets/LLM.jpg
  - raw/engineering/02-理论-Transformer.md
  - raw/engineering/03-理论-LLM核心机制.md
created: 2026-04-25
updated: 2026-04-25
---

# Decoder-only 前向传播

> 一句话定义：Decoder-only LLM 的前向传播，是把输入 token 序列经过 N 层"因果注意力 + 前馈网络"的堆叠，最终输出下一个 token 的概率分布，并将预测结果循环送回输入，实现自回归生成。

## 第一性原理

- 语言生成的根本问题是：**给定历史，预测下一个词的概率分布**。
- Decoder-only 的设计选择是：只用一种模块（因果 Self-Attention + FFN），反复堆叠 N 次，让深度取代宽度，用层数换取对序列的逐步抽象理解。
- 每一层看到的是"上一层对整个历史的理解结果"，而不是原始 token——这是深度堆叠有效的根本原因。

## 完整前向传播流程

以下流程对应博士老师手绘图（`assets/LLM.jpg`）从左到右的完整路径：

### Step 1：输入 Embedding

```
原始 token（A, B, C...）
    ↓ Token Embedding（词表维度 → d_model）
    ↓ + Positional Encoding（注入位置信息）
    ↓ 得到每个位置的初始向量表示
```

[[概念-Tokenizer]] 负责把文本切成 token，这是进入 Embedding 层之前的工作。

### Step 2：单个 Decoder Block 内部（重复 N 次）

一个 Decoder Block 包含两个子层，每个子层都有残差连接和 LayerNorm：

```
输入 x
    ↓
┌─────────────────────────────────────────────┐
│  子层1：因果多头自注意力                         │
│                                             │
│  对每个 Head h（共 H 个，并行执行）：            │
│    Q_h = x · W_Q_h                          │
│    K_h = x · W_K_h                          │
│    V_h = x · W_V_h                          │
│    head_h = softmax(Q_h · K_h^T / √d_k) · V_h │
│    （加 causal mask：只能看自己和之前的位置）    │
│                                             │
│  Concat(head_1, head_2, ..., head_H) · W_O  │
└─────────────────────────────────────────────┘
    ↓ Add（残差）& Norm（LayerNorm）
┌─────────────────────────────────────────────┐
│  子层2：Feed Forward Network                  │
│    FFN(x) = max(0, x·W_1 + b_1)·W_2 + b_2   │
│    （两层线性 + ReLU/GELU，维度先升后降）        │
└─────────────────────────────────────────────┘
    ↓ Add（残差）& Norm（LayerNorm）
    ↓ 输出传给下一个 Decoder Block
```

**图中的关键细节**：每个 Head 有**独立的** W_Q、W_K、W_V 投影矩阵，输出后 Concat 而非相加，再经过 W_O 混合。H=96 意味着并行学习 96 种不同的依赖关系。

### Step 3：N 层堆叠

图中标注 **96**（底部大括号）——整个 Decoder Block 重复叠加 96 次（对应 GPT-3 规模）。

```
输入 → [Decoder Block 1] → [Decoder Block 2] → ... → [Decoder Block 96] → 最终表示
```

每一层都是相同结构但**独立参数**，越深的层捕获越抽象的语言模式。

### Step 4：输出层

```
最后一个 Decoder Block 的输出（最后一个位置的向量）
    ↓ Linear（d_model → 词表大小 V）
    ↓ Softmax
    ↓ 概率分布 P(next token | context)
    ↓ 取概率最高（或采样）→ 预测 token
```

### Step 5：自回归循环

图左下角的回指箭头是关键——**预测出的 token 被追加到输入序列末尾**，作为下一步的输入，重新经过全部 96 层，预测再下一个 token，直到生成结束标记。

```
输入: [A, B, C]
→ 预测: D
→ 新输入: [A, B, C, D]
→ 预测: E
→ 新输入: [A, B, C, D, E]
→ ...
```

这个循环就是 **[[机制-KV-Cache]] 存在的根本原因**：每次推理只有最后一个 token 是新的，历史 token 的 K/V 已经算过，缓存后避免重复计算。

## 关键权衡

- 堆叠越深（96层）→ 表达能力越强，但参数量和计算量线性增长
- 自回归生成（每步一个token）→ 天然串行，吞吐受限，KV Cache 是关键优化手段
- 因果 mask（只看历史）→ 保证生成合法，代价是不能像 BERT 那样双向理解
- H=96 个 Head → 子空间丰富，但 KV Cache 随头数线性增大（GQA 是折中方案）

## 与其他概念的关系

- [[机制-Self-Attention]]：Decoder Block 中注意力子层的核心计算单元
- [[机制-多头注意力]]：H 个 Head 并行执行 Self-Attention，输出 Concat 后投影
- [[机制-KV-Cache]]：在 Step 5 自回归循环中，缓存每层历史 token 的 K/V 避免重算
- [[机制-FlashAttention]]：对 Step 2 中每个 Head 的注意力计算做 IO 优化，不改变结果
- [[概念-Tokenizer]]：Step 1 之前的工作，决定序列长度和 token 粒度
- [[主题-Transformer架构原理]]：包含 Pre-LN vs Post-LN、位置编码等更多架构细节

## 应用边界

- 适用：文本生成、对话、代码生成——所有"给历史，预测下一步"的任务
- 不适用：需要双向理解的任务（文本分类、NER）——应用 Encoder-only（BERT类）
- 规模效应：N 越大（层数）、H 越多（头数）、d_model 越大，能力越强，但推理成本随之上升

## 来源

- [手绘架构图](../../assets/LLM.jpg) — 博士老师手绘的 GPT-3 规模 Decoder-only 完整前向传播图（96层，96头）
- [02-理论-Transformer.md](../raw/engineering/02-理论-Transformer.md)
- [03-理论-LLM核心机制.md](../raw/engineering/03-理论-LLM核心机制.md)
