# Transformer 模型详解

这是一份按知识结构重写后的 Transformer 原理总结，覆盖原文中的核心内容：

- Transformer 为什么出现
- Encoder-Decoder 整体结构
- Embedding 与张量流动方式
- Self-Attention 的直觉、公式与矩阵实现
- Multi-Head Attention
- Position-wise Feed Forward Network
- 残差连接与 LayerNorm
- 位置编码
- Decoder 的生成过程
- Padding Mask 与 Sequence Mask

目标不是保留 PDF 转写痕迹，而是把 Transformer 的关键原理整理成适合学习、复习和面试表达的版本。

## 一、Transformer 为什么重要

2017 年，Google 在论文 `Attention Is All You Need` 中提出了 Transformer。它最大的突破在于：

- 用 Self-Attention 替代了 RNN 的递归结构
- 显著提升并行计算能力
- 更容易建模长距离依赖
- 为后来的 BERT、GPT、T5、LLaMA 等模型奠定了基础

如果用一句话概括 Transformer：

它是一种基于注意力机制的序列建模架构，核心思想是让每个位置都能直接“看到”序列中的其他位置。

## 二、Transformer 的整体结构

### 1. 黑盒视角

最初的 Transformer 用于机器翻译，输入是一个源语言句子，输出是目标语言句子。

从黑盒角度看，它就是：

- 输入一个序列
- 编码上下文信息
- 逐步生成另一个序列

### 2. Encoder-Decoder 架构

Transformer 本质上是一个 Encoder-Decoder 架构。

#### 2.1 Encoder

编码器负责：

- 读取输入序列
- 将每个 token 编码成带上下文信息的表示

编码器通常由多层相同结构堆叠而成。原始论文中使用了 6 层。

每个 Encoder Layer 包含两个子层：

- Multi-Head Self-Attention
- Position-wise Feed Forward Network

#### 2.2 Decoder

解码器负责：

- 基于已经生成的内容
- 结合编码器输出
- 逐步生成下一个 token

原始论文中 Decoder 也是 6 层堆叠。

每个 Decoder Layer 包含三个子层：

- Masked Multi-Head Self-Attention
- Encoder-Decoder Attention
- Position-wise Feed Forward Network

### 3. 为什么 Encoder 和 Decoder 都要堆叠多层

因为不同层会逐步提取不同层次的信息：

- 底层更偏局部模式和词法关系
- 中间层更偏语法结构
- 高层更偏语义与任务相关信息

## 三、Embedding 与输入表示

### 1. 词嵌入

和大多数 NLP 模型一样，Transformer 首先把离散 token 转换成连续向量。

原始论文中：

- `d_model = 512`

也就是说，每个 token 被表示为一个 512 维向量。

### 2. 输入矩阵

如果一句话有 `n` 个 token，那么输入可以写成一个矩阵：

- 每一行对应一个 token
- 每一列对应一个 embedding 维度

因此输入张量通常可以看成：

- `X ∈ R^(n × d_model)`

### 3. 为什么后续层维度要保持一致

Transformer 中：

- Embedding 输出维度
- Attention 输出维度
- FFN 输入输出维度
- 残差连接两端维度

通常都围绕同一个 `d_model` 设计。

原因很直接：

- 便于残差连接
- 便于层与层堆叠

## 四、Self-Attention 的直觉理解

### 1. 它解决了什么问题

RNN 在处理序列时，当前位置的信息必须通过多步传递才能影响远处位置。

Transformer 的 Self-Attention 不一样：

- 每个位置可以直接和所有位置建立联系

这使得模型在编码某个词时，不只依赖自己，而是可以参考整个句子的其他词。

### 2. 经典例子

例如句子：

`The animal didn't cross the street because it was too tired.`

这里的 `it` 指的是 `animal`，而不是 `street`。

Self-Attention 的作用就是让模型在处理 `it` 时，自动把注意力分配给更相关的词，比如 `animal` 和 `tired`。

### 3. 一句话理解 Self-Attention

Self-Attention 的本质是：

- 当前 token 用自己的查询向量
- 去和全序列的键向量做匹配
- 再对值向量做加权汇总

## 五、Q、K、V 是什么

### 1. 从输入向量得到 Q、K、V

对于输入矩阵 `X`，Transformer 会通过三组可训练参数得到：

- `Q = XW_Q`
- `K = XW_K`
- `V = XW_V`

其中：

- `W_Q`
- `W_K`
- `W_V`

都是训练出来的线性变换矩阵。

### 2. Q、K、V 的直觉含义

- Query：我当前想找什么信息
- Key：我这里能提供什么信息
- Value：我真正携带的内容是什么

这三个概念是注意力机制最关键的抽象。

### 3. 为什么需要三组矩阵

因为注意力不只是“相似度计算”，还包含两件事：

- 计算谁和谁相关
- 决定聚合什么内容

其中：

- Q 和 K 用来算相关性
- V 用来承载最终被加权聚合的信息

## 六、Scaled Dot-Product Attention

### 1. 标准公式

缩放点积注意力公式是：

`Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V`

这个公式是 Transformer 的核心。

### 2. 每一步在做什么

#### 2.1 `QK^T`

计算每个 query 和所有 key 的相似度分数。

结果是一个注意力分数矩阵。

#### 2.2 `/ sqrt(d_k)`

做缩放。

原因：

- 如果维度较大，点积值会变大
- softmax 会变得过于尖锐
- 梯度会不稳定

因此要除以 `sqrt(d_k)`。

#### 2.3 `softmax`

把分数归一化成概率分布。

含义：

- 当前 token 对每个位置分别分配多少注意力

#### 2.4 再乘 `V`

根据注意力权重对 Value 做加权求和，得到当前位置新的表示。

### 3. 逐 token 的直觉流程

以某个 token 为例：

1. 用它的 Query 去看整句的 Key
2. 计算和每个词的相关性
3. softmax 后得到一组权重
4. 用这组权重加权所有 Value
5. 得到当前 token 的新表示

## 七、Self-Attention 的向量级计算过程

原始材料里重点讲了这个过程，可以整理为 6 步。

### 1. 为每个输入 token 生成 Q、K、V

每个输入 embedding 经过三组线性变换，得到：

- Query 向量
- Key 向量
- Value 向量

在原始论文常见设定中：

- `d_model = 512`
- 单头的 `d_k = d_v = 64`

### 2. 计算注意力分数

对当前 token，用它的 Query 和所有 Key 做点积，得到一组分数。

### 3. 缩放

把每个分数除以 `sqrt(d_k)`。

### 4. Softmax

把分数变成概率分布，所有权重和为 1。

### 5. 权重乘 Value

每个权重乘对应位置的 Value 向量。

### 6. 加权求和

把这些向量加起来，得到当前 token 的最终 attention 输出。

## 八、Self-Attention 的矩阵实现

实际实现不会逐词循环算，而是用矩阵并行。

### 1. 输入矩阵

设：

- `X ∈ R^(n × d_model)`

通过线性映射得到：

- `Q ∈ R^(n × d_k)`
- `K ∈ R^(n × d_k)`
- `V ∈ R^(n × d_v)`

### 2. 分数矩阵

`QK^T` 的结果形状是：

- `n × n`

表示：

- 每个 token 对所有 token 的注意力分数

### 3. 并行优势

因为所有 token 都能同时计算：

- 不需要像 RNN 那样按时间步递归
- 可以充分利用 GPU 并行能力

这正是 Transformer 高效的关键原因之一。

## 九、多头注意力 Multi-Head Attention

### 1. 为什么要多头

如果只做一次注意力，模型只能在单一表示空间里建模相关性。

多头注意力的想法是：

- 把 Q、K、V 映射到多个不同子空间
- 每个子空间各算一遍 attention
- 再把结果拼接起来

这样做的好处是：

- 不同头可以关注不同关系
- 提升表示能力

### 2. 结构流程

多头注意力大致过程：

1. 对输入做多组线性变换
2. 每组分别计算 attention
3. 把各头结果拼接
4. 再经过一次线性变换得到最终输出

### 3. 数学形式

如果有 `h` 个头，则：

- 每个头单独计算 `head_i = Attention(Q_i, K_i, V_i)`
- 然后 `Concat(head_1, ..., head_h)`
- 最后乘一个输出矩阵 `W_O`

### 4. 为什么参数量不一定明显增加

因为多头注意力通常会把总维度拆开。

例如：

- 总维度仍是 512
- 8 个头时，每个头只负责 64 维

因此本质上是：

- 在不同子空间中建模
- 再做融合

### 5. 多头注意力的直觉理解

可以把它理解成：

- 一个头关注指代关系
- 一个头关注主谓关系
- 一个头关注修饰关系
- 一个头关注远距离依赖

最后把这些不同视角汇总起来。

### 6. 为什么多头有助于效果

因为序列中的关联并不是单一类型的。

多头注意力允许模型同时捕获：

- 词法关系
- 句法关系
- 语义关系
- 远距离依赖

## 十、Position-wise Feed Forward Network

### 1. FFN 是什么

在每个 Encoder / Decoder Layer 中，Attention 后面都会接一个前馈网络。

这个网络对每个位置独立地做同样的非线性变换。

### 2. 结构形式

原始 Transformer 中通常写作：

`FFN(x) = max(0, xW_1 + b_1) W_2 + b_2`

也就是：

- 两层线性变换
- 中间一个激活函数

原始论文里常用 ReLU。

### 3. 为什么需要 FFN

Attention 的作用更偏向：

- 信息交互
- 上下文融合

FFN 的作用更偏向：

- 特征变换
- 非线性表达
- 提升每个位置的表示能力

## 十一、残差连接与 LayerNorm

### 1. 为什么需要残差连接

深层网络训练容易出现：

- 梯度传播困难
- 表示退化

残差连接的作用是：

- 保留原始信息通路
- 让优化更容易
- 提高深层训练稳定性

### 2. 为什么需要 LayerNorm

LayerNorm 用于：

- 稳定数值分布
- 加快训练收敛
- 提高训练稳定性

### 3. Transformer 中的典型结构

原始 Transformer 子层通常写成：

`LayerNorm(x + Sublayer(x))`

也就是说：

1. 子层先计算输出
2. 与输入做残差相加
3. 再做 LayerNorm

### 4. 为什么所有子层输出维度要一致

因为残差连接要求：

- 输入和输出形状一致

因此：

- Attention 输出维度
- FFN 输入输出接口维度
- Embedding 维度

都要和 `d_model` 兼容。

## 十二、位置编码 Positional Encoding

### 1. 为什么需要位置编码

Self-Attention 本身不包含顺序信息。

换句话说：

- 它知道有哪些 token
- 但如果不额外注入位置信息，它并不知道这些 token 的前后顺序

因此必须加入位置编码。

### 2. 基本做法

位置编码向量和 token embedding 逐元素相加。

得到的新输入表示同时包含：

- 词语语义
- 位置信息

### 3. 原始论文的正弦余弦编码

Transformer 使用固定的正弦余弦函数生成位置编码：

- 偶数维用 `sin`
- 奇数维用 `cos`

特点：

- 不需要额外学习
- 对不同位置有唯一模式
- 具有一定相对位置表达能力

### 4. 这种设计的好处

- 可以泛化到训练中没见过的更长序列
- 不同频率帮助模型表达不同尺度的位置关系

### 5. 一句话理解

位置编码是给每个 token 附加一个“我在第几位”的信号。

## 十三、Decoder 的工作过程

### 1. Decoder 输入是什么

在训练时，Decoder 输入的是：

- 已知目标序列的前缀

在推理时，Decoder 输入的是：

- 已经生成出来的 token

### 2. Decoder 为什么比 Encoder 多一个注意力层

因为 Decoder 不仅要看自己已经生成的内容，还要参考 Encoder 提供的源序列信息。

因此 Decoder Layer 有两类 attention：

- 自注意力：看目标端历史
- Encoder-Decoder Attention：看源端表示

### 3. Encoder-Decoder Attention 是怎么工作的

它与普通 attention 非常类似，不同点在于 Q/K/V 来源不同：

- Query 来自 Decoder 当前层输入
- Key 和 Value 来自 Encoder 输出

作用：

- 在生成目标 token 时，让 Decoder 能对齐并利用源序列相关部分

## 十四、Mask 机制

Transformer 中原文重点提到了两种 mask。

### 1. Padding Mask

当一个 batch 中句子长度不一致时，较短句子通常会补 pad。

问题是：

- pad 位置没有真实语义
- 不应该参与注意力计算

Padding Mask 的作用就是：

- 屏蔽这些无意义位置

它在 Encoder 和 Decoder 中都会用到。

### 2. Sequence Mask

Sequence Mask 也叫：

- Causal Mask
- Look-Ahead Mask

作用：

- 防止 Decoder 在生成当前位置时看到未来 token

例如在预测第 `t` 个位置时，只允许看到：

- `1...t`

不能看到：

- `t+1...n`

### 3. 为什么 Decoder 必须有 Sequence Mask

因为 Decoder 的目标是自回归生成。

如果训练时能直接看到未来词：

- 就破坏了生成任务的因果约束
- 推理时和训练时就不一致

### 4. 两种 Mask 的区别

Padding Mask：

- 屏蔽无效补齐位置

Sequence Mask：

- 屏蔽未来信息

二者可以同时存在。

## 十五、Transformer 的解码流程

### 1. 自回归生成

Transformer Decoder 一般按下面流程生成：

1. 输入起始符
2. 生成第一个 token
3. 把它接回输入
4. 再生成下一个 token
5. 重复直到生成结束符

### 2. 为什么是逐步生成

因为当前位置的输出依赖前面已生成内容。

所以虽然训练时可以并行算整段目标序列，但推理时通常仍然要自回归逐步生成。

### 3. 训练和推理的差异

训练：

- 已知完整目标序列
- 可并行计算

推理：

- 目标序列未知
- 只能一个 token 一个 token 地生成

## 十六、Transformer 相比 RNN 的优势

### 1. 并行能力更强

RNN 天然按时间步递归，难以并行。

Transformer 在训练时可以对整段序列并行计算。

### 2. 长距离依赖更容易建模

RNN 需要多步传递信息。

Transformer 中任意两个位置都能直接建立连接。

### 3. 更适合大规模训练

因为：

- 结构规则
- 易并行
- 易扩展

这使它成为大模型时代最核心的基础架构。

## 十七、Transformer 的局限

为了完整理解，也要知道它的代价。

### 1. Attention 复杂度高

标准 Self-Attention 需要构造 `n × n` 的注意力矩阵，因此：

- 时间复杂度高
- 显存开销大

序列越长，问题越明显。

### 2. 推理阶段仍然有顺序依赖

虽然训练能并行，但自回归生成时仍然要逐步解码。

### 3. 长文本成本高

长序列会带来：

- attention 计算量膨胀
- KV cache 增长
- 显存压力上升

## 十八、面试高频问法速答

### 1. 为什么要除以 `sqrt(d_k)`

- 防止点积值过大
- 避免 softmax 过度饱和
- 保持梯度稳定

### 2. 为什么需要 Q、K、V 三个矩阵

- Q/K 用于算相关性
- V 用于携带最终被聚合的信息
- 把匹配和内容表达解耦

### 3. 多头注意力为什么有效

- 不同头能在不同子空间关注不同关系
- 最终融合得到更丰富表示

### 4. FFN 有什么作用

- 对每个位置做非线性特征变换
- 增强表达能力

### 5. 为什么要做位置编码

- 注意力本身不感知顺序
- 必须显式加入位置信息

### 6. Padding Mask 和 Sequence Mask 区别是什么

- Padding Mask 屏蔽无效 pad
- Sequence Mask 屏蔽未来 token

### 7. 为什么 Transformer 比 RNN 更适合大模型

- 训练可并行
- 更易扩展
- 更容易建模长距离依赖

## 十九、学习建议

如果你是第一次系统学习 Transformer，建议按这个顺序理解：

1. 先看整体结构：Encoder 和 Decoder 各做什么
2. 再看 Self-Attention：Q、K、V 和 softmax
3. 再看 Multi-Head Attention：为什么要多个头
4. 再看 FFN、残差、LayerNorm
5. 最后看位置编码、mask 和解码过程

一句话总结：

Transformer 的核心不是“注意力”这个词本身，而是它用注意力机制把序列中任意位置之间的依赖关系变成了可并行、可训练、可扩展的统一计算结构。
