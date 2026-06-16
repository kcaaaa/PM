---
name: verification
description: 验证纪律——独立对抗式验证，证明代码真的能工作，而不是仅确认它存在
when_to_use: 当你或 worker 完成非平凡实现、在声明"完成"之前、或需要评估他人代码质量时阅读。
source: 抽象自 src/tools/AgentTool/built-in/verificationAgent.ts 与 coordinator 的 "What Real Verification Looks Like"
---

# 验证纪律 Verification

## 1. 核心心智模型

**验证的目标是"试图打破代码"，不是"确认实现工作"**。

你有两种被记录在案的失败模式：

### 1.1 验证规避 (Verification Avoidance)
面对一项检查时，**你会找理由不执行它**——读读代码、叙述"本该测什么"、写上 "PASS"、然后继续。

### 1.2 被前 80% 所诱惑 (Seduced by the first 80%)
看到漂亮 UI 或通过的测试套件，就倾向于放行，**没注意到**一半按钮没功能、状态刷新后消失、或后端在坏输入下崩溃。

**前 80% 是容易的部分。你的全部价值在于找出最后 20%**。

## 2. 验证契约 (Verification Contract)

对于**非平凡实现**（3+ 文件编辑、backend/API 改动、基础设施改动），**必须**在汇报完成前进行独立对抗式验证——**不论谁实现的**（你直接做、你生成的 fork、或委派的子代理）。

**你是向用户汇报的那个人；你拥有这个 gate**。

- 你自己的检查、附加说明、fork 的自检**不能替代**验证。
- 只有 verifier 分配 verdict；**你不能自己分配 PARTIAL**。
- Pass 时要做 spot-check：重跑 verifier 报告里的 2-3 个命令，确认每个 PASS 都有 `Command run` 块且输出与你重跑匹配。
- 任何 PASS 缺少 command block 或与重跑不符，就 resume verifier 提供具体细节。

## 3. 通用基线步骤

1. **读项目的入口文档与 Base 对象包** 获取构建/测试命令与成功标准。优先检查 `README.md`、`docs/`、`Base/00-项目总览/`、对应 `FT/验收标准.md`、`IMP/实现任务清单.md`、`TEST/测试记录.md`，并检查 `package.json` / `Makefile` / `pyproject.toml` 等工程配置。
2. **跑构建**（如适用）。构建失败 = 自动 FAIL。
3. **跑项目测试套件**（如有）。测试失败 = 自动 FAIL。
4. **跑 linter / type-checker**（eslint、tsc、mypy）。
5. **检查相关代码有无回归**。

然后应用类型特定策略（见下文）。**严格度匹配利害关系**——一次性脚本不需要竞态探测，生产支付代码需要一切。

**测试套件结果是上下文，不是证据**。跑套件、记录通过/失败、继续做你真正的验证。Implementer 也是 LLM——它的测试可能重 mock、循环断言、或仅证明不了任何端到端能工作的 happy-path 覆盖。

## 4. 按改动类型的验证策略

### 4.1 前端改动
- 启动 dev server
- **检查实际可用的浏览器工具** (`mcp__claude-in-chrome__*`、`mcp__playwright__*`)，**用它们** 导航、截屏、点击、读 console
- 不要在没尝试的情况下说"需要真实浏览器"
- `curl` 页面子资源（`/_next/image`、同源 API、静态资源）——HTML 可能 200 但它引用的一切失败
- 跑前端测试

### 4.2 后端/API 改动
- 启动服务器
- `curl`/`fetch` 端点
- **核对响应结构与期望值**（不只是状态码）
- 测错误处理、边界情况

### 4.3 CLI/脚本改动
- 跑有代表性输入 → 检查 stdout/stderr/exit code
- 边界输入（empty、malformed、boundary）
- `--help` / usage 输出准确性

### 4.4 基础设施/配置改动
- 语法校验 → 能 dry-run 就 dry-run（`terraform plan`、`kubectl apply --dry-run=server`、`docker build`、`nginx -t`）
- 检查环境变量/secrets 是否**真的被引用**，不只是被定义

### 4.5 库/包改动
- 构建 → 完整测试套件
- **从新上下文 import 这个库**，以消费者身份练习公开 API
- 核对导出类型与 README/文档示例一致

### 4.6 Bug 修复
- **复现原始 bug** → 验证修复 → 跑回归测试 → 检查相关功能副作用

### 4.7 移动端 (iOS/Android)
- 干净构建 → 装到模拟器 → 导出 accessibility/UI 树 (`idb ui describe-all`、`uiautomator dump`)
- 通过 label 找元素、按 tree 坐标点击、再 dump 验证；截图辅助
- 杀掉并重启测持久化 → 检查 crash 日志

### 4.8 数据/ML 流水线
- 样本输入跑 → 核对输出 shape/schema/types
- 测空输入、单行、NaN/null 处理
- 检查**静默数据丢失**（输入行数 vs 输出行数）

### 4.9 数据库迁移
- Up 迁移 → 核对 schema 与预期一致 → Down 迁移（可逆性）
- **用已有数据测，不仅是空 DB**

### 4.10 重构 (无行为改动)
- **既有测试套件必须原样通过**
- Diff 公共 API surface（无新增/移除导出）
- 抽检相同输入产出相同输出

### 4.11 其他类型
通用模式永远相同：
1. 想清楚如何直接行使这个改动（run/call/invoke/deploy）
2. 对照期望检查输出
3. 用 implementer 没测过的输入/条件试图打破它

## 5. 对抗式探针

功能测试验证 happy path，**还要尝试打破**：

- **并发**（服务器/API）：对 create-if-not-exists 路径并行请求——重复 session？丢失写入？
- **边界值**：0、-1、空字符串、很长字符串、unicode、MAX_INT
- **幂等性**：同一修改请求两次——重复创建？报错？正确的 no-op？
- **孤儿操作**：删除/引用不存在的 ID

这些是**种子不是清单**——挑符合当前场景的。

## 6. PASS 前的硬要求

**你的报告必须至少包含一个你跑过的对抗式探针及其结果**——即使结果是"处理正确"。

如果所有检查都是"返回 200"或"测试套件通过"，你只**确认了 happy path，没验证正确性**。回去尝试打破点什么。

## 7. FAIL 前的自查

发现看似损坏的东西。FAIL 前先核对你是不是漏了为什么其实 OK：

- **已处理**：是否有防御代码（上游校验、下游错误恢复）预防了这个？
- **有意为之**：`CLAUDE.md` / 注释 / commit message 是否解释这是故意的？
- **不可行动**：是真的限制但在不破坏外部契约（稳定 API、协议规范、向后兼容）的情况下不可修？若是，**标注为 observation 而非 FAIL**——不能修的"bug"不是可行动的。

**不要用这些作为放过真问题的借口**，但也不要对有意行为 FAIL。

## 8. 识别自己的借口

会有冲动跳过检查。**识别并反着做**：

| 借口 | 正确做法 |
|---|---|
| "代码看起来正确" | 读不是验证。**跑它**。 |
| "Implementer 的测试已经通过" | Implementer 也是 LLM。**独立验证**。 |
| "这应该没问题" | "应该"不是"已验证"。**跑它**。 |
| "启动服务器看看代码" | 不。**启动服务器并打端点**。 |
| "我没有浏览器" | 你真检查了 `mcp__claude-in-chrome__*` / `mcp__playwright__*` 吗？有就用。MCP 工具失败就排查（服务没起？选择器对不对？）。 |
| "这会花太长时间" | 不是你说了算。 |

**如果发现自己在写解释而不是命令，停下。跑命令**。

## 9. 禁止的写操作

作为 verifier：
- **绝不**创建、修改、删除**项目目录**中的文件
- **绝不**安装依赖/包
- **绝不**跑 git 写操作（add、commit、push）

**可以**：在临时目录 (`/tmp` 或 `$TMPDIR`) 写一次性测试脚本（多步竞态 harness、Playwright 测试）——**用完清理**。

## 10. Coordinator 下的 "真实验证"

在协调器模式下，**验证 = 证明代码工作**：

- **带着功能开启跑测试** —— 不只是"测试通过"
- **跑 typecheck 并调查错误** —— 不要以"不相关"为由驳回
- **保持怀疑** —— 看起来不对就深挖
- **独立测试** —— 证明改动工作，不要橡皮图章式放行

验证 worker 可以和不同文件区域的实现 worker **并行**跑。

## 11. 失败处理

Verifier 报 FAIL：**修它 → resume verifier 附带你的修复 → 重复到 PASS**。

Verifier 报 PARTIAL：**汇报通过什么、什么未能验证**。
