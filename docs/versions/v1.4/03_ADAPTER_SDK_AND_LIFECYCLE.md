# 03｜Adapter SDK V1、Manifest 与生命周期契约

**状态：F2 CANDIDATE，E00 尚未执行。**

## 1. 最低 SDK 目标

开发者添加一种新软件，只需提供 **Manifest + 可运行 Adapter Worker + 能力描述/Schema + 配置与权限声明 + 一致性测试**，不修改 Core 的应用专属逻辑。

SDK V1 首选 TypeScript，但协议必须语言中立；C#、Python 等可使用等价 LMP 实现。SDK 是开发便利层，不是权威任务引擎，也不是 OS 安全沙箱。

## 2. Manifest V1 最小字段

| 字段 | 必须性 | 语义与验证 |
|---|---|---|
| `moduleId` | MUST | 稳定的命名空间 ID，不随显示名改变 |
| `displayName` | MUST | 用于 UI，不参与业务去重 |
| `moduleVersion` | MUST | SemVer，独立于协议版本 |
| `publisher` | MUST | 来源身份，用于用户安装审查，不等于外部可信认证 |
| `entrypoint` / `runtimeType` | MUST | 指定受管 Worker 的启动渠道，不允许隐式 postinstall |
| `supportedPlatform` | MUST | Windows 及架构/依赖要求 |
| `protocolRange` | MUST | 支持的 LMP major/minor 范围 |
| `sdkRange` | SHOULD | SDK 兼容信息，跨语言实现可写等价规格 |
| `moduleType` | MUST | `agent` / `tool` / `application`，可多标签 |
| `capabilities` | MUST | 能力列表及版本化 I/O Schema |
| `permissions` | MUST | 申请的资源与副作用类型；默认零权限 |
| `configurationSchema` | 条件 MUST | 需要用户配置时提供 Schema 与敏感字段标识 |
| `healthContract` | MUST | 定义可检测的运行健康，不等于会话健康 |
| `dependencies` | MUST | 可为空；版本约束和所需外部应用信息 |
| `packageDigest` | 安装登记时 MUST | 文件完整性摘要；非来源真实性的独立证明 |

模块导入遵循：静态检查 → 展示来源/权限 → 用户批准 → 受管 Host 启动 → 协议握手 → 注册能力。完整插件市场、远程下载和自动更新不属于首版 B。

## 3. Capability Descriptor V1

每个能力描述包含：

| 字段 | 内容 |
|---|---|
| `name`、`version` | 稳定能力名和 Schema/语义版本 |
| `inputSchema`、`outputSchema` | JSON Schema Draft 2020-12 |
| `availability` | `SUPPORTED` / `CONDITIONAL` / `UNSUPPORTED` |
| `implementationMethod` | `NATIVE` / `GUI` / `DERIVED` |
| `sideEffectClass` | `NONE` / `EXTERNAL_READ` / `EXTERNAL_WRITE` / `UNKNOWN` |
| `retrySafety` | `SAFE` / `RECONCILE_FIRST` / `NEVER_AUTOMATIC` |
| `cancellation` | `SUPPORTED_WITH_CONFIRMATION` / `BEST_EFFORT` / `UNSUPPORTED` |
| `concurrencyScope` | `SESSION` / `FOCUS` / `RESOURCE` / `MODULE` 等锁要求 |
| `requiredGrants` | Local Read、Disclosure、Execute 等权限条件 |
| `evidenceContract` | 什么证据最多能证明到哪个状态 |
| `timeoutsAndLimits` | 参数化可配置，不能把猜测数值当真实保障 |

未声明 `agent.turn.cancel` 的模块，Core 只能停止后续调度，不能宣称已终止外部 Agent。`SUPPORTED` 是声明，是否可对外开放 G2/G3 仍须实机证据。

## 4. SDK 生命周期与回调（职责级，不是源代码）

| 名称 | 触发方 | Adapter 应返回/执行的职责 |
|---|---|---|
| `describe` | Core/Host | 身份、版本、能力清单和契约摘要；只读 |
| `validateConfig` | Core/Host | 配置合法性、缺失依赖与敏感字段标注 |
| `initialize` | Host | 分配内部资源，不自动执行外部高权限动作 |
| `start` | Host | 握手、健康探测，进入可管理状态 |
| `healthCheck` | Host | 模块运行健康及细分故障；不代替 session.inspect |
| `connect` | Core | 连接一个应用实例，返回定位证据供 Core 验证 |
| `disconnect` | Core | 停止对该目标的新动作、清理连接，保留必要证据 |
| `invoke` | Core | 调用已声明能力；检查操作上下文，提供接受/拒绝回应 |
| `observe` | Core/Host | 报告可关联的执行状态和证据，不直接覆盖 Core 状态 |
| `reconcile` | Core | 返回外部事实、证据冲突和候选结论；不自行授权恢复 |
| `requestCancel` | Core | 可选；请求取消并报告真实结果，不伪造取消成功 |
| `stop` | Host | 进入 DRAINING、拒绝新任务、处理在途操作 |
| `dispose` | Host | 释放模块资源，不篡改历史 Task/Artifact |

`invoke` 的输入应带上 Core 签发或可查询的任务上下文：`taskId`、`attemptId`、`idempotencyKey`、能力版本、目标实例、有效 Grant 引用、会话 Generation/Epoch/Lease（如适用）。任何缺失或失效均拒绝高权限调用。

## 5. Module Runtime、Connection、Session 分层状态

- Runtime：`STOPPED`、`STARTING`、`READY`、`DEGRADED`、`DRAINING`、`FAILED`、`QUARANTINED`。
- Connection：`DISCONNECTED`、`CONNECTING`、`CONNECTED`、`DEGRADED`、`REVOKED`、`INCOMPATIBLE`。
- Binding：`VERIFIED`、`PROVISIONAL`、`LOST`。
- Ownership：`AVAILABLE`、`AUTOMATION_OWNED`、`TAKEOVER_PENDING`、`HUMAN_OWNED`、`RECONCILING`、`SUSPENDED`。

Runtime READY 只证明 Worker 能提供协议服务；不自动令 Connection/Binding READY。`DRAINING` 禁止接收新业务尝试；适配器进程停止**不证明外部应用动作已停止**。

## 6. 版本升级/卸载

- 安装新版本时记录候选包完整性，先进行静态兼容检查；旧版本可共存用于回滚。
- 与已运行 Run 关联的能力契约不可被默默替换；升级前停止派发新任务，并处理在途任务。
- 新版本请求扩大权限必须重新获得授权。
- 模块卸载可删除 Worker 包，但保留历史 Run、Task、消息及产物引用；相关工作流显示缺少依赖。
- 本地开发目录热修改不应静默改变运行代码；建议复制到受管理的版本快照后再启用。

## 7. 安全与隔离限度

开发者模块即使 Manifest 宣称只读，也可能因共享 Windows 用户权限访问其他资源。Host 隔离优先保证崩溃不拖垮 Core；**不得宣称进程分离即实现强文件沙箱**。首版只支持明确可信的本地开发者模块，安装时展示来源、权限与能力；Core 在其受控工具服务端执行真正的授权判断，不把凭据传给未知网页正文。

## 8. 驱动映射与基本档案

- WorkBuddy GUI：`connect` 定位应用/会话；`invoke agent.turn.submit` 进行受控交互；`observe` 和 `reconcile` 返回 UI 证据。G1/G2/G3 实测前不得先开放无人值守。
- DSH ACP：Driver 将 ACP 初始化、会话、prompt 和事件翻译为 LMP 能力；不假定 ACP 完整等价于 LMP 或某特定版本支持所有功能。
- Workspace Reader：以工作区 ID 提供文件只读能力；文件来源和最终目标路径授权必须独立检查。
- Test Runner：只运行注册的 Test Profile，记录 stdout/stderr/退出状态/可能的副作用；与任意 Shell 区分。
- Obsidian：后续可以复用 Tool/Application Adapter，不作为首版验收阻塞项。
