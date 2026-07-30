---
name: doc-governance
description: AI 编程项目文档治理专家。当用户需要诊断文档体系健康度、创建 ADR（架构决策记录）、优化文档架构、检测文档腐败、初始化文档体系或建立可迁移的文档管理规范时触发。触发词包括"文档治理"、"文档优化"、"新建ADR"、"文档架构审计"、"文档腐败检测"、"文档最佳实践"、"文档初始化"、"文档标准化"等。与 specforge 系列 Skill 衔接——负责文档体系的"治理"层，而非功能文档的"生成"层。
---

# 通用文档治理 Skill（Documentation Governance）

## 定位

本 Skill 是**项目无关**的通用文档治理引擎。可复制到任何项目的 `.codebuddy/skills/doc-governance/` 目录下直接使用。

与 `specforge-feature-dev`（功能文档生成）、`specforge-bugfix`（BUG 报告生成）互补——本 Skill 聚焦**治理层**：诊断、创建、审计、标准化、归档。

## 触发条件

当用户提到以下任一场景时自动加载：
- "文档治理"、"文档优化"、"文档清理"、"文档标准化"
- "新建 ADR"、"架构决策记录"、"为什么选这个方案"
- "文档架构审计"、"文档健康检查"、"文档评估"
- "文档腐败检测"、"文档和代码不一致"
- "初始化文档体系"、"建立文档规范"
- "文档最佳实践"、"文档规范"

## 五维工作流

---

### 工作流 1：文档体系诊断（Diagnose）

**目标**：使用三层记忆模型对项目文档体系进行全面评估，输出量化评分。

**步骤**：

1. **热层审计（Hot Memory Check）**
   - 读取 `AGENTS.md` 或 `CLAUDE.md`（若存在）
   - 读取所有 `alwaysApply: true` 的规则文件
   - 统计 Hot Memory 总行数，判断是否 ≤ 300 行上限
   - 逐条审查：用"删除这条，Agent 行为会改变吗？"测试必要性
   - 输出：行数 / 必要规则 / 冗余规则 / 优化建议

2. **温层审计（Warm Memory Check）**
   - 读取所有 `globs` 限定的规则文件
   - 检查每条规则是否声明了 `globs`（路径限定原则）
   - 检查单文件是否超过 200 行
   - 检查是否有无 `globs` 但非 `alwaysApply` 的规则（无法激活）
   - 输出：规则数 / 路径覆盖度 / 大小分布 / 合并拆分建议

3. **冷层审计（Cold Memory Check）**
   - 扫描 `documents/` 目录结构
   - 检查 `documents/README.md` 是否包含所有文档的索引条目
   - 抽样检查文档是否有 YAML frontmatter（元数据完整性）
   - 检查每个文档的 `role` 标注与所在目录是否一致
   - 检测超过 30 天未更新但引用了可能已变更代码的文档（腐败信号）
   - 统计 `[State]` / `[Delta]` / `[Cold]` 分布
   - 输出：文档数 / 索引覆盖率 / frontmatter 覆盖率 / 腐败风险

4. **综合诊断报告**

```markdown
# 文档体系诊断报告 — {date}

## 总体评分：XX/100
- Agent 友好度：XX/100（元数据 XX/25 + 角色 XX/15 + 链接 XX/20 + 一致性 XX/20 + 可检索 XX/10 + 时效 XX/10）
- 人类可读性：XX/100（排版 XX/25 + 目录 XX/15 + 示例 XX/20 + 图表 XX/20 + 语言 XX/20）

## Hot Memory（热层）
- 文件：{files}
- 总行数：XX/300
- 冗余建议：{items}

## Warm Memory（温层）
- 规则总数：X 条
- Globs 覆盖率：XX%
- 超大文件（>200 行）：X 个

## Cold Memory（冷层）
- 文档总数：X 份（State: X / Delta: X / Cold: X）
- 索引覆盖率：XX%
- Frontmatter 覆盖率：XX%
- 腐败风险：X 份

## 优先级排序
| 优先级 | 问题 | 影响 | 建议 |
|--------|------|------|------|
| P0 | ... | ... | ... |
```

---

### 工作流 2：文档创建（Create）

**目标**：引导用户创建符合规范的文档，自动选择模板、填充 frontmatter、注册索引。

**步骤**：

1. **意图识别**
   - 用户要创建什么文档？确定文档类型（ADR / Sprint Backlog / 设计文档 / 报告 / 调研）
   - 此文档属于 `[State]` 还是 `[Delta]` 还是 `[Cold]`？

2. **冲突检查**
   - 搜索 `documents/README.md`，是否已有覆盖该主题的文档？
   - 若有 → 建议更新现有文档，展示现有文档路径

3. **模板选择**

   | 文档类型 | 模板 | 目标目录 | role |
   |---------|------|---------|:--:|
   | ADR | `templates/adr-template.md` | `00-architecture/architecture-decisions/` | State |
   | Sprint Backlog | `templates/sprint-backlog-template.md` | `01-planning/` | Delta |
   | 设计文档 | `templates/universal-doc-template.md` | `00-architecture/` | State |
   | 审查报告 | `templates/audit-report-template.md` | `02-review/` | Delta |
   | BUG 报告 | `templates/bugfix-report-template.md` | `02-review/` | Delta |
   | 验证报告 | `templates/verification-report-template.md` | `04-reports/` | Delta |
   | 发布清单 | `templates/release-checklist-template.md` | `03-releases/` | Delta |
   | 调研/参考 | `templates/universal-doc-template.md` | `05-reference/` | Cold |
   | 通用文档 | `templates/universal-doc-template.md` | 按内容确定 | 按内容确定 |

4. **Frontmatter 生成**
   - 自动生成 YAML frontmatter（`doc_id` 按目录现有最大编号 +1）
   - 若用户未提供，自动推断 `tags`（从标题和内容提取关键词）
   - `related` 字段：自动搜索已有文档标题，推荐关联

5. **内容填充**
   - 基于用户提供的信息 + 代码库上下文填充模板各节
   - 确保所有代码块标注语言
   - 引用代码时使用相对路径

6. **索引注册**
   - 在 `documents/README.md` 对应分类表格中增加条目
   - 格式：`| [文档名](相对路径) | role | date | status |`

7. **输出创建摘要**

---

### 工作流 3：文档审计（Audit）

**目标**：对单份文档或整个目录进行全面质量审查，逐项评分。

**步骤**：

1. **确定审计范围**
   - 单文件 → 逐项检查 §2.3 + §5
   - 整个目录 → 批量抽样 + 统计报告

2. **执行检查清单**（来自 `documentation-governance.mdc`）

   | 检查项 | 来源 | 通过/失败 |
   |--------|------|:--:|
   | Frontmatter 完整（5 必填字段） | §2.1 | — |
   | 标题 = frontmatter title | §2.3 | — |
   | role 标注正确 | §3.2 | — |
   | TOC 目录（>3 节时） | §2.3 | — |
   | 代码块有语言标注 | §4.3 | — |
   | 链接有效性（内部相对路径） | §2.2 | — |
   | 无占位符 TODO/FIXME | §2.3 | — |
   | 已在 README.md 注册 | §3.3 | — |
   | 代码一致性 | §2.3 | — |
   | tags 含语义关键词 | §2.1 | — |

3. **Agent 友好度评分**
   - 按 §5.1 六维评分（元数据 25 + 角色 15 + 链接 20 + 一致性 20 + 可检索 10 + 时效 10）

4. **人类可读性评分**
   - 按 §5.2 五维评分（排版 25 + 目录 15 + 示例 20 + 图表 20 + 语言 20）

5. **输出审计报告**
   - 给出具体修复建议（不只是"需改进"，而是"在 §X.X 节增加 Y"）

---

### 工作流 4：腐败检测（Corruption Detection）

**目标**：检测文档是否与代码实现不一致，防止文档误导 Agent。

**步骤**：

1. **扫描代码变更**
   - `git diff --name-only HEAD~N` 或指定日期范围
   - 提取变更的类名、方法签名、接口路径、配置键

2. **搜索受影响的文档**
   - 在 `documents/` 中搜索引用上述变更实体的文档
   - 搜索模式：类名（`\bClassName\b`）、路径（`/api/xxx`）、配置键（`xxx.yyy`）

3. **比对**
   - 对比文档中的描述 vs 当前代码实际签名/路径/值
   - 标记不一致的文档章节和行号

4. **腐败信号清单**

   | 信号 | 严重度 | 示例 |
   |------|:--:|------|
   | 文档引用已删除的类/方法 | 🔥 P0 | `UserService.createUser()` 已改为 `UserService.register()` |
   | 接口路径不匹配 | 🔥 P0 | 文档 `POST /api/user` → 代码 `POST /api/users` |
   | 配置参数过期 | ⚠️ P1 | 文档 `timeout: 30` → 代码 `timeout: 60` |
   | 架构描述过时 | ⚠️ P1 | 文档说"单模块"→ 代码已是多模块 |
   | 字段名/类型不一致 | 🔵 P2 | 文档 `userId: String` → 代码 `userId: Long` |

5. **输出腐败检测报告**

---

### 工作流 5：文档体系初始化（Initialize）

**目标**：在新项目或现有项目中一键建立符合三层记忆架构的文档基础设施。

**步骤**：

1. **创建 `AGENTS.md`**（根目录）
   - 四段式：Setup & Commands / Coding Standards / Project Structure / Do Not
   - 控制在 80~120 行

2. **创建目录结构**
   ```
   documents/
   ├── README.md
   ├── images/
   ├── 00-architecture/architecture-decisions/
   ├── 01-planning/
   ├── 02-review/
   ├── 03-releases/
   ├── 04-reports/
   ├── 05-reference/
   ├── api/
   ├── archive/
   └── templates/
   ```

3. **复制模板文件**
   - `adr-template.md`
   - `universal-doc-template.md`
   - `sprint-backlog-template.md`
   - `audit-report-template.md`
   - `bugfix-report-template.md`
   - `verification-report-template.md`
   - `release-checklist-template.md`

4. **创建治理规则**
   - `.codebuddy/rules/documentation-governance.mdc`

5. **创建首份 ADR**（可选）
   - `001-documentation-architecture.md`：记录"为什么采用三层记忆架构"

6. **写入 `documents/README.md` 索引**
   - 包含场景查找表、目录结构图、ADR 索引区、模板索引区

7. **输出初始化完成清单**

---

## 与 SpecForge 工作流的衔接

| SpecForge 阶段 | 本 Skill 触发时机 | 推荐工作流 |
|---------------|-----------------|-----------|
| 需求澄清 | 用户提到"文档"关键词 | 工作流 1 诊断 + 工作流 5 初始化 |
| 技术设计 | 设计方案确定、技术选型敲定 | 工作流 2 创建 ADR |
| 任务规划 | Sprint Backlog 需要更新 | 工作流 2 创建 Sprint Backlog |
| 编码实现 | 代码变更完成 | 工作流 4 腐败检测 |
| BUG 修复 | BUG 修复报告生成 | 工作流 2 创建 bugfix 报告 |
| 阶段收尾 | Phase 完成 | 工作流 3 审计 + 更新索引 |

## 核心原则速查

| # | 原则 | 来源 | 操作 |
|---|------|------|------|
| 1 | **热冷分离** | 调研报告 §4.1 | Hot ≤ 300 行，Warm ≤ 200 行/文件，Cold 通过 README.md 索引按需检索 |
| 2 | **过时即毒药** | 调研报告 §4.2 | 修改代码 → 同步更新文档，过期文档比无文档更危险 |
| 3 | **解释两次就写下来** | 调研报告 §4.3 | 同一知识被反复解释 → 写入 Cold Memory，从上下文移除 |
| 4 | **State/Delta 显式标注** | 调研报告 §4.4 | 索引和 frontmatter 标注 `[State]` / `[Delta]` / `[Cold]` |
| 5 | **Frontmatter 必填** | governance.mdc §2.1 | 每份文档必须有 YAML frontmatter（5 必填字段） |
| 6 | **索引即入口** | governance.mdc §3.3 | README.md 是 Agent 检索文档的唯一起点 |

## 文件约定

| 约定 | 值 |
|------|-----|
| ADR 命名 | `NNN-kebab-case.md`（三位递增编号） |
| 索引文件 | 始终为 `documents/README.md` |
| ADR 目录 | 始终为 `documents/00-architecture/architecture-decisions/` |
| 治理规则 | 始终为 `.codebuddy/rules/documentation-governance.mdc` |
| 模板目录 | 始终为 `documents/templates/` |
| 通用模板 | `universal-doc-template.md` |
| 方法论参考 | `documents/05-reference/文档治理通用标准_v1.0.md` |
