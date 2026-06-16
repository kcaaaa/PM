---
name: skill-system
description: Skill 系统设计——如何定义、结构化、调用可复用的工作流技能
when_to_use: 当你需要创建可复用技能（SKILL.md）、理解 frontmatter 约定、或设计技能调用规则时阅读。
source: 抽象自 src/skills/loadSkillsDir.ts、src/skills/bundled/skillify.ts 及 bundled 系列
---

# Skill 系统 Skill System

> 当前项目适配：新增或调整 Skill 时，必须保持顶层 `SKILL.md`、`skills/pm/PM-skill.md`、`skills/code/code-skill.md` 与领域 Skill 的调度边界一致。索引型 Skill 只负责路由和调用条件，不能内联下级方法论或绕过 Base 对象包。

## 1. 什么是 Skill

**Skill = 可复用、结构化的工作流**。每个 skill 是一个带 YAML frontmatter 的 Markdown 文件 (`SKILL.md`)，描述：

- 何时被触发（auto-invoke 条件）
- 输入参数
- 需要的工具权限
- 分步流程（步骤、成功标准、人工检查点）
- 硬规则与约束

Skills 本质上是**把会话中反复出现的过程固化下来，让模型在合适时机自动调用或让用户显式调用**。

## 2. Skill 的存储位置

| 作用域 | 路径 | 用途 |
|---|---|---|
| This repo | `.claude/skills/<name>/SKILL.md` | 项目特定工作流 |
| Personal | `~/.claude/skills/<name>/SKILL.md` | 跨项目的个人工作流 |
| Plugin | `plugin/` 目录 | 插件提供的 skill |
| Managed | `.claude/skills/` (policy 路径) | 组织策略下发 |
| Bundled | 代码中注册 | 产品内置 (如 `/simplify`) |
| MCP | 来自 MCP server | 远程不可信，**禁止执行内嵌 shell** |

## 3. Frontmatter 结构

```yaml
---
name: skill-name                        # 技能标识（kebab-case）
description: "一行描述"                  # 必需；或由 Markdown 首段自动提取
when_to_use: "Use when... 触发短语举例"   # CRITICAL - 决定是否自动调用
allowed-tools:                          # 最小权限集合，用模式而非"Bash"
  - Bash(gh:*)
  - Bash(git:*)
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
argument-hint: "[提示占位符]"            # 可选；描述参数样式
arguments:                              # 可选；参数名列表
  - name_of_arg
context: fork                           # 可选；"inline"(默认) 或 "fork"
model: inherit                          # 可选；继承或指定模型
effort: medium                          # 可选；思考力度
version: "1.0"                          # 可选
disable-model-invocation: true          # 可选；禁止模型自动调用（只能用户显式）
user-invocable: true                    # 可选；默认 true
paths:                                  # 可选；限定作用路径
  - src/tools/**
hooks: {...}                            # 可选；hook 配置
shell: {...}                            # 可选；自定义 shell 注入
agent: "general-purpose"                # 可选；指定由哪个 agent 执行
---
```

### 3.1 `when_to_use` 是命脉
这是告诉模型**何时自动调用**的关键。必须：
- 以 "Use when..." 开头
- 包含**触发短语**
- 给出**用户消息示例**

**正例**：
> "Use when the user wants to cherry-pick a PR to a release branch. Examples: 'cherry-pick to release', 'CP this PR', 'hotfix'."

### 3.2 `allowed-tools` 用模式
- ❌ `Bash` （过宽）
- ✅ `Bash(gh:*)` / `Bash(git:*)` （最小权限）

### 3.3 `context: fork` 的选择
- **inline** (默认)：在当前对话中执行，用户可中途操控。
- **fork**：作为独立子代理，拥有自己的上下文。**仅**用于自包含、不需要中途用户输入的任务。

## 4. Skill 内容的标准结构

```markdown
# {{Skill Title}}
描述这个技能做什么

## Inputs
- `$arg_name`: 这个输入的描述

## Goal
清晰陈述工作流目标。最好有清晰定义的产物或完成标准。

## Steps

### 1. 步骤名
这步要做什么。具体、可执行。必要时包含命令。

**Success criteria**: 始终包含！表明这步完成可以继续。可以是列表。

### 2. 步骤名
...

### 3a. 并行子步骤 A
### 3b. 并行子步骤 B

### 4. [human] 需要用户操作
...
```

## 5. 每步的注解 (按需添加)

| 注解 | 说明 |
|---|---|
| **Success criteria** | **每步必需**。明确完成信号，让模型有信心往下走 |
| **Execution** | `Direct`（默认）、`Task agent`、`Teammate`（带通信的并行代理）、`[human]`（用户做） |
| **Artifacts** | 这步产生、后续步骤依赖的数据（PR 号、commit SHA） |
| **Human checkpoint** | 何时暂停向用户确认：不可逆动作（merge、发消息）、错误判断、输出审查 |
| **Rules** | 硬规则；用户在参考会话中修正你的地方特别适合写这里 |

### 5.1 步骤结构技巧
- 可并行步骤用子编号：`3a`、`3b`
- 需要用户操作的步骤标题加 `[human]`
- **简单 skill 保持简单**——2 步 skill 不需要每步都注解

## 6. 参数替换

- body 中用 `$arg_name` 表示参数占位符
- `${CLAUDE_SKILL_DIR}` 替换为 skill 自身目录（供 bash `!\`...\`` 引用打包脚本）
- `${CLAUDE_SESSION_ID}` 替换为当前 session ID
- **安全**：MCP skills（远程不可信）**永不执行**内嵌 shell 命令

## 7. 创建 Skill 的流程（skillify 思路）

创建 skill 的本质是**把会话中的可复用过程捕获下来**，标准流程：

### 第 1 步：分析会话
识别：
- 执行了什么可复用过程
- 输入/参数是什么
- 有序的步骤
- 每步的成功产物/标准（不是"写代码"，而是"一个 PR 创建好、CI 完全通过"）
- 用户在哪里修正或引导你
- 需要什么工具和权限
- 用了哪些 agent
- 目标和成功产物是什么

### 第 2 步：与用户访谈

**全部用 `AskUserQuestion`**，不要用纯文本。每轮迭代到用户满意。

**Round 1：高层确认**
- 基于分析建议 skill name 和 description，请用户确认或重命名
- 建议高层目标和具体成功标准

**Round 2：更多细节**
- 把识别的高层步骤作为编号列表呈现
- 需要参数则建议参数
- 不清楚时问：inline 还是 fork？
- 问保存位置：repo 还是 personal？建议默认值

**Round 3：逐步细化**
对每个主要步骤，除非明显，否则问：
- 这步产出后续步骤需要的什么（数据、产物、ID）？
- 什么证明这步成功、可以继续？
- 是否要在继续前问用户确认？（特别是 merge、发消息、破坏性操作）
- 有步骤独立可并行吗？
- 如何执行？（用 Task agent 做代码审查、或调用 agent 团队做并发步骤）
- 什么是硬约束或硬偏好？什么必须或必不能发生？

**重点关注用户在参考会话中修正你的地方**。

**Round 4：最终问题**
- 确认何时调用这个 skill、建议/确认触发短语
- 问任何其他 gotchas 或注意事项

**Stop 在已有足够信息时**。**简单流程不要过度问**！

### 第 3 步：写 SKILL.md
- 在用户选择的位置创建目录与文件
- **先把完整 SKILL.md 以 yaml 代码块输出**让用户审阅
- 用 AskUserQuestion 确认保存（简短问，不要用 body 字段）

### 第 4 步：保存后告知用户
- 文件保存位置
- 如何调用：`/{{skill-name}} [arguments]`
- 可直接编辑 SKILL.md 精化

## 8. Bundled Skill 范式 (内置 skill)

Bundled skill 通过 `registerBundledSkill({ name, description, allowedTools, userInvocable, async getPromptForCommand(args, context) {...} })` 注册。

**返回**一个字符串 prompt，被当作用户消息注入。通常引用会话状态（如 `getSessionMemoryContent`、`getMessagesAfterCompactBoundary`）来构建上下文感知的 prompt。

### 8.1 典型 Bundled Skills

| Skill | 作用 |
|---|---|
| `/simplify` | 启动 3 个 review agent (reuse / quality / efficiency) 并修问题 |
| `/debug` | 启用调试日志、读最近 20 行、帮用户诊断 |
| `/remember` | 审阅 auto-memory，提议晋升到 CLAUDE.md/CLAUDE.local.md/team memory |
| `/stuck` | 诊断同机其他 Claude Code 会话是否卡住 |
| `/skillify` | 把当前会话的可复用过程捕获为新 skill |
| `/verify` | 触发验证流程 |
| `/batch` | 批处理流程 |
| `/keybindings` | 键位配置 |
| `/loremIpsum` | 占位文本生成 |
| `/updateConfig` | 配置更新向导 |
| `/commit`, `/commit-push-pr` | git 提交与 PR（重度 skill，见 git 工作流技能） |

### 8.2 Bundled Skill 的特殊属性
- `disable-model-invocation`：只有用户显式 `/skillname` 才能触发，防止描述占用上下文
- `isEnabled` 回调：根据 feature flag 或环境动态显示/隐藏

## 9. Skill 发现与 Token 估算

- Skill 的 frontmatter（name、description、whenToUse）会自动随系统提示注入可被选择
- 完整内容**只在调用时才加载**——避免污染上下文
- `estimateSkillFrontmatterTokens` 基于 name + description + whenToUse 粗估 tokens
- 实验性：`DiscoverSkillsTool`（"Skills relevant to your task"）根据任务自动浮现相关 skill

## 10. 默认倾向与原则

- 每次更新都**保留简单性**——简单 skill 不需要复杂注解
- 捕获**用户修正你的地方**到 Rules 中
- **when_to_use 要具体**，不要泛泛
- **allowed-tools 最小权限**，用模式而非工具名
- **fork 用于自包含**，默认 inline 便于中途操控
