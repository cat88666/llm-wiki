---
name: bug
description: 基于 Kibana MCP + Yearning MCP + 代码仓库，对用户提供的 bug 描述执行全链路 AI 排障，输出结构化排障报告。触发命令：/bug <bug描述>
user-invocable: true
---

# bug 技能

## 核心目标

接收用户提供的 bug 描述，依次调用 Kibana MCP 查日志、Yearning MCP 查数据库、分析代码，最终输出结构化排障报告。AI 承担信息收集与分析，人类只做决策。

## 触发场景

- 用户执行 `/bug <bug描述>`
- 用户要求排查某个线上问题、分析 bug 根因

---

## 执行流水线

### 步骤 -1：环境自检与自动安装（每次启动必须执行）

在做任何排障之前，先检查 Kibana MCP 和 Yearning MCP 是否可用。
**判断方式**：尝试调用任意 Kibana/Yearning MCP 工具；若工具不存在（MCP server 未注册），则进入自动安装流程。

#### 1. 检查 Kibana MCP

```bash
# 检查 mcp-server-kibana 是否已安装
which mcp-server-kibana
```

- **未安装**：运行 `npm install -g @tocharianou/mcp-server-kibana`
- **已安装但未注册**：进入 Cookie 提取流程（见下）
- **已注册且可用**：跳过

**Cookie 提取流程**（仅在需要时执行）：

```bash
# 确认 Chrome Cookie 数据库位置
ls ~/Library/Application\ Support/Google/Chrome/Default/Cookies
ls ~/Library/Application\ Support/Google/Chrome/Profile\ 1/Cookies

# 从 macOS Keychain 取解密密钥
security find-generic-password -w -s "Chrome Safe Storage"
```

从 SQLite 数据库读取 `.dxit999.com` 的 `auth_pro` cookie 和 `new-dev-eslog.dxit999.com` 的 `sid` cookie，用 AES-CBC 解密（密钥 = PBKDF2(keychain_key, 'saltysalt', 1003 iter, 16 bytes)）。**注意：解密结果需要剥离前 32 字节前缀**，否则请求头非法。

提取到 cookie 后，注册到 Claude Code：

```bash
claude mcp add kibana mcp-server-kibana -s user \
  -e KIBANA_URL=https://new-dev-eslog.dxit999.com \
  -e "KIBANA_COOKIES=auth_pro=<值>; sid=<值>" \
  -e KIBANA_DEFAULT_SPACE=default \
  -e NODE_TLS_REJECT_UNAUTHORIZED=0
```

#### 2. 检查 Yearning MCP

```bash
# 检查编译产物是否存在
ls ~/.codex/mcpserver/yearning/dist/index.js
```

- **不存在，但 zip 在本地**：让用户指定 zip 路径，解压后执行：
  ```bash
  cd <解压目录> && npm install && npm run build
  ```
- **已编译但未注册**：读取 `~/.codex/config.toml` 中的 `[mcp_servers.yearning]` 节取出配置，然后注册：

  ```bash
  claude mcp add yearning node -s user \
    -e YEARNING_URL=<值> \
    -e YEARNING_SOURCE_ID=<值> \
    -e YEARNING_USERNAME=<值> \
    -e YEARNING_PASSWORD=<值> \
    -- ~/.codex/mcpserver/yearning/dist/index.js
  ```
- **已注册且可用**：跳过

#### 3. 注册完成后

若本次执行了任何 `claude mcp add` 命令，**必须告知用户重启 Claude Code** 才能加载新 MCP，然后暂停等待：

```
⚠️  已完成 MCP 注册，请重启 Claude Code 后重新执行 /bug 命令。
```

若所有 MCP 已可用，无需重启，直接进入步骤 0。

---

### 步骤 0：信息收集（前置）

读取 bug 描述，提取以下字段。若用户描述不完整，**逐项追问**，全部确认后再进入步骤 1：

| 字段 | 说明 | 是否必须 |
|------|------|---------|
| 问题现象 | 一句话描述 bug 表现 | 必须 |
| 关键业务 ID | 用户ID / 订单ID / 房间ID / 请求ID 等 | 必须 |
| 发生时间 | 精确到分钟，格式 `YYYY-MM-DD HH:MM` | 必须 |
| 涉及服务 | 哪个服务/模块 | 必须 |
| Kibana 索引 | 日志所在 index，例如 `app-logs-*` | 必须 |
| 预期结果 | 正常应该是什么 | 建议提供 |
| 实际结果 | 实际发生了什么 | 建议提供 |

收集完成后，向用户**输出一张确认表**，等用户确认后再继续。

---

### 步骤 1：Kibana 查日志

使用 `execute_kb_api` 调用 `api/console/proxy?path=<index>%2F_search&method=POST`。

#### 高效查询三原则（节省 token）

**原则 1：时间范围用 epoch_millis，不用 ISO 字符串**

```json
"range": {"time": {"gte": 1778225400000, "lte": 1778225700000}}
```

ISO 格式（如 `"2026-05-08T07:30:00.000Z"`）在部分集群的 range 查询中无效，会导致反复重试浪费 token。转换公式：将 CST 时间减 8 小时得 UTC，再乘以 1000 得毫秒。

**原则 2：第一次搜索就排除噪声**

以下日志在德信 texas 服务中大量存在，与业务逻辑无关，必须在每次查询中 must_not 排除：

```json
"must_not": [
  {"match_phrase": {"content": "requestTypeId\":3"}},
  {"match_phrase": {"content": "requestTypeId\":140"}},
  {"match_phrase": {"content": "I18nCache"}},
  {"match_phrase": {"content": "syncRiskData"}},
  {"match_phrase": {"content": "getTableUserDtoRMap"}}
]
```

**原则 3：_source 字段精简，size 控制在 20 以内**

```json
"_source": ["time", "content"],
"size": 20
```

#### 搜索升级路径

| 步骤 | 搜索目标 | 判断条件 |
|------|---------|---------|
| 第1步 | `match content: <用户ID>` + 排除噪声 + 精确时间窗口±5分钟 | 正常入口 |
| 第2步（若第1步只有心跳/风控） | **改用 tableId 搜索**，同样排除噪声 | 用户日志干净但游戏逻辑异常 → 服务端 WARN 通常出现在表级日志 |
| 第3步（若有 WARN/ERROR） | 取出 TID，用 TID 搜索完整调用链 | 定位堆栈 |
| 第4步（若日志完全为空） | 扩时间窗口至±30分钟，换关键词 | 时间对不上或索引错误 |

> **关键判断**：若用户级日志 30 分钟内只有 requestTypeId:3 和 140，说明客户端根本没有发出业务请求，"无法操作"是**客户端拦截**，不是服务端错误，应从产品逻辑（人数限制/状态校验）分析。

**整理输出**：

```
## 日志时间线

| 时间 | 级别 | 摘要 |
|------|------|------|
| 15:29:10 | WARN | ClubAndHallGameStartHandler: 校验游戏不符合开始继续等待 (code=10025) |

## 关键发现
- traceId / TID：...
- 错误码 / 异常类型：...
- 服务端是否收到该业务请求：是 / 否（客户端未发送）
- 异常首次出现时间：...
```

若日志为空，说明原因（索引不对 / 时间 epoch 换算错误 / 关键词不对），询问用户是否调整后重试。

---

### 步骤 2：Yearning 查数据库

使用 Yearning MCP 工具，根据步骤 1 中提取的业务 ID 查询相关表数据。

**查询策略**：
- 先从日志中提取所有业务 ID（订单号、用户 ID、流水号等）
- 根据涉及服务推断相关表名（若不确定，先用 `yearning_show_databases` / `yearning_show_tables` 确认）
- 只执行 **SELECT 查询**，禁止任何写操作
- 生成 SQL 后，**先展示给用户确认**，再通过 Yearning MCP 执行

**整理输出**：

```
## 数据库状态

### <表名>
| 字段 | 值 | 预期 | 是否异常 |
|------|-----|------|---------|
| status | 2 | 1 | ⚠️ 异常 |

### 关键发现
- 主表状态：...
- 关联表状态：...
- 时间戳是否合理：...
- 是否存在数据不一致：...
```

---

### 步骤 3：代码分析

结合日志和数据库状态，在代码仓库中定位相关逻辑。

**分析重点**：

| 维度 | 检查项 |
|------|--------|
| 状态机 | 是否存在未覆盖的状态分支 |
| 空值/边界 | NPE、除零、数组越界 |
| 并发/幂等 | 重试是否可能导致重复执行 |
| 缓存一致性 | 缓存与数据库是否可能不同步 |
| 外部服务 | 超时/降级后的异常处理是否正确 |
| 最近变更 | 近期上线代码是否引入回归（可用 `git log` 辅助） |

**输出**：精确到文件路径和行号，给出可疑代码片段及分析理由。

---

### 步骤 4：输出排障报告

汇总以上三步，输出结构化报告：

```markdown
# Bug 排障报告

## 问题结论
（一句话，例如：用户 X 因 Y 原因导致 Z 未生效）

## 证据链
1. 日志：[时间] [服务] 出现 [错误]，traceId=xxx
2. 数据库：t_xxx 表中 order_id=yyy 的 status 字段为 2，预期为 1
3. 代码：src/xxx/yyy.java:123 的分支逻辑未处理 status=2 的情况

## 影响范围
- 影响用户：（已知 / 估算）
- 影响时间段：
- 是否仍在持续：

## 临时止血方案
（可立即执行的操作，例如：手动修复数据、关闭某功能开关）

## 代码修复建议
（具体到文件路径和修改方向，不替代工程师判断）

## 需要人工确认的问题
- [ ] ...
- [ ] ...

## 建议补充的日志/监控/测试
- ...
```

---

### 步骤 5：等待人类决策

报告输出后，**不主动执行任何修复操作**。等待工程师：

1. 核实证据链
2. 评估修复风险
3. 决定是否需要止血
4. 确认后再修改代码

---

## 约束

- **禁止执行任何数据库写操作**（INSERT / UPDATE / DELETE）
- SQL 必须先展示给用户确认，再执行
- 若某步骤结果为空或不确定，明确告知用户，不猜测、不伪造数据
- Cookie / 密码等敏感信息不得出现在报告中
- 报告结论标注为"AI 建议"，最终由工程师判断
