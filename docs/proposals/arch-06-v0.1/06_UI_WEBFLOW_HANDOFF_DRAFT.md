# ARCH-06-06｜Webflow / UI 开发线交接草案（尚未发送）

**状态：HANDOFF_DRAFT_ONLY — 不创建、修改、克隆或发布现有 Webflow 页面。**

## 1. 设计目标

保持现有八页 P01–P08 和响应式 AppShell。把 ARCH-06 增量（Brain 对话、执行端/模型配置、能力连线、动态 Iteration、Brain 切换、长期任务监控）以**可单独验证的 UI 候选设计**融入，不重做现有已验收的 DS08 导航/键盘/迁移工作。

Webflow 用于视觉、响应式、静态 Mock 与交互原型；真正图编辑器/实时事件、权限、任务执行需要后续独立前端实现，不用虚构 Webflow 已具备 Core 后台。产品仍以 Windows 本机 loopback Core 为首版目标，移动端响应式 ≠ 远程跨网络访问。

## 2. 八页维持不变

| 页面 | ARCH-06 追加设计项 | 设计边界 |
|---|---|---|
| P01 总览 | 活跃 Run、待审批、Brain 所属、资源/预算警告 | 必须来源于 Core 投影或标记 Mock |
| P02 模块与连接 | Package/Runtime/Connection/Binding、能力目录、支持模型与验证等级 | 不把“模块已安装”写成“会话可用” |
| P03 工作区与工具 | 三轴 Grant、数据对外接收范围、Test Profile | 浏览器文件选择 ≠ Core 目录授权 |
| P04 工作流 | 对话 Copilot、步骤/画布、节点/端口、Inspector、发布预检 | Chat Draft ≠ 已批准 Revision |
| P05 任务中心 | Task Flow Monitor、Iteration、Brain 接管、Triple-State、Journal Timeline | UI 流断线 ≠ 外部 App 失败 |
| P06 审批与恢复 | 模型付费预算、Disclosure、人工接管恢复与对账 | Resume 必须用户显式确认 |
| P07 产物与审计 | TaskSpec/Verify/Artifact、证据来源、敏感过滤、模型变更历史 | 摘要不替代证据 |
| P08 设置 | Brain Providers/API Key 配置入口、Model Policy、资源策略、诊断 | 凭据原文不进入前端持久化/日志 |

## 3. P04 推荐交互路线

**Conversation**（用户描述）→ **Generated Draft**（GPT 提案）→ **Validator**（端口/权限/模型/预算/会话）→ **Step/Canvas Editor** → **Inspector**（Model/Connection/Parameters/Grants/Verification）→ **Publish Risk Review**（用户正式批准）→ **Run**（Core 权威确认）。

Desktop：左侧可收起 Brain 面板或模块库，中央主 Canvas/步骤视图，右侧 Node Inspector；避免默认把聊天栏/模块栏/属性栏全锁死导致 1024 宽度不可用。Tablet：Canvas 独立区域、Inspector 抽屉；Mobile：卡片式步骤列表和属性配置，不强求触屏自由拖线。

画布只呈现 Node/Port/Edge 的**草稿关系**；数据线与执行控制线在视觉上有区别，颜色不得为唯一语义；不可连接的端口应显示具体不兼容原因。

## 4. P05 推荐交互路线

Run Header（目标/状态/工作流 Revision） → Confirmed Node/Active Frontier → Iteration & Brain Ownership → 执行/交付/验证三轴 → 业务事件 Timeline → 展开诊断（Attempt/DeliveryAttempt/Message/journalSeq） → 证据和 Artifact。

关键文案：
- `最后确认的节点` 而非“绝对实时执行节点”。
- `结果已保存，尚未确认交付` 区分执行成功与送达。
- `主 Brain 不可用，备用 Brain 已接管` 需真实 Epoch/会话证据支持。
- `未知结果，等待对账` 不显示可一键盲重试副作用操作。
- `外部应用连接断开` 与 `Web Console 事件订阅断开` 分开显示。
- 若未来支持并行分支，使用 activeFrontier[]，不能只容纳一个 activeNode。

## 5. 必备 Mock 状态与风险行为

| 编号 | Mock 场景 | 期望表现 |
|---|---|---|
| M01 | 未安装模块 | 解释导入/来源审查入口 |
| M02 | 模块已安装但 Worker 未运行 | 分层状态，不显示 Connection Ready |
| M03 | Connection 可用但 Session PROVISIONAL | 自动写入按钮阻断 |
| M04 | Model Catalog UNKNOWN | 模型不可证实，不假定可用 |
| M05 | 免费模型不足，需授权付费 API | 明示预算与确认 |
| M06 | Brain API Key 缺失/失效 | 脱敏诊断，不展示明文 |
| M07 | 对话生成 Draft | 明示未发布、未执行 |
| M08 | 端口类型不匹配 | 连线阻断并说明原因 |
| M09 | 无 LOCAL_READ | Reader 受保护操作阻断 |
| M10 | 无 EXTERNAL_DISCLOSURE | 向 GPT/MJ/API 传输阻断 |
| M11 | 无 LOCAL_EXECUTE | 写入/Test Profile 阻断 |
| M12 | Brain 失联 / 备用模型接管 | 权威切换事件与当前 Owner 区分 |
| M13 | 原 GPT 恢复候选但尚未安全交接 | 保持备用 Brain 为 Owner |
| M14 | OUTCOME_UNKNOWN | 对账而不是自动重试 |
| M15 | 人工接管后对账通过但未确认恢复 | 明确“等待用户恢复确认” |
| M16 | Result Stored / Delivery Pending / Verification Failed | 三轴不合并显示 DONE |
| M17 | UI Event Stream 断开 | 展示最后可信序号与重连中提示 |
| M18 | 20 轮长期迭代但预算或停滞阻断 | 显示历史迭代与停止原因 |

注：M01–M18 是新版 UI **建议样例**，与旧扩展包 UI-EXP-M01～M16 不同，不能宣称旧 UI 用例已验收。

## 6. 响应式与质量门槛

至少核验 1440 / 1024 / 768 / 390 / 320 px 的布局及键盘导航；不能因为手机页面窄就隐藏必须处理的审批和紧急暂停入口。需要考虑标签、按钮、抽屉焦点、表格横向滚动、长中文文案、日志截断/复制及视觉状态不只依赖颜色。

## 7. 当前 Webflow 进度保护

已有另一 UI 对话正在推进 DS08 / P08 候选迁移与桌面导航验证。**本草案不推断该分支最终验收状态，不覆盖已有页面、不重新计数既有通过项、不发布站点**。真正交接前应先读取其最新验收日志和页面结构；在独立 Draft 设计 P04/P05 新交互，再做差距比对和候选迁移。

## 8. UI 交付定义（未来审查时）

- [ ] 现有设计系统/DS08 状态盘点，不复位已验收事项。
- [ ] P04 对话、步骤/画布、Inspector 与发布预检的 IA / 状态流。
- [ ] P05 与 P04 共用 Node 身份的运行证据视图与两层 Timeline。
- [ ] P02 模型/连接状态、P06 预算与恢复、P08 Provider 配置的候选界面。
- [ ] M01–M18 演示样例（全部标为 Mock）。
- [ ] 五档响应式、键盘/抽屉/焦点/长日志验证。
- [ ] 与 COD 共享字段一致性表；不在 UI 中发明已被 Core 确认的执行事实。
- [ ] 无正式页面变更/发布，等待专门的设计实施批准。

**停止点**：UI 草稿准备完成，不构成 Webflow MCP 写入授权。
