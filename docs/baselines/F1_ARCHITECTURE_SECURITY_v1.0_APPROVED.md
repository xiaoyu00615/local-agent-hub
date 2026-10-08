# Local Agent Hub — F1 架构与安全基线 v1.0

- **规范 ID：** `F1-BASELINE-001`
- **决策 ID：** `ADR-F1-001`
- **用户审批状态：** `APPROVED`（用户在 2026-10-08 的当前对话中明确批准全部 12 条修订条款）
- **Git 冻结状态：** 以本版本是否合并至 `main`、通过一致性检查并创建 `f1-baseline-v1.0` 标签为准；本文件生成本身不代表已经 Git 冻结
- **原始仓库基线：** `main@2d94d23`
- **适用项目：** `D:\Workspace\projects\local-agent-hub`
- **来源文件 SHA-256：** `68f1d36157791569bb6bfdf94a2f192d3f24c21fe54b3f7fb434ccfdbee283b0`（`F1_BASELINE_CANDIDATE_001.md` 的原始内容）
- **权限边界：** 仅架构与安全不变量 F1；**不批准 F2、T1 自动执行、真实应用写入或跳过 E00**

## 1. 生效说明

以下 12 条与用户明确批准的 `F1-BASELINE-001` 候选稿条款正文逐字一致（本文件仅调整标题及审批记录，不修改条款本身）。对历史 `docs/versions/v1.1`～`v1.4` 的 F1 摘要存在歧义时，本文件在 **F1 架构原则范围内** 优先；历史文件均保留以便追溯。

本文件批准的原则并非任何具体实现已经成功的证明；具体消息格式、状态转换、Runtime ID 签发、ACK 的持久化含义，以及实际浏览器/GUI/终端接入，均须对应 E00–E07 验证后逐项处理。

## 2. F1 层术语

| 对象 | 不变含义 | 不得混为 |
|---|---|---|
| Module Package / Installation | 已安装的分发物及本机安装记录 | 运行中的 Worker |
| Module Runtime | 一次受管 Worker 运行实例 | Package、连接实例 |
| LMP Endpoint | 经鉴权并绑定传输通道的端点 | 外部应用会话 |
| Connection Instance | Adapter 连接的真实应用或服务实例 | 单个 Session Binding |
| Session Binding | Core 管理的逻辑会话及其外部身份观察证据 | 仅靠 PID/HWND/Tab ID 判定的会话 |
| Workflow Revision | 不可变工作流定义 | 正在编辑的草稿 |
| Run / Node Execution / Task | 一次运行、某节点执行和逻辑操作，各自可关联 | 任意消息传输尝试 |
| Attempt | 某逻辑操作的一次可能发生副作用的业务尝试 | Delivery Attempt |
| Message / Delivery Attempt | 协议消息与某次传输/交付尝试 | 一次新的业务执行 |
| Grant | Core 掌握并可撤销的许可记录 | 消息中的 Grant 引用 |
| Evidence / Artifact | 经过来源、版本和访问策略标识的观察事实或产物 | 外部应用自述的必然成功 |

**身份分配原则**：Core 负责其权威登记与有效性校验；Host 提供真实进程/连接的观察证据。`moduleRuntimeId` 由谁生成、握手如何签发 `endpointId`、ID 的确切格式属于 F2 实验，不因本条而预先冻结。

## 3. 已获批准的 12 条架构与安全不变量

### F1-01｜Core 权威与外部真实世界分离

**MUST**：Core 是 Hub 受管任务、授权、Ownership、持久化事件、审计与恢复决策的权威来源。Adapter 不得自行改写 Core 的权威任务状态。

**边界**：Core 只能根据证据推断外部应用状态；不能因自己记录为已派发或已接收，就声称文件已经修改、Agent 已完成或外部动作已停止。

### F1-02｜身份分层及引用关系

**MUST**：Package、Runtime、Endpoint、Connection、Session Binding、Workflow Revision、Run、Node Execution、Task、Attempt、Message、Delivery Attempt 和 Grant 保有独立可追溯身份。任何 PID、HWND、Tab ID、窗口标题或网页 URL 均不得单独作为完整会话身份。

**F2 待验证**：ID 生成者、格式、会话身份判据、进程重启后的身份继承与协议字段名。

### F1-03｜能力调用统一管理及权限保证边界

**MUST**：Hub 调度的能力必须经过 Core 能力登记、参数验证、当时有效授权、资源控制和完成证据要求。模块不得经 Hub 内部受管工具入口绕开授权。

**明确不保证**：在与用户共享 Windows 操作系统权限的外部 Agent/开发者 Worker 上，Core 不能仅凭协议阻止其绕开 Hub 直接操作操作系统。不得宣传“所有外部写入均被 Core 物理阻止”。

### F1-04｜读取、外传与执行权限三轴分离

**MUST**：`LOCAL_READ`、`EXTERNAL_DISCLOSURE` 和 `LOCAL_EXECUTE` 互不隐含，默认拒绝。Grant 至少约束调用身份、资源、能力、有效期及版本；Disclosure 还约束接收者与目的会话或明确范围。

**MUST**：实际执行前以及结果向下游发送前重新检验相应许可；Grant ID 字符串不是许可证明。若用户撤销权限，不能借启动 Run 时的旧授权继续新操作。

### F1-05｜不可信内容不得提升权限

**MUST**：网页正文、文件内容、AI 回复、日志或工具输出均视为数据。它们可能提出候选请求，但不能提供授权、签发 Lease 或改变工作流安全规则。Schema 校验与用户授权独立执行。

### F1-06｜Session Ownership + 跨会话资源互斥

**MUST**：同一会话至多有一个有效自动写入所有者；写操作须验证 Binding Generation、Ownership Epoch、Lease 及相关 Grant。

**MUST**：此外考虑共享 GUI 焦点、文件、外部服务资源等冲突，所需资源锁由 Core 协调。Lease 过期只能禁止新的受控动作，不能撤销已提交的外部操作；直接的用户操作和 Hub 外程序不在锁的物理约束内。

### F1-07｜人工接管与明确恢复

**MUST**：人工接管或严重上下文冲突阻止新的自动写入；保留在途操作的实际状态并进行对账。曾发生人工接管时，即使对账成功，仍由用户本人在受信任 UI 明确确认，才可签发新的写入许可。

**MUST NOT**：因页面刷新、Core 重启、模块重连、Lease 续期或“对账看起来没问题”而自动清除恢复阻断。对检测到的异常变化不得无证据认定必然是用户操作。

### F1-08｜执行、存储、交付、验收严格分离

**MUST**：传输收帧、Adapter 接受、外部提交、运行、结果观察、Core 持久化、下游交付及独立业务验收分别保留证据。单一 ACK 不得跨阶段证明其他事实。

**MUST**：Core 未完成结果事务持久化前不得发出“结果已安全保存”的确认；对网页桥接模式，能证明“点击发送”不等于能证明 AI 端真正收讫或理解。

**F2 待验证**：`DELIVERED` 与 `ACKED` 的精准判据，`ack.command.accepted` 是否包含 Adapter 侧持久化，以及状态转换表。

### F1-09｜不确定外部效果禁止盲目重试

**MUST**：外部动作可能已提交但结果不确定时，保持 `OUTCOME_UNKNOWN` 或等价需对账状态；先观察或人工核对，不自动执行新的有副作用业务 Attempt。

**MUST**：传输重试、已持久化结果补交付、真正新业务 Attempt 是不同操作。Adapter 不支持幂等时，Core 不得宣称端到端 exactly-once。

### F1-10｜Workflow Revision 不可变且授权实时有效

**MUST**：启动后的 Run 固定 Workflow Revision 与能力契约；更改定义、替换模块或升级不得静默改变运行中的行为。

**MUST**：版本固定不代表权限永远有效；每个受保护节点执行时仍须按当前 Grant、Lease 和资源锁授权。旧模块不兼容时阻断并显式处理。

### F1-11｜本机 Web、浏览器扩展和 IPC 不默认可信

**MUST**：监听 loopback 不等于身份认证。Web Console、浏览器扩展、Core API、WebSocket 和本机 IPC 采用各自适配的身份绑定、来源校验和最低权限；网页脚本不能直接取得 Core 的高权限凭据。

**F2 待验证**：具体配对方案、CSRF/Origin 防御、重连事件续传、IPC 技术和凭据保护方式。

### F1-12｜本地开发者模块信任与进程隔离真实边界

**MUST**：MVP 仅允许平台内置或用户明确选择信任的本地开发者模块进入运行状态；导入前审查 Manifest、来源标识和权限申请，新增权限须重新批准。模块不应自动运行任意安装脚本。

**MUST NOT**：把 Worker 独立进程、包摘要、Manifest 声明和 SDK 审计描述为 Windows OS 级强沙箱。无明确隔离机制和对应实测时，按同用户可信程序处理。

## 4. F1 不含的决策

- F2 LMP V1 Message Envelope、字段名、兼容规则及 ACK 语义没有通过 E00；仍为 `F2 CANDIDATE`。
- Adapter SDK V1 与具体 Worker 生命周期状态机还需要 E00 检验。
- 网页 AI 的自动工具请求与原会话回传（E02），WorkBuddy GUI 的实际能力等级（E03），DSH 实际 ACP 能力（E04），Terminal/Test Profile 的副作用和隔离（E05），都尚未验证。
- 不因批准 F1 而赋予任何工作区读取、对外内容披露、命令执行或第三方模块运行权限。
- 经用户再次批准并完成工程门禁后，才可开展真实应用操作；E00 模拟测试亦必须在独立工程任务授权之后启动。

## 5. Git 留痕与修改规则

- 原始提交 `2d94d23` 不修改、不 amend。
- 本版本与决议文件 `docs/decisions/ADR-F1-001.md` 同时提交到新文档分支，经检查后合并 `main`；用 `f1-baseline-v1.0` 标签固定最终 Git 提交。
- 修改 F1 条款必须创建新 ADR 和新版本；不得直接改写已批准正文而不留记录。
- 经 E00 测试后冻结 F2 应采用单独的批准记录与版本标签，不与 F1 混用。
