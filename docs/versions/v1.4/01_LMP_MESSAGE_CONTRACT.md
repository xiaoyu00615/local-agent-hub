# 01｜LMP V1 最小消息契约

**状态：F2 CANDIDATE；待 E00 协议测试。**

## 1. 定位、传输和序列化

- LMP = **Local Module Protocol**，定义 Core ↔ Module Host/Adapter 的业务语义；不规定厂商 Agent 使用什么底层协议。
- V1 数据模型采用 UTF-8 JSON 和 **JSON Schema Draft 2020-12** 验证；具体传输载体可为安全 IPC、WebSocket 或受控本机调用，传输差异由 Host 处理。
- 大文件不在消息信封中内嵌，使用受授权的 Artifact 引用；禁止直接传递不限范围的绝对路径或不受控文件句柄。
- V1 单条消息有最大尺寸限制（数值经 E00 测试确定），超限返回 `PAYLOAD_TOO_LARGE`，不可静默截断命令参数。
- LMP V1 不假设 WebSocket 提供业务级“恰好一次”语义；发送到外部应用也不能仅靠 LMP 保证 exactly-once。

## 2. 身份层次及签发者

| ID | 含义 | 权威产生者 |
|---|---|---|
| `moduleId` | 软件包身份，跨版本稳定 | Module Registry 登记 / 检查命名空间 |
| `moduleRuntimeId` | 一次受管进程或服务实例 | Module Host |
| `endpointId` | 已通过握手的 LMP 通信端点 | Core/Host |
| `connectionInstanceId` | 与某具体软件/协议端的连接 | Core |
| `sessionBindingId` | 逻辑会话绑定，不等于 Tab/PID/HWND | Core Session Manager |
| `workflowRevisionId` | 不可变工作流版本 | Workflow Engine |
| `runId` | 一次工作流运行 | Workflow Engine |
| `taskId` | 一个逻辑任务/节点操作 | Task Engine |
| `attemptId` | 一次业务执行尝试，外部副作用可能发生 | Task Engine |
| `deliveryAttemptId` | 同一消息/结果的某次运输或交付尝试 | Transport/Delivery Manager |
| `messageId` | 每条 LMP 消息的身份 | 发送端按约束创建，Core 记录并检查 |
| `idempotencyKey` | 某项业务效果的去重键 | Core 生成，整个业务意图内保持稳定 |
| `grantId` | Core 存储中的权限记录引用 | Permission Gateway |

`attemptId` ≠ `deliveryAttemptId`。重新传递同一任务并不自动意味着可以开始第二次业务执行尝试。

## 3. Bootstrap 与握手（不用未认证信封伪造身份）

1. Core/Host 读取经静态验证的 Manifest 和包完整性数据，按批准的配置启动或连接 Worker。
2. 通过受保护的本地渠道提供短期一次性配对凭据/挑战；不得通过网页正文或日志暴露。
3. Worker 提交 `moduleId`、模块版本、LMP 支持范围、能力摘要、配置校验结果与所获引导凭据。
4. Host 校验调用进程/渠道、凭据、权限及模块注册信息；不匹配拒绝。
5. Core 记录 `moduleRuntimeId`、`endpointId`、协商协议版本、能力清单快照和允许的传输属性。
6. **仅在握手成功后**接受常规 LMP 消息；信封中 `sourceEndpointId` 只能与已认证通道绑定身份相符。

浏览器扩展采用单独配对机制和来源白名单；网页脚本不直接拥有 Core API 的执行凭据。Host 不得把仅有 localhost 地址视为认证。

## 4. 通用信封字段

| 字段 | 类型/必需性 | 含义 |
|---|---|---|
| `protocolVersion` | string / 必须 | 如 `1.0`；重大不兼容拒绝 |
| `messageId` | string / 必须 | 消息唯一 ID，重放同一消息须保留 ID |
| `kind` | enum / 必须 | `COMMAND` / `RESPONSE` / `EVENT` / `ACK` / `ERROR` |
| `name` | string / 必须 | 版本化的消息语义名称 |
| `sourceEndpointId` | string / 必须 | 经握手认证的实际发送端 |
| `targetEndpointId` | string / 必须 | 目标 Core 或 Worker 端点 |
| `correlationId` | string / 必须 | 请求-响应/异步事件关联用 ID |
| `createdAtUtc` | RFC3339 string / 必须 | 发送端时间，仅用于展示/排障，不用于决定权威事件顺序 |
| `payloadSchemaVersion` | string / 必须 | 当前 `name` 的载荷 schema 版本 |
| `payload` | JSON object / 必须 | 按 `kind + name + schemaVersion` 校验 |
| `causationId` | string / 条件 | 产生本消息的上游 messageId |
| `runId` / `taskId` / `attemptId` | string / 任务相关时必须 | 真实任务关联，不以字符串内容猜测 |
| `deliveryAttemptId` | string / 派发/投递时条件必需 | 传输/交付尝试 |
| `idempotencyKey` | string / 可能引起业务效果时必须 | 业务意图稳定键 |
| `sessionBindingId` / `bindingGeneration` | 条件必需 | 会话型操作绑定快照 |
| `ownershipEpoch` / `leaseId` | 条件必需 | 自动写入时的控制权凭证引用 |
| `grantId` | 条件必需 | 授权引用；**不等于权限本身** |

V1 将未知 `kind`、未知必需语义字段、错误 schema、无效版本或已失效的端点关联视为**拒绝**。允许忽略协议规定为可扩展的无语义附加字段，但不得忽略未知权限、状态或执行控制字段。

**事件顺序：** Core 为每条已成功持久化的事实分配单调递增的 `journalSeq`；`createdAtUtc` 不作为多进程排序依据。

## 5. Kind 与 name 的正式区分

| Kind | 用途 | 示例 name |
|---|---|---|
| `COMMAND` | 请求具有明确定义的操作 | `agent.turn.submit`、`files.readRange`、`module.health` |
| `RESPONSE` | 对请求的同步结果或已关联的最终操作结果 | `operation.result`、`module.describe.result` |
| `EVENT` | 不可变的观察事实，不直接授予权限 | `operation.progress`、`session.external_change` |
| `ACK` | 确认**某种特定事实**；见 02 | `ack.command.accepted`、`ack.result.stored` |
| `ERROR` | 无法完成的已识别故障，带副作用判断 | `operation.error`、`protocol.rejected` |

命令、响应、事件不能只因为名称相似就互换。所有 `RESPONSE`/`ACK`/`ERROR` 都必须关联目标命令/结果的 `messageId` 和 `correlationId`。

## 6. MVP 能力名称（首轮契约）

- 生命周期/连接：`module.describe`、`module.health`、`connection.open`、`connection.close`。
- Agent：`agent.session.bind`、`agent.session.inspect`、`agent.turn.submit`、`agent.turn.observe`、`agent.turn.collect`；可选 `agent.turn.cancel`。
- 工作区：`workspace.list`、`workspace.tree`、`files.find`、`files.search`、`files.readRange`、`files.stat`、`files.hash`。
- 工具：`terminal.profile.run`、`test.profile.run`；**不提供默认 `shell.exec(anyString)`**。
- 结果：`artifact.describe`、`artifact.export`（按实际能力可选）。
- Core 专有命令：`ownership.acquire`、`ownership.release`、`ownership.takeover.request`、`session.reconcile.request`、`workflow.resume.request`。

Agent Adapter 是否支持某项高级功能以 Capability Registry 中的动态声明为准；通用 Profile 不要求为不支持的功能构造伪结果。原来不一致的附件命名统一预留 `agent.turn.attach`，**不纳入 E00 必测 MVP**。

## 7. 典型一次调用的逻辑路径

1. Core 在数据库事务中保存 Run/Task/Attempt、权限检查、资源锁和 **Outbox 派发意图**。
2. 派发器向已经握手的 Adapter 发送同一业务请求，传输过程另记 `deliveryAttemptId`。
3. Adapter/Host 做入站去重及能力参数校验，回报 `ack.command.accepted`；这不代表外部应用已收到。
4. Adapter 在实际外部写入前重新检查会话身份、Epoch、Lease 和权限，随后提交动作并上报证据。
5. Agent 的结果由 `operation.result` 等响应/事件报告，Core 校验关联并持久化。
6. **持久化事务成功后** Core 返回 `ack.result.stored`。
7. 下游交付独立执行、独立鉴权，接收端证明满足交付契约后才记录 `ack.delivery.confirmed`。
8. Verification 按业务规则单独验收，不借用任何 ACK 宣布通过。

对迟到、乱序的 EVENT 可根据 ID 缓存/归并；不允许用迟到“RUNNING”覆盖已确认终态，证据冲突须转对账。
