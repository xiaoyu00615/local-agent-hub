# ARCH-06-07｜决策台账、待确认事项与最终交接检查

## 1. 已有正式冻结文件与本次产品方向的区别

| 编号 | 决策 | 级别 | 依据 |
|---|---|---|---|
| BASE-01 | Core 权威、F1 12 条不变量 | APPROVED_F1 | F1 已批准并 Git Tag |
| BASE-02 | F2 LMP/SDK | CANDIDATE / NOT RUN | E00 尚未验证，不可声称冻结 |
| PROD-01 | Local Agent Hub 独立开发，Relay 不进入产品运行架构 | USER_DIRECTION_CONFIRMED | 本轮用户明确说明 Relay 仅辅助开发 |
| PROD-02 | 统一多模态模块体系；App 内部独立专用 Driver | USER_DIRECTION_CONFIRMED | 用户确认 MJ 等属于执行模块、接口可统一 |
| PROD-03 | GPT 在授权模型范围内自主选择（方案 B） | USER_DIRECTION_CONFIRMED | 用户明确选择 B |
| PROD-04 | 免费模型优先，可引入授权 API Key | USER_DIRECTION_CONFIRMED | 用户明确偏好 |
| PROD-05 | 主 Brain 不可用，自动切换已授权备用 API Brain（方案 B） | USER_DIRECTION_CONFIRMED | 用户明确选择 B |
| PROD-06 | 原 Brain 在安全检查点、身份可证且权限有效时尝试自动切回（方案 B） | USER_DIRECTION_CONFIRMED | 用户明确选择 B |
| PROD-07 | 长期任务允许不预设固定总迭代轮数 | USER_DIRECTION_CONFIRMED | 用户明确需求 |
| PROD-08 | 对话生成 + 可视化画布 + 简单任务直接进入同一 Core | USER_DIRECTION_CONFIRMED（方向） | 用户同意按建议推进 |
| ARCH-01 | 认证 Brain TaskProposal，不信任纯 `USE` 字头 | RECOMMENDED | 结合 F1-05 提案与授权分离 |
| ARCH-02 | Model Catalog + Policy + Evidence + Safe Fallback | RECOMMENDED | 模型能力证据尚待实机验证 |
| ARCH-03 | 固定 Revision + 有约束的 DynamicPlan/Iteration | RECOMMENDED | 保证 F1-10 |
| ARCH-04 | Brain Planning Epoch + 安全交接 Checkpoint | RECOMMENDED | 需要并发/会话实测 |
| ARCH-05 | Journal/Outbox/Artifact/Checkpoint + Memory Source Index | RECOMMENDED | 需持久化与断线测试 |
| ARCH-06 | P04/P05 共用模型，先有限步骤/图，后自由画布 | RECOMMENDED | UI/工程共同对齐 |

用户批准产品方向不等于批准所有具体表结构、协议字段或无人值守执行。新的正式架构批准应在 Open Issues 收敛、F1 兼容核查及必要的 ADR 审查之后进行。

## 2. 当前不需要向用户反复询问的细节

Model Catalog 内部存储字段、UI 是否使用 React Flow 一类图库、Checkpoint 间隔、日志压缩算法、NodeJS/TypeScript 工具链版本、Brain 健康窗口和 Scheduler 的具体格式。这些可以由工程负责人提出候选并通过 Mock 实验测定，而不是要求用户现在拍板。

## 3. 真正需要用户以后明确授权的事项

- 允许开始 E00-001 工程编码和模拟测试。
- 允许向 COD 交付正式技术评审任务。
- 允许向 UI/Webflow 交付正式设计评审/修改任务（区分只评审与实机改页）。
- 允许具体 App/API 的真实接入、凭据配置、工作区读取、对外披露、执行副作用。
- 允许长期无人值守策略具体生效，并确定费用预算/资源范围/可使用提供商。
- 批准需要改变 F1 的新版 ADR、或测试完成后单独批准 F2 冻结。

## 4. 一次性交接前的统一审核清单

- [x] 独立产品 vs Relay 辅助开发边界书面澄清。
- [x] ARCH-01～05 统一术语/架构图。
- [x] F1 十二条原则逐项兼容设计检查。
- [x] F2/E00 尚未冻结与未运行状态明确标记。
- [x] COD/UI 待交接草案分别编写。
- [x] 早期有界轮次与现已确认的“无限总轮次”需求合并为受控迭代策略。
- [ ] 用户确认本包的统一架构设计方向。
- [ ] 关键 Open Issues 有明确负责人和验证方案。
- [ ] 新版本文档经仓库评审流程提交（当前未提交）。
- [ ] COD 实际签收（当前未通知）。
- [ ] UI 实际签收（当前未通知）。
- [ ] COD/UI 共享字段字典交叉验收。
- [ ] 对应工程和 Webflow 修改获得独立授权。
- [ ] 新增专项测试实际执行并提供证据。

## 5. 交接前的防误解声明

1. `ARCH-06 完成` 意味着**本轮评审材料形成**，不等于所有结构已最终冻结。
2. `用户选择方案 B` 是产品需求决策，不是 ChatGPT 网页会话自动切换能力已实现。
3. `允许长期迭代` 不是无授权调用无限费用 API、任意 Shell 或反复制造副作用的许可。
4. `画布拖线` 是编辑请求，尚未通过 Core 验证/审批时不能当作真实任务执行。
5. `Bridge/Connector` 只是 Local Agent Hub 自己的模块组件，不是旧 Relay 的依赖重命名。
