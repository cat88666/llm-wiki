---
type: entity
status: active
entity_type: tool
name: "vLLM"
aliases: ["vllm", "PagedAttention"]
domain: "LLM推理服务 / 生产部署"
related: ["KV-Cache", "分布式训练", "多头注意力"]
sources:
  - ../raw/llm-engineering/03-理论-LLM核心机制.md
  - ../raw/llm-engineering/10-LLM原理.md
  - ../raw/llm-engineering/07-工程-生产化.md
created: 2026-04-22
updated: 2026-04-22
lint_notes: ""
---

# vLLM

> vLLM 是一个高吞吐量的 LLM 推理和服务框架，其核心创新 PagedAttention 通过借鉴操作系统虚拟内存的分页机制管理 KV Cache，将显存利用率从约 60% 提升到 90%+，是目前开源 LLM 推理服务的事实标准。

## 基本信息
- **类型**：开源推理框架（Python，Apache 2.0 许可）
- **领域**：LLM 推理服务、批量推理、在线 API 服务
- **关联时间**：2023 年 UC Berkeley 团队发布，迅速成为生产推理首选

## 核心特点
- **PagedAttention**：将 KV Cache 划分为固定大小的物理块（默认 16 tokens/块），用逻辑块→物理块的映射表管理分配；不预留连续最大显存，按实际需要分配，消除内部碎片；显存利用率从约 60%（传统连续分配）提升到 90%+
- **Prefix Caching（前缀缓存）**：多个请求若共享相同前缀（如相同 System Prompt），其 KV Cache 物理块可以共享，大幅减少重复计算；对 RAG 系统中固定前缀的场景特别有效
- **Continuous Batching（连续批处理）**：传统静态 batching 需要等最长序列生成完才释放，vLLM 的连续 batching 允许新请求在已有请求生成过程中插入，大幅提升 GPU 利用率
- **TP 推理支持**：内置张量并行支持（TP=2/4/8），多 GPU 推理时自动分配注意力头，同步通信使用 NCCL
- **兼容 OpenAI API**：vLLM 的服务端实现了 `/v1/completions` 和 `/v1/chat/completions` 接口，可作为 OpenAI 的本地 drop-in replacement

## 在知识库中的出现
- [[KV-Cache]] concept：PagedAttention 是 KV Cache 显存管理的核心工程创新，解决传统连续分配的碎片化和利用率低问题
- [[LLM核心机制与推理优化]] summary：vLLM 是 KV Cache 优化的工程实现代表，将理论优化转化为生产可用的工具
- [[LLM生产化与评估]] summary：vLLM 是 LLM 生产化推理基础设施的首选方案

## 相关实体
- TensorRT-LLM：NVIDIA 出品的推理优化框架，深度优化 GPU 内核，峰值性能更高，但工程配置复杂，灵活性低于 vLLM
- Ollama：面向本地开发者的轻量推理工具，配置简单，但吞吐量和扩展性远低于 vLLM
- SGLang：斯坦福团队开发的推理框架，在复杂结构化输出场景（如多轮函数调用）有性能优势
- [[LangFuse]]：可观测性工具，与 vLLM 组合提供完整的生产推理监控链路

## 来源
- [03-理论-LLM核心机制.md](../raw/llm-engineering/03-理论-LLM核心机制.md)
- [10-LLM原理.md](../raw/llm-engineering/10-LLM原理.md)
- [07-工程-生产化.md](../raw/llm-engineering/07-工程-生产化.md)
