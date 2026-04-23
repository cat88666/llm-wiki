---
type: concept
status: active
name: "Tokenizer"
aliases: ["分词器", "BPE", "Byte-Pair Encoding", "WordPiece", "SentencePiece", "词表"]
related: ["Self-Attention", "KV-Cache", "Prompt工程", "分布式训练"]
sources:
  - ../raw/llm-engineering/03-理论-LLM核心机制.md
  - ../raw/llm-engineering/10-LLM原理.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# Tokenizer

> 一句话定义：Tokenizer 将原始文本映射为 token 序列（整数 ID），决定模型能看到多少文本信息（上下文利用率）、推理成本（token 数量）和对罕见词的处理能力（OOV）。

## 核心要点
- **Tokenizer 决定上下文利用率**：同样的文本在不同 Tokenizer 下产生不同长度的 token 序列，效率低的 Tokenizer（如对中文支持差的 GPT-2 词表）会"浪费"上下文窗口
- **BPE（Byte-Pair Encoding）**：从单字节出发，迭代合并频率最高的相邻 token 对，构建词表；训练语料的语言分布决定词表的语言覆盖效率；GPT 系列使用 BPE，词表约 50K-100K
- **WordPiece**：BERT 使用，类似 BPE 但合并判断标准不同（最大化训练数据似然而非频率）；对子词（subword）的处理类似 BPE，低频词被拆成更细的片段
- **SentencePiece**：与语言无关的实现（不依赖预分词），支持 BPE 和 Unigram 两种算法；LLaMA、T5 等模型使用，对多语言场景更友好
- **中文 Tokenizer 效率问题**：GPT 系列词表对中文覆盖不足，一个汉字或词可能占 2-4 个 token；专门优化的中文词表（如 ChatGLM、Qwen）可将中文 token 效率提升 2-3 倍，直接降低推理成本

## 详细说明

BPE 的训练过程：初始词表为所有字节（256 个），对训练语料统计相邻 token 对的频率，合并频率最高的 pair 为新 token（如 "t" + "h" → "th"），重复 N 次得到目标词表大小（通常 32K-100K）。推理时对新文本贪婪地应用学到的合并规则。BPE 保证了任何文本都可以被分词（通过 fallback 到字节级别），没有 OOV（Out-of-Vocabulary）问题。

Tokenizer 对模型行为的影响比直觉上更深远：同一个 Tokenizer 训练出来的模型对 token 边界有"感知"，比如数字的加法如果每位数字是独立 token，模型会分别处理每位，这与人类的数学直觉不同。最近的研究（如 Llama 3 的 128K 词表）发现更大词表可以提升数学和代码任务的性能，因为数字和符号有更完整的 token 表示。

Tokenizer 的特殊 token：`<eos>`（序列结束）、`<pad>`（填充）、`<unk>`（未知，BPE 理论上不需要但保留）、`<|system|>` 等指令模板 token，这些 token 在训练时有特殊含义，推理时不应该出现在用户输入中（否则可能干扰模型行为）。

## 与其他概念的关系
- 是 [[Self-Attention]] 的前置步骤：Tokenizer 的输出（token IDs）经过 Embedding 层转化为向量，才进入 Transformer 的注意力计算
- 影响 [[KV-Cache]] 大小：token 数量越多，KV Cache 越大；高效 Tokenizer 直接降低推理成本和 KV Cache 压力
- 关联 [[Prompt工程]]：Prompt 中的 token 数量直接影响推理成本，了解 Tokenizer 才能优化 Prompt 的 token 效率

## 应用场景
- 中文 LLM 部署：选择对中文有良好覆盖的词表（ChatGLM/Qwen 的 Tokenizer 优于 GPT-2 词表）
- Token 成本控制：在高频调用场景，压缩 Prompt 的 token 数量（避免重复、冗余描述）直接影响成本
- 多语言模型：SentencePiece + 多语言语料训练词表，保证跨语言的 token 效率相对均衡
- 代码模型：代码的 Tokenizer 需要对缩进、括号等符号有精细分词，GPT-4 的 tiktoken 对代码效率做了专门优化

## 来源
- [03-理论-LLM核心机制.md](../raw/llm-engineering/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../raw/llm-engineering/10-LLM原理.md)
