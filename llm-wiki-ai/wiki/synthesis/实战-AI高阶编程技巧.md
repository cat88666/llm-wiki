# 实战：AI 高效解 BUG 方法论

整理日期：2026-05-08  
来源：群聊实操记录 + Codex 接入 Yearning / Kibana 实践

---

## 核心理念

> 让 AI 承担**信息收集与分析**工作，人类只做**决策**。

传统排障：工程师手动查日志 → 查数据库 → 翻代码 → 分析 → 修复。每一步都要切换工具、复制粘贴、凭经验猜测。

AI 增强排障：**代码优先**，AI 先理解 bug、定位代码，仅在代码静态分析不够用时才调用 Kibana/Yearning，最终由 AI 直接修复代码，工程师只负责审核和提交。

```
Bug 描述
  ↓
AI 分析 + 向用户提问（一次性）
  ↓
定位相关代码（代码仓库）
  ↓
代码能确认根因？
  ├── 是 → 直接修复代码
  └── 否
       ├── 需要确认数据状态 → Yearning 查库
       └── 需要运行时信息  → Kibana 查日志
           ↓
       确认根因 → 修复代码
  ↓
人工审核 → 人工提交合并上线
```

---

## 一、工具链概览

| 工具 | 作用 | 安装方式 |
|------|------|---------|
| Yearning MCP | 通过 Yearning 代理查询数据库结构和数据 | 解压 zip 后让 Codex 编译注册 |
| Kibana MCP | 连接 Kibana 搜索生产日志 | `npm install -g @tocharianou/mcp-server-kibana` |
| Pincode MCP | 拉取 Bug 系统工单列表和详情 | 按各自公司的 bug 系统配置 |
| 代码仓库 | Codex 本地分析代码 | 无需额外安装 |

工具链搭建完成后，可以发出一条自然语言指令，让 AI 自动完成整条排障链路。

---

## 二、完整排障流程

### 步骤 1：描述 Bug，AI 初步分析

直接粘贴 bug 描述，AI 会立即给出初步判断和一次性追问：

```text
这个 bug 的现象是：用户ID 1037027917470683180 在 2026-05-07 20:30 点击游戏开始无响应。
错误发生在 gke-k8s-dx-pre-dx-game-texas*
```

AI 会：
- 复述理解（一句话）
- 初步判断根因方向
- **一次性**列出缺失信息，不逐条追问

---

### 步骤 2：AI 定位相关代码（最重要的步骤）

AI 在代码仓库中搜索相关逻辑，分析根因：

```text
帮我分析 gke-k8s-dx-pre-dx-game-texas* 点击游戏开始协议代码逻辑，
定位 点击游戏开始无响应 最可能的原因。
```

**AI 重点分析：**

| 分析维度 | 常见 bug 模式 |
|---------|-------------|
| 状态机分支 | 某个状态没有处理到 |
| 空值/边界 | NPE、除零、越界 |
| 并发与幂等 | 重试导致重复执行 |
| 缓存一致性 | 缓存未更新，数据库已改 |
| 外部服务 | 超时/降级后逻辑不对 |
| 最近变更 | 近期上线的代码引入了回归 |

**代码能确认根因 → 直接进步骤 5 修复，跳过 Kibana/Yearning。**

---

### 步骤 3a（按需）：Yearning 查数据库

**触发条件**：代码逻辑看起来正常，但怀疑数据状态异常。

```text
根据代码分析，怀疑 t_order 的 status 字段异常。
请查询 order_id=abc123 的订单状态和 t_point_record 的流水记录。
```

**AI 重点关注：**

| 关注点 | 说明 |
|--------|------|
| 状态字段 | 流转是否异常（停在中间态） |
| 时间戳 | create_time / update_time 是否合理 |
| 流水记录 | 操作记录表有无对应记录 |

**AI 只执行只读 SQL，执行前先展示给工程师确认。**

---

### 步骤 3b（按需）：Kibana 查日志

**触发条件**：需要运行时异常堆栈、请求链路、参数值等静态代码分析无法得到的信息。

```text
请在 Kibana 的 gke-k8s-dx-pre-dx-game-texas* 索引中，
搜索用户 1037027917470683180 在 2026-05-07 20:28~20:35 的日志，
重点找 ERROR/WARN 和完整异常堆栈。
```

**查询效率要点（避免 token 浪费）：**
- 时间范围用 epoch_millis，不用 ISO 字符串
- 第一次搜索就排除心跳/风控等噪声日志
- size ≤ 20，_source 只取 time 和 content
- 用户日志全是心跳 → 换 tableId/业务ID 搜表级日志

---

### 步骤 4：确认根因

综合代码 + 数据库/日志，输出根因结论和证据链：

```
根因：gke-k8s-dx-pre-dx-game-texas* 的 ClubAndHallGameStartHandler.java:93 校验游戏不符合开始继续等待 报错。

证据：
1. 代码：ClubAndHallGameStartHandler.java:93 报错
2. 数据库：t_order status=PROCESSING，t_point_record 无对应记录
3. 日志：20:30:01 INFO GameStartHandlerBase.check(GameStartHandlerBase.java:145) 检查抛异常
```

---

### 步骤 5：AI 直接修复代码

AI 修改代码，说明改了什么、为什么改：

```text
已修改 src/main/java/com/xx/PointService.java:234
新增 status=PROCESSING 分支，补充积分发放逻辑。
```

如需数据修复，给出 SQL，工程师确认后执行。

---

### 步骤 6：人工审核，人工提交合并

工程师负责：

1. Review AI 的代码修改
2. 补充单测（如有必要）
3. 自行 commit → PR → 合并上线

**AI 不执行 git commit / push / 创建 PR。**

---

## 三、Yearning MCP 安装配置

### 3.1 安装步骤

1. 将 `yearning.zip` 解压到目标目录，例如 `~/mcpserver/yearning`。
2. 在 Codex 中执行：

```text
我有一个 Yearning MCP server 包，目录在 ~/mcpserver/yearning。
请帮我安装依赖、编译，并注册到 Codex MCP。
配置如下：
YEARNING_URL=https://your-yearning.example.com
YEARNING_SOURCE_ID=<source_id>
YEARNING_USERNAME=<username>
YEARNING_PASSWORD=<password>
```

3. 重启 Codex，验证：

```text
帮我查一下当前有哪些数据库
```

返回数据库列表即配置成功。

### 3.2 配置模板

```toml
[mcp_servers.yearning]
command = "node"
args = ["~/mcpserver/yearning/dist/index.js"]

[mcp_servers.yearning.env]
YEARNING_URL = "https://your-yearning.example.com"
YEARNING_SOURCE_ID = "<source_id>"
YEARNING_USERNAME = "<username>"
YEARNING_PASSWORD = "<password>"
YEARNING_AUTH_PRO = "<sso_cookie_if_needed>"
```

### 3.3 source_id 获取

在 Yearning 页面查询时，从浏览器地址栏提取：

```
https://yearning.example.com/#/apply/query?source_id=6d2f80e9-51e4-47ad-bf85-323441d795f9
                                                                ↑ 这就是 source_id
```

---

## 四、Kibana MCP 安装配置

### 4.1 安装

```bash
npm install -g @tocharianou/mcp-server-kibana
```

NPM 包：`@tocharianou/mcp-server-kibana`

### 4.2 SSO Cookie 获取（关键步骤）

Kibana 使用 SSO 认证，需要从 Chrome 提取已登录的 cookie：

1. **手动登录**：在 Chrome 打开 Kibana 和 SSO，完成正常登录。
2. **让 Codex 读取 Chrome cookie**：

```text
帮我从 Chrome 浏览器读取 .example.com 域名的 auth_pro cookie，
以及 kibana.example.com 的 sid cookie，
用于配置 Kibana MCP 认证。
```

3. **Chrome Cookie 数据库位置**（macOS）：

```
~/Library/Application Support/Google/Chrome/Default/Cookies
~/Library/Application Support/Google/Chrome/Profile 1/Cookies
```

4. **解密注意事项**：新版 Chrome cookie 加密后，明文前 32 字节是校验前缀，需要剥离，否则请求头会包含非法字符导致认证失败。

### 4.3 配置模板

```toml
[mcp_servers.kibana]
command = "mcp-server-kibana"

[mcp_servers.kibana.env]
KIBANA_URL = "https://kibana.example.com"
KIBANA_COOKIES = "auth_pro=xxx; sid=yyy"
KIBANA_DEFAULT_SPACE = "default"
NODE_TLS_REJECT_UNAUTHORIZED = "0"
```

> `NODE_TLS_REJECT_UNAUTHORIZED = "0"` 仅适合内网受控环境，不能用于公网生产。

### 4.4 验证连接

```text
帮我检查 Kibana 连接状态
```

返回 Kibana 版本和 `status.overall.state: green` 即成功。

### 4.5 Cookie 时效

Cookie 有有效期，过期后需要重新登录 Chrome 并重新提取。建议在 Cookie 失效时直接让 Codex 重新读取。

---

## 五、Kibana 搜索效率技巧（实战踩坑）

以下技巧来自真实排障积累，不遵守会大量浪费 AI token。

### 技巧 1：时间范围必须用 epoch_millis

```json
// ✅ 正确
"range": {"time": {"gte": 1778225400000, "lte": 1778225700000}}

// ❌ 错误（ISO 字符串在部分 ES 集群的 range 查询中无效，导致 0 结果）
"range": {"time": {"gte": "2026-05-08T07:30:00.000Z", "lte": "2026-05-08T07:40:00.000Z"}}
```

转换公式：CST 时间 - 8h = UTC，再 × 1000 = epoch_ms。

### 技巧 2：第一次搜索就排除噪声日志

texas 服务每 2 秒一条心跳（requestTypeId:3）、每 5 秒一条风控同步（requestTypeId:140），加上 I18nCache、syncRiskData、getTableUserDtoRMap，占总日志 90%+，有效信息淹没其中。

**模板**（复制后直接用）：

```json
{
  "size": 20,
  "_source": ["time", "content"],
  "query": {
    "bool": {
      "must": [
        {"match": {"content": "<用户ID或关键词>"}},
        {"range": {"time": {"gte": <epoch_start>, "lte": <epoch_end>}}}
      ],
      "must_not": [
        {"match_phrase": {"content": "requestTypeId\":3"}},
        {"match_phrase": {"content": "requestTypeId\":140"}},
        {"match_phrase": {"content": "I18nCache"}},
        {"match_phrase": {"content": "syncRiskData"}},
        {"match_phrase": {"content": "getTableUserDtoRMap"}}
      ]
    }
  },
  "sort": [{"time": {"order": "asc"}}]
}
```

### 技巧 3：用户日志只有心跳 → 立即切换表级搜索

用户日志 30 分钟内只有 requestTypeId:3/140 → 客户端根本没有发出业务请求。  
此时应改用 `tableId` 作为关键词，搜索服务端的自动任务日志，WARN/ERROR 通常出现在这里。

```
用户级搜索 → 全是心跳
       ↓
表级搜索（match tableId）+ 噪声过滤
       ↓
发现 WARN: ClubAndHallGameStartHandler 自动开始异常 code=10025
       ↓
根因：服务端校验失败，客户端收到错误通知无法触发开始
```

### 技巧 4：Kibana MCP 必须通过 console/proxy 调 ES

直接路径 `/<index>/_search` 返回 404，必须用：

```
api/console/proxy?path=<index>%2F_search&method=POST
```

---

## 六、高效使用技巧

### 技巧 1：一步发起全链路排障

搭建好工具链后，可以用一条指令触发全部步骤：

```text
这个 bug：[粘贴 bug 描述]
请按顺序：
1. 到 Kibana 搜索相关日志（时间范围：XX~XX）
2. 根据日志里的 ID 到 Yearning 查数据库状态
3. 结合代码分析根因
4. 给出排障报告
```

### 技巧 2：让 AI 主动补充信息

遇到日志不够的情况：

```text
日志里缺少 XX 信息，你觉得还需要补充哪些查询才能定位问题？
```

### 技巧 3：让 AI 写 SQL 后人工审核

Yearning 支持 SQL 查询，但应先让 AI 生成 SQL、人工 review 后再执行，避免误操作：

```text
帮我写一条 SQL，查询 user_id=12345 在 2026-05-07 的所有积分流水记录，
我确认没问题后再执行。
```

### 技巧 4：固化常用排障模板

对于高频 bug 类型（充值未到账、数据不一致等），可以提前写好排障 prompt 模板，每次只需填入关键 ID 和时间即可。

---

## 六、安全注意事项

| 风险点 | 防范措施 |
|--------|---------|
| 密码/Cookie 泄露 | 不写入文档、PR、聊天记录；配置文件加入 `.gitignore` |
| AI 执行写操作 | 明确告知 Codex 只执行只读 SQL |
| Cookie 过期 | 定期检查，排障前先验证连接 |
| 生产数据权限 | 遵守公司数据权限和审计要求 |
| AI 结论误判 | 所有修改必须经工程师确认，AI 只提建议 |
| TLS 校验关闭 | `NODE_TLS_REJECT_UNAUTHORIZED=0` 仅用于内网环境 |

---

## 七、当前配置状态（本机）

| 工具 | 状态 |
|------|------|
| Yearning MCP | 已注册，基础配置完成；SSO `auth_pro` 待确认后完整验证 |
| Kibana MCP | 已安装、已配置、`get_status` 和 `get_available_spaces` 测试通过 |

配置文件位置：`~/.codex/config.toml`

修改配置后需重启 Codex，使 MCP 工具在新会话中生效。
