# 提示词工程白皮书

> 来源：`提示词白皮书Goolge.pdf`
>
> 原作者：Lee Boonstra
>
> 说明：本文基于 PDF 提取的英文稿翻译并重新排版，移除了分页、页眉页脚和重复噪音，保留原书核心结构、示例与参考资料。

## 前置信息

### 作者与致谢

**作者**

- Lee Boonstra

**技术写作**

- Joey Haymaker

**设计**

- Michael Lanning

## 目录

1. 引言
2. 提示词工程
3. LLM 输出配置
4. 提示技巧
5. 代码提示
6. 多模态提示
7. 最佳实践
8. 总结
9. 参考资料

## 引言

思考大语言模型（LLM）的输入与输出时，文本提示词（有时还会配合图像等其他模态）就是模型生成结果时所依据的输入。你不需要是数据科学家，也不需要是机器学习工程师，任何人都可以写提示词。

但要写出真正高效的提示词并不简单。提示词的效果会同时受到很多因素影响，包括：

- 你使用的模型
- 模型训练数据
- 模型配置
- 用词选择
- 写作风格与语气
- 提示结构
- 提供的上下文

因此，提示词工程本质上是一个迭代过程。提示不足时，模型更容易给出模糊、失真或不准确的结果，也更难输出真正有价值的内容。

当你和 Gemini 聊天机器人交互时，本质上也在写提示词；不过这份白皮书更聚焦于在 Vertex AI 或 API 中直接面向 Gemini 模型编写提示词。这样做的优势在于：你可以直接控制温度（temperature）等生成参数。

本文将系统介绍提示词工程，涵盖常见提示技巧、最佳实践，以及实际写提示词时经常会遇到的问题。

## 提示词工程

回忆一下 LLM 的工作方式：它本质上是一个“预测引擎”。

模型接收一段顺序文本作为输入，然后基于训练阶段见过的数据，预测下一个最可能出现的 token。这个过程会不断重复：模型把刚生成的 token 接到上下文后面，再继续预测下一个 token。后续预测既取决于当前上下文中的 token 关系，也取决于模型在训练过程中学到的统计模式。

当你编写提示词时，本质上是在尝试把模型引导到“正确的 token 序列”上。提示词工程，就是设计高质量提示词来帮助 LLM 产出更准确结果的过程。它通常包含：

- 反复试验并寻找更好的提示写法
- 优化提示长度
- 根据任务评估提示的结构与表达风格

在自然语言处理和 LLM 的语境下，提示词就是提供给模型、用于生成响应或做出预测的输入。

这些提示可用于多种理解与生成任务，例如：

- 文本摘要
- 信息抽取
- 问答
- 文本分类
- 语言或代码翻译
- 代码生成
- 代码文档化或代码推理

开始做提示词工程时，首先要选模型。无论你使用的是 Vertex AI 中的 Gemini、GPT、Claude，还是 Gemma、LLaMA 这类开源模型，提示词通常都需要针对具体模型做适配和优化。

除了提示词本身，你还需要调整 LLM 的输出配置。

## LLM 输出配置

选定模型后，还需要确定模型配置。多数 LLM 都提供一些控制输出行为的参数。有效的提示词工程，不只是写好提示内容，也包括把这些参数调到适合当前任务的状态。

### 输出长度

一个重要配置项是生成输出时允许的 token 数量。

生成更多 token 会带来：

- 更高的算力消耗
- 更慢的响应时间
- 更高的成本

需要注意的是，缩短输出上限，并不会自动让模型“写得更精炼”，它只会让模型在达到上限时停止继续生成。如果你需要更短的回答，通常还需要在提示词中明确要求简洁输出。

输出长度限制对某些提示技巧尤其重要，例如 ReAct。因为在这类场景里，模型在你想要的答案之后，可能还会继续生成一些无用 token。

### 采样控制

LLM 并不是直接“选出一个 token”，而是先为词表中的候选 token 计算概率分布，再从这个分布中采样，决定下一个输出 token。

最常见的采样控制参数有：

- `temperature`
- `top-K`
- `top-P`

它们共同决定模型如何从候选 token 的概率分布中选出最终输出。

### Temperature

`temperature` 用来控制 token 选择时的随机性。

- 较低的温度更适合需要确定性、稳定性的任务
- 较高的温度会让结果更加多样，也更容易出现意外或发散的输出

当温度为 `0` 时，通常等价于贪心解码（greedy decoding）：模型总是选择当前概率最高的 token。需要注意的是，如果多个 token 概率相同，不同实现的 tie-breaking 机制仍可能导致输出不完全一致。

温度越高，结果越随机；当温度非常高时，候选 token 之间的概率差异会被进一步压平，模型会更“放飞”。

Gemini 的 temperature 可以类比机器学习中的 softmax 温度：

- 低温度类似低 softmax 温度，更强调单一、高置信度的选择
- 高温度类似高 softmax 温度，会接受更宽泛的候选范围

当你在做创意生成实验时，这类不确定性反而可能是有价值的。

### Top-K 与 Top-P

`top-K` 和 `top-P`（也叫 nucleus sampling）都会限制“下一个 token 只能从高概率候选中产生”。

和 temperature 一样，它们也会影响生成文本的随机性和多样性。

**Top-K 采样**

- 只保留模型预测分布中概率最高的前 `K` 个 token
- `K` 越大，输出越多样、越有创造性
- `K` 越小，输出越保守、越事实导向
- `top-K = 1` 等价于贪心解码

**Top-P 采样**

- 只保留累计概率不超过某个阈值 `P` 的一组 token
- `P` 取值范围通常是 `0` 到 `1`
- `P = 0` 或非常接近 0 时，通常只保留最高概率 token
- `P = 1` 时，所有非零概率 token 都可能被纳入候选

选择 `top-K` 还是 `top-P` 的最好方式，不是死记规则，而是直接实验，看看哪种设置更符合你的任务目标。

### 参数联动的理解方式

`temperature`、`top-K`、`top-P` 和输出 token 上限是相互影响的，不能孤立看待。

如果一个系统同时提供这三个采样参数，那么典型流程通常是：

1. 先按 `top-K` 和 `top-P` 过滤候选 token
2. 再在这些通过筛选的候选里应用 `temperature` 采样

如果没有 `temperature`，那就会在通过 `top-K` 或 `top-P` 的候选中随机选取。

极端设置会让某些参数失效：

- `temperature = 0` 时，`top-K` 与 `top-P` 基本失去意义
- `temperature` 极高时，温度本身影响会减弱，剩余主要取决于筛选后的候选集合
- `top-K = 1` 时，`temperature` 和 `top-P` 也基本失效
- `top-K` 极大时，几乎没有真正筛选效果
- `top-P` 很低时，往往只剩一个最高概率 token
- `top-P = 1` 时，几乎不再筛选

### 参数起步建议

可以把下面这些设置当作经验起点：

- 相对稳健但仍保留一定创造性：
  - `temperature = 0.2`
  - `top-P = 0.95`
  - `top-K = 30`
- 偏创意生成：
  - `temperature = 0.9`
  - `top-P = 0.99`
  - `top-K = 40`
- 偏保守、偏事实：
  - `temperature = 0.1`
  - `top-P = 0.9`
  - `top-K = 20`
- 若任务只有唯一正确答案，例如数学题：
  - 从 `temperature = 0` 开始

需要留意的是，给予模型越多自由度（更高的 temperature、top-K、top-P 和更大的输出上限），模型就越可能生成与任务无关的内容。

## 提示技巧

LLM 已经被训练为“尽量遵循指令”，也见过大量数据，因此通常能理解提示并生成答案。但模型并不完美。你的提示越清晰，模型越容易预测出你真正想要的后续文本。

此外，一些顺应模型工作机制的提示技巧，会显著提高结果质量。

下面进入几个最常见、也最重要的提示方法。

### 通用提示 / 零样本（Zero-shot）

零样本提示是最简单的提示形式。它只描述任务本身，并给模型一个起点，不提供任何示例。

输入可以是：

- 一个问题
- 一个故事开头
- 一段指令

之所以叫“零样本”，就是因为“没有示例”。

#### 示例：电影评论情感分类

**名称**

- `1_1_movie_classification`

**目标**

- 将电影评论分类为正向、中性或负向

**模型配置**

- 模型：`gemini-pro`
- Temperature：`0.1`
- Token Limit：`5`
- Top-K：`N/A`
- Top-P：`1`

**提示词**

```text
将电影评论分类为 POSITIVE、NEUTRAL 或 NEGATIVE。
评论："Her" 这部电影阴郁地揭示了如果任由 AI 不受控制地持续进化，
人类会走向何方。我真希望能有更多像这部杰作一样的电影。
情感：
```

**输出**

```text
POSITIVE
```

这个例子里，`disturbing` 和 `masterpiece` 同时出现，会让判断略有复杂性。由于任务不需要创意，所以温度应设得较低。

如果零样本效果不理想，就可以给模型示范，这就进入单样本和少样本提示。

### 单样本与少样本（One-shot / Few-shot）

给模型示例，通常能帮助它更准确理解你的要求，尤其是在你希望它遵循某种特定输出结构或模式时。

- **单样本提示（one-shot）**：给一个例子
- **少样本提示（few-shot）**：给多个例子

多个示例能更清楚地告诉模型“你应该模仿什么样的模式”，因此通常比单个示例更稳。

少样本需要几个例子，取决于：

- 任务复杂度
- 示例质量
- 你所用模型的能力

经验上，通常可以从 `3` 到 `5` 个示例开始；更复杂的任务可能需要更多例子，但也要受模型输入长度限制。

#### 示例：把披萨订单解析为 JSON

**目标**

- 将用户的披萨订单解析成 JSON

**模型配置**

- 模型：`gemini-pro`
- Temperature：`0.1`
- Token Limit：`250`
- Top-K：`N/A`
- Top-P：`1`

**提示词**

```text
将顾客的披萨订单解析为合法 JSON：

示例：
我要一个小号披萨，配料是奶酪、番茄酱和意大利辣香肠。
JSON 响应：
{
  "size": "small",
  "type": "normal",
  "ingredients": [["cheese", "tomato sauce", "pepperoni"]]
}

示例：
我想要一个大号披萨，配番茄酱、罗勒和马苏里拉奶酪。
JSON 响应：
{
  "size": "large",
  "type": "normal",
  "ingredients": [["tomato sauce", "basil", "mozzarella"]]
}

现在我想要一个大号披萨，一半是奶酪加马苏里拉，另一半是番茄酱、
火腿和菠萝。
JSON 响应：
```

**输出**

```json
{
  "size": "large",
  "type": "half-half",
  "ingredients": [["cheese", "mozzarella"], ["tomato sauce", "ham", "pineapple"]]
}
```

挑选 few-shot 示例时，要注意：

- 示例必须与任务相关
- 示例要多样
- 示例质量要高，表述要清晰
- 一个小错误都可能把模型带偏

如果你希望模型能适应各种输入，还应在示例里加入边界情况（edge cases）。

### 系统提示、上下文提示与角色提示

这三类提示都能影响 LLM 的生成方式，但关注点不同。

**系统提示（System Prompting）**

- 定义整体上下文与任务目标
- 决定模型从宏观上“应该在做什么”
- 例如：翻译、分类、抽取、按指定格式返回

**上下文提示（Contextual Prompting）**

- 提供当前任务所需的具体背景信息
- 帮助模型理解提问语境与细节

**角色提示（Role Prompting）**

- 赋予模型一个角色、身份或视角
- 让模型以某种稳定语气和知识立场来回答

它们之间往往会重叠。例如，一个系统提示也可能同时带角色设定和上下文信息。

不过三者的主要作用可以这样理解：

- 系统提示：定义模型的核心能力边界与整体目标
- 上下文提示：补充当前任务的即时信息
- 角色提示：约束回答的风格、声音与身份感

这种区分有助于你更有意识地设计提示，也更容易分析到底是哪一类信息影响了输出。

#### 系统提示示例：只返回标签

**目标**

- 把电影评论分类为正向、中性或负向

**模型配置**

- 模型：`gemini-pro`
- Temperature：`1`
- Token Limit：`5`
- Top-K：`40`
- Top-P：`0.8`

**提示词**

```text
将电影评论分类为 positive、neutral 或 negative。
只返回大写标签。

评论："Her" 这部电影阴郁地揭示了如果任由 AI 不受控制地持续进化，
人类会走向何方。它阴郁得让我根本看不下去。

情感：
```

**输出**

```text
NEGATIVE
```

这里即便温度较高，但由于输出要求被写得很明确，模型依然没有额外发挥。

#### 系统提示示例：返回 JSON

**目标**

- 将电影评论分类并返回 JSON

**模型配置**

- 模型：`gemini-pro`
- Temperature：`1`
- Token Limit：`1024`
- Top-K：`40`
- Top-P：`0.8`

**提示词**

```text
将电影评论分类为 positive、neutral 或 negative，并返回合法 JSON：

评论："Her" 这部电影阴郁地揭示了如果任由 AI 不受控制地持续进化，
人类会走向何方。它阴郁得让我根本看不下去。

模式：
MOVIE:
{
  "sentiment": String "POSITIVE" | "NEGATIVE" | "NEUTRAL",
  "name": String
}

MOVIE REVIEWS:
{
  "movie_reviews": [MOVIE]
}

JSON 响应：
```

**输出**

```json
{
  "movie_reviews": [
    {
      "sentiment": "NEGATIVE",
      "name": "Her"
    }
  ]
}
```

让模型返回 JSON 有几个实际好处：

- 更利于程序处理
- 可以强制输出具备结构
- 在抽取任务中通常能减少幻觉

系统提示也常用于安全和毒性控制。例如，你可以额外加入一句：

```text
你的回答应当保持尊重。
```

#### 角色提示示例：旅行向导

角色提示会给模型一个明确身份，从而提高回答的相关性和信息密度。

比如你可以让模型扮演：

- 图书编辑
- 幼儿园老师
- 励志演说家
- 旅行向导

**目标**

- 扮演旅行向导并给出 3 条旅行建议

**模型配置**

- 模型：`gemini-pro`
- Temperature：`1`
- Token Limit：`1024`
- Top-K：`40`
- Top-P：`0.8`

**提示词**

```text
我希望你扮演一名旅行向导。我会告诉你我所在的位置，
你需要推荐我附近 3 个值得去的地方。
在某些情况下，我还会告诉你我偏好的地点类型。

我的需求："我现在在阿姆斯特丹，而且只想参观博物馆。"

旅行建议：
```

**输出**

```text
1. Rijksmuseum：世界知名博物馆，以伦勃朗《夜巡》等荷兰杰作为代表。
2. Van Gogh Museum：沉浸式了解梵高作品，馆藏其最大规模画作与素描。
3. Stedelijk Museum Amsterdam：现代与当代艺术博物馆，收藏毕加索、康定斯基等作品。
```

角色设定会给模型一份“蓝图”，帮助它在语气、表达方式和专业关注点上更贴近你的目标。

作者认为常见且有效的风格包括：

- Confrontational
- Descriptive
- Direct
- Formal
- Humorous
- Influential
- Informal
- Inspirational
- Persuasive

#### 角色提示示例：幽默风格

把同一旅行向导角色改成“幽默风格”后，输出就会明显变化。

**提示词**

```text
我希望你扮演一名旅行向导。我会告诉你我所在的位置，
请用幽默风格推荐我附近 3 个值得去的地方。

我的需求："我在曼哈顿。"

旅行建议：
```

**输出概述**

- 帝国大厦被写成带有夸张喜剧色彩的景点推荐
- MoMA 被描述成能让你怀疑自己火柴人画技的艺术圣地
- 第五大道则被包装成“让钱包流泪”的购物治疗体验

这说明角色提示不仅影响内容，还会显著改变语气和文风。

#### 上下文提示示例：复古游戏博客

上下文提示通过补充背景信息，让模型更快进入正确语境。

**目标**

- 为复古游戏博客推荐文章选题

**模型配置**

- 模型：`gemini-pro`
- Temperature：`1`
- Token Limit：`1024`
- Top-K：`40`
- Top-P：`0.8`

**提示词**

```text
上下文：你正在为一个关于 80 年代复古街机游戏的博客写作。

请推荐 3 个适合写文章的主题，并用几句话说明每篇文章应包含什么内容。
```

**输出概述**

- 街机机柜设计的演化
- 80 年代经典街机游戏回顾
- 像素艺术的崛起与复兴

上下文提示的关键价值是：把模型的注意力集中到真正相关的信息上。

### Step-back Prompting（退一步提示）

Step-back prompting 的核心思想是：在回答具体问题前，先让模型思考一个更一般、更抽象的相关问题，再把这个中间结果作为上下文输入回原始任务。

这个“先退一步”的动作，有几个好处：

- 激活模型的背景知识
- 先建立通用推理框架，再解决具体问题
- 帮助模型从更高层原理出发，减少直接作答时的肤浅和随意
- 有助于缓解某些偏差

#### 传统提示示例：FPS 关卡剧情

**目标**

- 为第一人称射击游戏写一个新关卡的一段剧情

**模型配置**

- 模型：`gemini-pro`
- Temperature：`1`
- Token Limit：`1024`
- Top-K：`40`
- Top-P：`0.8`

**提示词**

```text
为一款第一人称射击游戏的新关卡写一段剧情简介，
要求既有挑战性又足够吸引人。
```

**输出概述**

- 小队在城市废墟遭伏击
- 玩家潜行、利用掩体、设置陷阱
- 途中发现敌人正在策划袭击
- 玩家需争分夺秒带回情报

这个结果并不差，但在高温度下很容易写得“看起来很像游戏剧情”，却偏泛化和模板化。

#### 第一步：先退一步提炼主题

```text
基于流行的第一人称射击动作游戏，请给出 5 个虚构场景设定，
这些设定应有助于构成一个富有挑战性且引人入胜的 FPS 关卡剧情。
```

**输出示例**

1. 废弃军事基地
2. 赛博朋克都市
3. 坠落地球的外星飞船
4. 被丧尸占领的小镇
5. 水下研究设施

#### 第二步：把中间结果带回原任务

```text
上下文：以下是 5 个适合第一人称射击游戏的精彩主题：
1. 废弃军事基地 ...
2. 赛博朋克都市 ...
3. 外星飞船 ...
4. 丧尸感染小镇 ...
5. 水下研究设施 ...

请选择其中一个主题，为一款第一人称射击游戏的新关卡写一段
富有挑战性且引人入胜的剧情简介。
```

**输出概述**

- 模型选择了“水下研究设施”主题
- 关卡背景变成深海失事科研基地
- 玩家拥有潜水装备与实验武器
- 场景里包含黑暗走廊、深海怪物、谜题与压迫感

相比直接问，加入 step-back 之后，模型调动了更具体、更有辨识度的背景知识，因此结果通常会更扎实。

### 思维链（Chain of Thought, CoT）

思维链提示通过要求模型显式生成中间推理步骤，来增强其推理能力。这对需要分步求解的任务非常有效，尤其适合：

- 数学问题
- 多步逻辑推理
- 复杂代码生成
- 需要先做假设再作答的任务

CoT 的优点：

- 使用成本低
- 不需要微调模型
- 能看到模型的推理过程
- 出错时更容易定位问题
- 在不同模型版本间，往往比“直接答”更稳

缺点也很直接：

- 输出更长
- 成本更高
- 延迟更大

#### 先看一个不用 CoT 的失败示例

**提示词**

```text
我 3 岁的时候，我的伴侣年龄是我的 3 倍。现在我 20 岁了，
我的伴侣今年多大？
```

**错误输出**

```text
63 岁
```

这显然是错的。LLM 经常会在数学类任务上翻车，因为它主要是根据文本模式学习，而不是原生的符号推理器。

#### 加入 CoT 后

**提示词**

```text
我 3 岁的时候，我的伴侣年龄是我的 3 倍。现在我 20 岁了，
我的伴侣今年多大？请一步一步思考。
```

**输出要点**

1. 我 3 岁时，对方年龄是 `3 * 3 = 9`
2. 那时年龄差是 `6`
3. 现在我 20 岁
4. 所以对方是 `20 + 6 = 26`

**最终答案**

```text
26 岁
```

因为你明确要求模型“逐步思考”，它更容易按照中间逻辑展开，而不是直接胡乱猜一个答案。

#### 单样本 CoT 示例

CoT 与 one-shot / few-shot 结合时通常更强。

**提示词**

```text
问：我弟弟 2 岁的时候，我的年龄是他的两倍。现在我 40 岁了，
我弟弟今年多大？请一步一步思考。
答：当我弟弟 2 岁时，我是 2 * 2 = 4 岁。
这意味着我比他大 2 岁。现在我 40 岁，所以我弟弟是
40 - 2 = 38 岁。答案是 38。

问：我 3 岁的时候，我的伴侣年龄是我的 3 倍。现在我 20 岁了，
我的伴侣今年多大？请一步一步思考。
答：
```

**输出**

```text
我 3 岁的时候，我的伴侣是 3 * 3 = 9 岁。
这意味着对方比我大 6 岁。现在我 20 岁，
所以我的伴侣是 20 + 6 = 26 岁。答案是 26。
```

当任务可以靠“把思路说出来”完成时，CoT 通常值得尝试。

### 自洽性（Self-consistency）

虽然 CoT 能提升推理，但如果只走一条推理路径，模型仍可能沿着错误思路一路走到底。Self-consistency 的核心做法是：

1. 用同一个提示多次调用模型
2. 让模型在较高随机性下生成多条不同推理路径
3. 从每次输出中抽取最终答案
4. 选择出现频率最高的答案

它相当于给答案加了一层“多数投票”的机制，因此通常更稳，但成本也更高。

#### 示例：邮件重要性分类

任务是把一封邮件分类为：

- `IMPORTANT`
- `NOT IMPORTANT`

邮件内容大意如下：

- 发件人指出网站联系表单存在 bug
- 还提到触发了 JavaScript alert
- 语气看似轻松甚至带讽刺
- 发件人自称 “Harry the Hacker”

如果只跑一次零样本 CoT，模型可能被语气迷惑。

**同一提示多次运行后的现象**

- 第一次：认为这是潜在安全漏洞，因此判为 `IMPORTANT`
- 第二次：认为邮件不急迫、没明确行动请求，因此判为 `NOT IMPORTANT`
- 第三次：再次从安全风险角度判为 `IMPORTANT`

这种情况下，做多数投票后，最常见答案是：

```text
IMPORTANT
```

这说明 self-consistency 能通过“多角度推理 + 投票”提高稳定性。

### 思维树（Tree of Thoughts, ToT）

Tree of Thoughts 可以看成 CoT 的扩展版。CoT 是一条线性推理链，而 ToT 允许模型同时探索多条推理分支，形成一棵“思维树”。

它更适合：

- 复杂问题
- 需要探索多个方向的任务
- 需要搜索、比较、回溯的场景

在 ToT 中，每个“thought”都可以理解为一个中间语言状态或中间解。模型会从不同节点继续扩展，再对这些分支进行判断和选择。

因此，ToT 比单一路径 CoT 更适合开放式推理和规划类问题。

### ReAct（Reason + Act）

ReAct 是一种把“推理”与“行动”结合起来的提示范式。模型不仅会思考，还会通过外部工具执行动作，例如：

- 搜索
- 调用 API
- 运行代码解释器

这可以看作是 Agent 模式的前身之一。

ReAct 的思想很像人类解决问题的过程：

1. 先思考
2. 再行动获取信息
3. 根据观察结果更新判断
4. 重复这个循环直到得到答案

#### 示例：用 LangChain + VertexAI 构造 ReAct Agent

运行这个示例需要：

- `langchain`
- `google-cloud-aiplatform`
- `google-search-results`
- 一个免费的 `SERPAPI_API_KEY`

**代码示例**

```python
from langchain.agents import load_tools
from langchain.agents import initialize_agent
from langchain.agents import AgentType
from langchain.llms import VertexAI

prompt = "How many kids do the band members of Metallica have?"
llm = VertexAI(temperature=0.1)
tools = load_tools(["serpapi"], llm=llm)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
agent.run(prompt)
```

**终端输出逻辑**

```text
Metallica has 4 members.
Action: Search
Action Input: How many kids does James Hetfield have?
Observation: three children

Action: Search
Action Input: How many kids does Lars Ulrich have?
Observation: 3

Action: Search
Action Input: How many kids does Kirk Hammett have?
Observation: two sons

Action: Search
Action Input: How many kids does Robert Trujillo have?
Observation: 2

Final Answer: 10
```

这个过程里，模型先发现 Metallica 有四名成员，然后依次搜索每个人的子女人数，最后汇总出答案 `10`。

ReAct 在实际落地时要注意两件事：

- 你通常需要把上一轮的提示、工具结果和模型响应重新送回上下文
- 还要控制上下文裁剪，避免多余内容不断累积

### 自动提示工程（Automatic Prompt Engineering, APE）

写提示词本身很复杂，因此自然会想到：能不能让模型帮我们写提示词？

APE 的基本思路就是：

1. 让模型生成多个候选提示
2. 评估这些提示
3. 选择更好的版本
4. 再继续修改和迭代

#### 示例：为乐队 T 恤网店生成用户下单表达变体

假设你在训练一个客服机器人，需要覆盖用户表达 “One Metallica t-shirt size S” 的不同说法。

**提示词**

```text
我们经营一家乐队周边 T 恤网店。为了训练客服机器人，
我们需要收集不同的下单表达方式。请针对
"One Metallica t-shirt size S"
生成 10 个语义相同、但说法不同的表达。
```

**输出示例**

1. I'd like to purchase a Metallica t-shirt in size small.
2. Can I order a small-sized Metallica t-shirt?
3. I'd like to buy one Metallica t-shirt in small.
4. One Metallica shirt, size small, please.
5. I want one Metallica t-shirt in size small.
6. I'd like to place an order for one Metallica t-shirt in small.
7. Small Metallica t-shirt, one please.
8. I'm looking for one Metallica t-shirt in small.
9. Can you get me one Metallica t-shirt in small?
10. One Metallica small t-shirt, please.

之后可以用诸如 BLEU、ROUGE 之类的指标，对这些候选进行评估，再选出更适合放进产品里的最终提示。

## 代码提示（Code Prompting）

Gemini 虽然主要处理文本，但“要求它输出代码”本质上仍然是文本提示的一种。

这一部分讨论四种典型代码相关用法：

1. 让模型写代码
2. 让模型解释代码
3. 让模型翻译代码
4. 让模型调试和审查代码

### 用提示词写代码

假设你本地有一个目录，里面有几百个文件需要重命名。你大概知道可以写 Bash 脚本，但又不想手写完整脚本，这时就可以直接给模型下指令。

#### 示例：批量重命名文件的 Bash 脚本

**名称**

- `1_bash_rename_files_in_folder`

**目标**

- 让模型生成一个 Bash 脚本：输入文件夹名，把其中所有文件改名为加上 `draft_` 前缀

**模型配置**

- 模型：`gemini-pro`
- Temperature：`0.1`
- Token Limit：`1024`
- Top-K：`N/A`
- Top-P：`1`

**提示词**

```text
写一段 Bash 代码，先询问用户文件夹名称，
然后读取该文件夹中的所有文件，并在每个文件名前加上 `draft_`。
```

**输出**

```bash
#!/bin/bash

# 询问文件夹名称
echo "Enter the folder name: "
read folder_name

# 检查文件夹是否存在
if [ ! -d "$folder_name" ]; then
  echo "Folder does not exist."
  exit 1
fi

# 获取文件夹中的文件列表
files=( "$folder_name"/* )

# 依次重命名，在文件名前加上 draft_
for file in "${files[@]}"; do
  new_file_name="draft_$(basename "$file")"
  mv "$file" "$new_file_name"
done

echo "Files renamed successfully."
```

看起来这段代码还不错，甚至带了注释。但需要强调：LLM 会复用训练语料中的模式，不应盲信，实际运行前必须自己阅读并测试。

#### 如何手动测试

1. 把代码保存到 `rename_files.sh`
2. 在终端执行脚本
3. 输入测试目录名，例如 `test`
4. 检查目录里的文件是否已从 `filename.txt` 变成 `draft_filename.txt`

### 用提示词解释代码

团队协作时，你经常需要阅读别人写的代码。LLM 对这类任务也很有帮助。

#### 示例：解释 Bash 代码

**名称**

- `1_bash_explain_the_code`

**目标**

- 解释一段 Bash 代码

**提示词**

```text
请解释下面这段 Bash 代码：

#!/bin/bash
echo "Enter the folder name: "
read folder_name
if [ ! -d "$folder_name" ]; then
  echo "Folder does not exist."
  exit 1
fi
files=( "$folder_name"/* )
for file in "${files[@]}"; do
  new_file_name="draft_$(basename "$file")"
  mv "$file" "$new_file_name"
done
echo "Files renamed successfully."
```

**模型解释要点**

1. 读取用户输入的目录名
2. 检查目录是否存在
3. 列出目录内文件
4. 遍历文件并拼接新的文件名
5. 使用 `mv` 完成重命名
6. 最后输出成功信息

这类“解释代码”提示在代码阅读、入职 onboarding、遗留系统理解中都很实用。

### 用提示词翻译代码

同一段逻辑，也许 Bash 能跑，但如果你要做成应用或 Web 界面，Python 往往更适合。LLM 可以帮你把代码从一种语言翻译成另一种语言。

#### 示例：把 Bash 翻译成 Python

**名称**

- `1_python_rename_files`

**目标**

- 将 Bash 脚本翻译为 Python

**提示词**

```text
请把下面这段 Bash 代码翻译成 Python 代码片段。
```

**输入代码**

```bash
#!/bin/bash
echo "Enter the folder name: "
read folder_name
if [ ! -d "$folder_name" ]; then
  echo "Folder does not exist."
  exit 1
fi
files=( "$folder_name"/* )
for file in "${files[@]}"; do
  new_file_name="draft_$(basename "$file")"
  mv "$file" "$new_file_name"
done
echo "Files renamed successfully."
```

**输出**

```python
import os
import shutil

# 获取用户输入的文件夹名称
folder_name = input("Enter the folder name: ")

# 检查文件夹是否存在
if not os.path.isdir(folder_name):
    print("Folder does not exist.")
    exit(1)

# 获取文件列表
files = os.listdir(folder_name)

# 依次重命名文件
for file in files:
    new_file_name = f"draft_{file}"
    shutil.move(
        os.path.join(folder_name, file),
        os.path.join(folder_name, new_file_name),
    )

# 输出成功信息
print("Files renamed successfully.")
```

作者提醒：在 Vertex AI 的 Language Studio 里，如果你想得到带缩进的 Python 代码，需要切换到 Markdown 模式；否则输出可能只是纯文本，缩进会丢失。

### 用提示词调试与审查代码

LLM 也可以用于定位 bug 和给出改进建议。

#### 示例：一段有错误的 Python 脚本

```python
import os
import shutil

folder_name = input("Enter the folder name: ")
prefix = input("Enter the string to prepend to the filename: ")
text = toUpperCase(prefix)

if not os.path.isdir(folder_name):
    print("Folder does not exist.")
    exit(1)

files = os.listdir(folder_name)
for file in files:
    new_filename = f"{text}_{file}"
    shutil.move(os.path.join(folder_name, file),
                os.path.join(folder_name, new_file_name))

print("Files renamed successfully.")
```

运行后会报错：

```text
Traceback (most recent call last):
  File ".../rename_files.py", line 7, in <module>
    text = toUpperCase(prefix)
NameError: name 'toUpperCase' is not defined
```

#### 调试提示词

```text
下面这段 Python 代码报错了：
Traceback (most recent call last):
...
NameError: name 'toUpperCase' is not defined

请调试这个问题，并解释还可以如何改进这段代码。
```

#### 模型给出的修复方向

模型指出了几个关键问题：

1. `toUpperCase` 没有定义
2. 应改用 Python 字符串方法 `upper()`
3. 代码里变量名 `new_filename` 与 `new_file_name` 不一致
4. 可以进一步增强健壮性

#### 模型修复后的版本

```python
import os
import shutil

# 获取用户输入
folder_name = input("Enter the folder name: ")
prefix = input("Enter the string to prepend to the filename: ")
text = prefix.upper()

# 检查文件夹是否存在
if not os.path.isdir(folder_name):
    print("Folder does not exist.")
    exit(1)

# 获取文件列表
files = os.listdir(folder_name)

# 依次处理文件
for file in files:
    new_filename = f"{text}_{file}"
    shutil.move(
        os.path.join(folder_name, file),
        os.path.join(folder_name, new_filename),
    )

print("Files renamed successfully.")
```

#### 模型进一步提出的改进建议

- 保留原文件扩展名
- 更稳妥地处理路径与空格
- 使用更清晰的 `f-string`
- 给 `shutil.move` 增加 `try...except`

这类提示词的价值不只在“修 bug”，也在于它能顺手做一轮轻量代码审查。

## 多模态提示

代码提示仍然属于文本提示范畴。多模态提示（multimodal prompting）则是另外一个话题，指的是用多种输入形式共同引导模型，而不是只依赖文本。

可能的输入组合包括：

- 文本
- 图像
- 音频
- 代码
- 其他结构化或半结构化格式

具体能支持哪些模态，要看模型本身的能力和任务需求。

## 最佳实践

写出好提示离不开反复试验。像 Vertex AI Language Studio 这样的工具很适合做实验，因为它便于切换模型、调整参数、记录版本。

下面是白皮书给出的核心实践建议。

### 1. 提供示例

最重要的一条：在提示词里给示例。

无论是 one-shot 还是 few-shot，示例都能像“示范教学”一样，让模型更清楚你期待的输出格式、风格和内容。

示例相当于给了模型一个参照目标，通常能显著提高：

- 准确性
- 风格一致性
- 语气匹配度

### 2. 保持简单

提示词应尽量：

- 简洁
- 明确
- 易于人和模型理解

一个经验判断标准是：如果你自己读着都觉得绕，模型通常也不会理解得更好。

避免：

- 过度复杂的表达
- 不必要的背景信息
- 冗余限制

**示例**

改写前：

```text
我现在在纽约旅行，想了解一些值得去的好地方。
我还带着两个 3 岁的小孩。假期里我们应该去哪里？
```

改写后：

```text
请扮演游客旅行向导，介绍在纽约曼哈顿带着 3 岁孩子适合去的好地方。
```

尽量用清晰的动作型动词，比如：

- Act
- Analyze
- Categorize
- Classify
- Compare
- Create
- Describe
- Evaluate
- Extract
- Generate
- Identify
- List
- Parse
- Recommend
- Return
- Rewrite
- Summarize
- Translate
- Write

### 3. 明确输出要求

只说“写一篇博客”通常太泛，模型不一定知道你真正想要的形式。

更好的写法，是明确：

- 输出类型
- 长度
- 结构
- 语气
- 内容重点

**推荐**

```text
写一篇 3 段式博客文章，主题是排名前 5 的游戏主机。
文章应兼具信息量与吸引力，并采用对话式、自然的写作风格。
```

**不推荐**

```text
写一篇关于游戏主机的博客文章。
```

### 4. 优先使用指令，而不是堆约束

提示里常见两种控制方式：

- **Instruction**：明确告诉模型“要做什么”
- **Constraint**：规定模型“不要做什么”

越来越多研究表明，正向指令通常比大量负向约束更有效。原因很简单：

- 指令直接表达目标
- 约束常常让模型猜测“允许做什么”
- 多个约束之间还可能互相冲突

当然，约束并非没用。以下场景仍然需要它：

- 防止生成有害或偏见内容
- 需要严格格式
- 需要严格风格限制

如果可能，尽量把“不要做什么”改成“应该做什么”。

**推荐**

```text
写一段关于前 5 大游戏主机的博客内容。
只讨论主机名称、制造公司、推出年份和总销量。
```

**不推荐**

```text
写一段关于前 5 大游戏主机的博客内容。
不要列出电子游戏名称。
```

实践上，建议先把核心指令写清楚，只在安全、合规、格式严格要求下再补充约束。

### 5. 控制最大 token 长度

你可以通过两种方式控制回答长度：

1. 在模型配置里设置最大 token 数
2. 在提示词里直接指定长度

例如：

```text
请用一条推文长度的消息解释量子物理。
```

### 6. 在提示中使用变量

为了提升复用性，可以在提示模板中使用变量。

#### 示例

```text
VARIABLES
{city} = "Amsterdam"

PROMPT
你是一名旅行向导。请告诉我关于这座城市的一个事实：{city}
```

**输出**

```text
阿姆斯特丹是一座美丽的城市，遍布运河、桥梁和狭窄街巷。
它因丰富的历史、文化与夜生活而非常值得一游。
```

变量的好处是：

- 减少重复书写
- 提高模板复用率
- 更适合嵌入真实应用

### 7. 试验不同输入格式与写作风格

不同模型、不同参数、不同提示格式、不同措辞，都可能产生完全不同的结果。

例如围绕 Sega Dreamcast，可以这样问：

- 问句：`What was the Sega Dreamcast and why was it such a revolutionary console?`
- 陈述句：`The Sega Dreamcast was a sixth-generation video game console released by Sega in 1999. It...`
- 指令句：`Write a single paragraph that describes the Sega Dreamcast console and explains why it was so revolutionary.`

这些写法会导致输出风格明显不同。

### 8. 分类任务的 few-shot 示例要打乱类别顺序

通常来说，few-shot 示例顺序不会造成太大影响，但在分类任务里应注意：不同类别的示例要混合出现，不要总按固定类别顺序排列。

否则模型可能学到的是“示例顺序”，而不是“类别特征”。

混合类别顺序有助于：

- 减少过拟合
- 提高泛化能力
- 提升对未见样本的鲁棒性

经验起点是：从 `6` 个 few-shot 示例开始测试准确率。

### 9. 适应模型更新

模型架构、训练数据和能力都会变化，因此提示词也应持续演化。

建议：

- 主动测试新模型版本
- 调整提示以利用新能力
- 把不同版本的提示记录下来

像 Vertex AI Studio 这类工具，非常适合做版本管理、测试和归档。

### 10. 试验输出格式

除了改输入格式，也应该试输出格式。

对于非创意类任务，例如：

- 抽取
- 选择
- 解析
- 排序
- 排名
- 分类

通常更适合要求模型返回结构化格式，比如：

- JSON
- XML

结构化输出的好处包括：

- 更方便程序消费
- 更容易排序和字段访问
- 能约束模型按结构生成
- 往往能减少幻觉

### 11. 和其他提示词工程师一起实验

如果一个提示任务很重要，最好不要只让一个人写。

让多个人按同一套最佳实践独立尝试，通常会得到性能差异明显的多个候选提示。这样更容易比较方案，而不是过早锁死在某一种写法上。

### 12. CoT 的专项实践

对于 Chain of Thought，有几个额外要点：

- 最终答案应放在推理之后
- 你需要能把“最终答案”从推理文本中可靠提取出来
- CoT 通常建议把 `temperature` 设为 `0`

原因是：CoT 本质上依赖逐步推理，而多数推理题最终只有一个正确答案，所以应尽量降低随机性。

### 13. 记录每一次提示尝试

这是白皮书反复强调的一点：一定要完整记录你的提示实验。

原因是输出会因为很多因素变化：

- 模型不同
- 采样参数不同
- 同一模型的不同版本
- 甚至同一提示在同一模型上重复运行，也可能有细微差异

作者建议至少记录这些字段：

- 名称与版本
- 本次尝试目标
- 使用的模型与版本
- Temperature
- Token Limit
- Top-K
- Top-P
- 完整提示词
- 输出结果

此外还建议记录：

- 本次结果是否 OK
- 用户或团队反馈
- 提示在 Vertex AI Studio 中的保存链接

如果你在做 RAG 系统，还应额外记录：

- query
- chunk 配置
- chunk 输出
- 被插入提示的检索内容

#### 推荐记录模板

| 字段 | 内容 |
| --- | --- |
| Name | 提示名称与版本 |
| Goal | 本次尝试的目标 |
| Model | 使用的模型及版本 |
| Temperature | `0` 到 `1` |
| Token Limit | 输出上限 |
| Top-K | 采样参数 |
| Top-P | 采样参数 |
| Prompt | 完整提示词 |
| Output | 输出结果 |

当你觉得提示词已经足够成熟时，再把它放进代码库。并且最好把提示与代码分文件保存，方便维护、迭代和测试。

理想状态下，提示词应该进入一个被工程化管理的系统里，通过自动化测试和评估来验证它是否真正具备泛化能力。

## 总结

这份白皮书系统介绍了提示词工程，并梳理了多种常见方法，包括：

- 零样本提示
- 单样本与少样本提示
- 系统提示
- 角色提示
- 上下文提示
- Step-back 提示
- 思维链（CoT）
- 自洽性（Self-consistency）
- 思维树（ToT）
- ReAct
- 自动提示工程（APE）

同时，文中也强调了生成式 AI 的实际挑战：当提示词不足够清晰时，模型很容易产生模糊、失真或不可靠的结果。

最后给出的最佳实践可以浓缩为一句话：

**提示词工程不是一次写对，而是持续试验、记录、比较、优化的工程过程。**

## 参考资料

1. Google, 2023, Gemini by Google. Available at: https://gemini.google.com
2. Google, 2024, Gemini for Google Workspace Prompt Guide. Available at: https://inthecloud.withgoogle.com/gemini-for-google-workspace-prompt-guide/dl-cd.html
3. Google Cloud, 2023, Introduction to Prompting. Available at: https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/introduction-prompt-design
4. Google Cloud, 2023, Text Model Request Body: Top-P & top-K sampling methods. Available at: https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/text#request_body
5. Wei, J., et al., 2023, Zero-Shot Fine-Tuned Language Models Are Zero-Shot Learners. Available at: https://arxiv.org/pdf/2109.01652.pdf
6. Google Cloud, 2023, Google Cloud Model Garden. Available at: https://cloud.google.com/model-garden
7. Brown, T., et al., 2023, Language Models are Few-Shot Learners. Available at: https://arxiv.org/pdf/2005.14165.pdf
8. Zheng, L., et al., 2023, Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models. Available at: https://openreview.net/pdf?id=3bq3jsvcQ1
9. Wei, J., et al., 2023, Chain of Thought Prompting. Available at: https://arxiv.org/pdf/2201.11903.pdf
10. Google Cloud Platform, 2023, Chain of Thought and React. Available at: https://github.com/GoogleCloudPlatform/generative-ai/blob/main/language/prompts/examples/chain_of_thought_react.ipynb
11. Wang, X., et al., 2023, Self-Consistency Improves Chain of Thought Reasoning in Language Models. Available at: https://arxiv.org/pdf/2203.11171.pdf
12. Yao, S., et al., 2023, Tree of Thoughts: Deliberate Problem Solving with Large Language Models. Available at: https://arxiv.org/pdf/2305.10601.pdf
13. Yao, S., et al., 2023, ReAct: Synergizing Reasoning and Acting in Language Models. Available at: https://arxiv.org/pdf/2210.03629.pdf
14. Google Cloud Platform, 2023, Advanced Prompting: Chain of Thought and React. Available at: https://github.com/GoogleCloudPlatform/applied-ai-engineering-samples/blob/main/genai-on-vertex-ai/advanced_prompting_training/cot_react.ipynb
15. Zhou, C., et al., 2023, Automatic Prompt Engineering: Large Language Models are Human-Level Prompt Engineers. Available at: https://arxiv.org/pdf/2211.01910.pdf
