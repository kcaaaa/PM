---
name: tool-usage
description: 工具使用的基本规则——专用工具优先、并行执行、避免 Bash 重复造轮子
when_to_use: 当你在思考"该用哪个工具做这件事"时阅读。尤其在面临读文件、编辑、搜索、列目录这类有专用工具的操作时。
source: 抽象自 src/constants/prompts.ts 的 getUsingYourToolsSection 和各 Tool/prompt.ts
---

# 工具使用规则 Tool Usage

## 1. 黄金法则：专用工具 > 通用 Shell

**当有专用工具时，禁止用 Bash / PowerShell 做同样的事**。使用专用工具让用户能更好理解和审查你的工作。这一点对协助用户至关重要。

> 当前项目适配：默认运行环境为 Windows，终端命令按 PowerShell 语法执行；文件读取、搜索、创建、编辑仍必须优先使用专用工具，不得用 PowerShell 重定向、here-string、`Set-Content` 等方式绕过文件工具。

### 1.1 文件操作对照

| 意图 | 必须使用 | 不要使用 |
|---|---|---|
| 读文件 | `Read` | `cat` / `head` / `tail` / `sed` |
| 编辑文件 | `Edit` / `StrReplace` | `sed` / `awk` |
| 创建文件 | `Write` | heredoc / here-string / `echo >` / `Set-Content` |
| 按名称找文件 | `Glob` | `find` / `ls` / `Get-ChildItem` |
| 按内容搜索 | `Grep` | `grep` / `rg` / `Select-String`（除非是嵌入式工具） |

终端 **只保留给需要 shell 执行的系统命令与终端操作**（git、npm、docker 等）。不确定且存在相关专用工具时，默认用专用工具，只在绝对必要时退回终端。

### 1.2 常见专用工具的隐含规则

**Read 工具**：
- 文件路径必须**绝对路径**，不接受相对路径。
- 默认从文件开头读最多 2000 行，建议读整个文件而非分片。
- 可读图片（PNG/JPG 等）、PDF（大 PDF 须提供 pages 参数）、Jupyter notebook。
- 只能读文件不能读目录，读目录用专用目录列表工具；不要用终端命令替代。
- 编辑任何文件前，**至少必须 Read 过一次**，否则 Edit 会报错。

**Edit 工具**：
- 必须严格保留缩进（tab/spaces）。行号前缀不属于内容本身。
- `old_string` 必须唯一。不唯一时要么加更多上下文，要么用 `replace_all`。
- **默认使用最小唯一字符串**（通常 2-4 行即可），不要无脑加 10+ 行。

**Write 工具**：
- **优先编辑现有文件**，只有绝对必要才写新文件。
- **绝不主动创建文档文件** (`*.md`、README)，除非用户显式要求。
- 仅在用户显式要求时添加 emoji。

## 2. 并行优先

**可以在单次响应中调用多个工具**。如果多个工具调用**彼此独立**，**必须并行执行**以提高效率。

- 独立操作 → 批量并行（例如同时 `git status` / `git diff` / `git log`）。
- 有依赖关系 → 顺序执行（例如必须先创建目录再复制文件）。
- 使用 `;` 只在顺序执行且不在意前序失败时；其余用 `&&`。

**永远避免等价串行：** 先读 `file A`、拿到结果、再读 `file B`（但 B 不依赖 A 的结果），这是反模式，应一次性并行 Read 两个文件。

## 3. 长时任务与后台

- 命令默认 `block_until_ms` (30s) 内完成；超时会被移到后台，输出写到终端文件。
- 开发服务器、监听器、任何长跑进程 → 立即后台 (`block_until_ms: 0`)。
- 不要用 `&` 在命令末尾，改用 `run_in_background` 参数。
- 用 `AwaitShell` 轮询后台任务，首选指数退避的轮询策略；命令若能自带状态输出则更易监控。

## 4. 系统命令的 Shell 平台差异

Windows 平台使用 PowerShell 时：
- 注意路径分隔符差异；在 prompts 中倾向用 Unix shell 语法提示（`/dev/null` 而非 `NUL`，正斜杠路径）。
- 需要为 Windows 特有提供 PowerShell 工具。

## 5. 禁止的 Shell 模式

即使在 Bash 工具中也不应执行：
- `git rebase -i`、`git add -i` 等需要交互输入的 `-i` 命令——不支持。
- `git rebase --no-edit`——该标记在 rebase 中无效。
- 大仓库中 `git status -uall`——会 OOM。
- 使用管道/重定向作为文件写入手段（应用 Write）。

## 6. 搜索策略

**简单、明确的搜索**（某具体文件/类/函数）：直接用 `Glob` / `Grep`。

**广泛探索或深度研究**：使用 Agent (`subagent_type=Explore`) 委派。
- Explore 更慢，**只在简单直接搜索不够或预估查询次数会超过最小阈值 (约 3+) 时使用**。
- 对于**大文件 (>1K 行)**：用 `Grep` 精确查找已知符号，或用 SemanticSearch 聚焦该文件。

## 7. MCP 工具

- MCP 工具描述存储为 JSON 文件：`<mcps-root>/<server>/tools/<tool-name>.json`。
- **调用任何 MCP 工具前必须读其 schema**——这不是可选项，schema 包含必需参数、类型、用法。
- 发现 MCP server 有 `mcp_auth` 时，必须调用它以便用户授权。**一次只 auth 一个 server**。

## 8. 工具调用的文本礼仪

- 工具调用前**不要加冒号**：不要写 "Let me read the file:"，应为 "Let me read the file." 然后调用工具。
- 不要在面向用户的文本里展示工具名。用自然语言描述工具在做什么。

## 9. 工具调用失败的处理

- 用户拒绝工具时，**不要重复完全相同的调用**。思考为何被拒，调整方法。
- 不理解拒绝原因时，用 `AskUserQuestion` 问清楚。
