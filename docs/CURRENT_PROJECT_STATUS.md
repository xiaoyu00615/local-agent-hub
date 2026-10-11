# Local Agent Hub｜项目现行状态索引

> **文件建议位置：** `docs/CURRENT_PROJECT_STATUS.md`  
> **状态快照日期：** 2026-10-11（以本轮对话及用户提供的本地 Git 终端输出为依据）  
> **文档性质：** 当前状态导航与决策索引（INFORMATIONAL），**不替代 ADR、正式基线或测试证据，不构成工程执行授权**。  
> **项目根目录：** `D:\Workspace\projects\local-agent-hub`  
> **仓库：** `xiaoyu00615/local-agent-hub`

## 1. 当前状态总览

| 项目 | 权威状态 | 证据与边界 |
|---|---|---|
| **F1 架构与安全基线** | **APPROVED / FROZEN** | `F1-BASELINE-001` 的 12 条安全与架构不变量已经批准；`f1-baseline-v1.0` 为 Git 冻结标记。未经新 ADR、影响审查及再次批准不得修改其含义。 |
| **ARCH-06 V0.1** | **DESIGN REFERENCE / PROPOSAL** | 作为 Brain、模型路由、画布、动态计划、长期运行与记忆的**扩展设计依据**；具体实现契约未冻结。 |
| **ARCH-07 V0.3** | **APPROVED ARCHITECTURE & ROADMAP** | 项目所有者于 2026-10-11 明确批准其为后续工程的架构与路线基线；**不是已通过工程测试的技术规范**，亦不意味着自动允许实施。 |
| **ADR-ARCH07-001** | **APPROVED DECISION / LOCALLY COMMITTED** | 用户已将批准记录在本地审查分支提交，提交号 `0351648`；**尚无该分支远程推送或合并的确认记录**。 |
| **F2（LMP、ACK、SDK 具体契约）** | **CANDIDATE / NOT FROZEN** | 须按现有门禁实施 E00，完成强制测试并单独批准后才能冻结；ARCH-07 批准不改变这一状态。 |
| **E00–E07** | **PLANNED / NOT RUN** | 当前可用的项目记录没有实施或通过的证据；E00-001 任务书已准备不等于授权编码或运行测试。 |
| **COD 工程线** | **NOT AUTHORIZED FOR HANDOFF OR EXECUTION** | 尚未获得将本轮新规范交给 COD 或启动编码、E00 的授权。 |
| **UI / Webflow 设计线** | **NOT AUTHORIZED FOR NEW HANDOFF OR CHANGES** | 已有 UI 设计工作与文件不等于获准进行本轮新架构的交接、修改或开发。 |
| **文档审查分支** | **LOCAL ONLY AS FAR AS CONFIRMED** | `docs/arch-06-review` 的三项本地文档提交已由用户终端输出确认；没有该分支推送、PR 或合并 `main` 的确认。 |

## 2. 当前权威来源与读取顺序

1. **F1 安全边界（现行已冻结）：** [`baselines/F1_ARCHITECTURE_SECURITY_v1.0_APPROVED.md`](baselines/F1_ARCHITECTURE_SECURITY_v1.0_APPROVED.md) 与 [`decisions/ADR-F1-001.md`](decisions/ADR-F1-001.md)。F1 的 12 条不变量仍为不可绕过的安全约束。
2. **ARCH-07 批准依据与开发路线：** [`decisions/ADR-ARCH07-001.md`](decisions/ADR-ARCH07-001.md)；配套候选设计见 [`proposals/arch-07-v0.3/`](proposals/arch-07-v0.3/)。用户批准优先于这些提案中历史遗留的“待批准”字样。
3. **ARCH-06 扩展设计参考：** [`proposals/arch-06-v0.1/`](proposals/arch-06-v0.1/)。不得把其中的未来功能、字段草案、COD/UI 交接草案直接视为已冻结接口或开工指令。
4. **F2 协议候选：** [`versions/v1.4/`](versions/v1.4/)，尤其是 `01_LMP_MESSAGE_CONTRACT.md`、`02_TASK_ACK_AND_RECOVERY.md`、`03_ADAPTER_SDK_AND_LIFECYCLE.md`、`04_PERMISSION_SESSION_CONTRACT.md` 与 `05_E00_CONFORMANCE_AND_TESTS.md`。
5. **工程门禁：** [`engineering/E00_READINESS_AND_GATES.md`](engineering/E00_READINESS_AND_GATES.md) 和 [`engineering/E00-001_IMPLEMENTATION_TASK.md`](engineering/E00-001_IMPLEMENTATION_TASK.md)。上述文件是任务/门禁说明，**不是实际执行授权**。

**历史文档提示：** `docs/00_BASELINE_AND_STATUS.md`、部分 `docs/versions/v1.1–v1.4/` 文件可能保留当时的 `F1 PENDING APPROVAL`、`ARCH-07 PENDING APPROVAL` 等历史状态。不要改写历史记录；判断现行状态应以相应批准 ADR 与本索引中的时间/证据边界为准。**若本索引与正式 ADR、F1 基线或新发生的可核实 Git/测试证据不一致，应以更高权威且更新的事实为准，并更新本索引。**

## 3. 已确认的项目架构与路线

- **产品边界：** Windows 本地、单用户、响应式 Web Console；独立于既有项目开发，代码库为 `local-agent-hub`。
- **正式产品不依赖 Relay：** 既有 Relay 仅可作为辅助开发 Local Agent Hub 的外部工具，不属于产品运行时架构，也不作为其任务、权限或恢复权威。
- **核心架构：** 模块化单体 Local Core + LMP Module Host + App 专用 Adapter；Core 是受管任务、权限、Ownership、Journal 和恢复决策的唯一权威。
- **模块分发：** 首版为可信内置模块 + 经用户明确选择信任的本地开发者模块；远程市场不列为首版门槛。
- **首条端到端链路：** 经验证的网页 Brain 任务提案 → Core 授权/调度 → WorkBuddy 执行 → 独立验证 → 持久化证据 → 按可验证交付能力回传原 Brain 会话。另保留网页 Brain 经授权直接调用 Workspace Reader 的独立只读链路。
- **工作流：** Workflow Revision 不可变；先实现可验证的有限工作流和任务监控，后续扩展自由画布与动态规划。
- **高级路线：** 免费优先的授权模型路由、备用 API Brain、安全检查点切回、持久化记忆、无固定总轮数的受控长期迭代，以及更多执行器和多媒体模块，按 ARCH-07 分阶段实施。
- **技术默认方向：** Node.js LTS / TypeScript、SQLite + Artifact Store、React / Vite、JSON Schema Draft 2020-12、独立 Worker；IPC、Windows GUI Driver、网页 Bridge、最终打包与性能参数仍需实测裁决。

**特别注意：** 没有经验证的会话绑定或交付证据，不得宣称网页 GPT 已收到结果或可无人值守；人工接管后即使对账成功，也必须由用户明确批准才可恢复自动写入；外部副作用结果未知，不得盲目重试。

## 4. 工程验收与实施状态

| 阶段 | 范围 | 目前状态 | 准入/退出原则 |
|---|---|---|---|
| **L0 / E00 → F2** | LMP 身份、Handshake、ACK、幂等、授权、生命周期、恢复模拟 | **NOT RUN** | E00 现有 **31 项**强制测试按证据通过并处理未决语义；F2 另行批准，不得提前冻结。 |
| **L1 / E01** | 受限 Workspace Reader、越界/敏感文件防护 | **NOT RUN** | 验证授权边界、来源、撤销后阻断与外传独立审批。 |
| **L1 / E02** | 网页 Brain 识别原会话、请求收发与安全回传 | **NOT RUN** | 无法可靠定位会话/轮次则降级受监督模式，不能宣传自动桥接。 |
| **L1 / E03** | WorkBuddy 专用 GUI Adapter | **NOT RUN** | G0–G3 实际能力按证据分级；不可直接认定具备无人值守。 |
| **L1 / E05** | 受控 Terminal / Test Profile、独立业务核验 | **NOT RUN** | 真实副作用须单独授权；执行成功、结果保存和业务验收分别记录。 |
| **L2 / E04** | DSH 或另一种真实 Adapter 的能力契约映射 | **NOT RUN** | 无 Core 应用专属硬编码；以实测能力为准。 |
| **L2 / E06** | 崩溃/断线恢复、结果对账、人工接管 | **NOT RUN** | 不重复未知副作用；人工接管后不能自动恢复。 |
| **L2 / E07** | Web Console、模块安装、任务中心、审批与恢复 | **NOT RUN** | UI 状态来自 Core 真实数据；运行/交付/验证分离。 |
| **L3 / V1.x** | Brain Router、长期记忆、动态计划、循环与高级画布等 | **DESIGN ONLY** | 按能力门禁分阶段开发；不可将未来设计当作当前运行能力。 |

上述顺序是工程规划，实际并行或调整需尊重依赖与风险门禁，**不能跳过应有验证**。

## 5. 本地 Git 事实与未处理文件

以下来自用户于 2026-10-11 提供的本机终端输出，**非本轮实时读取用户工作区所得**：

| 记录 | 本地事实 |
|---|---|
| 当前工作分支（最后提供） | `docs/arch-06-review` |
| ARCH-06 归档 | Commit `bc6bac0`：9 份 `docs/proposals/arch-06-v0.1/` 文档 |
| ARCH-07 V0.3 归档 | Commit `3330958`：4 份 `docs/proposals/arch-07-v0.3/` 文档 |
| ARCH-07 ADR 归档 | Commit `0351648`：`docs/decisions/ADR-ARCH07-001.md` |
| 最近一次 `git status` | 无已跟踪文件的未提交修改；仍有未跟踪 `docs/proposals/arch-07-v0.2/`、`docs/ui-design/` |
| 远程同步/合并 | **未见推送审查分支、发起 PR、合并 `main` 或创建 ARCH-07 新标签的确认** |

**处理限制：** 不删除或误提交上述未跟踪目录；不要使用无范围的 `git add .`；不得把本地提交写成已经推送/合并。F1 的既有冻结状态与新文档分支的本地归档状态是两件不同的事。

## 6. 待办与授权门槛（当前全部尚未获本轮执行授权）

- [x] 用户正式批准 ARCH-07 V0.3 架构与路线。
- [x] 用户将 ARCH-06、ARCH-07 V0.3 和批准 ADR 分别提交至本地文档审查分支。
- [ ] 将**本索引**放入 `docs/CURRENT_PROJECT_STATUS.md`，审阅并单独提交（仅文档；由用户执行或另行明确授权）。
- [ ] 复核现行索引、ARCH-06、ARCH-07、ADR 与 F1 之间的引用和状态一致性。
- [ ] 经用户单独同意后，决定是否推送文档分支、提出 PR、合并 `main` 或创建新文档版本标签。
- [ ] 准备 COD/UI **待交接**任务书；发送、修改 UI/代码与执行任务均须另行授权。
- [ ] 获得用户明确 E00-001 工程执行授权后，按原有 E00 门禁开始 Mock 工程；没有授权不得启动。
- [ ] E00 全项通过且取得单独 F2 批准后，才可宣称 F2 已冻结。
- [ ] 按 E01–E07 阶段门禁逐项实测、归档证据，再宣布相应能力可用。

## 7. 更新规则

1. 本文件是**状态快照与导航入口**，不是重新审批 F1、ARCH-07、F2 或真实操作的渠道。
2. 只能根据用户明确批准、实际 Git 结果、真实测试及可复核的证据更新状态；不得因为“文档已写好”就把测试改为 PASS。
3. 更新状态时保留日期、关联 ADR/Commit/Case ID，并区分：`APPROVED`、`FROZEN`、`CANDIDATE`、`NOT RUN`、`LOCALLY COMMITTED`、`PUSHED`、`MERGED`。
4. 对历史文档中的过期状态作索引说明即可，不覆盖旧版本的原始文本。
5. 本文件的创建或入库本身**不授予** COD/UI 交接、E00 执行、仓库合并或应用控制权限。
