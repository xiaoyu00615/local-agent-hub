# E00-001 — 最小 LMP 模拟通信与握手验证工程任务书

- **任务 ID：** `LAH-E00-001`
- **状态：** `READY FOR IMPLEMENTATION REVIEW`（任务已准备；尚未批准开始编写产品代码）
- **依据：** `F1-BASELINE-001` 已获用户批准；V1.4 LMP/SDK 属 `F2 CANDIDATE`
- **前置提交：** 初始 `2d94d23`；须先把 F1 审批文件合并 `main`
- **工程分支建议：** `feat/e00-001-lmp-mock-handshake`
- **执行位置：** 全新独立仓库 `D:\Workspace\projects\local-agent-hub`
- **测试种类：** 只允许合成数据、模拟 Adapter 和隔离测试夹具；不接真实网页、WorkBuddy、DSH、真实项目文件、CMD 或 PowerShell 自动执行

## 1. 本任务目标

构建一个最小、可扩展、可确定性复测的 **LMP V1 消息与握手模拟验证切片**。这个切片不是产品 Core 的最终实现，也不宣称完成全部 E00 31 项测试；它要证明在不接入外部应用的情况下，可以唯一识别模块/端点、协商协议版本、拒绝未经认证和不符合 Schema 的消息，并记录足以复测的执行证据。

**只交付可检验的协议骨架和 Mock 测试能力，不开发 UI、不执行真实操作、不默认授予 Local Read/Disclosure/Execute 权限。**

## 2. 建议技术与结构（实施前核对环境后选择）

- 优先 TypeScript 和适配当前 Node LTS 的轻量工具链；Node/TS 具体版本与包管理器在实施前读取环境并记录，不凭本文件假定已安装。
- 单仓库中隔离 `packages/lmp-contract`（协议类型与 Schema）、`packages/adapter-sdk`（本切片仅最小接口）、`tests/e00`（Mock、故障注入及测试）。实际目录可在评审时按仓库工程规范调整，但不得将应用专属逻辑直接放入 Core。
- 使用可替换的 `Transport` 接口及进程内模拟通道；测试不依赖 Windows GUI、网络云服务、第三方账号、WebSocket 实例或真实 SQLite 数据库。
- Mock Clock、Mock Identity Registry、Mock Event Journal 需确定性，以便重放相同测试并比较可观察结果。
- 敏感凭据全部用随机测试伪值；测试日志必须脱敏，不存真实 token/cookie。

## 3. 必须实现的最小职责（不是固定源码接口）

| 部件 | 必须承担的行为 |
|---|---|
| Contract Validator | 校验 `kind`、`name`、`protocolVersion`、载荷版本、必需字段与 Schema，拒绝未知控制字段 |
| Identity Registry | 区分 Package / Runtime / Endpoint / Connection / Binding / Task 等逻辑身份，不用模糊 `instanceId` 代替所有对象 |
| Mock Module Host | 在未认证前只允许 Bootstrap；握手后记录已认证 Endpoint 与能力快照 |
| Mock Transport | 可发送、丢弃、延迟、重复、乱序消息，不直接执行外部副作用 |
| Mock Adapter | 正常版、错误身份版、不兼容版本版；能力声明可验证 |
| Event Evidence | 为每个测试记录输入、预期、实际、拒绝原因和 Core 内单调 `journalSeq` |
| Conformance Runner | 单项运行与全集运行，失败退出码非零，报告 PASS/FAIL/BLOCKED |

对 `moduleRuntimeId` 的“生成者”必须先写入一份小型方案比选记录：`Core` 负责权威注册、`Host` 负责运行实例证据；实际唯一签发方式在 E00-ID/HS 验证后再确定。不能偷偷在代码中把争议约定当成已冻结。

## 4. 本任务首批必须通过的 Case ID

| Case | 验证要求 |
|---|---|
| `E00-ID-01` | Package、Runtime、Connection、Binding、Task 等身份独立，引用可追踪 |
| `E00-HS-01` | 正确来源及兼容协议的 Mock Adapter 成功握手，登记经认证 Endpoint |
| `E00-HS-02` | 凭据、调用来源或身份不匹配时拒绝，不能注册能力 |
| `E00-HS-03` | 不兼容 LMP major 时拒绝，不能静默降级 |
| `E00-MSG-01` | 必需字段缺失、消息/载荷 Schema 非法时拒绝并可审计 |
| `E00-MSG-02` | 未知权限/控制字段或未声明能力不能误当成文本命令转发执行 |
| `E00-SDK-02` | 已注册能力的输入 Schema 不合规时，在调用 Driver 之前拒绝 |

这些用例对应仓库现有 `docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md`。若该规范实际包含 **31 项**，本任务只覆盖上述 **7 项**，余下 **24 项**应另建任务；不得将这 7 项成功误称为 E00 全部 PASS。

## 5. 接受标准（任何一项失败都不得宣称完成）

1. 上述 7 个 caseId 都有自动化 PASS 结果及可复现运行指令；用例失败时自动化测试进程退出非零。
2. 未认证、错误版本、未知控制字段、不符合 Schema 的消息均不能执行任何模拟业务动作。
3. 正常握手的 Endpoint 与真实声明的 Module Runtime、版本和能力快照可关联；不得与外部应用 Session 混淆。
4. 每一条成功存储的测试事实有递增 `journalSeq`，不依赖外部时间戳决定权威顺序。
5. 所有测试在无网络、无实际 Agent 软件、无真实用户工作区情况下可运行。
6. 测试证据包含 `caseId`、运行环境指纹、刺激、预期、实际、证据引用、结论与缺陷 ID（失败时）。
7. 无权限提升、无秘密外泄、无对本机真实项目的写入；生成物仅限本独立仓库与受控临时测试目录。
8. README 说明本任务只覆盖 E00 第一切片；工作流、ACK、Outbox、恢复等仍未实现或验证。

## 6. 非目标（不得擅自扩大）

- 不实现真实 WorkBuddy GUI、DSH ACP、ChatGPT 浏览器扩展或外部应用通信。
- 不实现任意 Shell、文件系统全盘扫描、真实工作区授权或自动化写入。
- 不把 Runtime 身份方案、最终 ACK 枚举或完整状态机宣告 `F2 FROZEN`。
- 不修改其他仓库，不修改 `main` 历史，不上传本机凭据，不在没有用户批准时触发外部动作。

## 7. 建议执行顺序

1. 确认 F1 记录已归档并固定到 `main`，确认 Git 工作区 clean。
2. 在新工程分支实施，不在 `main` 直接写代码。
3. 先定义断言和合成数据，用测试驱动协议约束；再实现 Mock 与 Schema 组件。
4. 分别运行 7 个本切片 Case ID，提交通过/失败证据。
5. 检查文档术语与实际接口的差异，用 `F2 OPEN ISSUE` 或 ADR 候选记录，不悄悄更改 F1。
6. 提交 Pull Request，仅在所有首批门槛满足后允许合并。
7. 后续再拆分 E00-002（ACK 与三轴状态）、E00-003（幂等与 Outbox）、E00-004（Lease/权限/接管）、E00-005（崩溃恢复及全套回归）；分组可在工程评审时调整。

## 8. 执行者需要提交的交付物

- 最小 LMP Contract / Adapter SDK 模拟代码及 7 个自动化测试（**后续实际开发阶段**）。
- 环境及工具链版本、测试命令、测试结果摘要、完整失败清单。
- 合成的事件/证据日志，不包含真实用户内容。
- 实施阶段差异与需要进一步冻结的 F2 字段清单。
- Git 分支、提交 SHA、工作区状态和 PR 说明；无证据不得宣布 PASS。

## 9. 准入决定

**当前状态为 `TASK_PREPARED / NOT_EXECUTED`。** 本任务书是工作交接资料；用户可在完成 F1 Git 归档后另行授权开始 E00-001 工程实现。
