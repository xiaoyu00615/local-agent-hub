# Local Agent Hub — F1 架构与安全基线最终修订候选稿

- 决策编号：`F1-BASELINE-001`
- 审查基准：`main@2d94d23`（首次提交；已由用户推送至 `origin/main`）
- 文档状态：**REVIEWED / PENDING USER APPROVAL**
- 规范范围：架构、安全边界、状态归属与恢复不变量；**不包含实现协议字段的最终冻结**
- 适用项目：`D:\Workspace\projects\local-agent-hub`（本机路径由用户确认）
- 日期：2026-10-08

> **批准约束**：只有用户明确批准本修订稿后，F1 才能记录为 APPROVED；更新仓库并检查文档一致性后才记为 FROZEN。E00 尚未运行，不得将 LMP V1、Adapter SDK V1 或真实应用能力标记为 F2 FROZEN。

## 1. 规范来源与优先级

来源为首次提交的下列文件：

- `docs/versions/v1.3/02_PROTOCOL_FREEZE_CHECKLIST.md`：F1-01 至 F1-12 原始候选条款。
- `docs/versions/v1.4/00_FREEZE_AND_DECISIONS.md`：十条摘要及用户已确认方向。
- `docs/versions/v1.4/01_LMP_MESSAGE_CONTRACT.md`：LMP V1 消息候选。
- `docs/versions/v1.4/02_TASK_ACK_AND_RECOVERY.md`：ACK、三轴状态与异常恢复候选。
- `docs/versions/v1.4/03_ADAPTER_SDK_AND_LIFECYCLE.md`：SDK、Manifest 与生命周期候选。
- `docs/versions/v1.4/04_PERMISSION_SESSION_CONTRACT.md`：授权与会话控制候选。
- `docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md`：模拟验收测试清单。
- `docs/versions/v1.4/06_COMPATIBILITY_AND_OPEN_ISSUES.md`：兼容及待验证范围。

解释优先级：用户明确确认的产品决策不可被新草案默默覆盖；当本 F1 文档正式获批后，**仅就架构与安全原则**优先于以前的 F1 摘要；V1.4 F2 仍是候选，须以 E00 的实证及后续独立批准为准。

## 2. 统一术语（F1 层）

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

## 3. 十二项 F1 强制不变量

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

## 4. 必须划清的 F1 / F2 边界

| 可以在批准后冻结为 F1 | 仍留给 F2 与实测 |
|---|---|
| 谁拥有权威状态；外部证据不是权威数据库自身 | 数据库表、IPC 选型和 Outbox 事务实现 |
| 对象独立、会话不得靠单个 PID/HWND 判断 | ID 格式、签发者、跨重启迁移方式 |
| ACK 不等于业务完成；未知结果不重做 | 枚举完整状态机、ACK 字段及幂等窗口 |
| 人工接管需要用户恢复确认 | GUI 检测规则、确认 UI 的具体挑战/凭据机制 |
| 读取、外传、执行独立授权 | Grant Schema、文件路径竞态防御实现 |
| 用户信任模块 ≠ 强沙箱 | Windows 执行隔离实施方式与 T1/T2 验证 |
| Run 固定版本并持续检查权限 | Workflow/Capability 具体版本数据结构 |

## 5. 需在 E00 解决的协议疑点

1. `moduleRuntimeId`：V1.4 `00_FREEZE_AND_DECISIONS.md` 与 `01_LMP_MESSAGE_CONTRACT.md` 对产生者描述不同。F1 仅冻结 Core 登记权威与 Host 运行证据；具体签发模型用 E00-ID、E00-HS 确定。
2. 旧文档 `sourceInstanceId` vs V1.4 `sourceEndpointId`：统一采用实际已认证通信端点概念，最终字段通过 E00-MSG 冻结。
3. `ack.command.accepted`：必须明确受理/持久化语义，通过 E00-ACK、E00-DB 测试。
4. Execution / Delivery / Verification：三个维度继续分离，准确转换与冲突处理通过 E00-ACK、E00-REC 确定。
5. `Run`、`Node Execution`、`Task`：确定一对多关系和对象生存期，进入 E00-ID、E00-REC。
6. 重试与去重：`messageId` 与 `idempotencyKey` 是不同领域，进入 E00-DUP 验证。

## 6. 网页桥接与终端执行的产品验收范围

- 网页 AI 能直接请求授权本地文件并把结果返回原会话是用户的首版目标。手工复制/手动发送仅是 E02 试验回退，不得被记录为已满足原始自动闭环验收。
- 测试方案 T1 可以被配置为受信任执行、审批与审计能力，不能默认称为沙箱。没有真实的 OS/虚拟化隔离及拒绝证据，不得标记 T2。
- WorkBuddy / DSH 的实际能力等级由各自实机证据确定，不能通过 Adapter 自报解锁 G2/G3。

## 7. 版本治理与批准路径

1. 本文件作为候选稿提交审查（当前）。
2. 用户明确批准 F1-01～F1-12 的修订措辞；如需要更改其中之一，形成 ADR 后重新评审。
3. 在独立功能/文档分支将本稿与批准记录加入仓库；进行一致性检查，避免旧摘要被误解为最新权威。
4. 审查通过后方可把该条款状态记为 `FROZEN`。历史 V1.1–V1.4 保留而不篡改。
5. E00 在独立分支进行；E00 用例通过后单独办理 F2 冻结，不把模拟通过外推为真实工作区、网页、GUI 或终端可用。

## 8. F1 验收与签署信息

| 项目 | 当前值 |
|---|---|
| 用户确认的产品路线 | CONFIRMED |
| 十二维 F1 修订条款 | REVIEWED / PENDING USER APPROVAL |
| 跨协议字段 | F2 CANDIDATE |
| E00 Mock 测试 | NOT RUN |
| 真实网页/GUI/终端验证 | NOT RUN |
| 对主分支的修改 | NONE |

通过判据：用户明确批准该版本；所有已知 F1 文字矛盾都有一致解释或正式例外；批准决定可追溯到基线 `2d94d23` 和后续文档提交 SHA。不能凭一句“工作正常”擅自更改批准状态。
