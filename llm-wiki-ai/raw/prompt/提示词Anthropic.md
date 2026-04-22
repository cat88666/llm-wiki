---
title: "提示词最佳实践"
source: "https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices"
author:
published:
created: 2026-04-12
description: "面向 Claude 最新模型的提示词工程完整指南，涵盖清晰表达、示例、XML 结构化、思考模式与智能体系统。"
tags:
  - "clippings"
---
这是使用 Claude 最新模型进行提示词工程的单一参考文档，包括 Claude Opus 4.6、Claude Sonnet 4.6 和 Claude Haiku 4.5。本文涵盖基础技巧、输出控制、工具使用、思考能力和智能体系统。你可以直接跳到与你当前场景最匹配的部分。

关于模型能力概览，请参阅 [models overview](https://platform.claude.com/docs/en/about-claude/models/overview)。关于 Claude 4.6 的更新内容，请参阅 [What's new in Claude 4.6](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-6)。关于迁移指导，请参阅 [Migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide)。

## 通用原则

### 清晰且直接

Claude 对清晰、明确的指令响应很好。越具体地说明你想要的输出，效果通常越好。如果你希望它“超出预期”，应当明确提出，而不是指望模型从模糊提示中自行推断。

把 Claude 想象成一位非常聪明但刚入职的新员工，他并不了解你的规范和工作流。你解释得越精确，结果就越好。

**黄金法则：** 把你的提示词给一位几乎不了解任务背景的同事，让他照着做。如果他会困惑，Claude 也一样会困惑。

- 明确说明期望的输出格式和约束。
- 如果步骤顺序或完整性重要，请用编号列表或项目符号按顺序写出指令。

### 添加上下文以提升效果

为你的指令补充上下文或动机，例如告诉 Claude 为什么某种行为很重要，能帮助它更好地理解你的目标，并给出更有针对性的结果。

Claude 足够聪明，能从你的解释中进行泛化。

### 有效使用示例

示例是控制 Claude 输出格式、语气和结构最可靠的方法之一。少量精心设计的示例（即 few-shot 或 multishot prompting）就能显著提升准确性和一致性。

添加示例时，建议做到：

- **相关：** 尽量贴近你的真实使用场景。
- **多样：** 覆盖边缘情况，并避免让 Claude 学到无意中的模式。
- **结构化：** 用 `<example>` 标签包裹示例（多个示例用 `<examples>`），让 Claude 能区分示例与指令。

通常加入 3 到 5 个示例效果最好。你也可以让 Claude 评估这些示例是否足够相关和多样，或基于初始示例生成更多示例。

### 用 XML 标签组织提示词

XML 标签可以帮助 Claude 无歧义地解析复杂提示，特别是在你的提示里混合了指令、上下文、示例和变量输入时。把不同类型的内容分别放进独立标签中（例如 `<instructions>`、`<context>`、`<input>`），可以减少误解。

最佳实践：

- 在各个提示中使用一致、描述性强的标签名。
- 当内容存在天然层级时使用嵌套标签（例如多个文档放在 `<documents>` 中，每个文档再放在 `<document index="n">` 中）。

### 给 Claude 一个角色

在系统提示中设置角色，能够让 Claude 的行为和语气更聚焦于你的使用场景。哪怕只是一句话，也会有明显差异：

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    system="You are a helpful coding assistant specializing in Python.",
    messages=[
        {"role": "user", "content": "How do I sort a list of dictionaries by key?"}
    ],
)
print(message.content)
```

### 长上下文提示

处理大型文档或富数据输入（2 万 token 以上）时，需要更仔细地组织提示词，才能获得最佳结果：

- **把长文数据放在前面：** 将长文档和主要输入放在提示词顶部，位于问题、指令和示例之前。这能显著提升所有模型上的效果。
  在测试中，把查询放在末尾可让回复质量最高提升 30%，尤其是在复杂的多文档输入场景中。
- **用 XML 标签组织文档内容和元数据：** 使用多个文档时，用 `<document>` 包裹每个文档，并在其中用 `<document_content>`、`<source>` 以及其他元数据子标签明确结构。
- **让回答以引用为依据：** 对长文档任务，可以先让 Claude 引用相关片段，再执行实际任务。这能帮助它在大量噪声中抓住关键信息。

### 模型自我认知

如果你希望 Claude 在应用中正确识别自己的身份，或使用特定 API 模型字符串，可以加入：

```
The assistant is Claude, created by Anthropic. The current model is Claude Opus 4.6.
```

对于需要显式指定模型字符串的 LLM 应用，可以写成：

```
When an LLM is needed, please default to Claude Opus 4.6 unless the user requests otherwise. The exact model string for Claude Opus 4.6 is claude-opus-4-6.
```

## 输出与格式

### 沟通风格与详细程度

相较于以前的模型，Claude 最新模型的沟通风格更简洁、更自然：

- **更直接、更贴近事实：** 倾向于给出基于事实的进度说明，而不是自我表扬式更新
- **更像对话：** 表达更流畅、更口语化，没有那么“机器感”
- **更不冗长：** 除非你特别要求，否则可能为提高效率而省略长篇总结

这意味着 Claude 在工具调用后，可能不会再口头总结，而是直接进入下一步。如果你希望它显式说明做了什么，可以加入：

```
After completing a task that involves tool use, provide a quick summary of the work you've done.
```

下面几种方式尤其适合控制输出格式：

1. **告诉 Claude 应该做什么，而不是不要做什么**
   - 不要写：“Do not use markdown in your response”
   - 可以改成：“Your response should be composed of smoothly flowing prose paragraphs.”
2. **使用 XML 形式的格式指示**
   - 例如：“Write the prose sections of your response in <smoothly_flowing_prose_paragraphs> tags.”
3. **让提示风格匹配你想要的输出风格**
   你提示词本身的格式会影响 Claude 的回应风格。如果你仍然发现输出格式不好控制，就让提示词尽可能接近目标输出风格。比如，去掉提示词中的 markdown，通常可以减少输出中的 markdown。
4. **为特定格式偏好提供细致说明**
   如果你想更严格控制 markdown 和格式使用，可以明确这样写：

```
<avoid_excessive_markdown_and_bullet_points>
When writing reports, documents, technical explanations, analyses, or any long-form content, write in clear, flowing prose using complete paragraphs and sentences. Use standard paragraph breaks for organization and reserve markdown primarily for `inline code`, code blocks (```...```), and simple headings (###, and ###). Avoid using **bold** and *italics*.

DO NOT use ordered lists (1. ...) or unordered lists (*) unless : a) you're presenting truly discrete items where a list format is the best option, or b) the user explicitly requests a list or ranking

Instead of listing items with bullets or numbers, incorporate them naturally into sentences. This guidance applies especially to technical writing. Using prose instead of excessive formatting will improve user satisfaction. NEVER output a series of overly short bullet points.

Your goal is readable, flowing text that guides the reader naturally through ideas rather than fragmenting information into isolated points.
</avoid_excessive_markdown_and_bullet_points>
```

### LaTeX 输出

Claude Opus 4.6 默认会在数学表达式、公式和技术解释中使用 LaTeX。如果你更希望使用纯文本，可以在提示词中加入：

```
Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents).
```

### 文档创建

Claude 最新模型在生成演示文稿、动画和视觉文档方面表现出色，既有创造力，也能很好地遵循指令。大多数情况下，模型第一次生成就能给出完成度很高、可以直接使用的结果。

为了获得更好的文档生成效果，可以这样写：

```
Create a professional presentation on [topic]. Include thoughtful design elements, visual hierarchy, and engaging animations where appropriate.
```

从 Claude 4.6 系列模型以及 [Claude Mythos Preview](https://anthropic.com/glasswing) 开始，最后一轮 assistant 的预填充回复（prefilled responses）已不再受支持。在 Mythos Preview 中，请求若包含预填充 assistant 消息会返回 400 错误。模型的智能水平和指令遵循能力已经提升到大多数 prefill 用例不再需要它。旧模型仍继续支持 prefill，在对话其他位置加入 assistant 消息也不受影响。

下面是常见的 prefill 场景以及如何迁移：

## 工具使用

### 工具调用

Claude 最新模型经过训练，能够更精确地遵循指令，因此如果你希望它使用特定工具，就要明确说出来。比如，你说“can you suggest some changes”，Claude 有时会只是给出建议，而不是直接修改，即使你的真实意图是让它动手做。

如果你希望 Claude 默认采取行动，需要把话说得更直接：

若你希望 Claude 更主动、默认直接执行，可以在系统提示中加入：

```
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

相反，如果你希望模型默认更谨慎，不要一上来就修改或实现，只有在被明确要求时才执行，可以这样引导：

```
<do_not_act_before_instructions>
Do not jump into implementatation or changes files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

Claude Opus 4.5 和 Claude Opus 4.6 对系统提示的响应也比之前的模型更强。如果你以前为了减少工具或技能“触发不足”而设计了一些激进提示，那么这些模型现在可能会“触发过度”。解决方法是把语气放缓。比如，以前你可能会写“CRITICAL: You MUST use this tool when...”，现在更适合写成“Use this tool when...”。

### 优化并行工具调用

Claude 最新模型很擅长并行执行工具调用。它们会：

- 在研究任务中同时发起多个探索性搜索
- 一次读取多个文件，更快建立上下文
- 并行执行多个 bash 命令（甚至可能让系统性能成为瓶颈）

这种行为很容易通过提示词调节。即使不特别提示，模型在并行工具调用上的成功率已经很高；但如果你想把它提升到接近 100%，或者想降低并行激进程度，也可以显式说明：

最大化并行效率的示例提示：

```
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>
```

降低并行执行的示例提示：

```
Execute operations sequentially with brief pauses between each step to ensure stability.
```

## 思考与推理

### 过度思考与过度彻底

Claude Opus 4.6 会比以前的模型做更多前置探索，尤其是在较高 `effort` 设置下。这些前置工作通常有助于优化最终结果，但模型也可能在没有被要求的情况下搜集大量上下文，或者并行展开多条研究线索。如果你之前的提示词是为了鼓励模型更彻底，那么在 Claude Opus 4.6 上应当重新调节：

- **用更有针对性的指令替代笼统默认。** 不要写“Default to using [tool]”，而改成“Use [tool] when it would enhance your understanding of the problem.”
- **去掉过度提示。** 之前容易触发不足的工具，如今通常已经能正常触发；像“If in doubt, use [tool]”这样的指令，会导致过度触发。
- **把 effort 当作兜底调节杆。** 如果 Claude 仍然太激进，就降低 `effort` 设置。

在某些情况下，Claude Opus 4.6 会思考很久，这会增加 thinking token 并拖慢响应。如果你不希望如此，可以加入明确指令来约束它的推理，或者降低 `effort` 设置，从而减少整体思考和 token 消耗。

```
When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning. If you're weighing two approaches, pick one and see it through. You can always course-correct later if the chosen approach fails.
```

如果你需要对思考成本设定硬上限，那么带 `budget_tokens` 上限的 extended thinking 在 Opus 4.6 和 Sonnet 4.6 上仍然可用，但已被弃用。更推荐降低 [effort](https://platform.claude.com/docs/en/build-with-claude/effort) 设置，或者配合 [adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking) 使用 `max_tokens` 作为硬限制。

### 利用 thinking 与交错 thinking 能力

Claude 最新模型具备思考能力，这对“工具调用后进行反思”或“复杂多步推理”这类任务尤其有帮助。你可以引导它进行初始思考，或在过程中穿插思考，以获得更好结果。

Claude Opus 4.6 和 Claude Sonnet 4.6 使用 [adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)（`thinking: {type: "adaptive"}`），也就是 Claude 会动态决定何时思考、思考多少。它主要依据两个因素校准思考量：`effort` 参数和查询复杂度。`effort` 越高，思考越多；问题越复杂，思考也越多。对于不需要思考的简单问题，模型会直接回答。内部评测显示，自适应思考在效果上稳定优于 extended thinking。若你想得到更聪明的回答，建议迁移到 adaptive thinking。

对于需要智能体行为的工作负载，例如多步工具调用、复杂编码任务以及长周期 agent loop，推荐使用 adaptive thinking。旧模型则使用带 `budget_tokens` 的手动 thinking 模式。

你可以这样引导 Claude 的思考行为：

```
After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.
```

adaptive thinking 是否触发，也可以通过提示词来调节。如果你发现模型思考得比想要的更频繁，这种情况通常会在大型或复杂系统提示下出现，可以加入如下引导：

```
Extended thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multi-step reasoning. When in doubt, respond directly.
```

如果你正从带 `budget_tokens` 的 [extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) 迁移过来，需要把 thinking 配置改掉，并把预算控制迁移到 `effort`：

**之前（extended thinking，旧模型）：**

```python
client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=64000,
    thinking={"type": "enabled", "budget_tokens": 32000},
    messages=[{"role": "user", "content": "..."}],
)
```

**之后（adaptive thinking）：**

```python
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # or max, medium, low
    messages=[{"role": "user", "content": "..."}],
)
```

如果你没有使用 extended thinking，则不需要做任何修改。省略 `thinking` 参数时，thinking 默认关闭。

- **优先使用一般性指导，而不是规定死步骤。** 像“think thoroughly”这样的提示，通常比人工编写的逐步计划更能激发高质量推理。Claude 的推理过程常常超出人类事先能规定的流程。
- **multishot 示例可以与 thinking 搭配使用。** 你可以在 few-shot 示例中加入 `<thinking>` 标签，展示所需的推理模式；Claude 会把这种风格泛化到自己的 extended thinking 块中。
- **手写 CoT 可作为兜底方案。** 当 thinking 关闭时，你依然可以通过要求 Claude 分步思考来促进推理。使用 `<thinking>` 和 `<answer>` 这样的结构化标签，有助于把推理过程和最终输出清晰分开。
- **让 Claude 自检。** 在提示末尾追加类似“Before you finish, verify your answer against [test criteria].”的话，能稳定减少错误，尤其在编程和数学任务中效果明显。

当 extended thinking 被关闭时，Claude Opus 4.5 对 “think” 及其变体特别敏感。此时可以考虑改用 “consider”、“evaluate” 或 “reason through” 之类的表达。

关于 thinking 能力的更多信息，请参阅 [Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) 和 [Adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)。

## 智能体系统

### 长周期推理与状态跟踪

Claude 最新模型在长周期推理任务中表现出色，尤其擅长状态跟踪。Claude 会通过关注增量进展，在同一时间稳定推进少数几件事，而不是试图一次做完所有事情。这种能力在多个上下文窗口或多轮任务迭代中尤为明显：Claude 可以处理复杂任务、保存状态，并在新的上下文窗口中继续推进。

#### 上下文感知与多窗口工作流

Claude 4.6 和 Claude 4.5 模型具备 [context awareness](https://platform.claude.com/docs/en/build-with-claude/context-windows#context-awareness-in-claude-sonnet-4-6-sonnet-4-5-and-haiku-4-5)，可以在对话过程中追踪剩余上下文窗口（即 “token budget”）。这使 Claude 能根据可用空间，更有效地执行任务和管理上下文。

**管理上下文限制：**

如果你在一个会压缩上下文、或允许把上下文保存到外部文件的 agent 框架中使用 Claude（例如 Claude Code），建议把这件事明确写进提示词里，这样 Claude 才会据此行动。否则，当它接近上下文上限时，有时会自然倾向于“收尾”。下面是一个示例提示：

```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

[memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) 与 context awareness 天然互补，适合做无缝的上下文切换。

#### 多上下文窗口工作流

对于会跨越多个上下文窗口的任务：

1. **第一次上下文窗口使用不同提示词：** 第一轮上下文窗口可以先搭好框架（写测试、创建初始化脚本），之后的窗口再围绕 todo 列表迭代。
2. **让模型以结构化格式写测试：** 开始工作前先让 Claude 创建测试，并用结构化格式跟踪（例如 `tests.json`）。这会显著提升后续长期迭代能力。还可以提醒 Claude 测试的重要性：“删除或修改测试是不可接受的，因为这可能导致功能遗漏或引入 bug。”
3. **准备提升效率的工具：** 鼓励 Claude 生成初始化脚本（如 `init.sh`），用于优雅地启动服务、运行测试套件和 linter。这样在新的上下文窗口继续工作时，可以避免重复劳动。
4. **重新开始 vs 压缩：** 当上下文窗口被清空时，可以考虑直接从全新的上下文窗口重新开始，而不是使用压缩。Claude 最新模型非常善于从本地文件系统自行发现状态。在某些场景里，这比压缩更划算。你需要明确规定它该如何开始：
   - “Call pwd; you can only read and write files in this directory.”
   - “Review progress.txt, tests.json, and the git logs.”
   - “Manually run through a fundamental integration test before moving on to implementing new features.”
5. **提供验证工具：** 随着自主任务时长增加，Claude 需要在没有持续人工反馈的情况下自行验证正确性。像 Playwright MCP server 或 computer use 这类 UI 测试能力会很有帮助。
6. **鼓励充分利用上下文：** 提示 Claude 在推进下一部分前，尽可能完整地完成当前组件：

```
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

#### 状态管理最佳实践

- **结构化状态数据使用结构化格式：** 当你要跟踪结构化信息（如测试结果或任务状态）时，使用 JSON 或其他结构化格式，方便 Claude 理解 schema 要求。
- **进度记录使用非结构化文本：** 自由文本形式的进度笔记很适合记录一般进展和上下文。
- **用 git 跟踪状态：** Git 能记录已完成的工作，并提供可恢复的检查点。Claude 最新模型尤其善于借助 git 在多次会话间跟踪状态。
- **强调增量推进：** 明确要求 Claude 跟踪自己的进展，并专注于一步步推进。

### 在自主性与安全性之间取得平衡

如果没有额外引导，Claude Opus 4.6 可能会执行一些难以撤销、或会影响共享系统的操作，例如删除文件、强制推送，或向外部服务发消息。如果你希望 Claude Opus 4.6 在执行潜在高风险动作前先确认，可以在提示词中加入：

```
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example, don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

### 研究与信息收集

Claude 最新模型展现出很强的智能体式搜索能力，能够有效地从多个来源中查找并综合信息。为了获得更好的研究效果：

1. **提供清晰的成功标准：** 明确什么样的答案才算成功回答了你的研究问题。
2. **鼓励来源校验：** 要求 Claude 在多个来源之间交叉验证信息。
3. **复杂研究任务使用结构化方法：**

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

这种结构化方法让 Claude 能在任意规模的语料中查找、综合并迭代审视几乎任何信息。

### 子智能体编排

Claude 最新模型在原生子智能体编排方面有显著提升。它们能够识别哪些任务适合委派给专门的子智能体，并且往往会主动这样做，而不需要你显式下达命令。

为了利用这种能力：

1. **确保子智能体工具定义清晰：** 让相关子智能体工具可用，并在工具定义中描述清楚。
2. **让 Claude 自然编排：** Claude 通常会在合适的时候自行委派，而不需要显式指示。
3. **留意过度使用：** Claude Opus 4.6 对子智能体有明显偏好，有时会在直接方法更简单时仍然创建子智能体。例如，针对代码探索，它可能会启动子智能体，而实际上直接 `grep` 往往更快、更够用。

如果你发现子智能体用得过多，可以显式补充子智能体适用与不适用的边界：

```
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating.
```

### 链式复杂提示

借助 adaptive thinking 和子智能体编排，Claude 已能在内部处理大多数多步推理。但在你需要检查中间输出，或强制执行某种特定流水线结构时，显式的 prompt chaining（把一个任务拆成顺序 API 调用）仍然有价值。

最常见的链式模式是 **自我纠错**：先生成草稿，再让 Claude 按标准审阅，然后根据审阅结果进行修改。每一步都是独立 API 调用，因此你可以在任意节点记录、评估或分叉。

### 在智能体编码中减少文件创建

Claude 最新模型有时会为了测试和迭代新建文件，尤其是在编程任务中。这样做可以把文件，特别是 Python 脚本，当成一种“临时草稿板”，再在最终阶段输出结果。对于智能体编码场景，这种临时文件策略有时确实能提升效果。

如果你希望尽量减少净新增文件，可以这样提示 Claude：

```
If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task.
```

### 过度积极

Claude Opus 4.5 和 Claude Opus 4.6 有时会“过度设计”，例如创建额外文件、增加不必要的抽象，或引入并未被要求的灵活性。如果你不希望出现这种行为，可以明确要求解法保持最小化。

例如：

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:

- Scope: Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability.

- Documentation: Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.

- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs).

- Abstractions: Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical future requirements. The right amount of complexity is the minimum needed for the current task.
```

### 避免只盯着通过测试和硬编码

Claude 有时会过于专注于“让测试通过”，而牺牲更通用的解决方案；或者在复杂重构时，用辅助脚本之类的变通手段代替直接使用标准工具。为了避免这种行为、确保方案稳健且可泛化，可以这样提示：

```
Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable.
```

### 在智能体编码中尽量减少幻觉

Claude 最新模型更不容易产生幻觉，能基于代码给出更准确、更扎实、更智能的回答。如果你想进一步强化这种行为、尽量减少幻觉，可以加入：

```
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>
```

## 按能力划分的建议

### 更强的视觉能力

相较于以往的 Claude 模型，Claude Opus 4.5 和 Claude Opus 4.6 的视觉能力更强。它们在图像处理和数据提取任务上表现更好，尤其是在上下文中包含多张图片时。这些提升同样体现在 computer use 上，模型能更可靠地理解截图和 UI 元素。你还可以把视频拆成帧后交给这些模型分析。

一个被证明很有效的进一步增强手段，是给 Claude 提供一个裁剪工具或 [skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)。测试表明，当 Claude 能对图像相关区域“放大查看”时，图像评测表现会稳定提升。Anthropic 还提供了一个关于裁剪工具的 [cookbook](https://platform.claude.com/cookbook/multimodal-crop-tool)。

### 前端设计

Claude Opus 4.5 和 Claude Opus 4.6 在构建复杂、真实世界的 Web 应用以及高质量前端设计方面表现突出。但如果没有额外引导，模型往往会落回通用模板，形成用户所谓的 “AI slop” 审美。若想让前端更鲜明、更有创意、更让人眼前一亮：

关于如何改进前端设计的详细指南，请参阅这篇博客：[improving frontend design through skills](https://www.claude.com/blog/improving-frontend-design-through-skills)。

下面是一段可用于鼓励更好前端设计的系统提示：

```
<frontend_aesthetics>
You tend to converge toward generic, "on distribution" outputs. In frontend design, this creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive frontends that surprise and delight.

Focus on:
- Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics.
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw from IDE themes and cultural aesthetics for inspiration.
- Motion: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions.
- Backgrounds: Create atmosphere and depth rather than defaulting to solid colors. Layer CSS gradients, use geometric patterns, or add contextual effects that match the overall aesthetic.

Avoid generic AI-generated aesthetics:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Clichéd color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the context. Vary between light and dark themes, different fonts, different aesthetics. You still tend to converge on common choices (Space Grotesk, for example) across generations. Avoid this: it is critical that you think outside the box!
</frontend_aesthetics>
```

你也可以参考 [完整 skill 定义](https://github.com/anthropics/claude-code/blob/main/plugins/frontend-design/skills/frontend-design/SKILL.md)。

## 迁移注意事项

从更早版本迁移到 Claude 4.6 模型时：

1. **更具体地描述期望行为：** 尽量明确说明你希望在输出中看到什么。
2. **用修饰语加强你的指令：** 适当加入能鼓励 Claude 提升质量和细节的修饰语，有助于更好地塑造输出。例如，不要只写 “Create an analytics dashboard”，而可以改成 “Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.”
3. **显式请求具体功能：** 如果你需要动画或交互元素，应明确提出。
4. **更新 thinking 配置：** Claude 4.6 模型使用 [adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)（`thinking: {type: "adaptive"}`），而不是使用 `budget_tokens` 的手动 thinking。用 [effort 参数](https://platform.claude.com/docs/en/build-with-claude/effort) 控制思考深度。
5. **迁移 away from prefilled responses：** 从 Claude 4.6 开始，最后一轮 assistant 的预填充回复已弃用。替代方案详见 [Migrating away from prefilled responses](#migrating-away-from-prefilled-responses)。
6. **调整“防偷懒”提示：** 如果你以前的提示词会鼓励模型更彻底、或更激进地使用工具，现在应当减弱这类引导。Claude 4.6 模型明显更主动，以前为了旧模型设计的提示，在这里可能会导致过度触发。

详细迁移步骤请参阅 [Migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide)。

### 从 Claude Sonnet 4.5 迁移到 Claude Sonnet 4.6

Claude Sonnet 4.6 默认 `effort` 等级为 `high`，而 Claude Sonnet 4.5 当时还没有 `effort` 参数。因此，当你从 Claude Sonnet 4.5 迁移到 Claude Sonnet 4.6 时，应考虑同步调整 `effort`。如果不显式设置，你可能会因为默认高 effort 而遇到更高延迟。

**推荐的 effort 设置：**

- **Medium** 适合大多数应用
- **Low** 适合高吞吐或对延迟敏感的工作负载
- 在 medium 或 high effort 下，设置较大的最大输出 token 预算（推荐 64k），给模型足够空间进行思考和执行

**什么时候改用 Opus 4.6：** 如果你处理的是最难、最长周期的问题，例如大规模代码迁移、深度研究或长时间自主工作，那么 Opus 4.6 仍然是更合适的选择。Sonnet 4.6 更适合那些看重响应速度和成本效率的工作负载。

#### 如果你没有使用 extended thinking

如果你在 Claude Sonnet 4.5 上没有使用 extended thinking，那么迁移到 Claude Sonnet 4.6 后也可以继续不使用。你只需要显式设置适合自己场景的 effort 等级。在 `low` effort 且禁用 thinking 的情况下，通常可以得到与 Claude Sonnet 4.5 无 extended thinking 相当或更好的效果。

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8192,
    thinking={"type": "disabled"},
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)
```

#### 如果你正在使用 extended thinking

如果你在 Claude Sonnet 4.5 上配合 `budget_tokens` 使用 extended thinking，那么在 Claude Sonnet 4.6 上它仍可工作，但已被弃用。建议迁移到 [adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)，并配合 [effort 参数](https://platform.claude.com/docs/en/build-with-claude/effort) 使用。

##### 迁移到 adaptive thinking

adaptive thinking 特别适合以下工作负载模式：

- **自主多步智能体：** 比如把需求转成可运行软件的编码 agent、数据分析流水线、独立运行多步流程的 bug 查找任务。adaptive thinking 允许模型在每一步按需校准推理强度，在更长轨迹上保持方向。对于这类工作负载，建议从 `high` effort 开始；如果你担心延迟或 token 消耗，再降到 `medium`。
- **computer use agents：** Claude Sonnet 4.6 在使用 adaptive 模式时，在 computer use 评测中取得了同类最佳准确率。
- **双峰工作负载：** 同时包含简单任务和复杂任务；adaptive 会在简单查询上跳过思考，在复杂查询上深入推理。

使用 adaptive thinking 时，建议在你的真实任务上评估 `medium` 和 `high` effort。最佳设置取决于你的工作负载在质量、延迟和 token 用量之间的权衡。

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}],
)
```

##### 迁移期间暂时保留 budget_tokens

如果你在迁移过程中必须暂时保留 `budget_tokens`，那么大约 16k token 的预算通常能为更难的问题留出足够余量，同时又不至于 token 使用失控。不过，这种配置已经弃用，并会在未来模型版本中移除。

**针对编码场景**（agentic coding、重工具工作流、代码生成），建议从 `medium` effort 开始：

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16384,
    thinking={"type": "enabled", "budget_tokens": 16384},
    output_config={"effort": "medium"},
    messages=[{"role": "user", "content": "..."}],
)
```

**针对聊天和非编码场景**（聊天、内容生成、搜索、分类），建议从 `low` effort 开始：

```python
client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8192,
    thinking={"type": "enabled", "budget_tokens": 16384},
    output_config={"effort": "low"},
    messages=[{"role": "user", "content": "..."}],
)
```

这个页面对你有帮助吗？
