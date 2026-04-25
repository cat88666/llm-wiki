---
type: synthesis
subtype: 策略
status: active
title: "LLM应用开发入门路线"
goal: "从AI使用者转型为LLM应用开发者的最短路径"
related_concepts:
  - 概念-RAG
  - 概念-Agent
  - 概念-Function-Calling
  - 框架-Context-Engineering
  - 方法-Prompt工程
related_entities:
  - 工具-LangFuse
  - 工具-Qdrant
  - 工具-vLLM
  - 工具-Reranker
  - 方法-LoRA
related_summaries:
  - 主题-RAG系统设计
  - 主题-Agent与工具调用
  - 主题-LLM生产化与评估
related_synthesis:
  - 对比-RAG-vs-微调-vs-Agent
  - 判断-LLM工程全栈知识体系
created: 2026-04-25
updated: 2026-04-25
---

# 策略：LLM 应用开发入门路线

> 核心原则：LLM 应用开发的门槛比传统软件开发低，但上线门槛比想象的高。先能做出来，再让它可靠。

---

## 转型的本质是什么

从"使用AI"到"开发AI应用"，核心差距只有两个：

1. **从单次对话到系统设计**：不是写一条好 Prompt，而是设计能在不同输入下稳定运行的 Prompt + 检索 + 工具编排体系
2. **从效果感知到可测量**：不是"感觉回答不错"，而是有评估集、有指标、有告警

其他技能（Python、API 调用、向量数据库）都是工具，很快能学会。

---

## 入门路线：三个阶段

### 阶段1：能接入（1-2周）

**目标**：会调用 LLM API，能构建最简单的对话应用。

核心技能：
- 用 Python 调用 OpenAI/Claude/Gemini API
- 理解 system prompt / user message / assistant 的角色分工
- 会做流式输出（streaming）
- 能计算 token 用量和成本

最小可用项目：一个命令行版的"个人助手"，能读取上下文、多轮对话、计算费用。

**需要掌握的 wiki 知识**：
- [[方法-Prompt工程]] — 系统提示词设计
- [[概念-Tokenizer]] — token 成本估算

---

### 阶段2：能用好（2-4周）

**目标**：构建一个实用的 RAG 知识库问答系统，这是 LLM 应用的标准起点。

**RAG 系统开发路径**：

```
1. 文档加载（PDF/Markdown/网页）
   └── 工具：LangChain Document Loaders / LlamaIndex

2. 文本分块（Chunking）
   └── 策略：语义分块优先，固定大小兜底，chunk size 需要实验

3. 向量化（Embedding）
   └── 工具：OpenAI text-embedding-3 / BGE（开源）

4. 存储与检索（Vector DB）
   └── 工具：[[工具-Qdrant]]（本地/云端都支持）

5. 混合检索（Hybrid Search）
   └── 向量检索 + BM25 + RRF 融合，参见 [[方法-混合检索]]

6. 精排（Reranker）
   └── [[工具-Reranker]]：Cross-Encoder，RAG 质量提升最大杠杆

7. Prompt 组装
   └── 最相关文档放首位，3-10 个 chunk，来源引用

8. 接入可观测性
   └── [[工具-LangFuse]]：每次请求都要有 trace
```

**里程碑**：做一个能回答自己私有文档问题的 RAG 应用，用 RAGAS 评估 Faithfulness > 3.5。

**需要掌握的 wiki 知识**：
- [[概念-RAG]] — 完整链路理解
- [[主题-RAG系统设计]] — 工程全链路
- [[对比-RAG-vs-微调-vs-Agent]] — 什么时候用 RAG

---

### 阶段3：能上线（持续）

**目标**：让系统可靠、可维护、可迭代。

**必须做的事**：

**评估体系**
- 建立 100+ 条评估集（正常问题 + 边界问题 + 恶意问题）
- RAG 用 RAGAS 四指标：Faithfulness / Answer Relevancy / Context Precision / Context Recall
- 每次改动 Prompt 或检索参数必须跑评估集

**可观测性**
- 接入 [[工具-LangFuse]]
- 最小追踪字段：trace_id / model_version / prompt_version / tokens / latency / feedback
- 建立成本告警（每日/每请求超阈值报警）

**安全与降级**
- 超时机制（每个 LLM 调用有 timeout）
- 输入检测（长度限制 + 注入检测）
- 降级策略（报错时返回安全的默认响应）
- 权限隔离（多租户场景必须在检索层做 metadata filter，不能依赖 Prompt）

参见 [[主题-LLM生产化与评估]] 上线前 8 项检查清单。

---

## 典型应用类型与对应路径

| 应用类型 | 核心技术 | 入门项目 |
|---------|---------|---------|
| 知识库问答 | RAG | 公司文档/个人笔记问答 |
| 智能客服 | RAG + 多轮对话管理 | FAQ 自动回复 |
| 代码助手 | Prompt工程 + Function Calling | 代码审查/生成工具 |
| 数据分析 | Text2SQL + Agent | 自然语言查数据库 |
| 自动化工作流 | Agent + 工具调用 | 邮件处理/日历管理 |
| 内容生成 | Prompt工程 + 微调 | 营销文案/报告生成 |

**建议第一个项目**：知识库问答——技术链路最清晰，效果可量化，是其他所有应用的基础。

---

## 工具选型建议（入门阶段）

| 层次 | 推荐工具 | 理由 |
|------|---------|------|
| LLM API | Claude Sonnet / GPT-4o | 能力均衡，API 文档完善 |
| 框架 | LangChain（起步）→ 直接 API（深入） | LangChain 上手快，但生产上有时太重 |
| 向量数据库 | [[工具-Qdrant]] | 支持混合检索，本地部署方便 |
| Embedding | BGE-M3（开源）/ OpenAI text-embedding-3 | BGE 中文效果好，OpenAI 方便集成 |
| Reranker | BGE-Reranker / [[工具-Reranker]] | 最高 ROI 的质量提升 |
| 可观测性 | [[工具-LangFuse]] | 开源，功能完整，本地可部署 |
| 推理服务 | [[工具-vLLM]]（自部署模型时） | 显存利用率高，PagedAttention |

---

## 常见误区

**误区1：先学框架（LangChain/LlamaIndex）**
→ 框架是封装，不理解底层机制，出问题无法调试。先理解 RAG 的每个步骤，再用框架提速。

**误区2：跳过评估直接上线**
→ 没有评估集 = 不知道系统在哪里失效 = 每次改动都是赌博。参见 [[主题-LLM生产化与评估]]。

**误区3：过早引入微调**
→ 80% 的问题可以通过更好的 Prompt + RAG 解决。微调是最后手段，不是起点。参见 [[对比-RAG-vs-微调-vs-Agent]]。

**误区4：单独优化某个组件**
→ RAG 系统是管道，失效通常在最上游。诊断顺序：文档解析 → 分块 → 检索 → Reranker → Prompt → 模型。

---

## 下一步（知识库待补充）

以下方向是当前 wiki 的空白区域，后续摄入资料后可扩展：

- LangChain / LlamaIndex 工程实践
- Text2SQL 应用架构
- 多 Agent 编排模式（Orchestrator-Worker）
- LLM 应用的测试策略（单元测试 Prompt，集成测试 RAG 管道）
- 私有化部署（vLLM + 开源模型）

---

## 参考页面

- [[判断-LLM工程全栈知识体系]] — 全局知识地图
- [[对比-RAG-vs-微调-vs-Agent]] — 路径选型
- [[策略-LLM核心知识掌握路径]] — 知识学习顺序
- [[策略-AI高效使用技巧全景]] — 使用技巧基础
