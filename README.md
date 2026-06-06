# JapanLearning

JapanLearning 是一个基于 Obsidian 的外语学习系统框架。当前仓库发布的是作者日语学习 vault 中可复用的自动化工具、Codex skills 和公开说明，不是完整私人 vault 备份。

项目的开箱主线是日语学习；英语和其他外语可以复用这套方法，但需要改造模板、skill 说明、词典 / 发音资源和例句规则。

## 适合谁

- 已经使用或愿意使用 Obsidian 管理学习资料。
- 愿意让 Codex 或类似 AI agent 协助整理词汇、语法、听力笔记和复习材料。
- 想把“原始材料 -> 学习笔记 -> 复习卡 -> 定期复习”做成可维护流程。
- 能接受少量命令行操作，用于转写、校验和安全提交检查。

如果你只想要一套静态日语笔记，本仓库不是最合适的选择。它更像一个可改造的学习系统骨架。

## 仓库包含什么

- `codex-skills/`：维护学习系统的本地 Codex skills。
- `tools/`：适合公开复用的自动化工具和测试。
- `系统配置/`：路径角色、卡片模板和复习流程说明。
- `学习系统/总训练.base`：Obsidian Bases 主训练面板 scaffold。
- `README.md` 与 `docs/USER_GUIDE.md`：公开使用说明。
- `AGENTS.md`：给 AI agent 的项目规则。

## 仓库不包含什么

- 私人日记、课堂笔记和复习记录。
- Obsidian 本地工作区状态，例如 `.obsidian/`。
- 教材音频、视频、PDF、截图或完整转写稿。
- 临时转写产物、缓存和本地媒体文件。
- 通用 ASR / 音频导入实现。

通用音视频导入和转写由 sibling 项目 [ListenKit](https://github.com/feiyanqiqiao/ListenKit) 负责。本仓库只保留外语学习系统和 Obsidian note 生成逻辑。

## 快速开始

1. 下载或 clone 本仓库。
2. 用 Obsidian 打开一个你自己的学习 vault。
3. 参考 `系统配置/` 和 `学习系统/总训练.base` 建立自己的学习结构。
4. 参考 `codex-skills/` 安装或改造需要的 Codex skills。
5. 如需听力转写，把 `ListenKit` 放在本仓库旁边，或设置 `LISTENKIT_ROOT`。
6. 阅读完整指南：[docs/USER_GUIDE.md](docs/USER_GUIDE.md)。

听力转写默认查找 sibling 路径：

```bash
../ListenKit/cli/generate-markdown.sh
```

如果你的 ListenKit 在其他位置：

```bash
export LISTENKIT_ROOT=/path/to/ListenKit
```

## 常见学习流程

- 词汇 / 语法 / 发音 / 错题卡：使用 `jp-review-material-maintainer`。
- 固定听力笔记：使用 `jp-listening-script-generator`。
- 视频、音频或脚本生成灵活学习笔记：使用 `jp-source-note-generator`。
- 生活口语句库：使用 `jp-survival-speaking-card-generator`。
- 每日复习 rollover：使用 `jp-next-day-review-updater`。

这些 skill 当前以日语学习为主。如果你要学习英语或其他外语，建议先复用工作流思想，再按目标语言重写模板和 skill 规则。

## 工具说明

公开工具入口见 [tools/README.md](tools/README.md)。其中包括：

- 听力笔记生成工具。
- 离线日语词典检查和安装工具。
- vault 结构验证和迁移工具。
- 公开提交 allowlist 检查工具。

## 隐私与版权

不要把以下内容提交到公开仓库：

- 私人学习笔记、日记、课堂记录。
- 教材音频、视频、PDF、截图。
- 商业教材完整转写稿。
- Obsidian 本地工作区状态。
- 临时转写产物、缓存和媒体文件。

提交公开变更前，请运行：

```bash
bash tools/git/check-public-staged-files.sh
```
