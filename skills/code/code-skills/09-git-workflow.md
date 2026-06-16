---
name: git-workflow
description: Git 提交、PR 创建的完整工作流与安全协议
when_to_use: 当用户请求 commit、push、创建 PR、cherry-pick、rebase 等 git 操作时阅读。
source: 抽象自 src/tools/BashTool/prompt.ts 的 getCommitAndPRInstructions
---

# Git 工作流 Git Workflow

## 1. 核心原则

**只在用户明确要求时才创建 commit**。不清楚就先问。**不要**因为觉得"有价值"就主动提交——用户会觉得你越权。

## 2. Git 安全协议 (硬约束)

### 绝不
- 更新 git config
- 跑破坏性 git 命令（`push --force`、`reset --hard`、`checkout .`、`restore .`、`clean -f`、`branch -D`）**除非用户显式要求**
- 跳过 hooks（`--no-verify`、`--no-gpg-sign`）**除非用户显式要求**
- 对 main/master 强推 → 用户要求时**警告**
- 未经允许提交 → 主动会让用户反感
- 使用 `git add -A` / `git add .`：易误入 `.env`、凭证、大二进制。**按文件名具体加**

### CRITICAL: amend 规则
**始终创建新 commit，不 amend**，除非：
1. 用户明确要求；或
2. Commit **成功了**但 pre-commit hook 自动修改了文件需要包含
3. **且** HEAD commit 是本次对话你创建的 (`git log -1 --format='%an %ae'`)
4. **且** Commit **未推送** (`git status` 显示 "Your branch is ahead")

**Pre-commit hook 失败时绝不 amend**：hook 失败意味着 commit 没发生，amend 会改前一个 commit 破坏历史。正确做法：**修问题 → 重新 stage → 建新 commit**。

**已推送远程时绝不 amend**，除非用户明确要求（需要 force push）。

## 3. 标准 commit 工作流

### 第 1 步：并行收集状态
**单条消息**中并行调用多个终端工具：
- `git status`（**永不**用 `-uall`：大仓库会 OOM）
- `git diff`——看 staged + unstaged 改动
- `git log`——看近期 commit 消息以跟随风格

当前项目适配：终端命令按 Windows PowerShell 语法执行；不要使用 Bash heredoc、`&&` 链式语法或未加引号的中文/空格路径。

### 第 2 步：分析改动并起草消息
- **总结改动性质**：feat / enhancement / fix / refactor / test / docs。准确反映改动和目的。
  - "add" = 全新功能
  - "update" = 对现有功能的增强
  - "fix" = bug 修复
- **不要提交疑似含有 secret 的文件**（`.env`、`credentials.json`）。用户特意要求时要**警告**。
- 起草**简洁的 1-2 句** commit 消息，聚焦**为什么** 而非做了什么。
- 准确反映改动与其目的。

### 第 3 步：并行执行
- `git add <specific files>`
- `git commit -m "..."`（使用当前终端兼容写法）
- **之后** `git status` 验证成功（依赖 commit 完成，顺序执行）

### 第 4 步：处理 pre-commit hook 失败
若 commit 因 hook 失败：**修问题 → 建新 commit**（不 amend）。

### 重要 notes
- **绝不**运行额外命令去读或探索代码，除了必要的 git 终端命令
- **绝不**使用 `TodoWrite` 或 `AgentTool`（git 提交流程内）
- **不要**推到远程除非用户明确要求
- **绝不**用 `-i` flag 的 git 命令（`git rebase -i`、`git add -i`）——需要交互输入，不支持
- **绝不**用 `--no-edit` 与 `git rebase` 一起——该 flag 在 rebase 中无效
- **没改动就不要空 commit**
- **提交消息必须使用当前终端兼容的安全写法**。在 Windows PowerShell 中优先使用单行 `git commit -m 'Commit message here.'`；需要多行消息时先创建临时消息文件或使用平台支持的参数方式，避免 Bash heredoc。

## 4. PR 创建工作流

### 第 1 步：并行收集分支状态
单条消息中并行：
- `git status`（禁用 `-uall`）
- `git diff`——staged + unstaged
- 检查当前分支是否跟踪远程，是否与远程同步（决定是否要 push）
- `git log` **+** `git diff [base-branch]...HEAD`——理解从分叉点以来的完整 commit 历史

### 第 2 步：分析所有改动起草 PR
- **分析 ALL 相关 commits**，不只是最新一条！所有将被包含在 PR 中的 commits
- **PR title 短**（< 70 字符）
- 用 **description/body 放细节**，不要塞 title

### 第 3 步：并行执行
- 需要时创建新分支
- `git push -u` 推到远程（若需要）
- `gh pr create` 的 body 必须使用当前终端兼容的安全写法；在 Windows PowerShell 中优先使用 `--body-file` 指向临时 PR 描述文件，避免 Bash heredoc。

### PR 中重要注意
- **不要**使用 `TodoWrite` 或 `AgentTool`
- **返回 PR URL** 让用户看到
- 不要自己 merge PR，除非用户明确要求

## 5. 其他常见 GitHub 操作

统一用 `gh` via Bash：
- 查看 PR 评论：`gh api repos/foo/bar/pulls/123/comments`
- 处理 issues、checks、releases 也用 `gh`
- 若给了 GitHub URL，用 `gh` 拉需要的信息

## 6. 内部版 / Skill 化 (ant)

内部版会把上面流程进一步封装为 bundled skills：
- `/commit` — 用 staged 改动创建 commit
- `/commit-push-pr` — commit、push、创建 PR

这些 skill 自带 git 安全协议、消息格式、PR 创建逻辑。

**建 PR 前**：
- 先跑 `/simplify` 审阅改动
- 再做端到端测试（如 `/tmux` 测交互功能）

## 7. Undercover 模式（公开仓库安全）

在公开/开源仓库工作时（内部使用场景），**commit message、PR title、PR body 绝不能包含任何内部信息**：

### 绝不在 commit/PR 中写
- 内部模型代号（动物名 Capybara、Tengu 等）
- 未发布的模型版本号（`opus-4-7`、`sonnet-4-8`）
- 内部 repo/项目名
- 内部工具、Slack 频道、短链（`go/cc`、`#claude-code-*`）
- "Claude Code" 短语或 AI 暗示
- 任何模型版本提示
- Co-Authored-By 行或其他归属

**像人类开发者一样写 commit 消息——只描述代码改动做了什么**。

**正例**：
- "Fix race condition in file watcher initialization"
- "Add support for custom key bindings"

**反例**（绝不写）：
- "Fix bug found while testing with Claude Capybara"
- "1-shotted by claude-opus-4-6"
- "Generated with Claude Code"
- "Co-Authored-By: Claude Opus 4.6 <…>"

## 8. Commit/PR 归属 (Attribution)

- 外部版可能在 commit/PR 尾部自动追加标识行（由 `getAttributionTexts` 决定）
- 内部版 + Undercover 激活时**全部剥离**

## 9. 并行优化要点

git 工作流中大量可并行的命令：
- status + diff + log → 一次性并行
- add 具体文件 + commit + 随后 status（status 顺序）→ 并行主体部分
- push + create PR（若 push 先完成）→ 视依赖决定

## 10. 信息探索禁令

**一旦进入 commit/PR 流程：除 git 命令外不要再读或探索代码**。这个流程是关于"把已做的改动描述好"，不是"再做更多改动"。
