# 02｜任务、ACK、幂等与恢复契约

**状态：F2 CANDIDATE，E00/E06 未执行。**

## 1. ACK 四层，严格禁止串义

| 事实 | 标准名称 | 产生者 | 仅能证明 | 明确不能证明 |
|---|---|---|---|---|
| 传输收到帧 | `transport.frame.received`（诊断，可选） | Transport | 帧已抵达传输层 | Adapter 已接单或持久化 |
| Adapter 接单 | `ack.command.accepted` | Adapter/Host | 入站验证/去重后受理逻辑操作 | 外部消息已提交、Agent 已完成 |
| Core 安全保存结果 | `ack.result.stored` | **Core** | 关联结果已经成功提交 Core 持久化事务 | 下游已收到、业务正确 |
| 目标确认交付 | `ack.delivery.confirmed` | 目标接收端经 Core 验证记录 | 满足该接收端明确的交付判据 | AI 已理解、任务业务验收成功 |

网页 GUI 可能只能证明“内容已出现在发送区/对话区”，无法证明平台层面的真正收悉，此时交付停留在 `DELIVERY_UNCONFIRMED`，不得冒用 `ack.delivery.confirmed`。

每条 ACK 必须明确：`acknowledgesMessageId`、`ackType`、`correlationId`、确认端点、可验证证据引用和是否持久化。`RESULT_STORED` 只能在 Core 持久化事务成功后发出。

## 2. 三轴状态机

### 2.1 ExecutionState（针对 Task/Attempt）

| 状态 | 含义 | 合法后续方向 |
|---|---|---|
| `CREATED` | Core 已创建意图，尚未派发 | QUEUED / CANCELLED |
| `QUEUED` | 待能力、权限和资源就绪 | ACCEPTED / CANCELLED / BLOCKED |
| `ACCEPTED` | Adapter 已受理，尚未有外部提交的可靠证据 | SUBMITTED / RUNNING / FAILED / OUTCOME_UNKNOWN |
| `SUBMITTED` | 外部提交有明确证据 | RUNNING / SUCCEEDED / FAILED / OUTCOME_UNKNOWN |
| `RUNNING` | 有运行中的证据 | SUCCEEDED / FAILED / OUTCOME_UNKNOWN |
| `SUCCEEDED` | 操作本身已按该能力完成标准结束 | 终态；业务验证独立处理 |
| `FAILED` | 有明确失败证据，副作用须另判断 | 终态；可建立经批准的新 Attempt |
| `CANCELLED` | 外部动作已按规定证明取消 | 终态 |
| `OUTCOME_UNKNOWN` | 外部动作是否生效无法确认 | 原 Attempt 保持未知；通过独立对账后记录 resolution |
| `BLOCKED` | 缺少权限、依赖或资源 | 按具体原因可回到 QUEUED |

注意：由于事件可能缺失，`SUBMITTED` 可以直接进入 `SUCCEEDED`，不能要求所有外部程序一定先上报 `RUNNING`。`FAILED` 若表示“执行失败但副作用未知”，必须同时记录 `sideEffectStatus=UNKNOWN` 并进入人工或自动对账，不允许新 Attempt 自动运行。

### 2.2 DeliveryState（对应 result × recipient）

`NOT_REQUIRED → PENDING → SUBMISSION_ATTEMPTED → DELIVERED / DELIVERY_UNCONFIRMED → ACKED`。

- `DELIVERED` 表示按具体传输/应用判据已提交或交付；**不保证目标 AI 理解**。
- `ACKED` 只表示目标端满足约定的确认条件，不等于验收通过。
- 收件方、会话、结果 Artifact 与交付请求必须形成唯一组合；中途变更收件人属于新授权动作，不是旧结果的“自然续传”。

### 2.3 VerificationState（对应业务验收）

`NOT_CHECKED → VERIFYING → PASSED / REJECTED / INCONCLUSIVE`。

- Agent 文本自称“完成”不等于 `PASSED`；由业务核验器检查真实文件/测试证据。
- `INCONCLUSIVE` 表示证据不足，不得改写为 `PASSED`。

## 3. 事实事件不是状态枚举

`command.accepted`、`external.submitted`、`external.running.observed`、`result.observed`、`result.stored`、`delivery.confirmed`、`verification.passed` 是**记录在事件日志中的事实**。Execution/Delivery/Verification 为 Core 依据已验证事实派生的状态。

使用 Core 提交时分配的 `journalSeq` 决定本地持久化先后；处理传入的迟到事件时参考 `messageId`、`attemptId`、因果关系及证据，不能靠外部时间戳随意覆盖。

## 4. 幂等和重复策略

| 对象 | 去重键 | 重复发生时做什么 |
|---|---|---|
| 同一 LMP 消息 | `messageId` + 认证发送端 | 返回已经记录的确认或去重结果，不重新解释为新业务 |
| 同一业务效果 | `idempotencyKey` + `capability` + 目标实例 | Core 返回当前权威状态；若已可能提交外部，不重复外部写入 |
| 同一结果持久化 | `taskId` + `attemptId` + `resultIdentity` | 去重保存，发现内容不一致时标记冲突 |
| 同一结果对同一接收者 | `artifactId` + recipient binding + delivery intent | 按接收端支持的幂等语义处理；未知时停止，不默认多次发送 |

请求重试应区分：**运输重试**（同一业务尝试、重新发送帧）、**结果补交付**（已有结果的交付）、**业务重新执行**（真正的新 Attempt）。三者不能共享一个无差别 retry 按钮。

## 5. Outbox/Inbox 与崩溃窗口

1. Core 持久化 Task + Attempt + Outbox 意图的原子事务。
2. 派发器仅从已提交 Outbox 发送；完成情况独立记录。
3. Host/Adapter 以业务键和消息键记录接收事实，并对重复入站调用幂等处理。
4. 如果 Core 在发送后崩溃，而受理状态未持久化，恢复后必须核对外部事实，不能直接再执行有副作用动作。
5. 结果进入 Core 后，事务提交完成才允许产生存储 ACK；已存储但未交付的结果通过交付 Outbox 处理。
6. 实际 GUI/API 系统若无法对外部动作提供唯一约束，不承诺严格 exactly-once。

## 6. 人工接管及断线恢复

- `ownership.takeover.request` 被受理后：阻止会话新的自动写许可，递增 `ownershipEpoch`，保留当前外部执行事实。
- Adapter 检测到未知外部改变：报告证据并请求暂停；不能凭推测证明“由用户操作”。
- Lease 到期/撤销只阻止新的受控写入，**不能保证已提交动作已停止**。
- `RECONCILING` 由 Core 发起，Adapter 仅提供外部状态与证据，不可直接改动权威状态。
- 对账通过且曾发生人工接管：Core 仍保留 `resumeApprovalRequired=true`；用户显式批准才清除阻断并签发新的 Epoch/Lease。
- 用户点击取消：发出取消请求和观察结果，不自动设置 `CANCELLED`；外部状态未知则 `OUTCOME_UNKNOWN`。

## 7. 最小恢复决策表

| 已证实最后阶段 | 唯一安全默认处理 |
|---|---|
| 意图已创建，且能证明尚未对外提交 | 重新进入调度 |
| 是否已提交不明 | 暂停并核对；不得重发 |
| 外部提交成功且仍运行 | 仅恢复观察原任务 |
| 外部已完成、结果未保存 | 关联当前会话与轮次后补采集 |
| 结果已安全持久化、未交付 | 重验收件人/Disclosure Grant，再尝试按约定幂等交付 |
| 交付状态未知且目标网页不保证幂等 | 暂停，要求核对或人工操作 |
| 人工接管后申请恢复 | 核对成功后仍等待用户明确确认 |
| 结果证据相互矛盾 | 标记冲突并进入 MANUAL_REVIEW |

## 8. 错误契约

最小错误字段：`code`、`phase`、`taskId/attemptId`（如适用）、`sideEffectStatus`（NONE/CONFIRMED/UNKNOWN）、`retryDisposition`（SAFE / RECONCILE_FIRST / DENY）、`recoveryAction`、`evidenceRefs`、无敏感信息的 `message`。

最低统一错误码：`CONFIG_INVALID`、`VERSION_INCOMPATIBLE`、`SCHEMA_INVALID`、`PERMISSION_DENIED`、`SESSION_MISMATCH`、`RESOURCE_BUSY`、`CONNECTION_LOST`、`OUTCOME_UNKNOWN`、`USER_TAKEOVER`、`EXTERNAL_APP_FAILURE`、`DELIVERY_UNCONFIRMED`、`DUPLICATE_CONFLICT`、`PAYLOAD_TOO_LARGE`。
