# ARCH-06-02｜候选数据契约、状态与接口映射

**全部字段、名称和示例均处于 `CANDIDATE`；不替换 V1.4 F2，不代表实际网页/App 能力已验证。**

## 1. 统一术语与实体关系

| 现有 F1 实体 | 追加概念 | 关系 / 约束 |
|---|---|---|
| Module Package / Runtime / Endpoint / Connection / Session Binding | Model Catalog Entry | 模型能力归属于具体模块 + 连接配置；不借 URL/PID 单独确认会话 |
| Workflow → Revision → Run | Graph Node / Port / Edge | 静态定义属于 Revision；运行中冻结 |
| Run → NodeExecution → Task → Attempt | Iteration / Dynamic Plan Revision | Iteration 可包含多个业务 Task/Attempt；不替代原主键 |
| Message / DeliveryAttempt | TaskProposal / HandoffPackage | 提案是业务入口对象；传输重试不是新业务尝试 |
| Grant / Lease / Epoch | Brain Planning Ownership | Brain 规划者 Epoch 与 GUI/App 写入控制相互独立，仍由 Core 统一校验 |
| Evidence / Artifact | Memory Record / Context Snapshot | AI 摘要不是证据；跨服务上下文传播仍需 Disclosure |

建议保留 `nodeId`（定义时）与 `nodeExecutionId`（运行时）两个不同对象；对重复执行的同一 Node 使用不同 NodeExecution。规划者切换不改变既有 Attempt。

## 2. Brain 入口 `TaskProposal`（抽象业务 Schema）

```json
{
  "kind": "brain.task.propose",
  "proposalSchemaVersion": "0.1-draft",
  "clientRequestId": "example-001",
  "goal": "在授权工作区内修复并验证功能",
  "requestMode": "AUTONOMOUS",
  "requestedCapabilities": ["agent.turn.submit", "test.profile.run"],
  "preferredExecutor": "workbuddy",
  "modelPolicyRef": "policy-free-first",
  "workspaceRef": "workspace-approved-01",
  "autonomyPolicyRef": "policy-autonomy-01",
  "verificationRequired": true
}
```

**必须注意**：`clientRequestId` 仅为提交去重线索，不是具有副作用操作的幂等键；真实 `principalId`、Connector/Endpoint/Session 身份、Grant、Lease、权限和最终投递目的由可信通道与 Core 权威记录校验，不能相信 AI 自报字段。网页文本中的 `USE: WorkBuddy` 仅可能产生待审核草稿，不能触发无授权执行。对于 LMP 适配，按现有消息 Kind、name、correlationId、messageId 和实际握手绑定设计；不将本 JSON 直接替代 LMP Envelope。

## 3. Brain Connector 职责

- `describeCapabilities`：已声明/已验证的会话定位、发送、接收、模型选项和结果交付证据等级。
- `bindSession`：注册目标会话与外部身份观察，生成/更新 Core Session Binding。
- `submitProposal`：认证的结构化候选意图；入 Core 验证门禁。
- `deliverContext`：从 Core 取最小必要上下文并进行目标 Disclosure 校验。
- `collectDecision`：只接收当前规划 Epoch 对应的结构化建议。
- `observeDelivery`：报告发送动作、目标接收可见性和可验证结果；不将点击等同于收讫。

这些是 SDK **职责抽象**；正式实现需映射既有 SDK `describe/connect/invoke/observe/reconcile`，不能随意新增与 LMP 冲突的消息种类。

## 4. Model Catalog 与选择证据

```text
ModelCatalogEntry(candidate)
  modulePackageRef / connectionInstanceId
  modelOptionId / providerId? / displayName
  supportedTaskClasses / inputOutputModalities
  configScope = REQUEST | SESSION | APP_GLOBAL
  availability = AVAILABLE | UNAVAILABLE | UNKNOWN
  selectionMethod / verificationMethod
  evidenceLevel = DECLARED | OBSERVED | CONFIG_CONFIRMED | PROVIDER_CONFIRMED
  costStatus = FREE_CONFIRMED | PRICED | UNKNOWN
  rateLimitEvidence? / catalogVersion / observedAt / evidenceRefs[]
```

最低策略是先能力匹配、身份/会话可用、权限/Disclosure、费用与额度约束，再在**可行候选集合**中免费优先。未知费用不得推断为免费；网页菜单确认不等于服务提供商承诺使用指定模型。`APP_GLOBAL` 切换必须考虑其他任务共享的配置资源锁。

## 5. Workflow Graph 与动态计划

```text
WorkflowRevision (immutable)
  nodes[]: nodeId, nodeType, capabilityRef, capabilityVersion,
           inputPortDescriptors[], outputPortDescriptors[], routePolicyRef,
           modelPolicyRef, requiredGrants, verificationContract
  edges[]: sourceNode/Port, targetNode/Port, edgeKind=CONTROL|DATA,
           mappingSchemaVersion, sensitivityClass
  policyEnvelope: allowedCapabilities, allowedConnections, modelPool,
                  workspaceScopes, disclosureRecipients, budgetLimits,
                  iterationPolicy, stopConditions

DynamicPlanRevision (per iteration)
  planId, planRevision, runId, iterationId, sourceBrainEpoch,
  baseWorkflowRevisionId, permittedSteps[], evidenceRefs[],
  validationDecision, createdAt
```

发布与运行前独立校验：完整性、类型、能力/版本、参数、会话绑定、Model Policy、三轴权限、预算、资源锁和终止/异常策略。静态校验通过不能替代执行时动态检查。允许的动态计划只能**实例化和选择被 Revision 明确授权的能力**，不能直接改写已启动 Revision；需要扩大能力/接收方/资源范围时进入新的审批/发布流程。

## 6. Typed Port / EdgeMapping

- 数据类型示例：`Text`, `Prompt`, `TaskSpec`, `ArtifactRef`, `ArtifactSet`, `TestReport`, `Evaluation`, `StructuredData(schemaRef)`。
- 每个端口定义：`name`, `direction`, `typeRef`, `schemaVersion`, `required`, `cardinality`, `sensitivity`, `sizeLimit`。
- Control Edge 表示调度前置/条件；Data Edge 表示明确字段或 Artifact 引用映射。连接数据边**不隐含**自动外传权限。
- 大文件必须以经过访问控制的 ArtifactRef 表示；不能在 LMP 消息中内嵌不受限内容。
- 若 MJ 输出 ImageSet，下游仅接收单张图，则需要显式选择/转换节点；不能靠“兼容图片类型”忽略基数约束。

## 7. Long-Running AutonomyPolicy 与 Iteration

```text
AutonomyPolicy(candidate)
  modelPolicyRef / approvedBrainProviders[] / approvedExecutors[]
  workflowRevisionId / workspaceScopes / allowedCapabilities[]
  maxIterations = null (可不设固定总轮数)
  cumulativeBudgetLimit / cycleBudgetLimit / nonzeroResourceLimits
  maxConcurrentWork / stallDetectionRule / reviewEscalationRule
  onOutcomeUnknown = STOP_AND_RECONCILE
  onHumanTakeover = REQUIRE_EXPLICIT_USER_RESUME
  onMissingEvidence = BLOCK_OR_REVIEW
  completionCriteria / policyVersion / approvalEvidenceRef
```

`Iteration` 是一次完整业务反馈周期；`Attempt` 是某个 Task 的一次可能产生副作用的业务尝试；`DeliveryAttempt` 是同一消息/结果的传输尝试。无限总轮数≠无限预算；一次外部结果未知，即使下一轮、更换模型或更换执行器也不得盲目重复相同副作用。

## 8. Brain Ownership / Handoff

```text
BrainSession(candidate): brainSessionId, runRef, goalRef
BrainBinding(candidate): provider, connectionInstanceId, sessionBindingId,
                         bindingGeneration, capabilities, evidenceRefs
PlanningLease(candidate): planningOwnerBindingId, planningEpoch, expiry,
                          checkpointRef, status
HandoffPackage(candidate): checkpointId, runId, revisionId, journalSeq,
                           taskGoal, verifiedFacts[], openQuestions[],
                           inFlightOperations[], artifactRefs[],
                           dataClassification, allowedRecipient
```

核心流程：Primary 不可用 → 阻断旧规划新派发 → 记录 Checkpoint → 检查备用 API Brain/数据外传/预算 → 原子更新规划 Epoch → 当前 Brain 承接。原 Primary 恢复且已确认会话、权限有效 → 等待 Handoff-Safe Checkpoint → Prepare Context → Target ACK（准确含义待实测）→ Commit 新 Epoch。旧 Epoch 的迟到提案拒绝。**当网页无法证明正确目标会话及接收能力时，不得自动切回，仅继续备用 Brain 或等待人工。**

## 9. Memory / Checkpoint 与来源

```text
MemoryRecord(candidate): memoryId, projectScope, taskScope?, kind,
                         claim, sourceRefs[], evidenceStatus,
                         createdAt, supersedesId?, sensitivity
EvidenceStatus = VERIFIED_FACT | MODEL_INFERENCE | PROPOSAL | SUPERSEDED
ContextSnapshot(candidate): checkpointId, journalSeq, memoryVersion,
                            includedRefs[], recipientScope, contentDigest
RecoveryCheckpoint(candidate): runId, journalSeq, pendingOutbox[],
                               unresolvedEffects[], currentEpoch, policyRefs
```

Journal/Artifact 是权威事实基座，Summary / Memory 仅供检索和上下文编排。保留事件顺序与原始证据引用。每次向新 API 提供商发送上下文必须依照最小披露原则再次校验 EXTERNAL_DISCLOSURE。不能要求各模型暴露不可读取的内部隐藏推理。

## 10. 数据状态分离

- `RunState`：继续沿用 V1.1 候选 Run 状态；不因新增长期任务再创建同义的权威 RunState。
- `ExecutionState`：受管 Task/Attempt 的真实已确认阶段，与外部仍运行的观察记录分开。
- `StorageState`：结果是否持久化（不得凭 ACK 自动成功）。
- `DeliveryState`：发送动作 vs 接收证明，特别是网页 `DELIVERY_UNCONFIRMED`。
- `VerificationState`：独立测试/人工验收记录。
- `BrainPlanningState`：仅主动规划控制，不等于 Session 自动写入控制。
- `ConnectivityState`：UI 流断线、Adapter 断线、应用断线分别表示。

UI 的“三轴”可以显示 Execution/Delivery/Verification，但内部结果存储证据单列，不能被压缩掉。

## 11. 去重与恢复顺序

1. 对任务意图与业务 Attempt 分配不同稳定身份；事务化写入 Outbox。
2. 发送可重试不等于业务可重做；Transport DeliveryAttempt 不导致新增业务 Attempt。
3. 重启时先从 Journal/Outbox 重建权威状态，按真实外部证据分组对账。
4. 已保存未投递只补交付，不新 submit；已提交但结果未知进入 RECONCILING。
5. 已人工接管后即使恢复成功，也必须在受信 UI 明确恢复。

## 12. 此阶段暂不冻结的实现问题

`moduleRuntimeId` 实际签发方、Endpoint 握手字段、ACK 存储判据、消息大小、凭据保护技术、Model Catalog 可信度数值、Brain 健康窗口时长、Context Snapshot 大小限制、Web 会话切回成功判据、Iteration 并发及恢复事务边界。均需 E00 或新增专项测试决定。
