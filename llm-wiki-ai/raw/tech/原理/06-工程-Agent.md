# Agent 工程实操

> **本文定位**：工程实操层。Function Calling、MCP、Agent 编排的设计和实现。
> 面试题见 [11-面试-LLM应用.md](11-LLM应用.md)

---

## 一、三层能力模型

```
Function Calling  →  模型输出结构化参数，你的代码执行
MCP              →  标准化工具协议，跨应用共享工具
Agent            →  模型自主决策多步任务（规划 → 调用 → 观察 → 继续）
```

**什么时候用哪层**：
- 只需要调用 1-2 个确定性工具 → Function Calling 就够
- 需要跨多个系统共享工具 → MCP
- 需要模型自主完成复杂多步任务 → Agent

---

## 二、Function Calling 工程

### 2.1 基础流程

```
用户："查一下订单 #12345 的状态"
      ↓
LLM 决策："需要调用 getOrderStatus 工具"
      ↓
输出 JSON：{"tool": "getOrderStatus", "orderId": "12345"}
      ↓
你的代码执行：查数据库
      ↓
结果回填给 LLM："订单状态：已发货，预计明天到达"
      ↓
LLM 生成自然语言回答
```

### 2.2 工具描述设计（最重要的工程细节）

工具描述质量直接决定模型是否会正确调用。

**好的工具描述**：
```java
@Tool("查询指定订单的当前状态和物流信息。" +
      "需要提供准确的订单ID（格式：#数字，例如 #12345）。" +
      "返回订单状态、预计送达时间和快递单号。")
public OrderStatus getOrderStatus(
    @P("订单ID，格式为 #数字，例如 #12345") String orderId
) {
    return orderService.getStatus(orderId);
}
```

**差的工具描述**（避免）：
```java
@Tool("查订单")  // 太模糊，模型不知道何时调用
public OrderStatus getOrderStatus(String orderId) { ... }
```

**描述要包含**：
1. 这个工具**做什么**（动词 + 宾语）
2. **什么时候调用**（触发条件）
3. **参数格式**（具体示例）
4. **返回什么**

### 2.3 LangChain4j 完整示例

```java
// 工具类
public class CustomerServiceTools {

    @Tool("查询订单状态。输入订单ID，返回状态和物流信息。")
    public String getOrderStatus(@P("订单ID，如 #12345") String orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new NotFoundException("订单不存在: " + orderId));
        return String.format("订单%s状态：%s，%s", 
            orderId, order.getStatus(), order.getShippingInfo());
    }

    @Tool("处理退款申请。需要提供订单ID和退款原因。")
    public String requestRefund(
        @P("订单ID") String orderId,
        @P("退款原因") String reason
    ) {
        // 安全校验：验证当前用户是否有权限操作此订单
        validateOwnership(orderId);
        return refundService.apply(orderId, reason);
    }

    @Tool("查询用户账户余额和积分。不需要参数。")
    public AccountInfo getAccountInfo() {
        return accountService.getCurrentUserInfo();
    }
}

// Agent 构建
CustomerAssistant assistant = AiServices.builder(CustomerAssistant.class)
    .chatLanguageModel(model)
    .tools(new CustomerServiceTools())
    .chatMemory(MessageWindowChatMemory.withMaxMessages(20))
    .build();
```

---

## 三、工具调用可靠性工程

### 3.1 参数校验

模型可能生成格式错误的参数，必须做校验：

```java
@Tool("查询商品库存")
public StockInfo getStock(@P("商品ID，整数") String productId) {
    // 校验格式
    if (!productId.matches("\\d+")) {
        return StockInfo.error("商品ID格式错误，应为数字，收到：" + productId);
    }
    // 校验存在性
    if (!productRepository.exists(Long.parseLong(productId))) {
        return StockInfo.error("商品不存在：" + productId);
    }
    return stockService.getStock(Long.parseLong(productId));
}
```

**关键原则**：工具返回值要包含清晰的错误信息，而不是抛异常。模型需要读取错误信息来决定下一步。

### 3.2 危险操作防护

```java
@Tool("取消订单（仅限未发货状态）")
public String cancelOrder(@P("订单ID") String orderId) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    
    // 状态检查
    if (order.getStatus() != OrderStatus.PENDING) {
        return "无法取消：订单已" + order.getStatus().getDescription();
    }
    
    // 权限检查
    if (!currentUser().getId().equals(order.getUserId())) {
        return "无权操作此订单";
    }
    
    orderService.cancel(orderId);
    return "订单 " + orderId + " 已成功取消";
}
```

**绝对禁止**：
- 工具直接执行 DELETE 语句而不做校验
- 工具读取/修改不属于当前用户的数据
- 工具调用外部付款、发邮件等高风险操作不做二次确认

### 3.3 超时和重试

```java
@Tool("搜索互联网获取最新信息")
public String webSearch(@P("搜索关键词") String query) {
    try {
        return searchClient.search(query, Duration.ofSeconds(5));
    } catch (TimeoutException e) {
        return "搜索超时，请稍后重试或换一个更具体的关键词";
    } catch (Exception e) {
        log.error("Search failed for query: {}", query, e);
        return "搜索服务暂时不可用，已记录错误";
    }
}
```

---

## 四、Agent 编排设计

### 4.1 单 Agent vs 多 Agent

**什么时候用单 Agent**（首选）：
- 任务目标清晰，工具集合不超过 10 个
- 执行步骤在 5 步以内
- 所有工具由同一个 Agent 能力范围覆盖

**什么时候拆多 Agent**：
- 任务复杂到单个 Agent 容易迷失（超过 10 步）
- 不同子任务需要不同的专业工具集
- 需要并行执行互相独立的子任务

```
多 Agent 架构示例（研究报告生成）：

Orchestrator Agent（协调）
    ├── Research Agent（网络搜索 + 信息收集）
    ├── Analysis Agent（数据分析 + 图表生成）
    └── Writing Agent（报告撰写 + 格式化）
```

### 4.2 状态管理

复杂 Agent 必须做显式状态管理：

```java
public class AgentState {
    private String taskId;
    private String originalGoal;
    private List<String> completedSteps;
    private List<String> pendingSteps;
    private Map<String, Object> collectedData;  // 工具调用结果
    private int retryCount;
    private AgentStatus status;
}
```

**检查点机制**：长任务每完成一个关键步骤，保存状态到 Redis/数据库，支持断点续执行。

### 4.3 工具调用日志

生产环境必须记录每次工具调用：

```java
public class ToolCallLogger {
    
    public Object logAndExecute(String toolName, Object params, Supplier<Object> execution) {
        long start = System.currentTimeMillis();
        try {
            Object result = execution.get();
            log.info("Tool call success: tool={}, params={}, duration={}ms", 
                toolName, params, System.currentTimeMillis() - start);
            return result;
        } catch (Exception e) {
            log.error("Tool call failed: tool={}, params={}, error={}", 
                toolName, params, e.getMessage());
            throw e;
        }
    }
}
```

---

## 五、MCP（Model Context Protocol）

### 5.1 MCP 是什么

Anthropic 提出的标准化工具协议：定义 LLM 如何发现和调用工具，无论工具来自哪个系统。

**解决的问题**：每个 AI 应用都自己定义工具格式，互不兼容。MCP 让工具定义一次、所有支持 MCP 的 Agent/IDE 都能用。

### 5.2 MCP 架构

```
Claude Desktop / IDE / 自定义 Agent（MCP Client）
        ↕ MCP 协议
MCP Server（工具提供方）
  ├── Filesystem Server（文件读写）
  ├── Database Server（SQL 查询）
  ├── GitHub Server（代码管理）
  └── 自定义 Server（业务工具）
```

### 5.3 自定义 MCP Server（Node.js）

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
    name: "order-service",
    version: "1.0.0",
});

server.setRequestHandler(ListToolsRequestSchema, async () => ({
    tools: [{
        name: "get_order_status",
        description: "查询订单状态",
        inputSchema: {
            type: "object",
            properties: {
                order_id: { type: "string", description: "订单ID" }
            },
            required: ["order_id"]
        }
    }]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
    if (request.params.name === "get_order_status") {
        const orderId = request.params.arguments.order_id;
        const status = await orderService.getStatus(orderId);
        return { content: [{ type: "text", text: JSON.stringify(status) }] };
    }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## 六、Agentic 工作流（Dify/Coze 平台）

对于不需要完全自定义的场景，低代码平台可以快速验证：

**Dify 工作流节点**：
```
开始 → LLM 节点（理解意图）
     → 条件分支（是否需要查数据库）
     ├── 是 → 代码节点（执行 SQL）→ LLM 节点（生成回答）
     └── 否 → LLM 节点（直接回答）
     → 结束
```

**何时从平台迁移到自研**：
- 需要深度定制业务逻辑
- 需要集成内部系统（无法通过 HTTP 暴露的）
- 性能要求高（平台有调用链路开销）
- 需要完整的可观测和安全控制

---

## 七、工程设计原则

1. **工具描述比代码逻辑更重要**：模型选择调用哪个工具完全基于描述
2. **工具失败的处理比规划能力更重要**：90% 的 Agent 问题是工具失败后无法优雅恢复
3. **每步状态可追踪**：不可追踪 = 不可调试 = 不可上线
4. **限制危险操作**：工具不应该能做超出当前用户权限的事情
5. **单 Agent 优先**：多 Agent 带来更多协调开销，先证明单 Agent 不够用再拆分
