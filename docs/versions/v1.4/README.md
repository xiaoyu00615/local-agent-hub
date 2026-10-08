# Local Agent Hub V1.4 — LMP V1 与 Adapter SDK 最小契约

**规范状态：REVIEWED CANDIDATE / 待用户 F1 审批与 E00 实测；不是已通过验收的正式版本。**  
日期：2026-10-08　|　适用：Windows 单用户、本地 Core + 独立 Adapter Worker　|　范围：协议与设计文档，不含产品实现代码。

## 阅读顺序

1. `00_FREEZE_AND_DECISIONS.md` — 已确认需求、建议冻结项、阻断项及 ADR。
2. `01_LMP_MESSAGE_CONTRACT.md` — LMP V1 的身份、消息、请求关联、版本与传输边界。
3. `02_TASK_ACK_AND_RECOVERY.md` — ACK、三轴状态、持久化、幂等及恢复契约。
4. `03_ADAPTER_SDK_AND_LIFECYCLE.md` — Manifest、SDK 回调、能力、生命周期与模块安全。
5. `04_PERMISSION_SESSION_CONTRACT.md` — 授权、会话、所有权、工具与网页回传契约。
6. `05_E00_CONFORMANCE_AND_TESTS.md` — E00 模拟验证用例及各断言。
7. `06_COMPATIBILITY_AND_OPEN_ISSUES.md` — 兼容规则、仍需实机验证的问题。
8. `07_IMPLEMENTATION_HANDOFF.md` — 独立新项目开工边界与建议任务包。

## 术语

- **MUST / 必须**：实现此候选协议所需的强制要求；是否能宣布 F2 冻结，仍取决于验证。
- **SHOULD / 建议**：推荐设计，允许有书面理由的例外。
- **MAY / 可选**：非最低适配要求。
- **P0**：不通过便不得宣称相应自动化能力已验收。

## 权威原则

Core 是 Run/Task/Attempt、Grant、Ownership、审计与恢复决策的权威来源。Adapter 可报告外部事实与证据，但不能自己授予权限、宣布业务验收或在人工接管后自行恢复写入。独立 Worker 不构成 Windows OS 强隔离。

## 外部规范参考

- JSON Schema Draft 2020-12：https://json-schema.org/draft/2020-12
- Node.js child_process / IPC：https://nodejs.org/api/child_process.html
- Chrome Extension 生命周期：https://developer.chrome.com/docs/extensions/develop/concepts/service-workers/lifecycle
- Windows UIA 事件：https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-eventsoverview

以上第三方规范用于技术选型与兼容映射，**LMP 不是 MCP/ACP/JSON-RPC 的别名**。
