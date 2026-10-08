# 07｜后续实施交接规范（不代表现在开工）

## 1. 新项目边界

Local Agent Hub 必须创建全新独立项目与仓库，不能直接在其他进行中的项目里增加业务模块。未来只复用经过审查的通用设计或已获准的技术基础。V1.4 阶段**未批准开发实现**。

## 2. 进入实现前最小资料包

- 用户已确认需求表（00）和 F1 评审记录。
- LMP 消息契约（01）、状态/ACK（02）、SDK Manifest（03）、权限和 Ownership（04）。
- E00 测试矩阵（05）和 Gate 失败阻断规则。
- 所有开放问题（06），以及未来真实环境信息。

## 3. 建议首批任务包（仅任务定义，不含代码）

| 工作包 | 范围 | 验收出口 |
|---|---|---|
| WP-01 Identity & Schema | Identity Registry、Manifest、版本协商 | E00-ID/HS/MESSAGE |
| WP-02 Journaling | Task/Attempt/Outbox、journalSeq、ACK | E00-ACK/DB |
| WP-03 Ownership & Grant | Session Binding、Epoch/Lease、权限校验 | E00-OWN/PERM |
| WP-04 Adapter SDK Mock | Worker 生命周期、Capability、Mock Agent/Reader | E00-SDK/LIFE |
| WP-05 Recovery Simulation | 乱序、重复、崩溃窗口与结果补交付 | E00-DUP/REC |
| WP-06 Conformance Report | 自动执行 E00 并输出规范证据 | 全部 E00 PASS，零安全阻断 |

顺序可内部并行开发，但不应提前运行需要真实 GUI 或执行外部副作用的试验。

## 4. 完成定义 DoD

1. 新模块不改 Core 应用专属逻辑即可注册；删除该模块后其他模块正常。
2. 所有必填字段、Schema 和版本不兼容均被正确拒绝。
3. ACK 不串义，结果落库后才发持久化 ACK。
4. 幂等和结果未知处理符合 02；未确认外部提交不盲目重做。
5. 人工接管后即使模拟对账通过也需显式用户确认才能恢复。
6. Local Read / Disclosure / Execute 独立检查；不允许网页正文提升权限。
7. E00 每个用例都有 PASS/FAIL/BLOCKED 和脱敏证据。
8. 不将模拟测试通过写成真实 Web/WorkBuddy/DSH/Terminal 验收成功。

## 5. 阶段出口

- **本阶段文档出口：** V1.4 候选契约已整理，可供评审。
- **F1 正式冻结出口：** 用户确认十项架构和安全不变量。
- **F2 正式冻结出口：** E00 模拟协议测试全部通过并处理冲突，确定可实现版本。
- **实用功能出口：** 后续 E01–E07 分别通过，不可由 F2 替代。

## 6. 下一项建议

在用户批准 F1 后，进入独立新项目的 E00 模拟 Adapter 工程实现与验收；如果尚未批准开发，则先做最后一次规范审查，不创建仓库、不运行外部执行器。
