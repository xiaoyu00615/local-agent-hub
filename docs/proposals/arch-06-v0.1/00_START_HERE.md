# Local Agent Hub · ARCH-06 统一架构交接包 V0.1

- **文件状态**：`ARCHITECTURE_REVIEW_DRAFT`，不是实施授权、协议冻结或测试报告。
- **生成日期**：2026-10-11。
- **独立产品仓库**：`D:\Workspace\projects\local-agent-hub`；`https://github.com/xiaoyu00615/local-agent-hub`。
- **已核查远程基线**：`main@d666bfc8f3fea570871d6f51d7215881f325aa7c`；Annotated Tag `f1-baseline-v1.0` 存在。
- **已批准**：F1 架构与安全不变量 12 条；**尚未批准/实测**：F2 最终消息契约、E00–E07 的完成、本文提出的新架构字段。
- **外部开发工具**：既有 Relay **仅用于辅助开发 Local Agent Hub**，不属于新产品的运行链路；不迁移其代码、数据库、任务状态机。
- **本轮行为**：只交付文档；未联系 COD/UI、未修改 GitHub、Webflow 或产品代码；不得据此自动开启 E00。

## 阅读顺序

1. `01_MASTER_ARCHITECTURE.md`：完整产品结构与关键运行链路。
2. `02_DATA_CONTRACTS.md`：Brain / Model / Workflow / Task / Memory 的候选数据契约。
3. `03_F1_COMPATIBILITY_AND_RISKS.md`：12 条已冻结原则的逐项兼容检查、风险与开放问题。
4. `04_ROADMAP_GATES_TESTS.md`：实施顺序、验收门禁、可勾选进度。
5. `05_COD_HANDOFF_DRAFT.md`：仅供未来 COD 技术评审，**本轮不分派**。
6. `06_UI_WEBFLOW_HANDOFF_DRAFT.md`：仅供未来 Webflow UI 评审，**本轮不分派**。
7. `07_DECISIONS_AND_CHECKLIST.md`：用户已确认 vs 设计建议、待审批事项、检查表。
8. `08_DIAGRAMS_AND_EXAMPLES.md`：可复制的 Mermaid 图与三类任务案例。

## 阅读时必须理解的状态标识

- `APPROVED_F1`：用户已批准且 Git 冻结的既有安全原则。
- `USER_DIRECTION_CONFIRMED`：用户在当前 ARCH-01～05 讨论中明确选择的产品方向；尚**不是**新版冻结协议。
- `RECOMMENDED`：本次架构整理提出的建议，待统一确认。
- `OPEN`：必须通过设计裁决或实测确认。
- `NOT_RUN`：尚无测试通过证据；不得标为 PASS。
- `BLOCKED_BY_GATE`：未通过前置条件，不可宣称具有实机执行保证。

## 更新自更早期《模块化工作流编排扩展 V0.1》的重要差异

1. 删除 Relay 作为正式运行依赖的任何理解。
2. 增加 Brain Gateway、Brain Session Ownership、Model Catalog、免费优先 API Brain 故障切换和安全切回。
3. 长期迭代允许**不设置固定总轮数**，但逐轮授权及预算、资源、停滞/结果未知停止条件不可省略。
4. 将“有限次 Iteration Group”升级为“受控 Iteration Controller”；首个实现仍应从线性/有界的模拟流程验证开始。
5. 将动态计划与不可变 Workflow Revision 严格区分。
6. 不对已有 P01–P08、DS08 或原 E00 测试重新标记完成/失败。

## 正式来源（仓库内优先级）

- F1：`docs/baselines/F1_ARCHITECTURE_SECURITY_v1.0_APPROVED.md`（F1 领域优先）
- ADR：`docs/decisions/ADR-F1-001.md`
- LMP Candidate：`docs/versions/v1.4/01_LMP_MESSAGE_CONTRACT.md`
- SDK Candidate：`docs/versions/v1.4/03_ADAPTER_SDK_AND_LIFECYCLE.md`
- Permission：`docs/versions/v1.4/04_PERMISSION_SESSION_CONTRACT.md`
- E00 Cases：`docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md`
- 工程门禁：`docs/engineering/E00_READINESS_AND_GATES.md`
- 历史 UI 草案：`docs/versions/v1.1/05_WORKFLOW_AND_UX.md`

本包是供评审的**增量设计材料**；如与 F1 已批准正文冲突，**F1 优先**，先提出 ADR 而不是自行改写 F1。
