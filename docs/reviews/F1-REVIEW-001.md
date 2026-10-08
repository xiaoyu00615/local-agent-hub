# Local Agent Hub — F1 架构与安全基线逐项审查

- 审查编号：`F1-REVIEW-001`
- 日期：2026-10-08
- 用户提供的仓库提交：`2d94d23`（`main`，首次规范提交，工作区 clean）
- 审查材料：已整理的 `docs/versions/v1.1`～`v1.4` 文档包与本轮用户提供的 Git 输出；未直接读取用户 Windows 本机 Git 对象，未开展 E00–E07 实验。
- 状态：**REVIEWED / CHANGES REQUIRED / NOT APPROVED / NOT FROZEN**
- 范围：文档和架构审查；没有编写或执行产品代码，没有连接真实应用。

## 1. 总体结论

架构方向可以继续。建议不要直接批准原始 F1 全文；先修订两项关键安全描述（F1-03、F1-12），再补齐身份、状态、权限与写入恢复边界，形成 `F1 Architecture & Security Baseline v1.0` 待用户批准版本。

从既有 V1.3 的 12 项 F1 编号审查：**3 项原则可原样保留、7 项附条件保留、2 项必须修订**。这些是文本成熟度分类，不是软件测试结果。

### 原始 F1 条款版本差异

V1.3 `02_PROTOCOL_FREEZE_CHECKLIST.md` §B 给出 F1-01～F1-12；V1.4 `00_FREEZE_AND_DECISIONS.md` §2 将内容压缩为十条。两者不能在未经映射和审签时默认为同一版本。建议正式版本保留十二条编号，并把 V1.4 十条逐一映射过去；V1.4 没单列的 Web/IPC 安全和开发者信任边界不得遗漏。

## 2. 十二条逐项结论

| 条款 | 原始主题（以 V1.3 为准） | 结论 | 修订或补充条件 | 验证 Gate |
|---|---|---|---|---|
| F1-01 | Core 权威状态 | 附条件保留 | 内部权威记录≠外部真实状态；Module Host 不得直接改 Core 任务/Grant | E00/E06 |
| F1-02 | 独立身份 | 附条件保留 | 统一 module/package/runtime/endpoint/connection/binding/workflow/node/task/attempt/transport ID 词典及签发者 | E00-ID/HS |
| F1-03 | 能力、权限、证据及锁 | **必须修订** | “全部外部写入都经 Core”仅对 **Hub 调度的受控能力**可保证；不能约束共享 Windows 用户权限的外部进程 | E00/E05 |
| F1-04 | Read / Disclosure / Execute 三轴 | 附条件保留 | Grant 绑定 principal、workspace、resource、capability、recipient、有效期及 revocation version；执行和交付重新鉴权 | E00-PERM/E01/E02 |
| F1-05 | 不可信输入不可授予权限 | **保留** | Web 页面、Agent 消息、仓库文件与工具输出均为数据，不产生授权 | E00/E02 |
| F1-06 | 会话所有权和 fencing | 附条件保留 | Session 独占之外必须处理 GUI 焦点锁、共享文件锁及不同连接对同一资源的冲突；Lease 不能撤回外部已提交动作 | E00-OWN/E03/E06 |
| F1-07 | 人工接管和恢复 | **保留** | 检测到疑似冲突先暂停，但不得把推断称为确认人工接管；曾确认接管后的解除必须用户显式批准 | E00-OWN/E06/E07 |
| F1-08 | ACK/执行/交付/验收分层 | 附条件保留 | `ack.command.accepted` 的是否持久化语义需确定；`DELIVERED` 与 `ACKED` 分离；`FAILED` 伴随未知副作用仍禁止重做 | E00-ACK/E06 |
| F1-09 | 未知副作用不盲目重试 | **保留** | 区分运输重试、结果补交付、业务新 Attempt | E00-DUP/REC/E06 |
| F1-10 | Run Revision 固定 | 附条件保留 | 定义及能力契约冻结，不能冻结已撤销的 Grant；运行中每次执行和交付重验当前权限 | E00-SDK/E06 |
| F1-11 | localhost Web/扩展/IPC 安全 | 附条件保留 | 信任主体、浏览器配对、来源校验、CSRF、WebSocket 和进程端点认证分别建模；不可仅凭 loopback 赋权 | E00-HS/E02/E07 |
| F1-12 | 开发者模块信任边界 | **必须修订** | Manifest 和独立 Worker 只能描述声明/故障隔离；同用户 Worker 不受文件与网络强沙箱约束。首版是显式受信任模块，不可宣称能安全运行任意不可信插件 | E00-LIFE/E05/E07 |

## 3. 跨文档实际不一致与缺口

| 冲突/缺口 | 影响 | 推荐裁决 | 等级 |
|---|---|---|---|
| C-01：V1.3 F1 共 12 项、V1.4 汇总 10 项 | 审批追溯可能漏项 | 十二条作为唯一审签索引；建立十条映射表 | P0 文档 |
| C-02：`packageId`/`moduleId`、`runtimeInstanceId`/`moduleRuntimeId`、`revisionId`/`workflowRevisionId` 命名不一 | DB、Schema、SDK 不兼容 | V1.4 优先；列出废弃别名及迁移映射 | P1/F2 |
| C-03：V1.1/V1.2 `sourceInstanceId` 与 V1.4 `sourceEndpointId` | 无法区分发送 Worker 和真实应用连接 | 用 endpoint 作认证信道，用 connectionInstanceId 定位目标应用；不混用 | P0/F2 |
| C-04：V1.4 `00_FREEZE...` 中“Core 生成 Runtime ID”与 `01_LMP...` 中 Module Host 为 Runtime ID 权威生成者 | 进程身份签发者矛盾 | 明确 Core 登记权威与 Host 创建运行实体现象的关系，E00 固定唯一语义 | P0/F2 |
| C-05：V1.1 有 `nodeExecutionId`，V1.4 Task/Attempt 中未明确 Node Execution 映射 | 工作流节点和业务任务可能错位 | 明确 `NodeExecution` 是独立对象还是与 `Task` 一对一；首版作显式绑定，不假设恒等 | P1/F2 |
| C-06：V1.1 `DELIVERED` 已按确认契约，V1.4 `DELIVERED` 可能仅代表提交、`ACKED` 才收到确认 | UI 会过早宣布已交付 | 以 V1.4 三轴为准，定义每种通道 `DELIVERED`/`ACKED` 证据等级，支持 `DELIVERY_UNCONFIRMED` | P0/F2 |
| C-07：`ack.command.accepted` 未明确是否持久化的接受事实 | Host 崩溃后遗失已受理任务；Core 错误认为有人负责 | 定义“受理”是否为持久化承诺；若不是，必须显式声明 `VOLATILE_ACCEPTED` 且恢复核对 | P0/F2 |
| C-08：`FAILED`、`CANCELLED`、`OUTCOME_UNKNOWN` 与 sideEffectStatus 的组合 | 失败/取消可能被误当成未产生外部影响 | 明确“确定失败”与“副作用未知”；安全调度必须同时检查后者，不能只看终态 | P0/F2 |
| C-09：所有外部写入经 Core vs 本地 Worker 可直接访问 OS | 安全承诺超出系统实际控制范围 | 将 F1-03 限定为 Hub mediated action；不可信代码需独立强隔离专项 | **P0/F1** |
| C-10：一个 Session 只一位 Writer vs 跨 Session 共享 GUI/文件 | 目标资源仍可能出现并发写冲突 | 锁对象以真实资源身份规范化，Session/Focal GUI/文件锁协同，人工优先 | P0/F1 |
| C-11：Run 固定授权策略版本 vs 执行时 Grant 可能撤销 | 旧授权被错误复用 | 冻结规则版本，不冻结 Grant 的有效性；每次执行和传输重新鉴权 | P0/F1 |
| C-12：`ownership.takeover.request` 由用户主动请求或 Adapter“疑似变化”共用 | 误将环境变化标记为用户明确接管 | 分离 `user.takeover` 控制命令与 `session.external_change` 观察事件；疑似冲突应触发安全暂停 | P1/F2 |
| C-13：Web AI “自主读取并自动回传”是首版 P0，但 E02 允许降级成 Hub 手工复制/投递 | 可能把不满足承诺的半自动演示算作首版交付 | 允许测试阶段降级，不自动满足正式 P0 需求；若改变首版承诺须用户批准范围调整 ADR | **P0/产品** |
| C-14：SDK Manifest `packageDigest` 能验证完整性，不能证明代码来源和 OS 隔离 | 安装安全感虚高 | UI 分开“文件未变化”“发布者身份已核实”“已受用户信任”“具备 OS 隔离” | P1/F1 |
| C-15：工作区根目录校验 vs Junction、符号链接与竞态 | 文件读取可能越界 | F1 固定“不得越界”，E01 验证最终文件对象和句柄；不能证明则拒绝 | P0/F2 |
| C-16：V1.1 `RESULT_UNCERTAIN`、`OUTPUT_SCHEMA_INVALID` vs V1.4 `OUTCOME_UNKNOWN`、`SCHEMA_INVALID` | 错误类型及兼容实现不一致 | 用 V1.4 优先并建立废弃映射，错误码语义 E00 冻结 | P1/F2 |
| C-17：首版终端 P0 与 T1 未强隔离且需另行人工批准 | “受控”易被误解为“沙箱” | 将 T0 查询、T1 信任执行、T2 强隔离声明区分；不能自动无限制执行 Shell | P0/产品 |
| C-18：V1.4 E00 文字此前称 30 项，现表中有 31 行 caseId | 验收报表数量错误 | 以 caseId 集合为准；更新文档标题/统计，禁止通过率基于错误分母 | P2 文档 |

## 4. 建议 F1 的核心规范性措辞（审批前草案）

**AUTHORITY**：Core 是 Hub 内任务、Grant、Ownership、审计和恢复决策的唯一权威记录者；外部副作用以真实证据为准，证据不足记为 UNKNOWN。

**CONTROL SCOPE**：所有 **由 Hub 调度的受控能力** 均须经 Core 许可与执行前复核。Core 不宣称能限制同 Windows 用户权限的、独立运行的外部程序在 Hub 之外所做的操作。

**IDENTITY**：Package、Installation、Runtime、Endpoint、Connection、Session Binding、Workflow Revision、Run、Node Execution、Task、Attempt、Delivery Attempt、Message、Grant 与 Artifact 分层建模；互不以同一 ID 冒充。

**PERMISSIONS**：LOCAL_READ、EXTERNAL_DISCLOSURE、LOCAL_EXECUTE 独立、最小作用域、默认拒绝；读取/执行/对外传递前分别验证当前 Grant、目标、资源和授权版本。

**UNTRUSTED INPUT**：网页、仓库、Agent 回复及工具输出均为不可信数据，不赋予本机或外部写入权限。

**SESSION CONTROL**：同一 Session 同时最多一个受控自动写入 Owner；跨 Session 的共享资源通过规范化锁控制。Epoch、Generation、Lease 失效阻止新的受控写入，不保证已对外提交的动作已停止。

**HUMAN TAKEOVER**：用户接管优先；疑似冲突至少阻止受影响资源的新自动写入并核对；确认人工接管后，只有用户显式批准且已对账，才能恢复新自动写入。

**FACTS AND ACK**：Transport、Adapter accepted、External submitted、Execution completed、Core result stored、Delivery、Receiver acknowledgment 与 Verification 是不同事实，不能互为代称。

**RETRY SAFETY**：外部副作用存在发生可能但无法确认时禁止盲目重放；运输重试、结果补交付、新 Attempt 独立管理。

**IMMUTABLE REVISION**：Run 绑定不可变工作流定义和能力契约，但不能绕过运行时最新的 Grant 撤销、Lease 失效及资源占用。

**LOCAL SURFACE SECURITY**：localhost 监听不替代认证；控制台、扩展、IPC 和 Worker 端点必须在各自信任边界鉴别调用主体、来源与授权。

**TRUSTED MODULES**：内置及用户明确信任的本地开发者模块可按 B 方案接入；独立 Worker 和 Manifest 不意味着操作系统强沙箱。未知来源模块不可按“安全隔离可执行插件”宣称或自动启用。

## 5. 未决事项与责任界限

### 需要用户批准（不由审查者代签）

- U-01：是否采用以上十二条作为唯一 F1 审批索引（建议：是）。
- U-02：是否接受 F1-03 与 F1-12 修订后的真实信任边界（建议：是，不能承诺强隔离）。
- U-03：首版 E02 若只能人工/半自动回传，是否仍列为未完成核心目标（建议：是，不静默降低用户已确认需求）。
- U-04：将来 T1 用户显式批准的可信 Test Profile 是否允许在无强沙箱下使用（需要实施前单独批准；不影响 E00）。
- U-05：是否授权启动 E00 实际工程编码和模拟测试（本轮没有该授权）。

### 必须由 E00/E01/E02/E03/E05/E06 实验决定（不能靠文档声称通过）

- T-01：Message Envelope 最终字段、Identity 签发与版本兼容性。
- T-02：Adapter accepted ACK 的持久化保证、Inbox/Outbox 失败窗口。
- T-03：State 转换、Idempotency 和重复消息的真实恢复行为。
- T-04：Grant 重新检查与 Epoch/Lease 生效时点。
- T-05：文件系统路径/Junction/TOCTOU 边界。
- T-06：一个具体网页 AI 的请求识别、原会话确认和回传能力。
- T-07：WorkBuddy GUI 真实可观察的 G0–G3 等级。
- T-08：DSH ACP 的实际版本和会话语义。
- T-09：CMD/Test 受控执行的实际副作用及隔离程度。
- T-10：Core 恢复时未知外部结果的正确处理。

## 6. 最小修订路径（无产品代码）

1. 编写并独立保存当前审查记录，不修改 `docs/versions/v1.1`～`v1.4` 的历史原件。
2. 生成新版本化 `docs/reviews/F1-REVIEW-001.md` 与 `docs/decisions/ADR-F1-001.md` 候选决策；在 ADR 中明确“不等价于用户批准”。
3. 统一 12 项条款，形成一页最终 F1 正文；对照 10 项 V1.4 汇总建立映射。
4. 用户批准后由新的 Git commit 记录 F1_APPROVED，不要 amend 历史初始提交 `2d94d23`。
5. 后续如另行授权实施，先在独立工程分支完成 E00，再按结果冻结 F2。E00 通过不代表 E01–E07 实机验收通过。

## 7. 主要来源文档

- `docs/00_BASELINE_AND_STATUS.md`
- `docs/03_DECISION_TRACEABILITY.md`
- `docs/versions/v1.1/01_SYSTEM_ARCHITECTURE.md`
- `docs/versions/v1.1/03_SESSION_AND_RECOVERY.md`
- `docs/versions/v1.1/04_WORKSPACE_TOOLS_SECURITY.md`
- `docs/versions/v1.1/07_ADR_OPEN_ISSUES.md`
- `docs/versions/v1.3/02_PROTOCOL_FREEZE_CHECKLIST.md`
- `docs/versions/v1.3/05_CONTRACT_ALIGNMENT.md`
- `docs/versions/v1.4/00_FREEZE_AND_DECISIONS.md`
- `docs/versions/v1.4/01_LMP_MESSAGE_CONTRACT.md`
- `docs/versions/v1.4/02_TASK_ACK_AND_RECOVERY.md`
- `docs/versions/v1.4/03_ADAPTER_SDK_AND_LIFECYCLE.md`
- `docs/versions/v1.4/04_PERMISSION_SESSION_CONTRACT.md`
- `docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md`
- `docs/versions/v1.4/06_COMPATIBILITY_AND_OPEN_ISSUES.md`

**结论：F1 = REVIEWED / CHANGES REQUIRED。没有自动冻结，也未启动产品实现。**
