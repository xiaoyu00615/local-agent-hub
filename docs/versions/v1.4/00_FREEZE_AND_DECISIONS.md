# 00｜V1.4 冻结判定与约束

## 1. 已由用户确认的决策（CONFIRMED）

| ID | 决策 |
|---|---|
| C-01 | Windows 本机单用户，通过本机 Web 控制台操作。|
| C-02 | 第一条样板：网页 AI ↔ 桌面应用协作。|
| C-03 | Windows GUI 自动化是正式支持的接入类别，不因无公开 API 而排除。|
| C-04 | Agent Adapter 使用统一能力模型，能力按真实证据分级。|
| C-05 | 会话允许用户人工介入；检测一致性冲突时暂停自动化新写入并核对。|
| C-06 | 人工接管结束且核对通过后，**仍须用户主动明确确认才能恢复自动写入**。|
| C-07 | 第一版安装方式 **B：内置模块 + 本地开发者模块**。|
| C-08 | P0 内置 Workspace Reader、受控 CMD/PowerShell、Test Runner；网页 AI 可在权限范围内直接查询本地文件。|
| C-09 | WorkBuddy GUI、DSH ACP 为主要候选样板；Obsidian 后续可添加。|
| C-10 | 本轮仅设计协议和测试契约，不编写产品实现代码。|

## 2. 本轮建议 F1 冻结的架构不变量（PENDING APPROVAL）

1. Core 对持久化任务、授权和控制权拥有最终写入权。
2. Module Package、Runtime、Connection Instance、Session Binding、Workflow Revision、Run、Task、Attempt、Message、Delivery Attempt 各有独立身份。
3. 所有外部写入操作通过 Core 许可和执行前复核；网页文本不能授予执行权。
4. Local Read、External Disclosure、Local Execute 三维授权独立，默认拒绝。
5. 一个 Session 同时最多一个自动化写入所有者；旧 Binding Generation / Ownership Epoch / Lease 不允许新动作。
6. 人工接管后保留恢复禁止标志，不能因 Core 重启或模块重连自行清除。
7. 执行、结果持久化、下游交付、业务验收是不同事实；ACK 不得跨语义使用。
8. 外部操作是否发生未知时必须冻结并对账；不允许盲目重发有副作用动作。
9. 工作流 Run 固定不可变 Revision 和能力契约；模块升级须检查兼容。
10. 开发者模块可加载而无需改 Core 的应用专属逻辑；独立进程并不代表安全沙箱。

## 3. 本轮候选 F2 最小实现契约（需要 E00）

| 领域 | 候选内容 | 验收来源 |
|---|---|---|
| Identity | Core 生成 Runtime/Connection/Binding/Task/Attempt ID | E00-ID |
| Envelope | 消息 kind、name、IDs、Schema 版本 | E00-MSG |
| Handshake | Module Host 身份校验及协议协商 | E00-HS |
| ACK | 接单、结果持久化、交付确认分离 | E00-ACK |
| Persistence | Core 事务记录 + Outbox 派发意图 | E00-DB |
| Deduplication | messageId 与 idempotencyKey 分别去重 | E00-DUP |
| Ownership | Epoch/Generation/Lease 过期阻断 | E00-OWN |
| Adapter SDK | describe、initialize、invoke、observe、reconcile、stop | E00-SDK |
| Lifecycle | READY、DRAINING、FAILED、QUARANTINED 的合法含义 | E00-LIFE |
| Recovery | 未知外部结果不重执行；已持久化结果可补交付 | E00-REC |

## 4. 本轮不冻结的事项

- 任意网页 AI 是否提供原生工具调用；网页 DOM、回传完成证据：E02。
- WorkBuddy 的具体 GUI 会话定位规则及 G0–G3 能力档案：E03。
- DSH 版本、ACP 支持子集、取消/恢复和 Token 使用语义：E04。
- Windows 强隔离执行、可信测试配置细节和资源限制参数：E05。
- 精确心跳/超时、页面尺寸、资源上限、序列化二进制格式：待相关实测。

## 5. 冻结流程

`DRAFT → REVIEWED → APPROVED (用户确认) → FROZEN (必要实验通过、版本明确)`。

本文件为 REVIEWED CANDIDATE；不能自行把 F1 设为用户批准，也不能把 E00 等未运行实验标记 PASS。冻结须生成决策编号、证据 ID、兼容影响和保留例外。
