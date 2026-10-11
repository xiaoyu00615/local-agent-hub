# ARCH-06-05｜COD 开发线交接草案（尚未发送）

**状态：HANDOFF_DRAFT_ONLY — 禁止自动启动编码、执行、提交或部署。**

## 1. 交接目标

在已冻结 F1、未冻结 F2、原 E00 31 caseId 与现有独立仓库基础上，审查 ARCH-01～05 扩展：Brain Connector、受约束模型路由、Workflow Graph、动态计划、持续迭代、Checkpoint、Memory、只读 Task Flow Projection。**评审是在 F1 之上提出可兼容增量，而不是重建 Core、套用旧 Relay 代码。**

## 2. 不得改变的事实与边界

- 产品仓库：`D:\Workspace\projects\local-agent-hub`；远程 `main@d666bfc`、Tag `f1-baseline-v1.0`。
- Relay 是外部辅助开发工具，正式产品完全独立；禁止把旧 Relay 任务管理、重试、密钥、数据库迁入 Hub。
- F1 12 条已批准；F2 Candidate/E00 未实施验证。E00-001 原 7 项、E00 31 项不得扩容或冒充已通过。
- 暂不允许真实 WorkBuddy/DSH/MJ/browser/API 的执行、写入与数据上传；无用户单独授权，不创建实现分支/commit，不修改 F1 或扩展系统状态。

## 3. 未来 COD 的 10 项设计审查任务

| ID | 评审内容 | 输出 |
|---|---|---|
| COD-ARCH06-01 | Core/Brain Gateway/LMP/SDK 的分层及现有接口复用 | 边界图 + 无第二调度中心说明 |
| COD-ARCH06-02 | 认证 TaskProposal、来源绑定、去重、提案→Task 的门禁 | JSON Schema 候选 + 反例 |
| COD-ARCH06-03 | Model Catalog、Model Policy、模型真实生效证据 | 能力目录 + 证据等级 + 回退状态矩阵 |
| COD-ARCH06-04 | Graph Node/Port/Edge 与已有 Capability I/O Schema 映射 | 共享字段字典 + 静态 Validator 草案 |
| COD-ARCH06-05 | Workflow Revision/Run/DynamicPlan/Iteration/Attempt 身份关联 | ER 图 + 修改/冻结规则 |
| COD-ARCH06-06 | BrainSession/PlanningEpoch/安全交接和迟到提案 | Ownership 状态机 + race 用例 |
| COD-ARCH06-07 | Journal/Outbox/Checkpoint/Memory 的持久化与恢复 | 数据源权威声明 + 崩溃注入矩阵 |
| COD-ARCH06-08 | AutonomyPolicy（无限轮数、预算、停滞、权限） | 运行守卫与负面测试设计 |
| COD-ARCH06-09 | P04/P05 共享数据投影，journalSeq 与断线续传 | 查询/事件投影契约草案 |
| COD-ARCH06-10 | F1 兼容、F2 Open Issues、E00–E07 门禁与新增专项测试 | 风险差距报告 + 独立测试矩阵 |

## 4. 必须统一的字典

`ModulePackage`, `ModuleRuntime`, `Endpoint`, `ConnectionInstance`, `SessionBinding`, `CapabilityDescriptor`, `ModelCatalogEntry`, `ModelSelectionPolicy`, `BrainSession`, `BrainBinding`, `PlanningEpoch`, `TaskProposal`, `WorkflowDraft`, `WorkflowRevision`, `GraphNode`, `Port`, `EdgeMapping`, `Run`, `Iteration`, `DynamicPlanRevision`, `NodeExecution`, `Task`, `Attempt`, `DeliveryAttempt`, `ArtifactRef`, `Evidence`, `Grant`, `Checkpoint`, `MemoryRecord`, `TaskFlowProjection`。

核心冲突核查：`NodeDefinition`≠`NodeExecution`；`Iteration`≠`Attempt`≠`DeliveryAttempt`；`BrainPlanningEpoch`≠`AppSessionOwnershipEpoch`；`ModelCatalogEntry`≠模型真实提供商身份；`RunState`≠结果存储、投递或验收状态。

## 5. 强制的安全审核场景

1. ChatGPT 网页输出 `USE: WorkBuddy`，未经过可信提交，不得生成已授权 Task。
2. Brain A 失联、Brain B 接管，Brain A 的迟到提案必须被拒绝。
3. Brain B 想把项目文件发给新的模型提供商，缺新的 Disclosure Grant 则拒绝。
4. WorkBuddy 已可能修改文件但 ACK 丢失，不得用“换模型/换 App/新 Iteration”再次提交相同效果。
5. 用户操作 App 触发接管后，重启、重连、模型切回都不能解除人工恢复确认。
6. API 调用实际价格/剩余额度未知，不能默认为免费、可无限调用。
7. 已启动 Run 更换能力契约/收件方或超出许可资源边界，不得静默生效。
8. Module Worker 没有独立 OS 级安全保证时，不得对外宣称硬隔离。

## 6. 如果未来进入实现，最小切片建议

- S0：在 E00 既定 Mock 门禁中验证 LMP、身份/ACK/幂等/权限/恢复，**不加入新功能**。
- S1：只用模拟模块的“固定路由 → 单 Task → Verify → Result Delivery”，有持久化证据。
- S2：受认证的 Brain TaskProposal 候选入口，先不接真实网页。
- S3：Capability/Model Catalog + 连接范围验证，假模型无需真实付费 API。
- S4：Graph 线性定义/预检/发布 + P05 最小只读投影。
- S5：通过所有需要的 Gate 后才实接 API/Web/GUI Adapter；再做备用 Brain、记忆、长期迭代。
- S6：后续 Scheduler、多分支、子工作流、媒体批处理。

该顺序是建议，不能跳过真实文件、数据外传和外部 App Gate 直接把模拟测试当无人值守许可。

## 7. COD 交付定义（未来评审时）

- [ ] 一份 F1 / F2 / ARCH-06 逐项差距矩阵，不能模糊写“兼容”。
- [ ] 一份候选 Schema 字典（标版本、身份签发者、可空/必需、权限、来源）。
- [ ] 至少三份全 Mock 示例：Direct Task、Planner→Executor→Verify、Brain Failover/Handoff。
- [ ] 明确哪些需要 ADR、哪些归 E00、哪些另设 `ARCH-TST-*`；所有未运行保持 NOT_RUN。
- [ ] 告知 UI 需要的 Projection/Inspector Schema，而不是单独口头定字段。
- [ ] 无真实程序操作、无 F1 改写、无擅自 Git 提交；返回本讨论线统一裁决。

**停止点**：审查草案完成后等待用户授权；ARCH-06 此时并未分派任何 COD 工作。
