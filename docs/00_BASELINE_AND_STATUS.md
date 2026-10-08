# 00｜当前产品基线、状态与优先级

> 这是给未来工程执行者的**入口摘要**；不能取代各原始规范。本文只归纳，**不自行批准**任何未决技术决策。

## 一、已经明确的产品方向（CONFIRMED）

| 主题 | 已确认内容 | 首选出处 |
| --- | --- | --- |
| 产品范围 | 全新独立的 Windows 单用户本地项目，通过本机 Web 控制台管理 | [V1.1 PRD](versions/v1.1/00_PRODUCT_REQUIREMENTS.md) |
| 首个场景 | 网页 AI ↔ Windows 桌面应用；AI 分析、应用执行、真实结果验证和回传 | [V1.1 PRD](versions/v1.1/00_PRODUCT_REQUIREMENTS.md) |
| 模块分发 | **B：内置模块 + 本地开发者模块**；不以插件市场为 MVP 必需项 | [V1.4 决策](versions/v1.4/00_FREEZE_AND_DECISIONS.md) |
| 人工接管 | 允许人工介入；检测冲突后暂停新的自动写入、对账，且 **必须用户明确确认才能恢复写入** | [V1.1 会话](versions/v1.1/03_SESSION_AND_RECOVERY.md) |
| 本地工具 | Workspace Reader、受控 CMD/PowerShell、Test Runner 为首版能力 | [V1.1 工具安全](versions/v1.1/04_WORKSPACE_TOOLS_SECURITY.md) |
| 网页 AI 读文件 | AI 可请求查询已授权工作区文件；不必经过 WorkBuddy/DSH；数据外传单独授权 | [V1.2 评审](versions/v1.2/V1.2_P0_FEASIBILITY_AND_PROTOCOL_REVIEW.md) |
| 接入候选 | WorkBuddy GUI、DSH 协议 Adapter；Obsidian 后续模块 | [V1.1 架构](versions/v1.1/01_SYSTEM_ARCHITECTURE.md) |
| 工程边界 | 与其他既有项目分离，新仓库，不复制原项目做技术起点 | [V1.4 交接](versions/v1.4/07_IMPLEMENTATION_HANDOFF.md) |

## 二、当前冻结判定

| 项目 | 状态 | 含义 |
| --- | --- | --- |
| 用户明确选择的产品需求 | CONFIRMED | 可以写入项目需求基线 |
| F1 核心对象、授权与恢复等架构不变量 | **F1 CANDIDATE / PENDING APPROVAL** | 具有可供批准的规范，尚未作为用户最终审批的冻结结论 |
| LMP V1 消息字段、ACK、状态转移 | **F2 CANDIDATE** | V1.4 候选实现契约；须 E00 测试通过后逐项冻结 |
| Adapter SDK V1 | **F2 CANDIDATE** | 同上 |
| E00–E07 | **NOT RUN** | 已定义验收任务，并没有实机运行通过的证明 |
| WorkBuddy GUI / DSH / 网页 AI 实际接入 | **NOT VERIFIED** | 适配能力、稳定性和真实支持范围均需按具体目标验证 |
| Terminal / Test Runner 强隔离 | **NOT VERIFIED** | 工作目录、权限声明、白名单均不构成完整 OS 沙箱 |

## 三、全平台强约束（F1 候选）

1. Core 是任务状态、授权、Ownership、审计与恢复决策的权威来源。
2. Module Package / Runtime / Connection / Session / Task / Run 必须有各自的身份。
3. 本地读取、向外部 AI 传输数据、本地执行应独立授权。
4. 发送、受理、执行、结果持久化、交付、验证不可互相冒充。
5. 同一写入资源应由有效会话/资源控制权和锁保护；人工接管优先。
6. 结果不确定的外部副作用不得盲目重试。
7. 模块停止或 Lease 过期，不表示已经发出的外部操作自动停止。
8. SDK 权限声明与独立 Worker 不等于可信的 OS 强隔离。
9. 正在执行的 Run 固定版本和契约，不受模块升级暗中改变。
10. 新增/卸载应用 Adapter 不应要求修改 Core 应用专属代码。

详见：[V1.4 冻结决策](versions/v1.4/00_FREEZE_AND_DECISIONS.md) 、[V1.4 权限与会话](versions/v1.4/04_PERMISSION_SESSION_CONTRACT.md) 。

## 四、开发前必须先完成

- 正式批准 F1 候选中的逐条不变量（没有批准的保留 PENDING）。
- 检查最新 V1.4 契约与 E00 验证任务是否一致。
- 从独立项目仓库搭建 **E00 模拟 Adapter** 验证基础，不直接接入现有生产项目。
- E00 验证通过后才能正式冻结 F2 协议；之后按 E01～E07 阶段推进。
