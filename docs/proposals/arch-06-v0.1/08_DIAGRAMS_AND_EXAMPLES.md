# ARCH-06-08｜系统脑图、任务链与示例

图为架构候选，非产品运行截图；示例不需要接触真实 App、账号或文件。

## 1. 架构脑图

```mermaid
mindmap
  root((Local Agent Hub))
    User Experience
      Web Console P01-P08
      P04 Workflow Design
      P05 Task Flow Monitor
    Brain
      Web Connector
      API Connector
      Local Connector
      Session Ownership
      Free-first Model Router
      Context and Memory
    Local Core
      Capability Registry
      Policy Grants
      Ownership and Locks
      Durable Workflow Engine
      Task Attempt Outbox
      Journal Checkpoint
    Modules
      LMP Host
      WorkBuddy Driver
      DSH Driver
      MJ Driver
      Future App Drivers
    Verification
      Artifact Evidence
      Test Profile
      Delivery Confirmation
      Business Acceptance
    Governance
      F1 Frozen
      F2 E00 Candidate
      Adapter Actual Testing
      Review and ADR
```

## 2. 任务链 A：简单操作

```mermaid
flowchart LR
  B["Brain 提出只读目标"] --> C["Core 可信提案/Grant/Schema"]
  C --> T["Reader/Tool Adapter"]
  T --> E["保存证据"]
  E --> D["检查外传权限并交付原会话"]
```

## 3. 任务链 B：媒体生成与审查

```mermaid
flowchart TD
  P["Brain 提示词规划"] --> M["Model Router/选择已授权生成模型"]
  M --> J["MJ 专用 Adapter 生成"]
  J --> A["ImageSet ArtifactRefs"]
  A --> V["Vision / 人工验收"]
  V --> C{"质量达成且证据足够?"}
  C -->|Yes| D["交付资产"]
  C -->|No / 已确认无未知副作用| B["候选修订方案"]
  B --> S{"策略/预算/授权允许下一轮?"}
  S -->|Yes| P
  S -->|No| H["等待/暂停"]
```

## 4. 任务链 C：长期开发、主 Brain 失联

```mermaid
sequenceDiagram
  participant U as User
  participant H as Hub Core
  participant P as Primary GPT
  participant B as Backup API Brain
  participant W as WorkBuddy
  participant T as Test Runner
  U->>H: 批准目标/授权/预算策略
  H->>P: 请求规划
  P-->>H: 结构化 PlanProposal
  H->>W: 提交经验证 Task
  W-->>H: 工作结果与 Evidence
  H->>T: 按 Test Profile 验证
  T-->>H: 验证失败及证据
  Note over P: 原 Brain 不可用
  H->>H: 保存 Checkpoint/更新规划 Epoch
  H->>B: 检查 Disclosure 并发送精简上下文
  B-->>H: 当前 Epoch 的下一轮提案
  H->>H: 检查 Revision、Budget、Grants
  H->>W: 仅派发新的安全业务 Task
  Note over H: 若外部结果未知则暂停对账，不重新提交
```

## 5. 最小产品演示“完成”标准

- 能够区分 Proposal/Draft/Published Revision/Run，不以 GPT 文本触发未经授权的操作。
- 能显示明确绑定的模块能力、Connection、Session 和模型状态；未知状态不冒充 READY。
- 简单任务和线性流程经过同一 Core 权威 Task、Attempt、Journal。
- 运行结果有独立 Execution、Storage、Delivery、Verification 证据。
- 浏览器断线后，P05 能通过 Snapshot + journalSeq 展示可核实的最后状态。
- 结果未知、权限撤销、人工接管时有正确 Block/Approval/Recovery 体验。
- 以上均仅在对应测试真实通过后才能标记 PASS，未实现前只可演示 Mock。
