---
name: file-operations
description: 文件读/写/编辑的规则与安全约束
when_to_use: 当你准备读文件、编辑现有文件、创建新文件、或搜索文件时阅读。
source: 抽象自 src/tools/FileReadTool, FileEditTool, FileWriteTool, GlobTool, GrepTool 的 prompt.ts
---

# 文件操作 File Operations

## 1. 绝对路径原则

**所有文件操作必须用绝对路径**，不接受相对路径。

- Agent / 子代理线程在终端调用间可能**重置 cwd**——相对路径容易失败
- 工具调用必须使用绝对路径；面向用户分享最终产物时使用可打开的文件链接或明确文件名
- Windows 路径要注意空格、中文与引号规则；终端命令优先使用单引号包裹路径

## 2. Read 规则

### 2.1 基础
- `file_path` 必须**绝对**
- 默认从头读 2000 行；倾向**整体读**（不提供 offset/limit）
- 支持：`.ipynb`（Jupyter notebook，返回所有 cell 与输出）、图片（PNG/JPG，多模态呈现）、PDF（大 PDF 必须传 `pages`，最多 20 页/次）
- **只能读文件不能读目录**——读目录用专用目录列表工具；不要用终端命令替代
- 结果以 `cat -n` 格式返回，行号从 1 开始

### 2.2 行号前缀规则
Read 返回的每行形如 `LINE_NUMBER|LINE_CONTENT`：
- `LINE_NUMBER|` 是 **metadata 不是代码内容**
- 编辑时**严禁**把行号前缀包含进 `old_string` 或 `new_string`
- 缩进只看 `|` 之后

### 2.3 空文件
读空文件会收到 `"File is empty."` 系统提醒，而非文件内容。

### 2.4 Re-read 缓存
再次 Read 未改动的文件，会返回：
> "File unchanged since last read. The content from the earlier Read tool_result in this conversation is still current — refer to that instead of re-reading."

**不要重复读未变文件**。

### 2.5 大文件策略
对 > 1K 行的大文件：
- 已知具体符号 → 用 `Grep` 精确查
- 广义查询 → 用 SemanticSearch 聚焦该文件
- **不要盲目全读**

## 3. Edit 规则

### 3.1 前置条件
**必须至少 Read 过一次才能 Edit**——否则 Edit 会报错。

### 3.2 精确匹配
- `old_string` 必须**唯一** → 不唯一时：
  - 要么加更多上下文使其唯一
  - 要么设 `replace_all: true` 改所有实例（适合变量重命名）
- **保留精确缩进**（tabs/spaces 原样）——注意行号前缀之后的才是真实内容

### 3.3 最小唯一字符串
**倾向用"最小唯一片段"**，通常 **2-4 行**足够。**避免**无脑包含 10+ 行上下文。更短的匹配：
- 降低意外冲突概率
- 让 diff 更清晰
- 减少 token 消耗

### 3.4 编辑原则
- **优先编辑既有文件**，绝不写新文件除非明确要求
- 只在用户显式请求时添加 emoji
- 编辑时保留**精确缩进**

## 4. Write 规则

### 4.1 创建文件的门槛
- **除非绝对必要，不创建新文件**
- 始终**优先编辑现有文件**
- **绝不主动创建文档文件**（`*.md`、README）——只在用户显式要求时创建

### 4.2 覆盖行为
Write 会覆盖已存在文件。对敏感文件**先 Read 确认再 Write**。

## 5. Glob 规则（按文件名找）

- 模式不以 `**/` 开头会**自动前置** `**/` 启用递归
- 例子：
  - `"*.js"` → `"**/*.js"`
  - `"**/test/**/test_*.ts"`
- 结果按**修改时间**排序

## 6. Grep 规则（按内容搜）

### 6.1 特性
- 基于 ripgrep
- 支持完整正则（`log.*Error`、`function\s+\w+`）
- 通过 `glob` 参数（`*.{ts,tsx}`）或 `type` 参数（`js`、`py`）过滤文件
- 三种输出模式：
  - `content`（默认）：显示匹配行
  - `files_with_matches`：只显示文件路径
  - `count`：显示匹配次数
- **字面字符需转义**（Go 中 `interface{}` → `interface\{\}`）
- **多行匹配**：默认单行；跨行用 `multiline: true`

### 6.2 输出限制
- 结果封顶到几千行以保证响应；截断时显示 "at least" 计数但准确
- 用 `head_limit` + `offset` 分页
- `-A` / `-B` / `-C` 提供上下文行（需要 `content` 模式）

### 6.3 优先于 Shell 的 grep/rg
用 `Grep` 工具，除非嵌入式工具别名了 `find` / `grep` 到 bfs/ugrep。

## 7. 选择搜索工具

| 场景 | 工具 |
|---|---|
| 已知具体文件 | `Read` |
| 按名称模式找 | `Glob` |
| 按内容搜索 | `Grep` |
| 需要回答"怎么工作" | `SemanticSearch`（若可用）或委派给 `Explore` agent |
| 深度多步探索 | 委派给 `Explore` (`subagent_type=Explore`) |
| 单词搜索 | `Grep`（**不要**用 SemanticSearch） |

### 反例
- 搜索 "AuthService" 单词 → 用 `Grep` 不用 `SemanticSearch`
- 找 `Button.tsx` → 用 `Glob` 不用 `SemanticSearch`
- 问"AuthService 做什么 + 怎么工作" → **拆成两次并行** SemanticSearch，不要一次问多事

## 8. 专用工具优先总结

见《02-tool-usage.md》第 1.1 节对照表。核心：**Bash 只用于没有专用工具覆盖的 shell 操作**。

## 9. Notebook 编辑

`.ipynb` 用专用 `NotebookEdit`（不要用 `Edit`/`Write`）：
- 可编辑现有 cell、创建新 cell
- **cell 索引 0-based**
- 仔细设 `is_new_cell` flag
- **不支持 cell 删除**——可将内容置空
- `old_string` / `new_string` 应是有效 cell 内容，不带 JSON 语法
- `old_string` 必须唯一：包含**至少 3-5 行前后上下文**
- markdown cell 有时保存为 "raw"——正常，不要改

## 10. 在引用代码时

- 引用**已存在**代码 → 用 `startLine:endLine:filepath` 代码块格式（不加语言标签）
- 引用**新代码/示例** → 标准 markdown code block（带语言标签）
- **行内引用文件**用反引号：`` `src/file.ts` ``
- 不要混用两种格式
- 代码块前**始终空一行**
- 代码块中**不要内嵌行号**

## 11. 文件路径引用格式

在叙述中引用函数/代码位置时用 `file_path:line_number` 让用户可跳转：
> "bug 在 `src/auth/validate.ts:42`"

引用 GitHub issues/PRs 用 `owner/repo#123` 格式让它们渲染为链接。
