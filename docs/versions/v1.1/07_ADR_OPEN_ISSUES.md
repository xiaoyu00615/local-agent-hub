# 07｜架构决策日志与未决问题 V1.1

**日期：2026-10-08；本表区分已确认需求与待评审建议**

## A. 已确认决策
- ADR-C01：首版为 Windows 本机、单用户、localhost Web Console（CONFIRMED）。
- ADR-C02：网页 AI ↔ 桌面应用作为样板；没有公开 API 的 GUI 驱动是一等接入路径（CONFIRMED）。
- ADR-C03：人工可介入，冲突后暂停核对；人工接管结束必须用户明确确认恢复（CONFIRMED）。
- ADR-C04：模块交付方式 B：内置模块 + 本地开发者模块（CONFIRMED）。
- ADR-C05：本地文件只读、受控终端和测试首版可用；网页 AI 可请求授权文件查询（CONFIRMED）。
- ADR-C06：Obsidian 作为后续目标模块（CONFIRMED）。

## B. 建议但尚未冻结的选择
- ADR-P01：模块化单体 Core + 独立 Adapter Worker（PROPOSED）。
- ADR-P02：LMP V1 内部协议及统一消息模型（PROPOSED）。
- ADR-P03：HTTP/WebSocket/IPC、SQLite + Artifact Store（PROPOSED）。
- ADR-P04：TypeScript SDK 优先，跨语言协议（PROPOSED）。
- ADR-P05：步骤式工作流编辑器、版本化节点图（PROPOSED）。
- ADR-P06：受信任本地开发者模块，第三方模块市场延后（PROPOSED）。

## C. 跨模块审查问题清单
| ID | 优先级 | 问题 | 初步处置 | 状态 |
| --- | --- | --- | --- | --- |
| R-01 | P0 | 普通网页 AI 是否能可靠产出工具请求、接收结果及识别轮次？ | 分 Native Tool / Conversation Bridge；按目标站点实测、权限显式控制 | BLOCKER |
| R-02 | P0 | CMD/Test 的真实文件/网络隔离如何强制执行？ | 不承诺目录白名单构成沙箱；先限定已信任测试方案，敏感执行要隔离/审批 | BLOCKER |
| R-03 | P0 | GUI Agent 发送/完成/取消判据是否可辨识？ | 证据阶段拆开，未知状态冻结重发，按真实应用试验 | BLOCKER |
| R-04 | P0 | 模块状态、连接状态、会话控制权、任务状态是否混淆？ | 独立状态机和 ID 模型，强制评审状态转换 | OPEN |
| R-05 | P0 | 本机文件读取与向网页 AI 外传是否独立授权？ | Local Read/Disclosure/Execute 维度分离 | OPEN |
| R-06 | P1 | Capability、Command、Event 命名不一 | 建统一名称注册表与 Schema 版本 | OPEN |
| R-07 | P1 | 第三方模块实际权限无法由 Manifest 充分约束 | 明确可信来源；OS 级隔离另设验收，不宣称默认安全 | OPEN |
| R-08 | P1 | 工作流已运行时模块升级及能力变化 | 固定 Revision、兼容性检查与版本并存 | OPEN |
| R-09 | P1 | 文件搜索是否能阻止 Windows Junction/TOCTOU 越界 | 以实际对象+工作区策略验证，进行专项攻击测试 | OPEN |
| R-10 | P1 | 任务与结果 ACK 到底确认哪些事实 | 统一阶段语义；将执行、交付、验收分开 | OPEN |
| R-11 | P1 | 取消与强制停止是否可真实生效 | 定义请求取消 vs 已确认取消，显示不确定状态 | OPEN |
| R-12 | P2 | Obsidian、Blender 等新模块的专有 UI | 默认 Schema 驱动配置；复杂 UI 后置 | BACKLOG |

## D. 推荐的关键技术试验
1. 对一个具体网页 AI 页面测试：请求识别、绑定正确、文件片段回传、会话切换与人工输入冲突。
2. 对 WorkBuddy GUI 测试：已发送/运行/结果/取消等能否被可靠识别；用户介入触发何种证据。
3. 在 Windows 上做受控测试执行隔离试验：确认允许和禁止的文件/网络访问是否真实受限。
4. 模拟 Core 崩溃发生于外部任务提交前、中、后，验证没有盲目重发。

**待用户确认的实质性产品问题**：第一版允许哪些网页 AI 站点、首版终端执行信任级别，以及是否将“自动化会话优先独立于人工会话”设置为默认推荐但不强制。这些选择不影响已确认的人工恢复规则。
