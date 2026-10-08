# Local Agent Hub V1.1 — 产品规范评审包

**状态：Review Draft 1（评审稿，非全部冻结）**  
**日期：2026-10-08**  
**范围：Windows 本机个人版；无产品代码实现。**

## 使用顺序
1. `00_PRODUCT_REQUIREMENTS.md` — 产品总规范、确认项和非目标
2. `01_SYSTEM_ARCHITECTURE.md` — 组件边界、数据职责与信任边界
3. `02_LMP_PROTOCOL.md` — 统一消息、能力与任务协议草案
4. `03_SESSION_AND_RECOVERY.md` — 会话、人工接管、幂等和恢复
5. `04_WORKSPACE_TOOLS_SECURITY.md` — 文件、终端、测试与权限
6. `05_WORKFLOW_AND_UX.md` — 工作流定义、界面及操作流
7. `06_ACCEPTANCE_MATRIX.md` — 端到端验收矩阵和测试门槛
8. `07_ADR_OPEN_ISSUES.md` — 决策日志及未解决的设计问题
9. `08_MILESTONES.md` — 后续冻结与交付门槛

## 文档权威性规则
- **CONFIRMED**：用户明确确认的产品级决定，跨文档继承。
- **PROPOSED**：评审建议；可进入设计，但不得被误写成已批准。
- **OPEN**：需要验证或决策的关键问题。
- **BLOCKER**：开发相关能力前必须解除的阻塞项。
- **FROZEN**：完成逐项评审后才可标记；本包尚无整体冻结声明。

**统一原则**：不将 UI 状态、模块自报、模型输出或传输 ACK 直接当作最终业务成功；所有跨模块副作用必须经过 Core 的授权和持久化流程。
