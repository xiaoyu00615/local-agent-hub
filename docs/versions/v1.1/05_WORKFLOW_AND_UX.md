# 05｜工作流与 Web Console 交互规范 V1.1

**状态：REVIEW DRAFT**

## 1. 工作流数据对象
Workflow（可复用流程）、Revision（不可变发布版本）、Run（一次执行）、Node Execution（节点执行）、Attempt（尝试）、Artifact（输入输出/产物引用）独立建模。

首版节点：Start、Agent Turn、Tool Call、Condition、Approval、Verify、Deliver、End。暂不要求复杂无限画布、自动动态流程改写、子工作流及无界循环；内部定义预留 DAG/条件分支支持。

Run 启动时固定 Workflow Revision、所需 Capability Schema 和授权策略版本。编辑中新版本不得悄悄改变在运行的 Run。每节点执行前验证依赖、输入、连接、Grant、Ownership 和资源锁。

AI 可提出候选流程，但提议不是发布；发布、增加权限和执行有副作用操作按照用户和策略批准。

## 2. Run 状态
`CREATED`、`VALIDATING`、`RUNNING`、`WAITING_APPROVAL`、`WAITING_EXTERNAL`、`PAUSED`、`RECONCILING`、`BLOCKED`、`SUCCEEDED`、`FAILED`、`CANCELLED`。节点与结果交付/验收状态另记；不把“结果返回”自动视为整个 Run 成功。

## 3. 顶层页面
- 工作台：Core 健康、当前工作区、活跃任务、待用户处理事项。
- 模块与连接：Package、Runtime、Connection Instance、能力目录分开展示。
- 工作区与工具：目录管理、只读查询、授权、Test Profile、受控终端执行。
- 工作流：步骤式编辑、关系概览、运行前校验、Revision 管理。
- 任务中心：Run/Node/Attempt、事件时间线、执行证据、结果交付与验收状态。
- 审批与恢复：新操作授权、人工接管结束确认、结果不确定的对账决策。
- 产物与记录：Artifact、测试报告、审计日志和隐私过滤。
- 设置：本地服务、版本、备份、诊断和开发者模式。

## 4. 首次使用路径
Windows Launcher 启动 Core → Web Console 初始化 → 注册授权工作区 → 启用 Reader/受控工具 → 安装/授权网页连接器 → 绑定 AI 对话 → 连接 WorkBuddy 或 DSH → 安全连接测试 → 从模板运行样板流程。

Web 页面不应假设浏览器文件夹选择器直接授予 Core Windows 文件访问权；真实目录授权必须通过受信任的本机接口显式确认。

## 5. 确认语义
用户点击操作后 UI 首先显示请求提交/待确认；Core 成功持久化且已返回真实状态后才更改权威显示。暂停、请求取消、人工接管、阻止新执行是不同操作。人工接管释放后必须显式“确认恢复”而不是自动继续。

## 6. 错误/空状态
每个页面具备未安装、未配置、权限不足、连接断开、运行中、等待人工、结果未知、Core 断开等真实状态与对应可行操作。错误文案需指出影响对象与恢复方案，不仅提供错误码。

## 7. 可用性目标
Windows 桌面优先，支持常见窄桌面窗口和可折叠侧栏；长日志虚拟展示、可复制和筛选；颜色不是唯一状态表达；实时事件需要支持断线重连后与持久化记录对齐。
