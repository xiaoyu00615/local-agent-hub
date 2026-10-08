# 01｜系统架构与模块边界 V1.1

**状态：REVIEW DRAFT**

## 1. 分层
- Windows Launcher：负责 Core 启停、浏览器打开、启动失败诊断，可按发布策略延后独立托盘功能。
- Web Console：展示状态与发出控制请求，不拥有权威任务状态、不直接执行本机高权限操作。
- Local Core Runtime：Module Registry、Capability Router、Workflow Scheduler、Task Engine、Event Journal、Session Ownership、Permission Gateway、Artifact Store、Recovery Manager。
- Module Host：管理独立 Worker、启动参数、协议握手、健康检查、运行权限和版本切换。
- Adapter：具体站点或应用的连接驱动（Web、GUI、ACP、API、CLI），只报告证据并执行授权能力。
- Storage：SQLite 结构化状态/事件/待交付记录；受控文件系统存储 Artifact、结果摘要和快照。

## 2. 必须独立建模的身份
`packageId`：模块身份；`moduleVersion`：模块版本；`runtimeInstanceId`：Worker 进程；`connectionInstanceId`：连接实例；`sessionBindingId`：对话/工作会话；`workspaceId`：本地资源授权边界；`workflowId` / `revisionId` / `runId` / `nodeExecutionId` / `attemptId`：工作流及执行分层。

## 3. 三种 Adapter 形式
- Agent Adapter：会话、轮次、提交、观察、收集、可选取消/恢复。
- Tool Adapter：结构化读写工具能力、无副作用或明确副作用语义。
- Application Adapter：应用窗口/文档/业务资源接口；Agent Adapter 是其专业化类别，生命周期统一。

**内置/开发者**是来源属性，不是另一套通信协议。

## 4. 运行边界
- 建议模块化单体 Core + 独立 Adapter Worker。
- 模块 READY 不代表具体应用、会话或任务 READY；Package、Runtime、Connection、Session 状态不复用一个枚举。
- 受信任内置 Worker 可采用受控专用实现；开发者模块默认独立进程，按显式信任模型启用。
- 独立进程只提供部分故障隔离；Windows 同用户进程没有天然文件/网络隔离。需要强隔离的执行必须另外选用经过实际验证的 OS/VM/容器保护。

## 5. 数据控制权
- Core 是模块状态登记、任务执行与审批结果的权威记录者；Adapter 不是 Core 数据库的写入方。
- Adapter 的“执行完成”报告只证明其已观察到的事实；Workflow Verify 独立验证目标产物。
- 调度器与记录库采用先持久化执行意图、再派发的模式；发送与结果交付使用持久化待办记录，并用唯一业务键防止无声重复。
- 前端重连：先加载带序列号的状态快照，再续接事件，不以浏览器缓存决定成功状态。

## 6. 模块安装/运行/维护
Package 状态：DISCOVERED、VALIDATING、INSTALLED、CONFIG_REQUIRED、UNINSTALLED。
Runtime 状态：STOPPED、STARTING、READY、DEGRADED、DRAINING、FAILED、QUARANTINED。
Connection 状态：PAIRING、READY、DEGRADED、DISCONNECTED、REVOKED、INCOMPATIBLE。
运行中的 Agent 会话独立使用 Ownership 状态机。

安装流程：本地目录 → 静态读取 Manifest → 验证协议/依赖/能力及来源 → 权限确认 → 安装注册 → Worker 启动 → 创建连接实例 → 非破坏性测试。安装前不执行未经确认的脚本。

升级流程：停止新任务 → 处理/记录在执行任务 → 保留旧版本 → 候选版本分目录安装 → 检查权限差异和兼容性 → 健康验收 → 切换或回滚。回滚不等于撤销外部应用已执行的副作用。
