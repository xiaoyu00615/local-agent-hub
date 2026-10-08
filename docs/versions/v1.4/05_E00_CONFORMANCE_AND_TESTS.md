# 05｜E00 模拟 Adapter 规范符合性测试与证据门槛

**状态：TEST DESIGN / NOT RUN。** 此文件不是测试结果。

## 1. 目标与环境

E00 在**独立新项目、无真实外部应用、无真实用户文件**的模拟环境中验证 LMP 和 SDK 最小契约。

最小模拟组件：Mock Core/Host、Mock Agent Adapter、Mock Reader Adapter、Faulty Adapter、事件日志/模拟持久化、可注入断连/崩溃/延迟的传输层。

不操作真实 WorkBuddy、DSH、浏览器账号、真实仓库或 CMD。使用无害虚构资源和合成状态。

## 2. 测试矩阵

| Case ID | 前置及刺激 | 必须观察到的断言 |
|---|---|---|
| E00-ID-01 | Package/Runtime/Connection/Binding/Task 创建 | 全部获得独立 ID，绑定关系可追踪 |
| E00-HS-01 | 正确版本与来源的 Mock Adapter 握手 | 建立经认证 endpoint 与协商版本 |
| E00-HS-02 | 错误凭据/身份/来源 | 握手拒绝，未注册能力 |
| E00-HS-03 | 不支持的 LMP major | 阻止连接，不盲目降级 |
| E00-MSG-01 | 缺失必需字段、错误 payload Schema | 拒绝并生成可审计错误 |
| E00-MSG-02 | 未知控制字段/未受支持能力 | 拒绝执行，不能将其当文本转发成命令 |
| E00-MSG-03 | 时间戳倒置、迟到 EVENT | Core 以 journalSeq 和因果证据保证不回退终态 |
| E00-ACK-01 | Adapter `ack.command.accepted` | 任务不能因此显示已执行成功 |
| E00-ACK-02 | RESULT 到达但 Core 保存失败 | 不产生 `ack.result.stored` |
| E00-ACK-03 | Core 保存成功但接收者离线 | 结果为已保存、未交付，不标记已验收 |
| E00-ACK-04 | 浏览器模式仅有“发送动作完成” | 无法证明目标接收则 DELIVERY_UNCONFIRMED |
| E00-DUP-01 | 同 messageId 重送 | 不新增业务执行；允许确认重传 |
| E00-DUP-02 | 不同 messageId、同 idempotencyKey | 不创建重复外部副作用动作 |
| E00-DUP-03 | 同业务键不同实质输入 | 标记 `DUPLICATE_CONFLICT`，不静默接受 |
| E00-OWN-01 | 旧 bindingGeneration | 自动写入被拒绝 |
| E00-OWN-02 | 旧 ownershipEpoch/过期 Lease | 自动写入被拒绝 |
| E00-OWN-03 | 人工接管→对账成功，无用户恢复审批 | 仍不能自动写入 |
| E00-OWN-04 | 用户恢复审批后新的 Epoch/Lease | 只允许有效授权下的新操作 |
| E00-PERM-01 | 有 Local Read，无 Disclosure | 可本地读取，不能外传 |
| E00-PERM-02 | 撤销 Execute Grant 但保留消息引用 | 新执行被拒绝 |
| E00-DB-01 | Core 保存派发意图后、发送前崩溃 | 可以证明未提交外部时安全恢复 |
| E00-DB-02 | 派发已开始但 ACK 未持久化时崩溃 | 进入核对，不盲目重新执行 |
| E00-REC-01 | 已提交、外部仍运行 | 只恢复观察，不再 submit |
| E00-REC-02 | 已保存结果、下游未交付 | 补交付，不再调用 Agent |
| E00-REC-03 | 结果/进度乱序、终态冲突 | 冲突留证并进入 RECONCILING |
| E00-LIFE-01 | READY 模块 + LOST 会话 | 不派发会话写入任务 |
| E00-LIFE-02 | DRAINING 时新任务 | 新任务拒绝/排队，不接受新副作用 |
| E00-LIFE-03 | Adapter 崩溃/被卸载 | Core 继续可用，历史数据保留 |
| E00-SDK-01 | 不支持 optional Capability | 返回 UNSUPPORTED，不伪造成功 |
| E00-SDK-02 | 有效能力但输入 Schema 违规 | 拒绝，未调用外部 Driver |
| E00-SDK-03 | Module 更改 Capability 版本影响 Run | 正在运行 Revision 不被静默改写 |

## 3. 通过标准

- 每项必须提供 caseId、预期、实际、脱敏证据、权威 journalSeq、Run/Task/Attempt 及结果 `PASS/FAIL/BLOCKED`。
- 所有列为强制的 E00 测试通过才能宣布 LMP **MVP F2** 冻结。
- P0 安全违例（越权、错会话、未经人工确认恢复、未知副作用盲目重试）发生一次即整体阻断对应 Gate。
- E00 全通过**不代表** E01 文件边界、E02 网页桥接、E03 WorkBuddy、E04 DSH 或 E05 终端执行已验收。

## 4. 建议证据模板

| 字段 | 要求 |
|---|---|
| `caseId` / `testRunId` | 能唯一识别试验及复测 |
| `environmentFingerprint` | OS、Core、Host、Mock Adapter、协议、Schema 版本 |
| `stimulus` | 引发的动作/错误注入阶段 |
| `expected` / `actual` | 可观察断言与真实结果 |
| `journalTrace` | 核心事实顺序与必要 ID，不记录真实密钥 |
| `sideEffectAssessment` | NONE / CONFIRMED / UNKNOWN |
| `result` | PASS / FAIL / BLOCKED / NOT_RUN |
| `defectId` | 失败时必需，追踪修复和复测 |

## 5. Gate 依赖

E00 → E01 文件权限 → E02 网页只读桥接；E00 → E03 GUI 和 E04 DSH；E00 + E01 → E05 可信测试；所涉 Gate 通过后 → E06 恢复 → E07 UX。

真实应用尚未纳入时，允许先通过 E00 的模拟契约验证，但**不能将模拟结果作为真实 Agent 可用证据**。
