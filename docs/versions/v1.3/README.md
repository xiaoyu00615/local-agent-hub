# Local Agent Hub V1.3｜最小技术验证与实施前冻结包

- 版本：V1.3 REVIEW DRAFT 1
- 日期：2026-10-08
- 类型：产品/协议与工程验证计划；**不包含产品实现代码**
- 依赖：V1.1 产品总规范、V1.2 P0 技术评审
- 测试状态：**全部 NOT RUN**。技术资料核实不等于本机实验通过。

## 阅读顺序
1. `00_EXECUTIVE_DECISION.md`：产品基线、范围、准入结论
2. `01_VALIDATION_TASK_CARDS.md`：E00–E07 验证任务书
3. `02_PROTOCOL_FREEZE_CHECKLIST.md`：冻结前具体检查项与状态
4. `03_TEST_EVIDENCE_REQUIREMENTS.md`：环境、数据、留证模板和安全约束
5. `04_GATES_AND_FALLBACKS.md`：Gate、风险处置、降级决策及开工入口
6. `05_CONTRACT_ALIGNMENT.md`：对象、状态、确认与版本统一检查

## 状态词
- CONFIRMED：用户已明确批准的需求/策略。
- PROPOSED F1：建议冻结的架构不变量，尚未获得冻结批准。
- CANDIDATE F2：实验成功后才能冻结的具体实现契约。
- BLOCKED：因能力/环境条件缺失暂不能验证；**不是通过**。
- PASS/FAIL/NOT RUN：必须有有效实验记录与证据方可赋予 PASS/FAIL。

## 用户已确认
个人 Windows 本机+localhost Web 控制台；网页 AI↔桌面 Agent 首个样板；无 API 的 GUI 接入是正式路线；Agent 能力声明；人工接管后暂停对账且恢复写入必须用户明确确认；内置模块+本地开发者模块（B）；Workspace Reader/受控终端/Test Runner 为首版能力；Obsidian 为后续模块。

## 不可混淆
项目尚未正式开工；本文列出的实验步骤是未来实施阶段的任务书。不能把“协议支持”“模拟测试通过”误认为“WorkBuddy/Chrome/DSH 真机端到端通过”。
