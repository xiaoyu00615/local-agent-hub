# 02｜实施前冻结清单（Freeze Checklist）

**状态：待签署评审；除已确认用户需求外均为候选。**

## A. 用户确认的需求 / CONFIRMED
- [x] C01 Windows 本机单用户，localhost Web 控制台。
- [x] C02 网页 AI↔桌面应用为第一条样板闭环。
- [x] C03 无公开个人 API 的 Windows GUI Driver 是正式支持方案。
- [x] C04 Agent Adapter 使用统一能力声明。
- [x] C05 用户可直接人工介入，检测到冲突后阻止新自动写入并对账。
- [x] C06 人工接管结束后，即使对账成功也必须用户明确确认恢复。
- [x] C07 模块安装为内置模块 + 本地开发者模块 B。
- [x] C08 本地文件只读查询、受控 CMD/PowerShell、Test Runner 为首版功能。
- [x] C09 网页 AI 可请求直接读取已授权本机工作区，不必经桌面 Agent。
- [x] C10 Obsidian 后续接入，非首版闭环阻塞项。

## B. F1 架构/安全候选冻结（建议批准前逐条核查）
- [ ] F1-01 Core 权威记录：Module Registry、Run/Task/Attempt、Grant、Ownership、恢复决定、审计写入职责明确。
- [ ] F1-02 Module Package/Runtime/Connection Instance/Session Binding/Workflow Revision/Run/Task/Attempt 独立标识。
- [ ] F1-03 能力注册、参数 Schema、权限、完成证据、资源锁为模块统一契约。
- [ ] F1-04 Local Read、External Disclosure、Local Execute 三轴许可不可互相推导，默认拒绝。
- [ ] F1-05 任何来自网页/仓库文件/Agent 自由文本的命令只是数据，不可自己授权。
- [ ] F1-06 同会话最多一名自动写入所有者，Generation/Epoch/Lease 与 Core 权限记录组合核验。
- [ ] F1-07 用户接管后停止新自动写入，恢复必须用户主动确认；Core 重启、模块重连不得自动解禁。
- [ ] F1-08 发送、执行、收集、Core 持久化、交付、ACK、独立验收分别留证。
- [ ] F1-09 外部操作可能已提交但结果未知，拒绝盲目重试；补交付不等于重执行。
- [ ] F1-10 工作流 Run 固定不可变版本，模块替换及升级都要做契约兼容检查。
- [ ] F1-11 个人本机服务默认 loopback；控制台/扩展/IPC 需鉴权、来源检查、CSRF/WS 防护。
- [ ] F1-12 本地开发者模块可信来源有标记；独立进程与 Manifest 不是强沙箱。

## C. F2 实现协议冻结（实验前不可勾选）
### 模块和消息
- [ ] F2-01 模块 Manifest 必需字段、ID 格式、版本/兼容范围与签名/完整性策略确定（E00）。
- [ ] F2-02 LMP 消息 kind、字段、Schema/时间语义与传输 ACK 定义确定（E00）。
- [ ] F2-03 Core 事务提交与任务派发意图的边界明确，重启可核对（E00/E06）。
- [ ] F2-04 `messageId` 与 `idempotencyKey` 使用范围分别通过重复测试（E00/E06）。
- [ ] F2-05 Module Lifecycle STARTING/READY/DRAINING/FAILED 及 Module vs Connection 分层通过测试（E00/E07）。
### Session 与 Agent
- [ ] F2-06 会话绑定身份必需证据与失效规则明确（E02/E03/E04）。
- [ ] F2-07 所有 GUI 提交前检查 Generation/Epoch/Lease，控制权变化后阻断新写入（E03/E06）。
- [ ] F2-08 Agent Turn submit/observe/collect 的返回证据、错误与可选性语义确定（E03/E04）。
- [ ] F2-09 G0/G1/G2/G3 分档由具体实机证据确定，不能根据 Adapter 声明自选（E03）。
- [ ] F2-10 DSH ACP/SDK 当前版本行为与 LMP 能力映射实测通过（E04）。
### 工具和权限
- [ ] F2-11 工作区目录与最终文件对象边界经路径越界验证（E01）。
- [ ] F2-12 每次对外结果传输重新校验目标与 Disclosure Grant（E02）。
- [ ] F2-13 Native Tool 与 Conversation Bridge 分别定义自己的接收/交付保证（E02）。
- [ ] F2-14 Test Profile 的命令、参数、工作区、日志、超时和副作用处理契约通过实测（E05）。
- [ ] F2-15 若宣称 T2 沙箱保护，必须有 OS/虚拟化级拒绝文件/网络访问证据；否则不得使用该宣称（专项）。
### 工作流和界面
- [ ] F2-16 工作流节点依赖、输入输出类型、任务与结果三轴状态测试通过（E00/E06）。
- [ ] F2-17 Run 固定 Revision、升级不静默改变能力定义（E06/E07）。
- [ ] F2-18 用户接管、核对和人工恢复 UI 操作与 Core 权威记录一致（E06/E07）。
- [ ] F2-19 持久化事件快照+流式事件续传的顺序与去重语义明确（E06/E07）。
- [ ] F2-20 恢复按钮、取消按钮、停止模块分别具有不同且可测试的操作语义（E06/E07）。

## D. 只冻结语义，不立即冻结数值
- 连接心跳/失效阈值；长任务超时；队列并发数；搜索大小上限；测试 Profile 资源上限；具体 stdout 截断；内存预算；网页 DOM 判据。由环境试验和可配置默认值决定。

## E. 冻结审批记录模板
- 决策 ID：
- 规范/条款版本：
- 提交者及评审者：
- 证据附件和实验 caseId：
- 已排除的失败路径：
- 兼容影响：
- 回退/降级方式：
- 状态：PROPOSED → REVIEWED → APPROVED → FROZEN（有正式批准记录才改变）
