# 🔍 多语言架构深度比对与代码级评审报告（全面核查版）

**评审对象**：上游 `docs/lingotrace_multilingual_architecture_plan.md`
**代码核查对象**：我们本地与远端分支 `feature/multi-language-english` 中的实际代码（PR #14）

在对我们提交的代码进行逐行扫描后，发现我们的 V3 实现与上游的工业级多语言架构规划在哲学上不仅有冲突，在代码落地上更是留下了大量“烂尾”与致命隐患。以下是极其具体的代码级对照报告：

---

## 1. 数据抽象：代码证实了最严重的“伪通用抽象”

### 上游红线（章节 10.1 & 6.1）
> “风险：把 `accent_display` 改成 `pronunciation` 等宽泛名称，但字段仍然只适用于日语……控制：**保留语言专属字段**；只有生命周期和接口层进入核心。”

### 我们的代码证据 (`tools/vocab_note.py`)
我们在代码里硬编码了字段名的强制转换：
```python
# tools/vocab_note.py 第 19 行
FIELD_ALIASES: dict[str, list[str]] = {
    "pronunciation": ["reading", "accent_display"],
    "variants": ["kanji_diff_pairs"],
}

# 第 124 行，甚至会删除历史字段！
def set_pronunciation(frontmatter: dict[str, Any], value: str) -> None:
    frontmatter["pronunciation"] = value
    frontmatter.pop("reading", None)
    frontmatter.pop("accent_display", None)
```
**严重后果**：这正是上游严厉禁止的做法。我们的代码在保存卡片时，会**擦除**用户原有的 `reading` 和 `accent_display` 字段，强行缝合成一个 `pronunciation` 字段。上游的 4.6 节明确规定：“核心不得重写手工确认的发音和音调”。我们的代码会破坏几十万日语用户的存量数据格式。

---

## 2. 架构拓扑：未完成的语言包隔离

### 上游红线（章节 4.1 & 10.2）
> 上游要求“四层架构”，明确区分 `Core`（核心引擎）与 `Japanese / English Language Pack`（独立的语言包）。
> “控制：确定性公共逻辑只保留一份；**语言包通过稳定接口提供扩展数据和渲染规则**。”

### 我们的代码证据 (`agent-skills/`)
为了“支持多语言”，我们只是粗暴地把文件夹 `codex-skills/` 改名成了 `agent-skills/`，并去掉了目录前缀的 `jp-`。
但深入核查 Agent 配置文件：
```yaml
# agent-skills/survival-speaking-card-generator/agents/openai.yaml 第 2 行
interface:
  display_name: "JP Survival Speaking Card Generator"
  short_description: "Create and maintain conservative daily-life Japanese speaking cards."
  default_prompt: >-
    Use $jp-survival-speaking-card-generator to create, update, merge, or promote short survival-speaking cards in the Japanese-learning vault.
```
**严重后果**：我们在改名时只改了“皮”，没改“骨”。Prompt 里依然是写死的 `Japanese-learning vault` 和 `JP Survival Speaking`。这意味着如果英语用户使用了这个技能，Agent 仍然会被提示去写“日语”！这就证明了上游的判断：Prompt 和提示词属于**语言特征**，必须分化成两个独立的语言包，靠 `config.json` 里的参数注入是无法覆盖所有硬编码自然语言逻辑的。

---

## 3. Obsidian 查询层的大杂烩

### 上游红线（章节 3.1 单目标语言 Vault）
> “同一个人学习多种语言时，分别建立多个 Vault。不同语言的词汇、语法不在同一个 Vault 中混合。”

### 我们的代码证据 (`学习系统/总训练.base`)
我们在 `总训练.base` 文件里写的查询公式：
```markdown
# 学习系统/总训练.base
core_text: jp_text -> fr_text -> en_text -> pronunciation -> accent_display -> headword
```
**严重后果**：上游的理念是“单目标语言 Vault”，这意味着在英语 Vault 里根本就不该出现 `jp_text` 和 `fr_text` 的判断。按照上游的设计，`English Language Pack` 会自带一个专属于英语的 `总训练.base`（或者专属于英语的视图）。我们试图在一个配置文件里用 `->` 兼容世界上所有的语言，导致随着语言增加，公式会变得无限长且难以维护。

---

## 4. 流程与纪律：一次性梭哈的“危险 PR”

### 上游红线（章节 9.2 变更拆分原则）
> “同一个变更不得同时承担以下三类工作：1. 重构公共核心 2. 批量迁移现有日语数据 3. 增加新的英语学习功能。必要时按依赖顺序分成多个独立 Pull Request。”

### 我们的代码证据
我们的 PR #14 包含 `1121 insertions`，这 1121 行代码在一个 Commit 里同时做了：
1. 提取了新的 Python 工具 (`tools/*.py`) —— **对应重构公共核心**
2. 更改了 Agent 目录和依赖的 bash 脚本路径 —— **对应批量迁移结构**
3. 新增了 `模板/en/` 的五大模板文件 —— **对应增加新英语功能**

**严重后果**：上游的 Code Reviewer 根本无法审阅这个 PR。如果底层 Python 代码引发了日语卡片的复习周期错乱，Reviewer 根本找不出是因为目录重命名导致的，还是因为你新写的 Python 降级逻辑导致的。

---

## 💡 终局建议与后续行动

通过对代码的全面审视，我们的 V3 方案完全不具备合并进上游主分支的基础。它是一个为了让你快速用上“英语功能”而仓促拼凑出的补丁，存在严重的架构破坏性和历史数据毁灭风险。

**明确的行动指南：**
1. **立即自我关闭 PR #14**：向上游说明“我们看到了架构规划，发现当前的 PR 与四层架构严重冲突，且可能影响存量日语数据，因此主动关闭。”
2. **在本地放弃 V3 思路**：我们在本地写的那堆 `vocab_note.py` 的强行合并逻辑最好被抛弃。
3. **拥抱上游路线**：如果你想推动 LingoTrace 的多语言进展，必须严格按照上游文档的 **8.2 阶段 1**（先不要写任何与英语有关的模板和代码，只帮上游把“哪些是日语逻辑”、“哪些是公共引擎逻辑”拆解清楚，分批提交 PR）。
