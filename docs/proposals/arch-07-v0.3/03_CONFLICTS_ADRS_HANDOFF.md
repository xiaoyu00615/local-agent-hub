# ARCH-07-03｜规范冲突、待建 ADR 与 COD/UI 交接边界

**状态：审查材料；不修改任何历史 F1、V1.1–V1.4 或 ARCH-06 文件。**

## 1. 唯一权威与历史语境

| 主题 | 现行权威来源 | 解释 |
|---|---|---|
| F1 架构/安全 | `docs/baselines/F1_ARCHITECTURE_SECURITY_v1.0_APPROVED.md` + `docs/decisions/ADR-F1-001.md` | 12 条已批准；高于旧 V1.x 中的 F1 候选叙述 |
| F2 消息、SDK、ACK 候选 | `docs/versions/v1.4/01`–`06` | 仅 Candidate，E00 后方可单独冻结 |
| E00 31 项和测试门槛 | `docs/versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md`、`docs/engineering/E00_READINESS_AND_GATES.md` | E00 仍 NOT RUN；不修改 caseId 集合 |
| 已批准产品方向 | 对话中的用户明确决策 + 后续批准的 ADR | Relay 独立；授权模型池免费优先；备用 Brain；安全切回；受控长期迭代 |
| 新 ARCH-06/07 | `docs/proposals/...` | 设计候选，不得覆盖 F1 或伪装为正式工程命令 |
| UI 现况 | UI 对话的最新 DS08/P08 记录与 `docs/ui-design/` | 与 ARCH 提案并行；不得据旧资料认定当前页面完成/失效 |

`docs/00_BASELINE_AND_STATUS.md` 与部分 V1.4 历史标题中 F1=PENDING 为**编写当时的历史状态**；不改写原文，通过新索引注释当前 F1 已冻结。

## 2. 必须明确裁决的六处差异

| ID | 旧文本/惯性 | ARCH-07 裁决 | 处理 |
|---|---|---|---|
| C01 | V1.1 “未经批准的无限轮执行”列为非目标 | 用户后续允许**获批策略范围内**不设固定总轮数；仍限制资源/预算/风险 | 新需求 Scope ADR，历史原文保留 |
| C02 | 旧导读说 F1 待批准 | F1 12 条已批准冻结 | 更新当前状态索引，不改历史快照 |
| C03 | ARCH-06 广泛规划多 Brain、完整画布 | V1 只做必要抽象与有限工作流，V1.x 才实现高级自治/画布 | 通过实施分期确认，不删未来接口 |
| C04 | 模型选项在网页可见即认为可用 | Catalog 声明≠真实模型已生效 | 证据分级，无法确认时标 UNKNOWN |
| C05 | 旧 Relay 曾被讨论成桥接组成部分 | Relay 仅辅助开发 Hub，不进入产品运行时 | 清理后续交接术语，不搬迁状态机/代码 |
| C06 | Webflow 跳转/静态工作流图被视为功能进度 | Webflow 为原型；Core 状态和业务运行靠真正的前端/后端 | UI 与工程独立验收，对既有 DS08 不复位 |

## 3. 计划建立的决策记录（目前均未正式建立/批准）

| 候选 ADR | 决策内容 | 是否要求用户批准 |
|---|---|---|
| `ADR-ARCH07-SCOPE` | V1 必需范围、V1.x 推迟功能、原 V1.1 PRD 的受控连续运行差异 | 是（产品范围） |
| `ADR-ARCH07-TECH` | Node/TS/SQLite/React 默认栈及未来调整机制 | 技术提议可由工程评审裁决；重大范围/权限变更需用户批准 |
| `ADR-ARCH07-GATES` | 阶段准入、预览/正式版本降级与发布 DoD | 是（产品目标/验收标准） |
| `ADR-ARCH07-BRAIN` | TaskProposal、Brain Gateway/LMP 职责及安全交接接口预留 | 如要冻结新的协议字段/权限语义，单独批准且不能绕过 F2/E00 |
| `ADR-ARCH07-GRAPH` | 有限工作流与后续 Graph/DynamicPlan 兼容性约束 | 如改变已冻结不变量或协议，先走对应 ADR |

ADR 编号是**候选名称**，不是已创建的正式仓库决议文件。全部新字段与参数，仍需原型和对应门禁证据。

## 4. COD 与 UI 的未来交接切片（尚未分派）

**COD-00：** 核对 F1、F2、E00 用例、环境与状态；不得擅自开始编码。

**COD-01：** 只有得到用户专门授权，才开始 E00-001 七项 Mock 测试；代码与证据隔离、通过审查后继续 E00-002～005。

**COD-02：** 逐 Gate 完成 Reader、Web Brain、WorkBuddy、Test Profile 和可追踪业务闭环。

**COD-03：** 第二不同 Driver、故障恢复、有限 Workflow/P05 与 V1 正式 Web Console；复杂自治以独立切片后置。

**UI-00：** 核对已有 Webflow/P01–P08/DS08 状态与 `docs/ui-design/`，先对照不重做、不发布。

**UI-01：** 与 COD 共享字段字典：模块/连接分层、会话状态、三轴授权、执行/交付/验证状态、审批与对账。

**UI-02：** 首版 P04 以步骤式 Draft/关系视图为主、P05 以真实证据监控为主；高级自由画布和 Brain 多模型可视化后置。

**UI-03：** 如果用户另外授权，才在 Webflow 的独立候选 Draft 上进行修改、测试和迁移；此文档不是 Webflow 写入许可。

## 5. 工作流与权限的最低共享契约

- `ModulePackage` ≠ `ModuleRuntime` ≠ `ConnectionInstance` ≠ `SessionBinding`。
- `NodeDefinition` ≠ `NodeExecution`；`WorkflowRevision` 固定绑定当前 Run。
- `Iteration` ≠ `Attempt` ≠ `DeliveryAttempt`；业务尝试和运输补交付不同。
- `BrainPlanningEpoch` ≠ `AppSessionOwnershipEpoch`；两者均需符合 Core 的有效身份与当前权限。
- `TaskProposal` 仅是任务意图；LMP Envelope 才是已授权 Core↔Adapter 能力消息。
- UI“发送成功”≠ Adapter 接单≠外部已提交≠结果已存储≠目标已接收≠业务验收通过。
- 自动回到 GPT、切换 API Provider、发送结果均需要核实收件范围及 `EXTERNAL_DISCLOSURE`。

## 6. 正式交接与变更时禁止事项

- 不通知 COD/UI、不触发 Webflow MCP 变更、不执行 E00，不将本文件当作暗含授权。
- 不修改 F1 十二条正文；如出现不可兼容点先生成新 ADR 候选并等待批准。
- 不把 `docs/ui-design/` 与 ARCH 文档一起 `git add .`。
- 不把 `docs/arch-06-review@bc6bac0` 写成 GitHub 已推送/合并。
- 不把 E00 Mock PASS 直接当成 E02 网页、E03 GUI、E06 重启或长期无人值守 PASS。
- 不用未经实际核验的 Web GPT 会话复用、模型更换或 MJ 自动化能力做承诺。
