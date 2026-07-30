# 平台适配指南（Platform Adapter）

`doc-governance` Skill 的核心逻辑（三层记忆模型、Frontmatter 受控词表、`documents/` 目录结构、README 索引五要素、反模式清单、质量评分）是平台无关的。不同 AI 编程工具对"常驻指令 / 路径作用域规则 / 长期记忆"的承载文件不同，本文说明如何把治理规则与目标项目结构接入各平台。

---

## 一、三层记忆模型的平台映射

"三层记忆模型"是对所有 AI 编程工具共享概念（**热 / 温 / 冷**）的抽象，不绑定任何单一工具：

| 层级 | 概念 | 作用 |
|------|------|------|
| **热（Hot）** | 常驻指令文件 | 每次会话自动加载，定义项目命令、规范、硬约束 |
| **温（Warm）** | 路径作用域规则 | 仅在匹配特定路径/语境时激活，约束具体技术栈行为 |
| **冷（Cold）** | 长期记忆与文档语料 | 按需检索的知识库（`documents/` 文档体系） |

各平台对应文件：

| 平台 | 热层（常驻指令） | 温层（路径作用域规则） | 冷层（长期记忆/语料） |
|------|----------------|---------------------|---------------------|
| **CodeBuddy** | `.codebuddy/rules/*.mdc`（`alwaysApply: true`） | `.codebuddy/rules/*.mdc`（`globs:` 限定） | `.codebuddy/memory/` + `documents/` |
| **Claude Code** | `CLAUDE.md`（根目录） | `CLAUDE.md` 内 `@path` import 子文件 | `CLAUDE.md` 追加 / `documents/` |
| **Cursor** | `.cursorrules`（旧）或 `.cursor/rules/*.mdc` | `.cursor/rules/*.mdc`（`description` + `globs`） | `documents/` + `.cursor/rules/` 参考 |
| **Windsurf** | `.windsurfrules` | `.codeium/windsurf.memories` 或规则文件 | `documents/` |
| **Cline** | `CLAUDE.md` 或 `.clinerules` | `.clinerules` 分段 / `documents/05-reference/` | `documents/` |
| **Aider** | `CONVENTIONS.md` / `.aider.conf.yml` | 子模块 `CONVENTIONS.md` + `.aider` 上下文 | `documents/` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 仓库内 `**/*.md` 自然检索 | `documents/` |
| **OpenAI Codex** | `AGENTS.md`（或 `codex.md`） | `AGENTS.md` 内分节 / 子 agent | `documents/` |
| **通用标准** | `AGENTS.md`（多工具已支持） | `documents/05-reference/` 参考文档 | `documents/` |

> 趋势：越来越多工具支持根目录 `AGENTS.md` 作为通用常驻指令文件，可作为跨工具首选。

---

## 二、把治理规则接入各平台

治理规则本体是 `references/documentation-governance.md`（平台无关纯 Markdown）。接入方式：

| 平台 | 接入方式 |
|------|----------|
| **CodeBuddy** | 将规则保存为 `.codebuddy/rules/documentation-governance.mdc`，frontmatter 加 `globs: ["**/*.md"]`、`alwaysApply: false`、`version: "1.0.0"` |
| **Claude Code** | 在 `CLAUDE.md` 中以 `@references/documentation-governance.md` import，或把要点提炼进 `CLAUDE.md` |
| **Cursor** | 放入 `.cursor/rules/documentation-governance.mdc`，frontmatter 加 `description: 文档治理规则`、`globs: **/*.md` |
| **Windsurf** | 要点纳入 `.windsurfrules`，或作为记忆文件 |
| **Cline** | 纳入 `.clinerules` 或 `CLAUDE.md` |
| **Aider** | 纳入 `CONVENTIONS.md` |
| **GitHub Copilot** | 要点纳入 `.github/copilot-instructions.md` |
| **OpenAI Codex** | 要点纳入 `AGENTS.md` |

原则：保留 §1~§7 的实质约束（frontmatter 受控词表、命名、生命周期、索引五要素、反模式、评分），平台专属 frontmatter 仅是"如何被加载"的开关，不影响治理逻辑。

---

## 三、建立目标项目的文档体系（全平台通用）

`documents/` 目录结构与 `templates/` 与具体 AI 工具无关，所有平台一致：

1. 在目标项目根目录创建 `documents/`，按 `references/documentation-governance.md §3.1` 建立子目录（`00-architecture/`…`05-reference/`、`api/`、`archive/`、`templates/`、`images/`）。
2. 将本 Skill 的 `references/templates/` 六个模板复制到目标项目 `documents/templates/`。
3. 在 `documents/` 下创建 `README.md` 索引入口，按 §3.3 五要素维护。
4. （可选）在目标项目根目录建立 harness 文件：`AGENTS.md`（操作手册：Setup & Commands / Coding Standards / Project Structure / Workflow / Do Not）、`PROGRESS.md`（进度快照）、`DECISIONS.md`（决策日志）、`MEMORY.md`（长期记忆）。这些文件对应"热层"，承载项目上下文。
5. 将 `documents/` 接入对应平台的"冷层"检索机制（见第一节映射表）。

---

## 四、最小可用接入（推荐起点）

若只想快速启用治理，最小步骤：

1. 复制 `documents/` 结构 + `templates/` 到目标项目。
2. 创建根目录 `AGENTS.md`，写入"编辑 `*.md` 时遵循 `documents/README.md` 索引与 frontmatter 规范"一行约束。
3. 把 `references/documentation-governance.md` 的 §2.1、§3.3 要点贴入 `AGENTS.md` 或对应平台规则文件。

后续随项目成熟度再补全温层规则与评分审计工作流。
