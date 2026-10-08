# 03｜验证环境、证据模板与安全试验边界

**目的**：让每次试验可重复、可归属、可审查，不让误判通过混入后续协议冻结。

## 1. 环境记录（必填）
- Windows 版本、版本号/架构、RAM；虚拟化和安全隔离能力（仅识别，不自动启用）。
- 浏览器、扩展及自动化 Driver 版本；Profile 身份但不保存敏感路径/登录 Cookie。
- Core、Module Host、Adapter SDK/Driver、LMP 协议和 Schema 版本。
- WorkBuddy / DSH 具体版本与允许的连接方式；外部账号可用能力（不记录凭据）。
- 测试 Workspace 独立根目录、权限范围、快照版本、需要的工具和 Test Profile。
- 网络权限、数据对外传输目的地及授权记录 ID。

## 2. 合成样本集（不包含真实秘密）
1. `samples/`：2–5 个小型 Markdown 和源码文件，包含可唯一定位的词句与不同编码。
2. `long/`：大文本文件，验证读取范围、截断、页码/行号。
3. `excluded/`：模拟敏感命名，例如 `.env`，文件中仅放无效占位内容。
4. `links/`：只用于未来受控测试的安全目录链接，确保工作区外测试目标是无害文件。
5. `agents/`：独立试验会话，所有请求含可区分的 Test Turn ID 与期望回声数据。
6. `tool-profiles/`：无害受限测试方案，明确预期生成的临时产物。

## 3. 单次实验记录模板
| 字段 | 应填内容 |
|---|---|
| caseId | 如 E02-N-001；同一案例重试分 attempt |
| purpose | 要证明的唯一断言 |
| preconditions | 环境/连接/Grant 与初始外部状态 |
| action | 由用户/测试系统在何阶段触发什么操作 |
| expected | 明确可观察事实，含硬性禁止项 |
| actual | 实测的事实，不写推测性结论 |
| evidence | 脱敏日志、时间线、目标身份、摘要或测试产物引用 |
| sideEffect | 无 / 已知且受控 / 可能发生但未知 |
| result | PASS / FAIL / BLOCKED / NOT RUN |
| defect | 关联缺陷 ID、隔离条件与复测要求 |

## 4. 工具与会话事件至少保留
- `messageId`、`correlationId`、`runId`、`taskId`、`attemptId`（适用时）。
- Binding 身份证据、Generation、Epoch、Grant 版本或授权引用，不记录凭据明文。
- 传输与操作各阶段的观察证据、客户端/服务端时间顺序、核对结论。
- 文件读取范围、相对路径和内容指纹；不存放不必要的文件原文。
- 跨网页返回的接收方会话确认；明确是否只证明“已输入”而非对端实际接收。
- Test Profile 的退出码、耗时、stdout/stderr 脱敏摘要、明确的输出截断信息。

## 5. 故障注入安全约束
- 实验前备份样本数据，默认不接触真实工作项目和任何真实密钥。
- 不在日常工作会话里做消息错投与重试实验；使用新建或明确标记的测试会话。
- 测试权限撤销应在本地模拟数据上进行；不得测试真实凭据泄露。
- 不用危险命令来证明 Shell 拒绝：模拟请求、故意无效能力名、明确不可执行的占位目标足够。
- 对无强隔离的 Test Runner，优先选择可信测试脚本；不拿真实未知程序进行风险实验。
- 所有截图/日志先脱敏账号、绝对路径、工作区秘密和会话内容。

## 6. 测量规则
- “零违规”指该批已覆盖案例内未观察到违规，不等于现实世界零概率。
- 运行成功率与安全违规率分别计算；关键安全故障出现一次即触发 FAIL。
- 区分测试故障导致的工具不可用，与错误授权导致的安全失败；不能靠降低测试覆盖率获取 PASS。
- 某个 Driver 无法证明能力时标记 UNSUPPORTED 或 BLOCKED，不补造证据。
- 资源与性能基线先测得后定阈值，8GB Windows 是目标环境，不构成性能达标的推断。

## 7. 官方资料核实基线
- Chrome 扩展 MV3 Service Worker 生命周期与 WebSocket 行为：https://developer.chrome.com/docs/extensions/develop/concepts/service-workers/lifecycle
- Windows UIA 事件不能单独证明用户操作或真实状态变化：https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-eventsoverview
- Job Objects 管理进程与资源，不是文件沙箱：https://learn.microsoft.com/en-us/windows/win32/procthread/job-objects
- AppContainer 资源隔离能力：https://learn.microsoft.com/en-us/windows/win32/secauthz/appcontainer-isolation
- DSH ACP 具体能力：https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/acp/acp/README.md
- ChatGPT 自定义 MCP / 隧道接入前提：https://developers.openai.com/api/docs/guides/secure-mcp-tunnels
