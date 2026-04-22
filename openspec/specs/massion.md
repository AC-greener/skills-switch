# Mission

## 我们是谁

SkillSync 是一个面向开发者的 macOS 原生应用，用于统一管理跨 AI 代码 Agent 的 Skill 文件。

---

## 我们解决什么问题

Agent Skills 是一项正在快速成为事实标准的技术——以一个包含 `SKILL.md` 的文件夹为单位，向 AI Agent 注入可复用的专项能力。Claude Code、Codex CLI、Gemini CLI、Cursor 等主流工具都已采用这一格式。

**但格式统一，路径各异。**

| Agent | 全局 Skill 路径 | 项目级 Skill 路径 |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex CLI | `~/.codex/skills/` | `.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/` | `.gemini/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |

这意味着：

- **重复维护**：同一个 skill 需要手动复制到多个目录，更新时容易遗漏
- **版本失控**：各目录下的 skill 版本悄悄分叉，没有统一的 source of truth
- **项目分发困难**：团队协作时，项目级 skill 的安装和同步完全靠手工
- **没有全局视图**：无法一眼看清当前系统中有哪些 skill、它们分布在哪里、哪些在用

这些问题在 skill 数量少时可以忽略，但随着 Agent 工具链的成熟，每个开发者都会积累数十个 skill，痛点会急剧放大。

---

## 我们的解决方案

SkillSync 将你的 skill 库作为单一数据源（Single Source of Truth）进行管理，并通过 symlink 或文件同步自动分发到各 Agent 所需的路径。

核心能力：

- **统一库**：在一个地方浏览、编辑、搜索你所有的 skill
- **自动分发**：配置一次目标 Agent，skill 的变更自动同步到对应目录
- **版本管理**：每次保存自动快照，支持查看历史、对比差异、一键回滚
- **项目感知**：识别当前项目的 `.claude/skills/`、`.gemini/skills/` 等目录，提供安装/卸载操作
- **文件监听**：实时感知磁盘上的变更，双向保持同步

---

## 我们的信念

Agent Skills 作为一种开放的、基于文件系统的标准，其生命力在于可移植性。我们相信这一标准会继续扩展到更多工具，而开发者需要一个稳定的管理层来驾驭这种碎片化。

SkillSync 不绑定任何特定 Agent，不托管你的数据，不改变 skill 的格式。它只做一件事：**让正确的 skill 出现在正确的地方。**

---

## 目标用户

- 日常同时使用 2 个以上 AI 代码 Agent 的开发者
- 积累了超过 10 个自定义 skill 并开始感到管理困难的用户
- 需要在团队内分发和标准化项目级 skill 的 Tech Lead