## LLM Wiki 知识库
本项目是一个基于 [Karpathy 的 LLM Wiki 理念](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 构建的 Obsidian 知识库。

### 核心理念
将碎片化信息编译成**结构化、高度相互链接**的知识网络，便于 AI 辅助学习和研究。

### 目录结构
```
知识库文件夹 (LLM-Wiki-Vault)
├── assets                 ← 资源层：图片、PDF、附件
├── raw                    ← 原始资料层（文件处理后移到archive）
│   ├── 01-articles        ← 网页技术文章
│   ├── 02-papers          ← 论文、研报、PDF文档
│   ├── 03-transcripts     ← 视频、播客文本、会议记录
│   ├── 04-meeting_notes   ← 会议纪要
│   └── 09-archive         ← 已归档区：`/ingest` 执行后源文件档案层
│
├── wiki/                  ← 知识编译输出层（LLM完全写权限）
│   ├── index.md           ← 全局索引：wiki页面索引
│   ├── log.md             ← 行为流水线：Grep-friendly格式ingest/query历史
│   ├── concepts/          ← 抽象层：方法论、架构模式、第一性原理 
│   ├── entities/          ← 实体层：人名、公司、工具软件、项目 
│   ├── sources/           ← 摘要层：针对 raw 文件的一对一核心观点提炼 
│   └── syntheses/         ← 综合层：针对复杂提问生成的深度研究报告 
│
├── CLAUDE.md              ← 全局心智规范：定义语言协议、读写权限与Schema
└── .claude/               ← Claude Code官方配置目录
    └── 🛠️ skills/         ← Agent Skill中心
        ├── ingest/        ← 自定义：编译raw到wiki，并执行09-archive归档
        ├── query/         ← 自定义：检索wiki/index检索内容，生成带双链引用的回答
        ├── lint/          ← 自定义：知识体检，修复死链、补充 index、发现认知冲突
        ├── obsidian-cli/  ← Obsidian官方：调用Obsidian原生API进行检索、打开页面
        └── defuddle/      ← Obsidian官方：URL自动清理并转化Markdown存入raw/
```

### 使用方式
在 Obsidian 中打开本 vault，使用Claude Code或者Claudian插件执行操作。

### 常用命令
- `/query <问题>` — 在知识库中搜索相关内容
- `/ingest` — 将新的原始资料编译到知识库
- `/lint` — 检查知识库健康度（死链、孤儿页面）

## 知识来源
- Google Gemini API 官方文档
- Anthropic Claude 最佳实践
- 各机构发布的 Prompt Engineering 白皮书
- 学术论文（如 5C Prompt Contracts）
