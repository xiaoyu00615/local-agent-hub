# Local Agent Hub｜项目规范文档总入口

**适用仓库：** `D:\Workspace\projects\local-agent-hub`（建议路径，实际以你的项目目录为准）  
**资料范围：** V1.1–V1.4 已实际生成的 28 份原始 Markdown 文档，另附 4 份整合索引/交接说明及文件校验清单。  
**整理时间：** 2026-10-08  
**状态：** 规划/评审资料，不包含产品源代码或已通过的实机测试结果。

## 一分钟开始

1. 阅读 [`00_BASELINE_AND_STATUS.md`](00_BASELINE_AND_STATUS.md)：理解哪些决定已确认、哪些还没有冻结。
2. 阅读 [`01_DOCUMENT_INDEX.md`](01_DOCUMENT_INDEX.md)：按主题找到产品、LMP、SDK、权限、工作流和验收文档。
3. 阅读 [`02_ENGINEERING_HANDOFF.md`](02_ENGINEERING_HANDOFF.md)：查看新仓库初始化、F1 审批和 E00 起步建议。
4. 阅读 [`03_DECISION_TRACEABILITY.md`](03_DECISION_TRACEABILITY.md)：查看跨版本继承、覆盖和仍待批准的事项。

## 版本文档（完整原文）

- [`versions/v1.1/`](versions/v1.1/) — PRD、系统架构、早期 LMP/会话/权限、工作流与验收（10 份）。
- [`versions/v1.2/`](versions/v1.2/) — P0 技术可行性与冻结评审（2 份）。
- [`versions/v1.3/`](versions/v1.3/) — E00–E07 验证任务、冻结清单、证据与门禁（7 份）。
- [`versions/v1.4/`](versions/v1.4/) — 最新的 LMP V1/Adapter SDK 候选契约与 E00 测试（9 份）。

**优先级提示：** 产品级用户明确确认事项优先；就 LMP/SDK 具体契约而言，以 **V1.4 候选规范** 为主，以 V1.3 的测试/冻结要求校验，以 V1.2 评审限制为边界，V1.1 相应协议作为设计历史。任何版本冲突都要写 ADR，不要直接混用字段。

## 重要说明

- **CONFIRMED**：已由用户明确选择/认可的产品级方向。
- **F1 CANDIDATE**：待正式批准的架构与安全不变量。
- **F2 CANDIDATE**：待 E00 模拟测试及逐项通过后才能冻结的实现契约。
- **NOT RUN**：尚未实际执行测试。文档中的“通过标准”不是“已通过”。
- 本包不包含重复的 `V1.4-Complete-Contract.md` 合订本及 V1.1–V1.4 原始 ZIP，因为它们与分册内容重复。本包保留每一份原始 Markdown 分册及原始版本 README。
- V0.1–V1.0 的讨论历史已由 V1.1 PRD 归纳，但目前没有独立生成并核实的 V0.x 源文件，不应误认为本包包含全部早期聊天记录。
- `SHA256SUMS.txt` 提供内容指纹；迁入仓库后仍可通过 Git 追踪修改。

## 放入项目的方式

此 ZIP 的顶层只有 `docs/`。将 ZIP 解压到 **`D:\Workspace\projects\local-agent-hub\`**，即可得到 `...\local-agent-hub\docs\README.md`。不要解压到已有的 `docs\` 内，否则可能形成 `docs\docs\`。
