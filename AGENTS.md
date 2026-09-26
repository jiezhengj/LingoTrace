本仓库是位于用户私有 Obsidian Vault 之外的公共 LingoTrace 运行时。请将笔记、Frontmatter、Wikilinks、Bases、公共模板、Vault 初始化、运行时连接以及语言包 Agent Skill 均视为面向用户的学习系统的一部分。

# 主要操作入口

使用 `lingotrace/packs/japanese/agent_skills/SKILL.md` 作为日语日常学习任务的自然语言操作入口（natural-language operating entry）。

使用 `lingotrace/packs/english/agent_skills/SKILL.md` 作为英语日常学习任务的自然语言操作入口（natural-language operating entry）。已初始化的 Vault 根 `AGENTS.md`、`.lingotrace/vault-context.json` 以及当前平台的运行时连接会自动选择对应的操作入口，无需用户显式指定。

用户应当能够使用日常学习语言发起请求，例如：

- "请把这段音频做成精听稿。"
- "帮我把这篇材料整理成日语学习笔记。"
- "把这个词加入复习。"
- "这句话很实用，帮我做成口语卡。"
- "今天复习结束了，帮我结算。"

不要要求用户提及工作流入口（Do not ask users to mention workflow entrypoints）、函数名称、数据包络或写入模式术语。Agent Skill 会将自然语言请求映射到匹配的日语包能力。实际的文件变更仍必须经由 LingoTrace 核心与日语包（LingoTrace core and Japanese pack），包括上下文检查、能力检查、路径边界以及核心写入保护（core write guard）。

不要向本文档复制完整的模式定义或工作流细节。在修改对应子系统之前，请先阅读相关的 Agent Skill、对应的 `lingotrace/packs/japanese/` 模块以及公开测试。

# 用户旅程

- 仅希望进行学习的用户从 `docs/learner-agent-setup.md` 开始。仅安装最小公共运行时，将私有 Vault 保持在运行时目录外部，并将该 Vault 作为日常 Agent 工作区。
- 开发者从 `docs/developer-agent-setup.md` 开始，使用完整的代码检出和主题分支，然后在其真实的 Vault 中复用学习者配置。
- 不要要求学习者 fork 本项目、安装 GitHub CLI、阅读贡献者文档或运行公开开发测试套件。
- 在修改引导配置流程之前，请先阅读 `docs/installation-and-onboarding-design.md`，确保学习者与开发者的路径保持独立。
- 两条旅程均需执行 `docs/daily-runtime-update-design.md` 中定义的非阻塞每日更新检查。官方运行时仅在获得明确同意后方可更新；个人 fork 必须留给用户在开发者工作区中自行同步。

# 路径角色

不要将正文中的文件夹路径视为单一事实来源。运行时路径角色位于每个目标 Vault 的 `.lingotrace/paths.json` 中；语言包默认路径位于 `lingotrace/packs/japanese/paths.json`。仅在修改共享的日语模板时更新语言包默认路径，并在显式的本地操作期间更新私有 Vault 配置。

# 操作规则

- 优先使用支持 Obsidian 与 Markdown 规范的工作流进行笔记检索、笔记编辑、Frontmatter、Wikilinks 以及 `.base` 文件处理。
- 编辑词汇卡前先进行检索。在编辑基础词库之前，先检查重点复习层，避免创建重复卡片。
- 对于可能更新现有学习状态的面向用户任务，请使用日常白话描述计划中的变更，并在保存前征得确认；明确的每日复习结算请求除外。明确的复习结算执行内部预览，若确认接受则应用，随后通过二次预览进行验证。
- 保持改动范围收敛。在执行单一任务时，不要重新排序大量笔记、批量重写 Frontmatter 或规范化无关的 Markdown 文件。
- 保留人工整理的内容，特别是精听笔记中的选句、复习笔记和每日学习小结，除非用户明确要求重置。
- 避免修改生成的工具或辅助脚本，除非该任务专门针对自动化本身。
- 声调（アクセント）对比卡归属于发音声调（pronunciation accent）角色，不属于普通词汇。不要将其放入普通词汇或句子练习角色中；请遵循 `docs/multilingual/japanese-review-card-format-and-links.md` 中的具体卡片规则。
- 清音/浊音、送气、声带振动等音素对比卡属于发音音素（pronunciation phoneme）角色，不要放入句子练习角色中。
- **Changelog 规则**：在修改项目框架（如源代码、Manifests、公共模板或核心文档）时，必须确保更新项目的 `CHANGELOG.md`。
  - *例外*：日常用户内容创建任务（例如在 Vault 中生成笔记或词汇卡）不编写 Changelog。
  - *适当时机*：仅在所有代码变更均已完整实现并通过自动化测试之后、但在执行最终 `git commit` *之前* 编写 Changelog 条目。这确保 Changelog 能够反映最终真实状态并与代码原子性提交。

# 验证

对于纯文档变更，验证所引用的路径是否存在，且新指引不与相关的 `SKILL.md` 文件相冲突。

对于笔记或工作流变更，优先使用小范围的针对性检查，而不是对整个 Vault 进行全面扫描。当脚本提供干运行（dry-run）模式时，将其作为第一验证步骤。

<!-- PROJECT-SPEC-KIT-GOVERNANCE:START -->

# Spec Kit Governance

This repository uses the committed project-local Spec Kit governance package.

Read `docs/spec-kit/START_HERE.md` before substantive engineering work.

At entry to an existing Spec Kit project, run `tools/spec-kit-governance/governance.py check-capabilities`. If `specify`, the active integration, `assess`, or `bug` is missing, ask the user whether to install the missing capability through the native CLI. If the user declines, return `HANDOFF_TO_AGENT` and let the current Agent continue without applying Reference governance to that capability.

Whenever `.specify/` exists, independently of the central Reference or `GLOBAL_POLICY.md`, perform the upstream Spec Kit update check at most once per new Agent session before the first substantive action. Run the currently installed CLI's read-only `specify self check`; do not assume an exact version or installation source. Ask the user before `specify self upgrade` only when a newer CLI is reported. Regardless of whether that CLI upgrade is approved, declined, or not needed, inspect the current integration, installed extensions, and installed workflows using the CLI's actual help/status/list contracts and automatically run supported refresh commands such as `specify integration upgrade <active-key>`, `specify extension update`, and `specify workflow update` when components can be refreshed. If the CLI has no freshness field, a supported no-force update command may be run and its no-update result is sufficient. Do not install missing components, assume presets are covered, invent flags, or add `--force`. If refresh is blocked by modified managed files, a requested `--force`, an unsafe scope, or another irreversible choice, stop and ask the user with the exact reason and paths.

A conversational approval such as `the plan is acceptable` advances a direction into the upstream Spec Kit workflow; it does not authorize direct application-code edits before the current Spec Kit artifacts are aligned.

The governance package does not edit `.specify/**`, `specs/**`, or native Agent-generated integration files.

Do not replace the project baseline with personal global rules or a local Reference.

Project documentation language: `zh`.

Write new and substantively rewritten project documentation, including Spec Kit artifacts, in this language unless an explicit user or more specific project instruction overrides it. Do not translate existing documentation solely because this setting was selected.

<!-- PROJECT-SPEC-KIT-GOVERNANCE:END -->

<!-- PROJECT-SPEC-KIT-REFERENCE-UPDATE-CHECK:START version=1 -->

# Spec Kit Reference update check

This check is active only when the current Agent has loaded the global Spec Kit Policy and that Policy provides a readable `SPEC_KIT_GOVERNANCE_SOURCE` absolute path.

When `.specify/` and the committed project governance package are present, run the local governance manager's read-only `check-update --source <central-reference-path>` once before the first substantive task in a new Agent session. If the Policy or source locator is absent, skip this check silently; do not scan the computer for a Reference directory.

If a verified Reference update is available, run the exact hash-bound `auto-upgrade` operation without waiting for project-owner approval. The sync may update only Reference-owned governance files and this managed block; it must never edit `.specify/**`, `specs/**`, native Agent files, or business code. After the governance sync, let the upstream Spec Kit workflow decide whether any specification, plan, or task artifacts need updating.

A missing source, unclean source, invalid verification, offline check, or timeout is non-blocking in normal project work and must not be presented as an available update.

<!-- PROJECT-SPEC-KIT-REFERENCE-UPDATE-CHECK:END -->
