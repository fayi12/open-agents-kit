# Agent 协作工作流设计

## 1. Agent 通信机制

### 1.1 通信模式

| 模式 | 描述 | 适用场景 |
|------|------|----------|
| **同步消息** | 直接通过 `sessions_send` 实时发送消息 | 需要即时响应的任务 |
| **异步任务** | 通过 `cron` 或任务队列延迟处理 | 定时任务、后台处理 |
| **事件广播** | 通过 `message` 广播到多个Agent | 状态变更、通知分发 |

### 1.2 消息格式

```json
{
  "type": "task|event|response",
  "source": "<agent-id>",
  "target": "<agent-id>|*",
  "payload": {
    "action": "<action-name>",
    "data": { ... }
  },
  "timestamp": "<ISO-8601>",
  "correlationId": "<uuid>"
}
```

### 1.3 通信协议

- **会话路由**：通过 `sessionKey` 或 `label` 精确定位目标Agent会话
- **超时控制**：默认 60s，超时自动重试或上报
- **确认机制**：重要消息需回执确认（ACK）

---

## 2. 任务队列设计

### 2.1 队列结构

```
┌─────────────────────────────────────────┐
│              任务队列 (Task Queue)       │
├─────────────────────────────────────────┤
│ pending    → 待处理队列（FIFO）           │
│ running    → 执行中队列                  │
│ retry      → 重试队列（指数退避）          │
│ completed  → 完成记录（最近100条）        │
│ failed     → 失败记录（需人工介入）        │
└─────────────────────────────────────────┘
```

### 2.2 任务定义

```json
{
  "id": "<task-uuid>",
  "type": "agentTurn|systemEvent|webhook",
  "source": "<source-agent>",
  "target": "<target-agent>|queue",
  "payload": { ... },
  "schedule": {
    "kind": "now|immediate|cron|at",
    "expr": "<cron-expr>"
  },
  "retry": {
    "maxAttempts": 3,
    "backoff": "exponential",
    "intervalMs": 5000
  },
  "status": "pending|running|completed|failed",
  "createdAt": "<timestamp>",
  "completedAt": "<timestamp>",
  "error": "<error-message>"
}
```

### 2.3 调度策略

- **优先级队列**：高优先级任务优先调度（priority: 1-10）
- **负载均衡**：多Agent时轮询或最少任务优先
- **依赖管理**：支持 `dependsOn` 字段，等前置任务完成后再执行

---

## 3. 错误处理

### 3.1 错误分类

| 错误类型 | 处理策略 | 示例 |
|----------|----------|------|
| **瞬时错误** | 自动重试（指数退避） | 网络超时、服务暂时不可用 |
| **限流错误** | 等待后重试（尊重 Retry-After） | 429 Too Many Requests |
| **永久错误** | 记录并标记失败，通知上游 | 权限不足、参数错误 |
| **未知错误** | 记录堆栈，上报警报 | 未捕获异常 |

### 3.2 重试机制

```
初始重试延迟 = 5s
最大重试次数 = 3
退避策略 = exponential (5s → 10s → 20s)
最大重试延迟 = 60s
```

### 3.3 死信队列（Dead Letter Queue）

- 重试超过 `maxAttempts` 仍未成功的任务进入 DLQ
- DLQ 任务需要人工介入或手动触发重试
- 支持查看失败原因和重试历史

### 3.4 监控与告警

- **关键指标**：队列深度（pending/running/failed）、成功率、平均执行时间
- **告警阈值**：
  - 失败率 > 10%
  - 队列积压 > 100 条
  - 执行时间 > 5 分钟
- **告警方式**：通知到配置的消息通道（Webhook/Telegram等）

---

## 4. 扩展点设计

### 4.1 插件化架构

```
┌──────────────────────────────────────────┐
│              OpenClaw Core               │
├──────────────────────────────────────────┤
│  插件接口（Plugin Interface）              │
│  ├── onTaskStart(task)                   │
│  ├── onTaskComplete(task, result)        │
│  ├── onTaskFailed(task, error)           │
│  ├── onAgentRegister(agent)              │
│  └── onAgentUnregister(agent)            │
└──────────────────────────────────────────┘
           ↑ 插件实现
┌──────────────────────────────────────────┐
│  内置插件                                  │
│  ├── 调度插件（Scheduler）                 │
│  ├── 监控插件（Monitor）                   │
│  ├── 告警插件（Alerter）                   │
│  └── 日志插件（Logger）                   │
└──────────────────────────────────────────┘
```

### 4.2 自定义调度器

实现 `CustomScheduler` 接口以替换默认调度逻辑：

```typescript
interface CustomScheduler {
  selectNextTask(queue: Task[]): Task | null;
  shouldRetry(task: Task, attempt: number): boolean;
  getNextRetryDelay(task: Task): number;
}
```

### 4.3 扩展通信协议

- 支持接入第三方消息队列（RabbitMQ / Kafka）
- 支持 gRPC / WebSocket 等传输层扩展
- 支持自定义序列化格式（JSON / Protobuf / MessagePack）

### 4.4 新增Agent类型

| 类型 | 描述 | 示例 |
|------|------|------|
| `worker` | 执行具体任务 | 爬虫、翻译、数据处理 |
| `router` | 负责任务分发和路由 | 接收任务→分配给worker |
| `aggregator` | 聚合多源结果 | 收集多个worker结果并汇总 |
| `monitor` | 监控和健康管理 | 检查Agent存活状态 |

---

## 5. 典型工作流示例

### 5.1 任务下发到完成

```
用户 → Router Agent → [任务入队]
                      ↓
                 Worker Agent (从队列取任务)
                      ↓
                 执行任务
                  ↙    ↘
              成功       失败
                ↓         ↓
          更新状态为   重试（指数退避）
          completed     ↙
                       最终失败 → DLQ → 告警
```

### 5.2 并行任务处理

```
                    ┌──→ Worker A ──→ 结果A
用户 → Router ───┤
                    ├──→ Worker B ──→ 结果B
                    │
                    └──→ Worker C ──→ 结果C
                          ↓
                    Aggregator（汇总结果）
                          ↓
                    返回给用户
```

---

## 6. 配置参考

```yaml
scheduler:
  queue:
    pendingLimit: 1000
    completedHistory: 100
    dlqThreshold: 3  # 重试次数超过此值进入DLQ
  
  retry:
    maxAttempts: 3
    initialDelayMs: 5000
    maxDelayMs: 60000
    backoff: exponential
  
  monitor:
    enabled: true
    intervalSeconds: 30
    alertOnFailureRate: 0.1
    alertChannels:
      - telegram
      - webhook

  workers:
    - id: worker-1
      type: worker
      maxConcurrent: 5
    - id: worker-2
      type: worker
      maxConcurrent: 5
```

---

*本文档由调度崽（⏰）生成 | OpenClaw Agent Collaboration Framework*
