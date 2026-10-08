# 02｜独立项目入库与 E00 前的交接说明

## 本包应放在哪里

推荐仓库：`D:\Workspace\projects\local-agent-hub`

解压之后应为：

```text
local-agent-hub/
└─ docs/
   ├─ README.md
   ├─ 00_BASELINE_AND_STATUS.md
   ├─ 01_DOCUMENT_INDEX.md
   ├─ 02_ENGINEERING_HANDOFF.md
   ├─ 03_DECISION_TRACEABILITY.md
   ├─ SHA256SUMS.txt
   └─ versions/
      ├─ v1.1/  (10 份 Markdown)
      ├─ v1.2/  (2 份 Markdown)
      ├─ v1.3/  (7 份 Markdown)
      └─ v1.4/  (9 份 Markdown)
```

`docs/` 内只有规范、索引和校验文件，没有可执行程序，也不会修改你的电脑或其他仓库。

## 首次入库前检查

1. 核对新仓库根目录与 Git 工作区，确保不是其他项目的子仓库。
2. 仅解压这个包到项目根目录；不要再重复导入 V1.1–V1.4 的旧 ZIP。
3. 打开 [`README.md`](README.md) 和 [`00_BASELINE_AND_STATUS.md`](00_BASELINE_AND_STATUS.md)。
4. 检查并确认“用户已确认 / F1 候选 / F2 候选 / NOT RUN”的区分正确。
5. 将规范作为首次 Git 提交的文档部分；不要提交密钥、生产数据、个人文件或本地日志。

## 推荐的下一步工程阶段

**Gate F1：** 审阅并批准基础架构、安全、会话与恢复不变量；逐条记录需要调整的条款。

**Gate E00：** 在新仓库中建立最小 LMP Schema、SDK 模拟 Host/Adapter、协议一致性测试与故障注入；E00 用例见 [V1.4 E00](versions/v1.4/05_E00_CONFORMANCE_AND_TESTS.md)。

**Gate F2：** E00 实测证据通过后，才冻结具体消息字段、状态转换、兼容规则与 SDK 行为。

**E01～E07：** 依次验证本地文件、网页桥接、GUI、DSH、终端测试、恢复和 Web Console，详见 [V1.3 验证任务](versions/v1.3/01_VALIDATION_TASK_CARDS.md)。

## 任何执行者必须知道的限制

- 当前没有在新仓库中实际执行 E00–E07。
- 这些规范是候选开发契约，不能宣称真实 Adapter 已验证通过。
- 优先从模拟器和合成数据开展验证；未经明确授权，不连接或修改既有项目。
- 若文档冲突，保留历史原文，通过 ADR 决策解决，不要静默删除原则或扩大权限。
