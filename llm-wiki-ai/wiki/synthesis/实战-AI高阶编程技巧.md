# 实战：AI 高效解 BUG 方法论

整理日期：2026-05-08  
来源：群聊实操记录 + Codex 接入 Yearning / Kibana 实践

---

## 核心理念

> 让 AI 承担**信息收集与分析**工作，人类只做**决策**。

传统排障：工程师手动查日志 → 查数据库 → 翻代码 → 分析 → 修复。每一步都要切换工具、复制粘贴、凭经验猜测。

AI 增强排障：把 Kibana、Yearning（数据库代理）、代码仓库、Bug 系统全部接入 AI 工具链，AI 自动完成信息聚合和分析，工程师只负责确认和决策。

```
Bug 信息
  ↓  (Pincode/需求 MCP)
Kibana 日志
  ↓  (Kibana MCP)
数据库查询
  ↓  (Yearning MCP)
代码对比分析
  ↓  (代码仓库)
AI 综合结论
  ↓
人类决策 → 修复上线
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

### 步骤 1：获取 Bug 信息

方式一：直接粘贴 bug 描述给 Codex：

```text
这个 bug 的现象是：用户 ID 12345 在 2026-05-07 20:30 充值后积分未到账。
错误发生在 payment-service。
```

方式二（推荐）：用 Pincode MCP 让 AI 自动拉取：

```text
帮我用 Pincode 拉取 BUG-6789 的详情，包括现象、复现步骤、相关 ID。
```

**关键定位字段（优先提取）：**

- 用户 ID / 房间 ID / 牌局 ID / 订单 ID / 请求 ID
- 发生时间（精确到分钟）
- 涉及服务名
- 预期结果 vs 实际结果

---

### 步骤 2：Kibana 查日志

```text
根据以上 bug 信息，到 Kibana 搜索用户 12345 在 2026-05-07 20:28~20:35 
之间的日志，整理错误栈、请求链路和关键字段。
```

**AI 应重点关注：**

| 关注点 | 说明 |
|--------|------|
| `error` / `exception` | 完整错误堆栈 |
| `traceId` / `requestId` | 串联完整请求链路 |
| 业务 ID | 用户、房间、订单、局号等 |
| 上下游参数 | 入参和返回值 |
| 时间顺序 | 各节点时间戳，定位卡在哪一步 |

**排障提示词模板：**

```text
以下是 bug 基本信息：[粘贴 bug 描述]
请先在 Kibana 中搜索相关日志，时间范围 [发生时间前后10分钟]，
关键词包括 [用户ID/订单ID]，整理成时间线格式输出。
```

---

### 步骤 3：Yearning 查数据库

```text
根据日志里的订单 ID order_abc123，
到 Yearning 查询 t_order 和 t_point_record 表的相关记录，
和日志中的参数做对比，看数据状态是否符合预期。
```

**AI 应重点关注：**

| 关注点 | 说明 |
|--------|------|
| 业务主表状态 | 订单/积分/余额等核心表 |
| 关联表状态 | 关系是否正确 |
| 状态字段 | 流转是否异常（如停在中间态） |
| 时间戳 | create_time / update_time 是否合理 |
| 流水记录 | 操作记录表有无对应记录 |

**注意：只让 AI 执行只读 SQL，禁止 AI 自动执行写操作。**

---

### 步骤 4：代码对比分析

```text
结合上面的日志和数据库状态，
分析 payment-service 中积分发放的逻辑，
定位最可能出问题的代码路径，给出证据链。
```

**AI 应重点分析：**

| 分析维度 | 常见 bug 模式 |
|---------|-------------|
| 状态机分支 | 某个状态没有处理到 |
| 空值/边界 | NPE、除零、越界 |
| 并发与幂等 | 重试导致重复执行 |
| 缓存一致性 | 缓存未更新，数据库已改 |
| 外部服务 | 超时/降级后逻辑不对 |
| 最近变更 | 近期上线的代码引入了回归 |

---

### 步骤 5：获取 AI 分析结论

让 AI 输出标准排障报告：

```text
请综合以上所有信息，给出：
1. 问题结论（一句话）
2. 证据链（日志+数据+代码行号）
3. 影响范围
4. 临时止血方案
5. 代码修复建议
6. 需要人工确认的问题
7. 建议补充的日志/监控/测试
```

---

### 步骤 6：人类决策

AI 分析结果只是建议，工程师需要：

1. 核实 AI 给出的证据链是否可信。
2. 评估修复方案的风险。
3. 决定是否需要临时止血。
4. 最终确认修改代码并提测上线。

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

## 五、高效使用技巧

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
