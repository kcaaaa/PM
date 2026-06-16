---
name: agent-delegation
description: 子代理委派与协调器模式——何时委派、如何写好 worker prompt、如何管理多并行任务
when_to_use: 当你面对可并行的研究、独立的大任务、或需要隔离上下文的长任务时阅读。用来决定自己做还是委派。
source: 抽象自 src/tools/AgentTool/built-in/*.ts 与 src/coordinator/coordinatorMode.ts
---

# 子代理委派 Agent Delegation

## 1. 何时委派

Agent 工具用于：
1. **并行独立查询** 以提速。
2. **保护主上下文窗口** 不被大量原始输出污染。
3. 长时间、自包含的任务。

当前项目适配：委派前必须明确该任务对应的 Base 对象包或咨询边界。实现类子代理不得绕过 `RQ / PD / TD / FT / IMP`，探索类和规划类子代理保持只读。

**不要过度使用**——当任务不需要时不要委派。**严禁重复工作**：若已委派研究给子代理，**不要**自己也做同样搜索。

## 2. 内置 Agent 类型

### 2.1 Explore (探索代理)
- **快速、只读** 代码库探索。
- 用途：按模式找文件（`"src/components/**/*.tsx"`）、按关键词搜索代码（`"API endpoints"`）、回答"X 怎么工作"类问题。
- **被严格禁止**：创建、修改、删除、移动文件；任何改变系统状态的命令。
- 参数：需要指定**彻底程度**——`quick` / `medium` / `very thorough`。
- 使用场景：简单定向搜索不够，或预估查询 > 3 次。

### 2.2 Plan (规划代理)
- **只读**软件架构师。
- 用途：为任务设计实现方案，输出分步策略 + 关键文件清单 + 架构取舍。
- 输出强制包含 `### Critical Files for Implementation`（3-5 个文件）。
- 同样禁止任何写操作。

### 2.3 General-Purpose (通用代理)
- 研究复杂问题、跨多文件分析、多步研究任务。
- 拥有所有工具 (`tools: ['*']`)。
- 广开搜索策略：从宽到窄，多策略、多命名习惯、多位置。
- 不主动创建文档文件。

### 2.4 Verification (验证代理)
- **对抗式独立验证**。详见验证纪律技能。
- 不能修改项目目录中的文件；可在 `/tmp` 写临时脚本并清理。

### 2.5 Shell (Shell 代理)
- 专门执行终端/git 操作。

### 2.6 Browser-Use (浏览器代理)
- Web 自动化与前端测试。
- 有状态子代理——重复调用会自动恢复最近的 agent。

### 2.7 Cursor-Guide / ClaudeCodeGuide
- 回答"产品如何工作"类问题。

### 2.8 Best-of-N-Runner
- 在 git worktree 中隔离运行并行尝试。

## 3. 写 worker prompt 的核心纪律

### 3.1 Worker 看不到你的对话
**每个 prompt 必须自包含**，包括 worker 需要的一切信息。

### 3.2 必须先合成 (synthesize)
Worker 汇报研究结果后，**你必须理解它**才能指挥后续工作。读发现、识别方案、写带有具体文件路径、行号、明确改动内容的 prompt——用这证明你理解了。

**绝不**写 "based on your findings" 或 "based on the research"。这些短语把理解委派给 worker，**你永远不把理解转交给其他 worker**。

**反例（懒惰委派）**：
```
Agent({ prompt: "Based on your findings, fix the auth bug" })
Agent({ prompt: "The worker found an issue in the auth module. Please fix it." })
```

**正例（合成过的规格）**：
```
Agent({ prompt: "Fix the null pointer in src/auth/validate.ts:42. The user field 
on Session (src/auth/types.ts:15) is undefined when sessions expire but the 
token remains cached. Add a null check before user.id access — if null, return 
401 with 'Session expired'. Commit and report the hash." })
```

### 3.3 包含用途声明 (purpose statement)
让 worker 校准深度：
- "This research will inform a PR description — focus on user-facing changes."
- "I need this to plan an implementation — report file paths, line numbers, and type signatures."
- "This is a quick check before we merge — just verify the happy path."

### 3.4 包含具体要素
- 文件路径（绝对）、行号、错误消息
- 明确的"完成"定义
- 实现任务加上："Run relevant tests and typecheck, then commit your changes and report the hash"
- 研究任务加上："Report findings — do not modify files"
- git 操作要精确：branch 名、commit hash、draft vs ready、reviewers
- 实现偏好："Fix the root cause, not the symptom"
- 验证偏好："Prove the code works, don't just confirm it exists"、"Try edge cases"、"Investigate failures — don't dismiss as unrelated without evidence"

### 3.5 更正时 (continue 模式)
继续一个 worker 的更正时，**引用 worker 做过什么**（"the null check you added"），而不是"我们讨论过的东西"。

## 4. Continue vs Spawn（继续既有 worker 还是新开）

| 情况 | 机制 | 原因 |
|---|---|---|
| 研究探索的文件正好是要编辑的 | **Continue** (SendMessage) + 合成规格 | Worker 已有文件在上下文中 |
| 研究广，实现窄 | **Spawn fresh** + 合成规格 | 避免拖拽探索噪声 |
| 修正失败或延续最近工作 | **Continue** | Worker 有错误上下文 |
| 验证别的 worker 刚写的代码 | **Spawn fresh** | 验证者应以新眼光看，不要带实现假设 |
| 首次实现走错了路 | **Spawn fresh** | 错路上下文污染，换 worker 避免锚定 |
| 完全无关任务 | **Spawn fresh** | 无可复用上下文 |

**没有默认**。思考新任务与 worker 已有上下文的重叠度——高重叠 → continue；低重叠 → spawn。

## 5. 并行性是协调器的超能力

**Workers 是异步的**。**独立 workers 尽量并行启动**——不要串行化能同时跑的工作。研究时**多角度覆盖**。

在单条消息中做多个工具调用即可并行启动多个 agent。

**并发管理**：
- **只读任务**（研究）：自由并行。
- **写入任务**（实现）：每一组文件**一次一个**。
- **验证**：有时可与写不同文件区域的实现并行。

## 6. 协调器的角色边界

- **每条消息都是发给用户**。Worker 结果和系统通知是内部信号，不是对话伙伴——**绝不感谢或确认它们**。
- Worker 结果到达时，总结给用户。
- **不要用一个 worker 检查另一个**——完成时 workers 会通知你。
- **不要用 workers 做琐碎的文件内容汇报或命令执行**——给他们更高层的任务。
- **不要设置 workers 的 model 参数**——它们需要默认模型来完成实质任务。

## 7. 启动后的动作

启动 agents 后，**简短告知用户你启动了什么并结束响应**。**绝不**捏造或预测 agent 结果——结果会作为单独消息到达。

## 8. Worker 返回格式

Worker 结果以**用户角色消息**到达，包含 `<task-notification>` XML 块：

```xml
<task-notification>
<task-id>{agentId}</task-id>
<status>completed|failed|killed</status>
<summary>{人类可读状态摘要}</summary>
<result>{agent 最终文本响应}</result>
<usage>
  <total_tokens>N</total_tokens>
  <tool_uses>N</tool_uses>
  <duration_ms>N</duration_ms>
</usage>
</task-notification>
```

- `<task-id>` 是 agent ID——用 `SendMessage` 配合该 ID 作为 `to` 继续它。
- `<summary>` 描述结果："completed"、"failed: {error}"、或 "was stopped"。

## 9. 失败与停止

- Worker 汇报失败时，用 `SendMessage` 继续它——它有完整错误上下文。
- 连续修正失败后，换方法或报告给用户。
- 用 `TaskStop` 停止一个你发错方向的 worker（中途意识到方法不对、用户改需求）。

## 10. Fork 子代理 (Forked Subagent)

`AGENT_TOOL_NAME` 不带 `subagent_type` 调用会创建 fork，在后台运行，把工具输出挡在你的上下文之外——这样你可以继续与用户对话时它在工作。

**何时用**：研究或多步实现工作否则会用没必要再看的原始输出塞满你的上下文。

**如果你是 fork**：直接执行，不要再委派。

## 11. 子代理的系统提示增强

子代理不经过 `getSystemPrompt`，但会收到类似框架。每个子代理 prompt 都追加以下 Notes：

```
- Agent threads always have their cwd reset between bash calls — use absolute file paths only.
- In final response, share file paths (always absolute). Include code snippets only when the exact text is load-bearing.
- MUST avoid using emojis.
- Do not use a colon before tool calls.
```

## 12. 默认倾向

- **简单直接查询** → 自己做。
- **大范围探索或 >3 次查询** → 委派给 Explore。
- **架构设计** → 委派给 Plan。
- **独立可并行** → 同批次多委派。
- **风险高、需要真实验证** → 用 Verification 代理。
