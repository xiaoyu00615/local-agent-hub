# ARCH-06-04｜工程路线图、阶段门禁与可打勾任务表

## 1. 总体推进规则

当前是**架构研究/交接准备**。实施前用户必须单独明确授权；已有 E00–E07 不会因为新架构被替换、挤占或宣称已通过。第一条真实闭环的安全/证据能力优先于自由画布与高级调度。

### 路线图（依赖优先，不做时间承诺）

| 阶段 | 交付 | 依赖与退出门槛 | 当前 |
|---|---|---|---|
| F1 基线 | 12 条已批准不变量，Git Tag | `main@d666bfc`, `f1-baseline-v1.0` | DONE（已核验远程） |
| ARCH-01～05 | 新扩展候选架构讨论 | 用户方向、F1 兼容评估 | REVIEWED（非实现） |
| ARCH-06 | 整合总规范与待交接包 | 本包完成、待统一确认 | DRAFT |
| E00-001～005 | 现有 LMP/SDK Mock 验证（31 原始 caseId） | 单独开发授权；全部适用测试 PASS + F2 裁决 | NOT_RUN / PAUSED |
| E01～E05 | 工作区、网页、GUI、DSH、受控 Test 实机验证 | 分 Gate 真实证据 | NOT_VERIFIED_HERE |
| EXP-01 | 固定路由 + 线性 Workflow + TaskSpec/Artifact + P05 最小投影 | E00 + 涉及的权限/接入 Gate | FUTURE |
| EXP-02 | Brain Gateway / API model catalog / FREE_FIRST | 已授权 API / Disclosure / 验证等级 | FUTURE |
| EXP-03 | Checkpoint / Backup Brain / Memory Context Handoff | 错会话、Epoch、历史证据/恢复专项测试 | FUTURE |
| EXP-04 | 动态 Plan + 持续 Iteration Controller | unknown effect/预算/接管/停滞专题验收 | FUTURE |
| EXP-05 | Scheduler、并行、子工作流、历史回放 | 真实资源竞争、错过触发、GUI 无人值守验证 | LATER |
| E06/E07 | 真实恢复与 Console 验收（涉及能力逐项） | 对应真实场景 PASS | NOT_VERIFIED_HERE |

E06/E07 的实际安排按所需功能前置 Gate 规划，表格不暗示 E00–E07 能全部线性一次性完成。F2 的冻结需要独立用户决策和测试证据，不由本包批准。

## 2. E00 保持原样

- E00 原始测试集合：`docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md` 的 **31 项**，全部必须以真实结果更新。
- 现有 E00-001 已计划首批 7 项：`E00-ID-01`、`E00-HS-01/02/03`、`E00-MSG-01/02`、`E00-SDK-02`。
- E00-002：ACK / 顺序；E00-003：去重/Outbox/Inbox；E00-004：权限、Ownership；E00-005：恢复、SDK、全量回归。
- 新增 ARCH 专项测试另建 `ARCH-TST-*` 集合；不覆盖此前 `EXP-TST-01..18` 也不把它们算作 31 项的一部分。所有未实测条目均 `NOT_RUN`。

## 3. ARCH 专项测试候选（全部 NOT_RUN）

| caseId | 关键断言 | 依赖 |
|---|---|---|
| ARCH-TST-01 | 伪造网页 `USE:` 文本不产生授权 Task | Brain Gateway mock + F1-05 |
| ARCH-TST-02 | 同一 clientRequestId 不产生第二个业务 Attempt，冲突入审计 | Core Inbox/Task |
| ARCH-TST-03 | Graph Port 类型或 Artifact 基数不匹配时不能发布 | Workflow Validator |
| ARCH-TST-04 | 无 Disclosure Grant，跨模型发上下文被阻断 | Policy + Handoff |
| ARCH-TST-05 | Model Catalog UNKNOWN 或未达最低证据时拒绝错误模型宣称 | Router + Adapter evidence |
| ARCH-TST-06 | APP_GLOBAL 模型配置冲突被资源锁拒绝/排队 | Resource Scheduler |
| ARCH-TST-07 | 旧 Brain Epoch 延迟提案不能发布新 Task | Brain Ownership |
| ARCH-TST-08 | 主 Brain 不可用时在授权范围内选择备用 API 模型 | Brain Router + Grants |
| ARCH-TST-09 | 原 GPT 会话不可证实时不自动切回 | Web Bridge Gate |
| ARCH-TST-10 | 仅在 Handoff-Safe Checkpoint 切回，Snapshot 与 Delta 对齐 | Checkpoint + Journal |
| ARCH-TST-11 | 结果已保存而未交付只补交付，不新增外部 Attempt | Outbox/Delivery |
| ARCH-TST-12 | 外部结果未知，循环、fallback 或重启都不重发副作用 | Reconcile / Attempt |
| ARCH-TST-13 | 人工接管阻断在重启与 Brain 切换后仍保持 | F1-07 |
| ARCH-TST-14 | 无限总轮数模式仍受硬预算/停滞/资源控制 | Iteration Controller |
| ARCH-TST-15 | AI 记忆摘要无法改写有证据的事实 | Memory / Journal |
| ARCH-TST-16 | UI 快照 + journalSeq 断线续传不漏/不重复应用事件 | P05 Projection |
| ARCH-TST-17 | P04 Node 定义与 P05 多次 NodeExecution 映射正确 | Graph Runtime |
| ARCH-TST-18 | Core 关闭/电脑睡眠后恢复不会虚报无人值守执行 | Recovery + UI 状态 |

这些是**测试设计建议**，未分配实际执行人、环境或通过证据；需要后续单独制定 caseId、输入、期望、日志、复测及阻断门槛。并非“原扩展包的 EXP-TST 18 项”逐条原文替代。

## 4. 验收证据格式

每项工程测试必须提供：Case ID、版本/Commit、环境指纹、预期、实际、日志 `journalSeq`、Run/Task/Attempt 关联、外部副作用风险、脱敏证据、结论 `PASS/FAIL/BLOCKED/NOT_RUN`、失败的 defectId。UI 演示数据只能标 Mock，不算真实系统 PASS。

## 5. 项目级检查表（只有真实事实才打勾）

### 已确认的仓库事实
- [x] F1 12 条经用户批准，标签 `f1-baseline-v1.0` 已存在。
- [x] 初始规范及 F1 审查记录已提交到 GitHub `main@d666bfc`。

### ARCH-06 讨论成果
- [x] 对照 F1、既有 F2 Candidate 和 E00 门禁完成本轮书面兼容评审。
- [x] 统一 Brain、Core、Adapter、Workflow 与 Journal 的职责边界形成候选规范。
- [x] 形成 COD/UI 双线待交接草案（**未交接**）。
- [ ] 用户确认整套 ARCH-06 架构草案作为后续评审基线。
- [ ] 完成关键 Open Issues 的正式裁决/实验证据。
- [ ] 形成受批准的新版 ADR 或工程切片授权。

### 工程状态
- [ ] E00-001 开发授权与模拟实现。
- [ ] E00 原始 31 项符合性测试全部实际通过。
- [ ] F2 经过独立审批冻结。
- [ ] 真实网页 Brain 和原会话回传实测。
- [ ] 真实 GUI / DSH / MJ 等各 Adapter 对应能力等级实测。
- [ ] 长期任务恢复、模型故障切换和预算守卫专项测试通过。
- [ ] COD 正式签收、评审和提交实现方案。
- [ ] UI 正式签收、评审并对现有 Webflow 验收资产进行保护性扩展。

## 6. P0 阻断清单

任何阶段发现：未经授权操作、错误会话自动写入、人工接管未确认恢复、外部结果未知盲重试、ACK/投递/验收混淆、假称独立 Worker 是强沙箱，都必须阻断对应 Gate，不能通过降级 UI 文案或跳过测试放行。
