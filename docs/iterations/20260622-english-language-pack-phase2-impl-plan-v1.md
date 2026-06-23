# LingoTrace English Language Pack 实施计划 (Phase 2 第一步)

本文档阐述了在遵循上游最新的四层多语言架构规范下，创建并引入 `English` 语言包第一版（骨架声明层）的实施细节。

## 🎯 目标描述

在不破坏现有日语包、不复用日语特有逻辑，且不修改通用核心引擎的前提下，建立标准合规的英语语言包基础结构。本阶段不实现完整的 Python 工作流，仅建立语言身份、字段契约、路径定义及 Agent 指令边界。

> [!IMPORTANT]
> **User Review Required**
> 请仔细审阅 `fields.json` 中拟定的英语专属字段是否符合你的英语学习规划（我们引入了音标、搭配等概念）。如需增加或修改字段名称，请直接提出。

## ❓ Open Questions

- 我们在 `manifest.json` 中为默认的转写 Locale 设置了 `en-US` (美式英语)，是否需要改为 `en-GB` 或保留为空？
- 当前拟定的核心字典：`meaning_zh`（中文释义）、`pronunciation_ipa`（音标）、`collocations`（常用搭配/词组）。这些是否能覆盖初期英语单词卡片的主要诉求？

## 🛠️ Proposed Changes

所有的修改都将严格限制在新建的 `lingotrace/packs/english/` 目录中。

### English Language Pack Skeleton

建立英语包的四大核心契约文件。

#### [NEW] [manifest.json](file:///Users/jiezhengj/Documents/Project/LingoTrace/lingotrace/packs/english/manifest.json)
- **标识声明**：`language_pack_id: lingo-english`, `target_language: en`, `transcription_locale: en-US`。
- **能力认领**：
  - 声明 `source_notes`、`review_materials`、`review_rollover` 为初始能力（状态标为 `experimental`）。
- **不支持能力声明**：
  - 将 `listening_notes` 和 `speaking_cards` 列入 `unsupported_capabilities`，并给出用户友好的 `failure_reason` (例如: "English listening transcription and speaking card validation are not yet implemented in Phase 2.")。
- **无依赖隔离**：绝不引用 `japanese_dictionary` 或任何日语相关的外部工具。

#### [NEW] [fields.json](file:///Users/jiezhengj/Documents/Project/LingoTrace/lingotrace/packs/english/fields.json)
脱离日语体系，定义一套纯正的英语知识模型：
- `pronunciation_ipa`: 用于记录英文单词的国际音标 (IPA)。
- `meaning_zh`: 保留用作中文释义（遵循解释语言分离原则）。
- `collocations`: 记录英文的常见搭配/词组（替代日语的汉字变体）。
- `word_family`: 记录同源词/词根信息（英语学习特有）。

#### [NEW] [paths.json](file:///Users/jiezhengj/Documents/Project/LingoTrace/lingotrace/packs/english/paths.json)
声明英语 Vault 内部的目录结构。由于目录结构是跨语言高度通用的知识组织形式，我们将保持与标准相近的扁平化英文路径，例如：
- `focus_vocab_root`: `review/focus/vocab`
- `source_notes_root`: `sources`
- *其余以此类推，保证路径命名干净且自洽。*

#### [NEW] [SKILL.md](file:///Users/jiezhengj/Documents/Project/LingoTrace/lingotrace/packs/english/agent_skills/SKILL.md)
为 AI Agent 编写的英语语种操作手册：
- 将用户的自然语言指令（如：“帮我整理这篇英语阅读材料”、“把这个生词加入复习”）映射到对应的能力项。
- 明确指出当用户要求处理听力（Listening）或口语卡（Speaking）时，明确向用户道歉并拒绝（遵守不支持声明）。
- 强调在保存英语卡片前，要抓取音标和常用的英文搭配，并在生成材料时强制不使用日语假名或罗马音规则。

## 🧪 Verification Plan

本阶段没有任何业务逻辑被执行，因此验证以**合规性测试 (Compliance Testing)** 为主。

### Automated Tests
- 执行 `python -m unittest discover -s tests/lingotrace -p 'test_*.py'`
- 执行 `python -m unittest discover -s tools/architecture-baseline/tests -p 'test_*.py'`
- 运行 `bash tools/git/check-public-staged-files.sh` 确保所有文件被白名单正确接纳。

### Manual Verification
- 肉眼检查提交的文件中是否意外混入了 `reading`、`accent_display` 等日语专属字段。
- 确认没有在英语包内引入任何的 Python 脚本或模板逻辑（本期不包含）。
