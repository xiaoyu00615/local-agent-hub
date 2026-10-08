# 05｜跨模块术语、状态与消息边界对齐清单

## 1. 系统对象词典
| 对象 | 谁分配/维护权威 ID | 不得混为 |
|---|---|---|
| Module Package | Module Registry | 运行进程或会话 |
| Module Runtime | Module Host | 模块安装记录 |
| Connection Instance | Core + Adapter 绑定证据 | 具体 Agent Session |
| Session Binding | Session Manager | 窗口 PID/Tab ID |
| Workflow Revision | Workflow Engine | 正在编辑的草稿 |
| Run | Workflow Engine | 一次 Agent Turn |
| Task | Task Engine | 消息投递尝试 |
| Attempt | Task Engine | 业务操作唯一性键 |
| Grant | Permission Gateway | 网页/Adapter 自报权限 |
| Artifact | Artifact Store | AI 文本中提到的文件名 |

## 2. 建议分层状态（具体枚举仍待 F2）
- Module Runtime：STOPPED/STARTING/READY/DEGRADED/DRAINING/FAILED/QUARANTINED。
- Binding：VERIFIED/PROVISIONAL/LOST。
- Ownership：AVAILABLE/AUTOMATION_OWNED/TAKEOVER_PENDING/HUMAN_OWNED/RECONCILING/SUSPENDED。
- Execution：CREATED/QUEUED/ACCEPTED/SUBMITTED/RUNNING/SUCCEEDED/FAILED/CANCELLED/OUTCOME_UNKNOWN。
- Delivery：NOT_REQUIRED/PENDING/DELIVERED/ACKED/DELIVERY_UNCONFIRMED。
- Verification：NOT_CHECKED/VERIFYING/PASSED/REJECTED/INCONCLUSIVE。
- Workflow Run：CREATED/VALIDATING/RUNNING/WAITING_APPROVAL/WAITING_EXTERNAL/PAUSED/RECONCILING/BLOCKED/SUCCEEDED/FAILED/CANCELLED。

**注意**：这些枚举中包含汇总态和事件事实；具体合法转换仍需 E00/E06 验证，不能直接拿来称作已冻结状态机。

## 3. 消息事件证明边界
| 事件/事实 | 最多能证明什么 | 明确不能证明 |
|---|---|---|
| Transport Received | 下层收到该数据帧 | Adapter 已持久化/受理 |
| Adapter Accepted | Adapter 声明接单 | 已发送到目标 AI |
| External Submitted | 有证据表明提交给目标 | AI 开始或完成操作 |
| External Running | 目标正在运行的可观察证据 | 文件已完成修改 |
| Result Observed | 发现关联轮次的结果 | Core 已安全保存 |
| Result Persisted | Core 提交成功 | 下游已收到 |
| Delivery Confirmed | 交付契约定义的对端接收 | AI 已理解/业务已成功 |
| Verification Passed | 独立业务验证条件符合 | 历史所有步骤都无风险 |

## 4. 命名空间建议
- Module/Connection：`module.describe`、`module.health`、`connection.open`、`connection.close`。
- Agent：`agent.session.bind`、`agent.session.inspect`、`agent.turn.submit`、`agent.turn.observe`、`agent.turn.collect`，`agent.turn.cancel` 可选。
- Workspace：`workspace.list`、`workspace.tree`、`files.find`、`files.search`、`files.readRange`、`files.stat`、`files.hash`。
- Tool：`terminal.profile.run`、`test.profile.run`。不默认暴露 `shell.exec(anyString)`。
- Ownership/Recovery：`ownership.acquire`、`ownership.takeover.request`、`session.reconcile.request`、`workflow.resume.request`。

## 5. 消息信封候选字段（不是已冻结 API）
通用：protocolVersion、messageId、kind、sourceInstanceId、targetInstanceId、correlationId、timestamp、payloadSchemaVersion、payload。与 Run 关联时携带 runId/taskId/attemptId/idempotencyKey。会话自动写入需 bindingId/generation/epoch 和 Core 可验证的授权引用。**消息的授权引用不是授权本身；Core 必须查自己的有效许可记录。**

## 6. 关键时序不变量
- 派发意图持久化与真正发送的间隙必须能识别，重启后未确认的副作用不直接重发。
- 结果只有在 Core 已成功持久化后才对模块确认“已保存”。
- 交付时重验接收者会话身份与数据外传授权，不仅在任务创建时检查。
- 人工接管导致当前 Epoch 失效，下一动作提交前需重新核验；无法追回已提交外部任务。
- UI 按钮调用 Core 命令后先显示处理中，以权威事件/查询结果显示成功，不由前端直接改权威状态。
- Module DRAINING/STOPPED 不等于目标应用已停止，可能仍需会话对账。
