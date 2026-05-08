---
name: bug
description: 代码优先的全链路 AI 排障。先理解 BUG、定位代码，代码无法确认时才按需调用 Yearning/Kibana，最终修复代码等待人工审核合并。触发命令：/bug <bug描述>
user-invocable: true
---

# bug 技能

## 核心目标

接收 bug 描述 → AI 分析 → **代码定位优先** → 按需查库/查日志 → 修复代码 → 人工审核合并。  
AI 承担分析与修复，人类负责审核与提交。

## 触发场景

- 用户执行 `/bug <bug描述>`
- 用户要求排查线上问题、分析 bug 根因

---

## 执行流水线

### 步骤 -1：环境自检与自动安装（每次启动必须执行）

在需要调用 Kibana/Yearning 之前，先确认 MCP 是否可用。若后续步骤不需要这两个工具，可跳过。

#### 检查 Kibana MCP

```bash
which mcp-server-kibana
```

- **未安装**：运行 `npm install -g @tocharianou/mcp-server-kibana`
- **已安装但未注册**：从 Chrome Cookie 提取认证信息并注册：

  ```bash
  # 确认 Chrome Cookie 数据库位置
  ls ~/Library/Application\ Support/Google/Chrome/Default/Cookies
  # 从 macOS Keychain 取解密密钥
  security find-generic-password -w -s "Chrome Safe Storage"
  ```

  读取 `.dxit999.com` 的 `auth_pro` 和 `new-dev-eslog.dxit999.com` 的 `sid`，AES-CBC 解密（PBKDF2 密钥，1003 iter，16 bytes），**剥离前 32 字节前缀**后注册：

  ```bash
  claude mcp add kibana mcp-server-kibana -s user \
    -e KIBANA_URL=https://new-dev-eslog.dxit999.com \
    -e "KIBANA_COOKIES=auth_pro=<值>; sid=<值>" \
    -e KIBANA_DEFAULT_SPACE=default \
    -e NODE_TLS_REJECT_UNAUTHORIZED=0
  ```

#### 检查 Yearning MCP

```bash
ls ~/.codex/mcpserver/yearning/dist/index.js
```

- **不存在**：让用户指定 zip 路径，解压后 `npm install && npm run build`
- **已编译未注册**：从 `~/.codex/config.toml` 取配置并注册：

  ```bash
  claude mcp add yearning node -s user \
    -e YEARNING_URL=<值> -e YEARNING_SOURCE_ID=<值> \
    -e YEARNING_USERNAME=<值> -e YEARNING_PASSWORD=<值> \
    -- ~/.codex/mcpserver/yearning/dist/index.js
  ```

若执行了任何 `claude mcp add`，**告知用户重启 Claude Code** 后重新执行 `/bug`，然后暂停。

---

### 步骤 0：理解 BUG，向用户提问

读取 bug 描述，**立即分析**：

1. 复述对问题的理解（一句话）
2. 初步判断可能的根因方向（状态机？并发？数据异常？配置？）
3. 列出需要补充的信息，**一次性提问**，不要逐条追问：

| 可能需要补充的信息 | 何时必须 |
|-----------------|---------|
| 关键业务 ID（用户/订单/房间等） | 需要查日志或数据库时 |
| 发生时间（精确到分钟） | 需要查日志时 |
| 涉及服务/模块名 | 代码仓库有多个服务时 |
| Kibana 索引名 | 需要查日志时 |

> **原则**：能从 bug 描述推断的不问，只问真正缺失且代码分析无法绕过的信息。

---

### 步骤 1：定位相关代码

**这是最重要的步骤，优先于一切外部查询。**

根据 bug 描述中的服务名、功能点、错误信息，在代码仓库中搜索相关逻辑：

```bash
# 按关键词搜索
grep -r "关键词" src/ --include="*.java" -l
# 查看最近变更
git log --oneline -20 -- src/相关路径/
git diff HEAD~5 -- src/相关路径/
```

**分析维度**：

| 维度 | 检查项 |
|------|--------|
| 状态机 | 是否存在未覆盖的状态分支 |
| 空值/边界 | NPE、除零、数组越界 |
| 并发/幂等 | 重试是否可能导致重复执行 |
| 缓存一致性 | 缓存与数据库是否可能不同步 |
| 外部服务 | 超时/降级后异常处理是否正确 |
| 最近变更 | 近期上线代码是否引入回归 |

**输出**：精确到文件路径和行号，给出可疑代码片段和分析理由。

---

### 步骤 2：判断——代码能否直接确认根因？

```
代码分析后：
├── 能确认根因 → 直接进入步骤 5（修复代码）
├── 需要确认数据状态 → 步骤 3a（Yearning 查库）
└── 需要确认运行时行为/异常堆栈 → 步骤 3b（Kibana 查日志）
```

不要在不需要时调用 Kibana/Yearning。

---

### 步骤 3a（按需）：Yearning 查数据库

**触发条件**：代码逻辑正常，但怀疑数据状态异常（数据不一致、状态卡中间态、流水缺失等）。

**查询规范**：
- 只执行 **SELECT**，禁止任何写操作
- SQL 先展示给用户确认，再执行
- 若不确定表名，先用 `yearning_show_tables` 确认

**整理输出**：

```
## 数据库状态

### <表名>
| 字段 | 值 | 预期 | 是否异常 |
|------|-----|------|---------|
| status | 2 | 1 | ⚠️ 异常 |

### 关键发现
- 主表状态：...
- 时间戳是否合理：...
- 是否存在数据不一致：...
```

---

### 步骤 3b（按需）：Kibana 查日志

**触发条件**：需要运行时异常堆栈、请求链路、参数值等代码静态分析无法得到的信息。

使用 `execute_kb_api` 调用：
```
api/console/proxy?path=<index>%2F_search&method=POST
```

#### 高效查询规范（避免 token 浪费）

**时间范围必须用 epoch_millis**（ISO 字符串在此 ES 集群 range 查询无效）：
```json
"range": {"time": {"gte": 1778225400000, "lte": 1778225700000}}
```
转换：CST 时间 -8h = UTC，×1000 = epoch_ms。

**首次搜索就排除噪声**（占日志 90%+）：
```json
"must_not": [
  {"match_phrase": {"content": "requestTypeId\":3"}},
  {"match_phrase": {"content": "requestTypeId\":140"}},
  {"match_phrase": {"content": "I18nCache"}},
  {"match_phrase": {"content": "syncRiskData"}},
  {"match_phrase": {"content": "getTableUserDtoRMap"}}
]
```

**精简 _source，size ≤ 20**：
```json
"_source": ["time", "content"], "size": 20
```

**搜索升级路径**：
1. 用户 ID 搜索 + 噪声过滤 + ±5 分钟
2. 若只有心跳/风控 → 改用 tableId/业务ID 搜索（服务端 WARN 在表级日志）
3. 发现 WARN/ERROR → 取 TID 搜索完整调用链
4. 日志为空 → 扩时间窗至 ±30 分钟，换关键词

> **关键判断**：若用户日志 30 分钟内只有 requestTypeId:3/140，说明**客户端未发出业务请求**，问题在客户端拦截逻辑，不是服务端错误。

---

### 步骤 4：确认根因，输出结论

综合代码分析 + 数据库/日志（如有），输出：

```
## 根因结论
（一句话：用户 X 因 Y 原因导致 Z 未生效）

## 证据链
1. 代码：src/xxx/yyy.java:123 的分支逻辑未处理 status=2 的情况
2. 数据库：t_xxx 表中 order_id=yyy 的 status=2（预期为 1）
3. 日志：15:29:10 WARN ClubAndHallGameStartHandler code=10025

## 影响范围
- 影响用户：已知 / 估算
- 是否仍在持续：
```

---

### 步骤 5：修复代码

根据根因直接修复代码，**不只是给建议**。

修复后说明：
- 修改了哪些文件（路径+行号）
- 修改逻辑是什么
- 是否需要数据修复（如有，给出 SQL，用户确认后由 Yearning 执行或手动执行）
- 是否需要配置变更

---

### 步骤 6：等待人工审核与合并

代码修改完成后**停止操作**，由工程师：

1. Review 代码变更
2. 补充单测（如有必要）
3. 自行提交 commit、发起 PR、合并上线

AI 不自动执行 `git commit` / `git push` / PR 创建。

---

## 约束

- **禁止执行任何数据库写操作**（INSERT / UPDATE / DELETE）
- SQL 必须先展示给用户确认，再执行
- 若某步骤结果为空或不确定，明确告知，不猜测、不伪造数据
- Cookie / 密码等敏感信息不得出现在任何输出中
