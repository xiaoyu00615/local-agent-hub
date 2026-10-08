# 02｜Local Module Protocol（LMP）V1 草案

**状态：REVIEW DRAFT；协议名称与字段尚待逐项冻结**

## 1. 边界
LMP 定义模块与 Core 的能力调用、消息与事件语义，独立于 HTTP、WebSocket、stdio、IPC 等传输。ACP、MCP 或厂商 API 由 Adapter 翻译；不得假设它们与 LMP 一一对应。

## 2. 统一命名提案
- **命令/能力**：`module.describe`、`module.health`、`connection.open`、`connection.close`、`agent.session.bind`、`agent.session.inspect`、`agent.turn.submit`、`agent.turn.observe`、`agent.turn.collect`、`agent.turn.cancel`、`files.find`、`files.search`、`files.readRange`、`files.stat`、`terminal.profile.run`、`test.profile.run`、`artifact.describe`、`artifact.export`。
- **控制命令**：`ownership.acquire`、`ownership.release`、`ownership.takeover.request`、`session.reconcile.request`、`workflow.pause`、`workflow.resume.request`。
- **领域事件**：`module.ready`、`connection.changed`、`session.external_change`、`task.accepted`、`task.progress`、`task.completed`、`approval.requested`、`result.persisted`、`result.delivered`、`result.acked`。
- 原讨论中 `agent.turn.attach` 与 `agent.input.attach` 统一建议为可选 `agent.turn.attach`，但是否作为独立能力需后续验证。

命令、返回结果、事件具有不同的 Schema，不得只根据相似的字符串假设等价。

## 3. 消息信封
必须字段建议：`protocolVersion`、`messageId`、`kind`（COMMAND/RESPONSE/EVENT/ACK/ERROR）、`sourceInstanceId`、`targetInstanceId`、`correlationId`、`timestamp`、`payloadSchemaVersion`、`payload`。在任务上下文中增加 `runId`、`taskId`、`attemptId`、`causationId`、`idempotencyKey`。涉及会话写入需 `sessionBindingId`、`bindingGeneration`、`ownershipEpoch`、`leaseId` 或可验证的短期授权引用。

消息字段内容不得用于替代 Core 中的权限记录。外部 Adapter 无权自行为自己签发授权。

## 4. 接受、执行、核验、交付
- `RECEIVED`：传输侧已接收，不证明持久化或执行。
- `ACCEPTED`：Adapter 确认受理，不证明已经向外部应用提交。
- `SUBMITTED`：有证据表明输入已发送到目标会话；证据本身有来源/局限。
- `RUNNING`：外部操作进入运行状态。
- `RESULT_OBSERVED`：看到对应结果，不证明已存入 Core。
- `RESULT_PERSISTED`：Core 持久化成功。
- `DELIVERED`：投递到指定接收端并按交付契约确认。
- `ACKED`：接收方确认对应结果；未必说明业务验收通过。
- `VERIFIED`：独立验收器证实约定业务条件达成。

执行状态、交付状态、验收状态三轴独立存储。失败和 `OUTCOME_UNKNOWN` 必须保留未知而不编造成功。

## 5. 请求重试
- `messageId` 表示单个消息，`idempotencyKey` 表示同一业务效果请求；不能互相替代。
- 同一业务键已经进入“可能发送到外部应用”的阶段时，禁止不经对账就重新执行副作用操作。
- 已持久化结果可按接收者要求进行幂等重新交付，不重新生成结果。
- 外部产品不支持操作幂等时，不承诺端到端 exactly-once。

## 6. 能力语义
每个 Capability 必须声明：名称、Schema、最小权限、副作用级别、幂等语义、可取消性、可并行性、完成证据以及错误类别。使用 JSON Schema 检查输入输出，但 Schema 不是安全沙箱。

## 7. 错误分类
`CONFIG_INVALID`、`VERSION_INCOMPATIBLE`、`PERMISSION_DENIED`、`SESSION_MISMATCH`、`RESOURCE_BUSY`、`CONNECTION_LOST`、`RESULT_UNCERTAIN`、`USER_TAKEOVER`、`EXTERNAL_APP_FAILURE`、`DELIVERY_UNCONFIRMED`、`OUTPUT_SCHEMA_INVALID`。
每个错误包含 phase、涉及的实例/任务、外部副作用是否可能发生、可否安全重试、所需恢复动作、证据引用。

## 8. 版本与兼容
协议大版本不兼容须拒绝握手或走已测试转换层。模块版本、Capability Schema 版本与 Workflow Revision 分别记录。工作流执行中的模块升级不得悄悄改变已有能力语义。
