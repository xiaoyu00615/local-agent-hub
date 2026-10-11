# Local Agent Hub · ARCH-07 最终架构裁决包 V0.3

- 日期：2026-10-11
- 状态：`TECHNICAL_DECISION_RECOMMENDED / USER_APPROVAL_PENDING`（技术裁决建议已形成；未自动构成用户批准、工程授权或 Git 冻结）
- 适用项目：`D:\Workspace\projects\local-agent-hub`
- 建议存放路径：`docs/proposals/arch-07-v0.3/`
- 本轮工作：只生成独立文档；未通知 COD/UI，未修改产品代码、GitHub 分支或 Webflow。
- 历史基线：F1 12 条已冻结；F2 LMP/SDK 仍为 Candidate；E00 31 项及 E01–E07 未见通过证据。
- 已有本地文件（由用户终端输出证明）：`docs/arch-06-review` 分支 `bc6bac0` 已新增 9 份 ARCH-06 提案；未确认该分支已推送。
- 保护：`docs/ui-design/` 仍未跟踪，未经授权不得移动、删除或混入 ARCH 文档提交。

## 阅读顺序

1. `01_FINAL_ARCHITECTURE_DECISION.md`：整体架构、功能边界、技术选型、MVP 范围与项目退出定义。
2. `02_STAGE_GATES_CHECKLIST.md`：分阶段可勾选的开发和验收清单、停线条件。
3. `03_CONFLICTS_ADRS_HANDOFF.md`：原规范冲突与权威优先级、待建立的 ADR、COD/UI 交接边界。

## 决策级别

- `FROZEN_F1`：已经用户正式批准且 Git 冻结，不能由本文修改。
- `USER_DIRECTION_CONFIRMED`：用户明确表达过的产品需求；不是外部 App 已通过实测的证明。
- `ARCH07_RECOMMENDED`：本轮技术负责人的最终方案建议；需要用户确认后成为后续新需求基线。
- `F2_CANDIDATE`：消息字段、SDK、ACK 的真实运行契约需通过 E00 和后续审批。
- `NOT_RUN`：没有实际测试证据，不能写为 PASS。

## 下一步的权限边界

如用户仅批准 ARCH-07 规划，并不等于授权 E00-001 代码编写、COD/UI 对话交接、Webflow 变更、真实 GUI 写入、文件外传或收费 API。以上每一类动作按约定另行授权。
