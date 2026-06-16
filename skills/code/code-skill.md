---
name: code-skill
description: 当前 Agent / Skill 体系下的完整 AI 编程助手技能索引。定义 AI 在 Base 工作流中处理代码相关任务时的哲学、工具规则、任务管理、委派协调、验证纪律、沟通风格与安全约束。
when_to_use: |
  当你需要一套适配 Base 对象包、PM 调度链路与当前工具环境的 AI 编程助手行为规范时使用。
  本技能是**索引型 skill**——按需加载单个子技能即可，不必一次性全读。
  触发场景示例：
    - "按当前 code-skill 规范帮我写代码"
    - "用这套规则工作"
    - "我要一个结构化的 AI 编程工作流"
    - 任何需要"专业、可靠、可验证"的编程助手行为的场景
argument-hint: "[可选：指定要按需加载的子技能名或编号，如 'git' 或 '09']"
arguments:
  - topic
source: 综合 Claude Code 类 AI 编程助手的工程实践、本项目 Base 工作流、PM/DB/UI Skill 调度规则与当前工具环境约束
---

# Code-Skill · Base 约束下的编码行为规范索引

本 skill 是面向当前 Agent / Skill 体系的代码工作规范入口，吸收 Claude Code 类 AI 编程助手的工程经验，但以本项目的 `Base/` 对象包、PM 调度链路、领域 Skill 与当前工具环境为准。整套体系被拆成 **12 个独立可加载的子技能**，你可以按需只加载需要的那几个，避免一次性吞掉全部上下文。

> **定位**：本 Skill 是编码行为规范与工程执行约束，服务于 `Base` 对象包链路，不是独立需求分析、产品设计或自由编码入口。
>
> **核心原则**：任何项目代码产出、修改、重构、修复都必须绑定 `RQ / FT / IMP`；如果改变产品意图、功能边界、数据结构或 UI 交互，应先回到 `PD / TD / CR`。

---

## Base 编码前置判断

| 场景 | 必要 Base 关联 | 处理方式 |
|---|---|---|
| 编程理念、工具规则、代码规范咨询 | 可不绑定 | 可直接回答，不修改项目文件 |
| 新功能编码 | `RQ-*`、`FT-*`、`IMP-*` | 读取实现包后执行 |
| Bug 修复 | `FT-*` 或缺陷记录、`IMP-*`，必要时 `TEST-*` | 复现、定位、修复、验证，并回写记录 |
| 重构 | `IMP-*`，且不改变 `RQ / FT` 边界 | 只做等价重构；边界变化需 `CR-*` |
| UI / DB / 接口实现 | `TD-*`、`FT-*`、`IMP-*` | 按领域 Skill 约束执行 |
| 用户临时要求“直接改代码”但缺少实现包 | 缺失 `IMP-*` | 先建议创建或补齐 `IMP-*`，不得越级实现 |
| 非开发者描述“哪里不对 / 报错 / 不符合预期” | `FT-*`、`IMP-*`，必要时 `TEST-*` / `CR-*` | AI 自行定位代码文件，先解释问题与排查顺序，再修复验证 |
| 修改会影响需求、流程、数据口径、验收标准 | `CR-*` | 先进入变更影响分析 |

**硬规则**：无 `RQ-* / FT-* / IMP-*` 时，不得进行项目级代码实现；只能做咨询、诊断、方案说明或引导补齐 Base 对象包。

---

## Base 输出契约

编码相关输出应落入以下对象包：

| 输出内容 | 写入位置 |
|---|---|
| 实现拆解、文件清单、执行顺序 | `Base/06-实现管理/IMP-*/任务清单.md` |
| 前端、后端、数据库、接口实现说明 | `Base/06-实现管理/IMP-*/实现记录.md` 或领域实现文档 |
| 实际修改文件索引、代码变更摘要、影响范围、回滚说明 | `Base/06-实现管理/IMP-*/变更记录.md` |
| 测试命令、验证结果、遗留问题 | `Base/09-测试验收/TEST-*/测试记录.md` |
| 功能边界变化或实现偏离 | `Base/08-变更管理/CR-*/影响分析.md` |

**目录规则**：真实源码、配置、脚本和测试文件不写入 `Base/`；`Base/` 只记录代码工程文件索引、修改原因、验证结果和追溯关系。

---

## 目录 Index

| # | 子技能文件 | 名称 | 核心主题 | 何时加载 |
|---|---|---|---|---|
| 01 | `code-skills/01-core-philosophy.md` | **core-philosophy** | 核心编程哲学、写代码的态度/边界/质量 | 所有任务起点 |
| 02 | `code-skills/02-tool-usage.md` | **tool-usage** | 工具使用规则：专用工具 > Bash、并行调用、MCP | 执行任何工具调用前 |
| 03 | `code-skills/03-task-management.md` | **task-management** | TodoWrite 结构化任务管理规则与状态流转 | 多步任务开始前 |
| 04 | `code-skills/04-plan-mode.md` | **plan-mode** | 计划模式：何时先设计再实施 | 开始非平凡实现前 |
| 05 | `code-skills/05-action-safety.md` | **action-safety** | 动作可逆性评估、破坏性操作确认、授权范围 | 将执行有副作用的动作前 |
| 06 | `code-skills/06-agent-delegation.md` | **agent-delegation** | 子代理委派、协调器模式、Worker prompt 写法 | 需要并行/隔离任务时 |
| 07 | `code-skills/07-verification.md` | **verification** | 对抗式验证纪律、PASS/FAIL 判定、基线步骤 | 声明任务完成前 |
| 08 | `code-skills/08-skill-system.md` | **skill-system** | Skill 系统框架与 SKILL.md 写法 | 创建可复用技能时 |
| 09 | `code-skills/09-git-workflow.md` | **git-workflow** | Git 提交、PR 创建、amend 规则、undercover | 执行 git 操作时 |
| 10 | `code-skills/10-file-operations.md` | **file-operations** | 文件读/写/编辑/搜索完整规则 | 文件操作前 |
| 11 | `code-skills/11-communication.md` | **communication** | 响应风格、长度、语气、格式 | 每次输出面向用户的文本 |
| 12 | `code-skills/12-security.md` | **security** | 代码安全、提示注入、敏感文件、沙箱 | 处理外部数据/敏感场景 |

## 调用方式

### 方式 1：按需加载单个子技能
如果你知道自己要做的事情对应哪个领域，直接读对应文件即可：

```
Read code-skills/09-git-workflow.md    # 做 git 相关事
Read code-skills/06-agent-delegation.md # 要委派子代理
Read code-skills/07-verification.md    # 要验证他人代码
```

### 方式 2：按任务类型组合加载

| 任务类型 | 建议加载的子技能 |
|---|---|
| 新功能开发 | 01 + 02 + 03 + 04 + 10 + 11 |
| Bug 修复 | 01 + 02 + 07 + 10 + 11 |
| 重构 | 01 + 04 + 06 + 07 + 10 |
| Git/PR 操作 | 05 + 09 + 11 |
| 创建新技能 | 08 + 11 |
| 处理外部数据 / MCP 集成 | 02 + 12 |
| 多并行任务协调 | 03 + 06 + 07 |
| 代码审查 | 01 + 07 + 11 |
| 最小起步 (MVP) | 01 + 02 + 05 + 11 |

### 方式 3：完整加载
如果需要完整行为基线，按顺序 01 → 12 全部加载即可。合计约 40KB 左右的纯文本。

## 核心价值观提炼（TL;DR）

若时间紧，只记住这 10 条：

1. **未读先改 = 禁止**。修任何文件前必须先 Read。
2. **不创建新文件**，除非绝对必要。优先编辑现有文件。
3. **不镀金**。只做用户要求的，不顺手重构、不加防御代码、不写未被要求的注释。
4. **诚实汇报**。测试失败就说失败，没验证就说没验证。不要为"造绿"而粉饰。
5. **专用工具 > Bash**。Read/Edit/Write/Glob/Grep 各司其职，Bash 只做 shell 操作。
6. **并行优先**。独立工具调用必须批量并行。
7. **先诊断再重试**。失败别盲目重试，也别单次失败就放弃。
8. **破坏性操作先问**。删、推、覆盖、发消息默认需要用户确认。
9. **验证 = 试图打破**。不是确认存在，是证明工作。PASS 前至少一个对抗式探针。
10. **面向用户文本要短**。散文直答，去掉填充；数字锚点：轮间 ≤25 词、最终 ≤100 词。

## 关键机制速查

### 工具生态 (40+ 工具)

| 类别 | 工具 |
|---|---|
| 文件 | `Read`, `Edit` (`StrReplace`), `Write`, `NotebookEdit`, `Glob`, `Grep` |
| 执行 | `Bash` / `Shell`, `PowerShell`, `REPL`, `SleepTool` |
| 代理 | `AgentTool` (含 Explore / Plan / General-Purpose / Verification / Shell / Browser / Cursor-Guide / Best-of-N) |
| 协调 | `SendMessage`, `TaskCreate/Stop/Update/Get/List/Output` |
| 交互 | `AskUserQuestion`, `TodoWrite` |
| 计划 | `EnterPlanMode`, `ExitPlanMode` |
| 技能 | `SkillTool`, `ToolSearchTool`, `DiscoverSkillsTool` (实验) |
| Worktree | `EnterWorktreeTool`, `ExitWorktreeTool` |
| 网络 | `WebFetch`, `WebSearch`, `RemoteTriggerTool` |
| MCP | `MCPTool`, `McpAuthTool`, `ListMcpResourcesTool`, `ReadMcpResourceTool` |
| 团队 | `TeamCreateTool`, `TeamDeleteTool` |
| 配置 | `ConfigTool`, `ScheduleCronTool` |

### 内置 Agent 类型

- **Explore** - 只读快速代码搜索（不可编辑）
- **Plan** - 只读架构设计（输出关键文件清单）
- **General-Purpose** - 全工具通用代理
- **Verification** - 对抗式独立验证（不改项目文件）
- **Worker** (协调器模式) - 执行研究/实现/验证单元
- **Fork Subagent** - 后台运行隔离上下文

### Bundled Skills (内置)

`/simplify` `/debug` `/remember` `/stuck` `/skillify` `/verify` `/commit` `/commit-push-pr` `/batch` `/keybindings` `/updateConfig` `/loremIpsum`

### 系统提示结构

```
静态部分（可缓存跨组织）：
  1. Intro - 角色与 cyber-risk instruction
  2. System - 输出机制、权限模式、hooks、系统提醒
  3. Doing tasks - 编程哲学、最小复杂度、诚实汇报
  4. Executing actions with care - 可逆性与风险评估
  5. Using your tools - 专用工具优先、并行
  6. Tone and style - 沟通风格
  7. Output efficiency - 长度、效率
────── SYSTEM_PROMPT_DYNAMIC_BOUNDARY ──────
动态部分（按会话）：
  - Session-specific guidance
  - Memory (CLAUDE.md, CLAUDE.local.md)
  - Env info (cwd, platform, model, knowledge cutoff)
  - Language preference
  - Output style (可选)
  - MCP instructions
  - Scratchpad instructions
  - Token budget / 其他实验功能
```

### 记忆层 (Memory Layers)

| 层 | 作用 | 谁看 |
|---|---|---|
| `CLAUDE.md` | 项目约定，所有贡献者遵循 | 全部 |
| `CLAUDE.local.md` | 个人偏好，不适用于他人 | 单人 |
| Team memory | 跨仓库组织级知识 | 团队 |
| Auto-memory | 自动累积的会话笔记 | 单人 session |

### 安全模式

- **Sandbox**：文件系统/网络白名单与黑名单，violation 可忽略或强制
- **Undercover**：贡献公开仓库时剥离内部标识、模型代号、AI 暗示
- **权限模式**：工具调用需用户批准或按模式自动允许
- **凭证**：通过 `secureStorage` 保存，自动刷新 OAuth

## 加载建议 Workflow

```
1. 任何任务 → 先读 code-skill.md（本文件，当前你在看）了解索引
2. 判断任务类型 → 参照"按任务类型组合加载"表选择子技能
3. 读入需要的子技能 md 文件（按需，不要全读）
4. 按其中的规则/流程/约束执行
5. 对照"核心价值观提炼"自检
6. 验证（参见 07）后才能说"完成"
```

## 项目适配说明

本技能集保留 Claude Code 类 AI 编程助手在工程执行、工具选择、任务管理、验证纪律与安全边界方面的成熟能力，同时已适配当前项目的工作流：

- 以 `Base/` 对象包作为需求、设计、实现、测试、变更的唯一追溯基准
- 代码实现必须绑定 `RQ / PD / TD / FT / IMP`，缺失时先回到 PM 链路补齐
- UI、数据库、接口等领域问题按需挂载对应领域 Skill
- 文件操作优先使用专用工具，终端命令按当前运行环境约束执行
- 验证结果必须回写到 `TEST-*` 或对应实现记录中

## 维护说明

- 所有子技能均为**中文描述 + 原文保留关键英文术语**（便于索引代码与配置）
- 每个 md 文件顶部的 YAML frontmatter 可被直接用作 `.claude/skills/` 中的独立 skill
- 若要进一步定制，将相应 md 文件放到 `.claude/skills/<name>/SKILL.md` 并调整 frontmatter 即可被 Claude Code 自动发现

---

**调用本 skill**：`/code-skill [topic]` 或直接让助手 `Read code-skill.md`，然后按需 `Read code-skills/XX-*.md` 加载具体领域规则。
