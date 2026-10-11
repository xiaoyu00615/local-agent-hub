# ADR-ARCH07-001｜批准 Local Agent Hub ARCH-07 V0.3 架构与实施路线

> **文件性质：经用户明确批准的决策内容之归档草案；尚未提交项目仓库。** 这里的“APPROVED”指用户对架构与路线的批准，不指 Git 合并、协议冻结、软件实现或测试通过。

| 元数据 | 记录 |
|---|---|
| Decision ID | `ADR-ARCH07-001`（建议正式编号，入库前检查是否冲突） |
| Decision type | 架构、产品范围、实施分期与技术路线 |
| Decision date | `2026-10-11` |
| Project | `Local Agent Hub` |
| Local project root | `D:\Workspace\projects\local-agent-hub` |
| User decision | **APPROVED — ARCH-07 V0.3** |
| ADR document state | **DRAFT FOR ARCHIVE / NOT COMMITTED** |
| Proposed repository path | `docs/decisions/ADR-ARCH07-001.md` |
| Existing security authority | `ADR-F1-001` + `F1-BASELINE-001`；不变更 F1 |
| F2 / implementation status | `F2 CANDIDATE`；`E00–E07 NOT RUN` |
| Implementation authorization | **NOT GRANTED** |

## 1. 用户明确批准的证据与范围

用户于 2026-10-11 明确表示：

> 正式批准 ARCH-07 V0.3 作为 Local Agent Hub 后续工程的架构与路线基线。仅授权准备 ADR，不通知 COD/UI，不修改代码或启动 E00。

批准对象为下列**已经准备并由用户本地 Git 提交的 ARCH-07 V0.3 文档组合**：

1. `docs/proposals/arch-07-v0.3/00_README.md` — 版本、来源与阅读说明；其中先前的 `USER_APPROVAL_PENDING` 为批准发生前的历史状态。
2. `docs/proposals/arch-07-v0.3/01_FINAL_ARCHITECTURE_DECISION.md` — D07-01～D07-18、产品范围、技术默认选型和完成定义。
3. `docs/proposals/arch-07-v0.3/02_STAGE_GATES_CHECKLIST.md` — Gate A～H、任务入口和退出条件。
4. `docs/proposals/arch-07-v0.3/03_CONFLICTS_ADRS_HANDOFF.md` — 与旧版 PRD/F1/F2、Webflow 和 Relay 的差异处理与交接限制。

**本地版本来源（据用户提供的终端输出）：**

- `docs/arch-06-review` 分支上的 ARCH-06 归档提交：`bc6bac0`（9 份文档）。
- 同一分支上的 ARCH-07 V0.3 归档提交：`3330958`（4 份文档，343 行新增）。
- 用户最新 `git status` 仍有未跟踪的 `docs/proposals/arch-07-v0.2/` 和 `docs/ui-design/`，不属于本 ADR 授权处理范围。
- **本 ADR 不声称**上述文档分支已推送 GitHub、合并 `main`、打 Git 标签或做过 E00 实测。

## 2. 正式架构决策（APPROVED）

### 2.1 产品与权威边界

1. 产品保持为全新独立的 **Windows 单用户本地模块化 AI 协作与编排平台**，通过响应式 Web Console 管理。
2. 采用 **模块化单体 Local Core + LMP Module Host + 独立专用 Adapter Worker** 的总体结构。
3. **Local Core 是受管任务、权限、Ownership、持久化事件、派发和恢复的唯一权威**。Brain、Web Console、Adapter 不能建立竞争性调度或绕过它批准操作。
4. 现有 **Relay 仅用于辅助开发 Local Agent Hub**，不作为产品运行时依赖；不复制其代码、任务队列、数据库或状态机作为产品基线。
5. 内置模块与经用户明确信任的本地开发者模块为首版安装范围；远程插件市场延期。

### 2.2 Brain 与模块调用

6. **Brain Gateway / Brain Connector** 统一处理网页、API 与本地 Brain 的会话绑定、结构化任务提案、结果交付；它不是第二个 Scheduler。
7. `TaskProposal` 是经过可信入口验证的**任务意图或候选**，不是 LMP 执行命令，更不是授权凭证；`USE:` 之类文本不能授予操作权限。
8. Core 通过 Capability Registry 和各 Adapter 声明的版本化能力、I/O Schema 与真实证据实现接入；每个 App 的网页/GUI/协议差异由其专用 Driver 解决。
9. 模型选择遵守授权模型池、真实可用能力、收件人权限和预算；未来 `FREE_FIRST` 优先选择可胜任任务的免费模型，不自动扩大付费和数据披露授权。
10. 主 Brain 不可用时，可在后续实测支持、已有有效权限与授权备用 API Brain 的条件下故障切换；原 Brain 恢复后**仅在下一个可验证的安全检查点**自动切回，且必须避免双 Brain 同时拥有有效规划控制权。

### 2.3 工作流、记忆与长期任务

11. Workflows 使用 Draft → **不可变 Workflow Revision** → Run 的关系。P04 是设计/编辑视图，P05 依据 Core Journal 投影真实运行证据；UI 呈现不能替代后端验收。
12. **首版先做步骤式/有限工作流与关系视图**；完整可拖动图画布、复杂子工作流、动态拓扑编排在 V1.x 分功能实施。提前保留端口、能力引用和版本兼容性设计。
13. Dynamic Plan / Iteration 属于受批准 Workflow Revision 约束下的后续能力；**不能静默重写运行中的 Revision** 或自行增加新权限。
14. 长期任务允许在获批准的自主策略下**不预设固定总迭代轮数**，但必须服从预算、调用额度、授权、外部效果未知、人工接管、停滞及资源冲突守卫。
15. 首版先实现 Event Journal、Outbox/Inbox、Evidence、Artifact 与安全恢复；跨 Brain 长期记忆、Snapshot/Delta、备份切回等在 V1.x 独立验证后开放。

### 2.4 证据和安全

16. 严格区分：传输、Adapter 接受、外部提交、外部运行、结果观察、Core 持久化、目标交付及独立业务验收。
17. `LOCAL_READ`、`EXTERNAL_DISCLOSURE`、`LOCAL_EXECUTE` 相互独立、默认拒绝且实时有效；网页文本、AI 回复、日志与工具输出均不得提升权限。
18. 人工接管后自动写入必须停止；对账通过也仍须用户本人在可信界面**明确批准恢复**。外部副作用是否发生未知时先对账，禁止盲目创建新业务 Attempt。
19. 独立 Worker、Manifest、同用户 Windows 进程不构成可证明的 OS 强沙箱；不得宣传外部 App 自动操作的端到端 exactly-once。

> 上述 19 点是批准文件 `01_FINAL_ARCHITECTURE_DECISION.md` 中 D07-01～D07-18 和相关约束的审查性归纳；若归纳与被批准的提案原文有细微差异，应以获批准的 V0.3 原文及上位 F1 为准，并在入库前修订本 ADR，不得默默改变语义。

## 3. 技术选型批准的精确含义

**获批准的是“默认评审与实施方向”，不是尚未测试的具体底层协议或依赖版本。**

| 子系统 | 采用的默认方向 | 尚未冻结的事项 |
|---|---|---|
| Core | Node.js LTS + TypeScript 模块化单体 | 具体 LTS 版本、进程组织、压力指标 |
| Storage | SQLite + 受控本地 Artifact Store | Schema、WAL/备份策略、迁移与事务边界 |
| Protocol | LMP、UTF-8 JSON、JSON Schema Draft 2020-12 | F2 具体 Envelope、ACK、SDK 语义，待 E00 |
| Host / IPC | 经过身份认证的本机通信与独立 Worker | Named Pipe / stdio / WebSocket 的最终选择 |
| Web Console | React + TypeScript + Vite | 组件库、具体 Webflow 迁移方案和打包细节 |
| Workflow Editor | 首版步骤式；后续 React 节点图组件候选 | React Flow 等具体库的采用、Graph Schema |
| Browser Brain | 官方工具接口优先，必要时逐站 Bridge | 目标网页是否允许和可靠支持自动收发，待 E02 |
| GUI Agent | 专用 Windows GUI Driver，优先研究 UIA | WorkBuddy 实际 G0–G3 能力，待 E03 |
| Executor / Test | 注册且经授权的 Profile | 是否及如何具备更强 OS 隔离，待 E05 |
| Memory | SQLite 索引、结构化摘要与证据引用 | V1.x 记忆清理、Context Builder 具体契约 |
| Launcher | 轻量 Windows 本地启动器 | 安装器、更新、回滚与具体技术 |

重大替代方案涉及安全、产品范围、数据兼容、迁移或成本边界时，应通过后续技术评审/ADR 明确处理，不得仅凭“已批准 Node/TS”就擅自选定全部实现细节。

## 4. 产品范围和分期（APPROVED）

### L0 — 可靠底座

- 保留已批准并 Git 冻结的 F1-01～F1-12。
- E00 31 项模拟一致性测试按原用例集合完成；F2 必须在证据满足后**单独批准和冻结**。

### L1 — 首个真实、受控的完整闭环

- E01：授权工作区 Reader 与安全文件查询。
- E02：真实网页 Brain 的可信来源、会话关联、候选请求及结果回传。
- E03：WorkBuddy GUI Driver 的真实会话身份、提交与结果证据。
- E05：经批准的 Test Profile 与独立业务验证。
- 目标链路：`Web Brain → Core → WorkBuddy → Verify → Core 持久化 → 原 Brain`。
- 独立 Reader 链路：`Web Brain → Core Reader → 独立 Disclosure 校验 → 原 Brain`。
- 能力等级若不足，只可标明受监督 Alpha/Preview，不可虚报“完整自动闭环”。

### L2 — 可交付的 V1 产品

- E04：第二种真实不同 Driver（优先 DSH ACP；无法实现时需新的范围 ADR）。
- E06：真实故障窗口、断线、人工接管与安全恢复。
- E07：Launcher、Web Console、模块管理、有限工作流、P02/P04/P05、审批和真实状态显示。
- 满足相应 Gate 后，由用户**单独批准正式 V1 发布**。

### L3 — 后续 V1.x 自主与多媒体扩展

- Model Catalog / FREE_FIRST、备用 API Brain、安全检查点切回、持久化记忆、动态计划与长期 Iteration。
- 完整图形画布、媒体生成 Adapter（包括 MJ）、可复用子工作流、并行/定时等按功能分批实现。
- 各扩展单独设置能力与安全门禁；不得因早期 Mock 通过而自动开放无人值守。

## 5. 与既有文件的权威关系及冲突处理

1. **F1 安全语义最高且保持不变**：`docs/baselines/F1_ARCHITECTURE_SECURITY_v1.0_APPROVED.md` + `docs/decisions/ADR-F1-001.md`。ARCH-07 不修改原 F1 正文；如发现不可兼容，停止相关设计并准备新的 F1 变更 ADR 供用户批准。
2. **ARCH-07 V0.3 已批准为新的产品架构/实施路线基线**；其 `docs/proposals/arch-07-v0.3/` 原文可保留为审查时的来源快照，本 ADR 建议作为“批准状态指针”。它不自动改写或删除 ARCH-06 的探索性设计。
3. **F2 仍为候选**：`docs/versions/v1.4/01`～`06` 中的具体 LMP/SDK/ACK 字段、版本和状态必须经 E00 及独立审批，不能由本 ADR 提前冻结。
4. **历史文档保留当时状态**：如旧 PRD 所说“未经批准的无限轮自主执行”不属于其当时目标；新批准策略是**在实际授权的预算、资源和副作用约束内，不预设固定迭代次数**。不要静默修改旧版。
5. **Webflow 设计线保留独立状态**：现有 `docs/ui-design/` 和 DS08/P08 的真实工作状态单独审查，不因新技术路线自动复位或替换。
6. **权限外传仍需逐次/按有效范围验证**：批准新架构本身不授予任何工作区读取、外部提供商数据发送、GUI 操作或 API 消费权限。

## 6. 实施门禁与停线条件

工程应遵守已批准的 `02_STAGE_GATES_CHECKLIST.md` 中 Gate A～H 的顺序、准入/退出与真实证据要求。基础流程：

`Gate A（架构/资料） → Gate B（环境/授权） → Gate C（E00/F2） → Gate D（E01/E02） → Gate E（E03/E05 与闭环） → Gate F（E04/E06） → Gate G（E07 与 V1 发布） → Gate H（V1.x）`

下列事件直接阻断对应 Gate：无授权访问或外传、错误目标会话写入、旧 Epoch 仍可写入、人工接管后未获恢复批准却继续、未知外部副作用的盲重试、将 ACK 误记为实际业务成功、将普通 Worker 伪称为强沙箱。

每个工程 Gate 必须有 caseId、实际运行环境、输入/输出和失败注入证据、相关 Commit/Artifact、权限记录、验收结果；证据不足不得标 `PASS`。

## 7. 尚未冻结、留给后续验证的事项

- 具体 Node.js、SDK、React 及 GUI Driver 库版本；性能和 8GB Windows 机器上资源目标。
- LMP 消息字段、Session Binding 判据、ACK/Delivery 确认语义和 SDK 生命周期完整状态机（F2/E00）。
- Browser Native Tool / Conversation Bridge 的真实接入方式、应用平台支持范围、会话轮次及交付证明（E02）。
- WorkBuddy GUI 是否能达到 G2/G3；DSH ACP 实测能力和模型配置（E03/E04）。
- IPC 配对、来源认证、Windows Credential 保护及真实进程隔离机制（相应 E00/E05）。
- Brain Planning Ownership Epoch 的字段、备用 API Provider 配额、上下文外传与切回检查点判据（V1.x Gate）。
- Typed Graph、动态 Plan、Iteration、Budget Ledger 的 Schema 和 UI 编辑细节（V1.x Gate）。
- 软件安装、升级、回滚以及完整端到端发布性能指标（E07）。

## 8. 明确**未授权**的行为

本次用户仅授权准备本 ADR，**未授权**以下任何行为：

- 通知、分派任务给 COD 或 UI/Webflow 对话及人员/代理。
- 修改 Local Agent Hub 的产品代码、配置、设计页面或 Webflow 站点。
- 运行 E00–E07 测试、真实 WorkBuddy/DSH/MJ 操作或任何本机执行任务。
- Git `add`/`commit`/`push`/`merge`/`tag`、创建 PR、改动原有 F1 或其他仓库文件。
- 安装插件、扩展、连接第三方账号、消耗收费 API、向外部提供商发送项目内容。
- 移动、删除或提交未跟踪的 `docs/ui-design/` 与 `docs/proposals/arch-07-v0.2/`。

文件生成不代表已经在用户的 Windows 项目根目录写入任何东西。

## 9. 建议的归档步骤（未来需另行批准/由用户操作）

1. 复核此 ADR 与已批准的四份 ARCH-07 V0.3 原文是否一致；核查 `ADR-ARCH07-001` 编号未被占用。
2. 由用户另行决定是否将此文件放入 `docs/decisions/ADR-ARCH07-001.md` 并在当前文档分支建立独立提交。
3. 若未来需要同步远端、合并 `main` 或创建新冻结标签，应由用户分别明确授权。不要将 Git 版本动作与用户已给予的架构批准混为一谈。
4. 不改写 `docs/proposals/arch-07-v0.3/` 已归档的历史审批前状态行；通过此 ADR 提供新的当前状态指针。

## 10. 决策结论

- **产品架构与路线：APPROVED**（用户明确批准 `ARCH-07 V0.3`）。
- **F1：FROZEN**（继续有效，不因 ARCH-07 改写）。
- **F2：CANDIDATE / NOT FROZEN**。
- **工程测试与实现：NOT RUN / NOT AUTHORIZED**。
- **ADR 文件 Git 状态：NOT COMMITTED；待用户决定归档方式**。

决策人为用户本人；本记录不声称具有独立数字签名，也不将架构批准解释为执行权限。
