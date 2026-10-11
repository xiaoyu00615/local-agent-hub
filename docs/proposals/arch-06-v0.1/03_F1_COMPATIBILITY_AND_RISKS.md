# ARCH-06-03｜F1 十二条不变量兼容性与风险审查

**状态：DESIGN REVIEW / NOT VERIFIED；不是对新功能完成安全验收。**

## 1. 与 F1 已批准条款的逐项映射

| F1 编号 | 不可违背的不变量 | ARCH-06 设计约束与检查点 | 判断 |
|---|---|---|---|
| 01 Core 权威 | Brain/Adapter 不改权威状态；Journal 存储后才承认已保存；Monitor 只读 | 无结构性冲突，需事务实测 |
| 02 身份分层 | Node/Iteration/BrainSession 是新增身份；严格分离 Package/Runtime/Endpoint/Connection/Binding/Revision/Run/NodeExec/Task/Attempt/Message/DeliveryAttempt/Grant | 可兼容；需 ID 字典 |
| 03 能力统一 | Model Router 只是选择器；一切受管操作经 Core Capability、Schema、Grant、锁、证据检查 | 可兼容，不能宣称约束 Hub 外程序 |
| 04 三轴授权 | LOCAL_READ、EXTERNAL_DISCLOSURE、LOCAL_EXECUTE 分开；切换 API Brain、回传 MJ 资产前重新验证目的服务/会话与内容范围 | 强制要求 |
| 05 不可信内容 | GPT 文本、`USE` 标头、网页正文、文件不授予权；结构化 Proposal 仍要来源认证和用户策略检查 | 强制要求 |
| 06 写入排他与锁 | Planning Epoch 与外部 App Session Ownership 分离；GUI 焦点/模型全局设置/工作区锁按资源控制 | 可兼容，双写冲突为 P0 |
| 07 人工接管 | Brain 自动故障切换/安全切回不得解除人工接管后写入阻断；需显式用户 Resume | 强制要求 |
| 08 执行/存储/交付/验收 | Adapter ACK、外部提交、持久化、Brain Delivery、Verify 逐层独立，P05 不合并为 DONE | 强制要求 |
| 09 结果未知禁止盲重 | Iteration、Fallback、Scheduler、脑切换不得开启未知副作用的第二个业务 Attempt | 强制要求 |
| 10 Revision 不可变 | 允许在发布范围内选择模型、节点能力及路由；DynamicPlan 只实例化合法计划；扩大范围需新审批/修订 | **必须严审候选动态规划语义** |
| 11 Loopback 不等于可信 | Web Console、扩展、IPC、WS 等各有鉴权和 Origin/CSRF 处理；网页脚本不持有高权凭据 | 强制要求 |
| 12 Worker 非强沙箱 | 首版仅内置/明确可信本地开发模块；Module Host 分进程不宣传 OS 级文件/网络强隔离 | 强制要求 |

**审查结论**：没有发现必须修改 F1 正文才能表达的架构扩展；但这只能证明“设计层可兼容”，**不能证明具体实现或外部接入已安全**。F2 字段、生命周期、握手和 ACK 必须遵循 E00 真实测试裁决。

## 2. 四个必须预先排除的架构错误

**X1：双重 Core。** 把 Relay、Brain Coordinator 或 Adapter Worker 做成另一个权威任务系统，造成两套重试、授权与数据库。拒绝：它们是接口、逻辑服务或外部开发工具。

**X2：把 AI 文本当权限。** `USE: MJ`、模型自述、网页日志看似结构化，都不能直接改 Grant、Lease、Workflow Revision 或自动授权。拒绝：仅认证来源的候选提案进入 Core 策略门禁。

**X3：动态计划篡改已发布流程。** 运行时任意加节点、更换接收者、增加新权限而不审批。拒绝：固定 Revision 许可边界 + 每轮新 PlanRevision + Core 验证；超范围新审批。

**X4：无限循环等于无限重试。** 外部生成已提交、网络断开，随后 Brain 用新模型再次提交相同任务。拒绝：以外部业务效果为边界的对账与去重，Iteration 绝不能覆盖 Attempt 安全规则。

## 3. 风险登记册（候选）

| ID | 严重度 | 风险 | 建议约束和可复核证据 |
|---|---|---|---|
| R01 | P0 | Brain 旧 Epoch 迟到提案造成双派发 | Epoch/decisionId CAS 校验 + race 注入测试 |
| R02 | P0 | 执行结果未知却跨执行器重发 | idempotencyKey/Attempt/外部 Evidence；对账门禁 |
| R03 | P0 | API Brain 切换向未授权提供商披露文件 | Disclosure Grant 含 recipient/session/content scope；负面测试 |
| R04 | P0 | 人工接管后 Brain 自动恢复写入 | 恢复阻断独立持久化 + 人工批准记录 |
| R05 | P0 | 修改正在运行的 Revision 或能力契约 | Revision 哈希/版本引用，拒绝无审批变更 |
| R06 | P1 | 站点更新使模型选择错误 | Model Catalog 证据等级、有效期、验证机制，失败保守阻断 |
| R07 | P1 | 无限迭代造成 API 费用、额度或资源过度使用 | 硬预算、原子预留、周期上限、进展守卫、停止条件 |
| R08 | P1 | Web 连接只是发出点击却显示“已交付” | Delivery 证据等级、UNCONFIRMED、目标会话关联 |
| R09 | P1 | 对话摘要过时导致错误下一步 | Context Snapshot + journalSeq + delta，保留来源引用 |
| R10 | P1 | WorkBuddy 与 MJ 或不同任务冲突 GUI 焦点 | 焦点/会话/工作区/全局模型锁、运行前二次检查 |
| R11 | P1 | UI 动画越权代表外部运行事实 | 最后可信事件 + observedAt + Projection 只读 |
| R12 | P1 | 共享 Windows 用户权限被误认为 OS 沙箱 | 文档/UI 提示真实边界、可信模块策略与实测隔离 |
| R13 | P1 | 电脑休眠/关闭导致“持续运行”失效 | Core 恢复状态与补交付；不作后台物理持续保证 |
| R14 | P2 | 复杂自由画布阻碍第一条真实闭环 | 先步骤式/有限图，后高级画布、多分支、子流程 |

## 4. 尚待裁决的关键开放问题

| Issue | 问题 | 建议临时结论 | 冻结前必须有的证据 |
|---|---|---|---|
| OI-01 | Brain 提案是否使用独立网络协议？ | 业务层 Proposal + 认证 Connector；Core→Adapter 保留 LMP | Schema + trust boundary review |
| OI-02 | 同一个 Brain Session 多模型同时规划？ | 每个控制槽位只允许一个有效规划 Owner；Verifier 可独立只读 | 并发/旧 Epoch race test |
| OI-03 | Web GPT 原会话自动切回是否可靠？ | 未满足会话收发证据不切回 | E02 真实站点/轮次/送达实测 |
| OI-04 | API Key 存在哪里？ | 系统保护的本地密钥存储；不得写明文日志 | 平台实现威胁建模和恢复测试 |
| OI-05 | 模型真实身份能证实到哪级？ | 提供证据等级；达不到策略门槛则阻断/降级 | App/API 单独实测 |
| OI-06 | 动态 Plan 可以改变哪些路由？ | 固定可用能力与模型池；跨接收者需独立批准的 Disclosure Scope | Core Validator 测试 |
| OI-07 | 没有最大总轮数如何停滞处理？ | Budget + 限速 + stall watchdog + 明确终止规则 | 长跑 mock/成本闸门测试 |
| OI-08 | Context Snapshot 如何增量追平？ | checkpoint journalSeq + delta cursor + 原子 epoch cutover | 并发事件/中断测试 |
| OI-09 | Core/Event Journal 实际事务语义？ | 沿用 Outbox，ACK 仅证实单一事实 | E00-002/003/005 |
| OI-10 | App 无支持查询原任务时如何恢复？ | RECONCILING / 人工，不盲重试 | E03/E04/MJ 专项实测 |
| OI-11 | 首版画布复杂度 | 步骤/有限图优先，编辑器仅数据图的前端视图 | UI 原型用户验收 |
| OI-12 | Scheduler 是否首版上线？ | 暂不实施，预留 Trigger | 离线、时区、重叠及接管测试 |

## 5. 版本变化约束

任何与 F1 条款相抵触的需求→提出新的 ADR、影响评估和用户明确批准；**不得**静默改写 `f1-baseline-v1.0`。若是 F2 的实现字段争议→保持 OPEN，到 E00 模拟测试和 F2 单独裁决再冻结。新增 ARCH-06 概念自身要单独设立版本、兼容策略及专项验证，不能以“整体架构已讨论”为依据声称上线。
