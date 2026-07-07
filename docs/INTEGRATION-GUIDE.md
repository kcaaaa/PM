# Agent / Skill / Base 集成指南

## 概述

本项目是一套面向非开发者、产品经理、业务负责人和 AI 编程工具的纯 Markdown 文件型 Agent / Skill 协作系统。架构以 `Base/` 作为唯一项目认知数据库，通过 Orchestrator、PM Agent、QA Agent、Dev Agent、Domain Agent 与 BA / PM / Code / DB / UI / Council / Extension / Prompt 等 Skill 协同工作。

默认使用者不需要懂代码、路径或技术实现方式，只需要描述业务想法、问题现象、使用感受和期望效果；AI 负责识别意图、维护 Base、定位代码工程文件、执行实现与验证，并用用户能理解的语言说明结果。

本项目无需安装依赖、无需 CLI、无需运行服务。AI 编程工具直接读取根目录 `SKILL.md`、`智能体提示词.md`、`agents/`、`skills/`、`protocols/`、`docs/` 中的 Markdown 规则即可工作。

核心模式：

```text
人类输入方向
→ AI 识别意图与当前 Base 状态
→ AI 输出 3-5 个候选方案并推荐最优方案
→ 人类选择 / 修正 / 否定
→ AI 写入或更新 Base 对象包
→ PM 推进 RQ → VR → PD → TD → FT
→ QA Agent 执行实现前验证
→ QA 通过并经人类确认后创建 IMP
→ Dev Agent 执行实现与验证
→ TEST 测试验收
→ FB 反馈 / CR 变更 / 回归测试
```

---

## 当前目录结构

```text
全流程产品/
├── README.md                     # 项目总览
├── SKILL.md                      # 顶层调度入口
├── 技能使用场景介绍.md             # 使用场景与触发方式
├── 智能体提示词.md                 # 可挂载到 AI 工具的完整提示词
├── agents/                       # Agent 角色规则
│   ├── orchestrator/              # 意图识别、Base 扫描、方案推荐、调度
│   ├── pm-agent/                  # 产品链路、Base 对象包与闸口管理
│   ├── qa-agent/                  # QA 验证、用户模拟、风险评估、运营分析
│   ├── dev-agent/                 # 基于 IMP 的开发执行与验证记录
│   └── domain-agent/              # DB / UI / 技术方案领域专家
├── skills/                       # 可按需加载的能力库
│   ├── BA/                        # 业务分析、访谈、隐性需求挖掘与 Council
│   ├── council/                   # Advisor 动态匹配能力
│   ├── pm/                        # PM 流程、对象包模板、变更与 BUG 工作流
│   ├── code/                      # 21 个编码行为、验证、调试、架构与工程子技能
│   ├── db/                        # 数据库设计、SQL 模板、性能优化
│   ├── ui/                        # UI/UX、设计系统、多端落地与检查清单
│   ├── extension/                 # TypeScript、移动端、系统优化、代码审查增强
│   └── prompts/                   # 阶段提示词与关节节点提示词
├── protocols/                    # Agent 间通信与错误处理协议
└── docs/                         # 文档索引、操作手册、集成指南
```

---

## Base 运行时产出结构

`Base/` 是运行时项目认知数据库，不是源码目录。真实源码、配置、脚本和测试文件应放在与 `Base/` 平级的代码工程目录中；`Base/` 只记录需求、方案、实现索引、验证结果和变更追溯。

```text
Base/
├── 00-项目总览/              # 项目目标、状态看板、阶段进度、决策记录、项目结构基线
├── 01-需求管理/RQ-*/         # 需求说明、AI候选方案、人类确认记录、需求状态
├── 02-验证管理/VR-*.md       # 需求验证记录
├── 03-产品设计/PD-*/         # 页面流程、用户路径、交互规则、设计验收
├── 04-方案设计/TD-*/         # 方案说明、接口设计、数据结构、安全策略、方案评审
├── 05-功能管理/FT-*/         # 功能说明、边界、异常场景、验收标准、关联关系
├── 10-QA验证/QA-*/           # QA报告、用户模拟记录、风险评估矩阵、运营分析、人类确认记录
├── 06-实现管理/IMP-*/        # 实现任务清单、代码文件索引、变更摘要、验证结果
├── 07-反馈管理/FB-*.md       # 用户反馈与实现偏差记录
├── 08-变更管理/CR-*/         # 变更说明、影响分析、AI建议、人类决策、执行记录
└── 09-测试验收/TEST-*/       # 测试用例、测试记录、问题清单、验收结论
```

---

## 模块职责

| 模块 | 职责 |
|---|---|
| `SKILL.md` | 顶层入口，基于 Base 状态调度 Agent / Skill |
| `智能体提示词.md` | 可挂载到 AI 工具的完整系统提示词，与顶层入口、Agent、协议保持一致 |
| `agents/orchestrator/` | 识别真实意图，扫描 Base，生成候选方案，推荐最优路径，调度后续 Agent |
| `agents/pm-agent/` | 维护 `RQ → VR → PD → TD → FT → QA → IMP → TEST → FB → CR` 产品链路与关键闸口 |
| `agents/qa-agent/` | 在实现前执行文档解析、用户模拟、风险评估、运营分析和 QA 报告输出 |
| `agents/dev-agent/` | QA 通过后，基于 `IMP-*` 执行代码任务并回写验证结果 |
| `agents/domain-agent/` | 提供 DB / UI / 技术方案 / 性能等专业判断，但不绕过产品链路 |
| `skills/BA/` | 业务分析、隐性需求挖掘、访谈和候选方案生成 |
| `skills/council/` | 为复杂判断匹配 Advisor 视角 |
| `skills/pm/` | 产品链路、对象包模板、变更、BUG 与测试规则 |
| `skills/code/` | 编码行为规范索引，覆盖工具、任务、验证、调试、性能、API、CI/CD、架构等 21 个子技能 |
| `skills/db/` | 数据库设计、SQL、命名规范、索引与性能调优 |
| `skills/ui/` | UI/UX、设计系统、Web / 多端技术栈和检查清单 |
| `skills/extension/` | TypeScript、移动端、系统优化、代码审查等扩展能力 |
| `skills/prompts/` | 阶段提示词与关键节点提示词 |
| `protocols/` | Agent 交接字段、确认点、异常分类与处理规则 |
| `docs/` | 文档索引、操作手册、集成说明和同步维护规则 |

---

## 调度原则

1. 项目相关请求先扫描 `Base/`，没有 Base 时先识别为项目初始化或模板库维护任务。
2. 新需求、新功能、设计取舍、实现偏好、反馈和变更必须先输出 3-5 个候选方案并推荐当前最优方案。
3. 人类确认后，才能写入或更新 Base 对象包。
4. 无上级 `RQ-*`，不得创建 `FT-*` 或 `IMP-*`。
5. 无已确认的 `AI候选方案.md` 与 `人类确认记录.md`，不得进入产品设计。
6. 无 `PD-*` 与 `TD-*`，不得进入实现。
7. `FT-*` 确认后，必须经 QA Agent 验证并生成 `QA-*`。
8. QA 未通过或未经人类确认，不得创建 `IMP-*`。
9. 修改已有功能必须创建 `CR-*`，不得直接覆盖原需求意图；重大变更必须重新 QA。
10. AI 修改代码前必须声明当前对应的 `RQ / FT / QA / IMP`。

---

## Agent 交接关系

| 来源 | 目标 | 条件 | 交接重点 |
|---|---|---|---|
| Orchestrator | PM Agent | 涉及需求、设计、方案、功能、变更、测试 | Base 状态、候选方案、推荐方案、人类确认点 |
| Orchestrator | QA Agent | FT 已确认准备进入 IMP，或 CR / TD 风险较高 | RQ / VR / PD / TD / FT 范围、风险关注点 |
| Orchestrator | Dev Agent | QA 通过且 IMP 已创建，任务为实现、修复或验证 | RQ / FT / QA / IMP、实现边界、验证要求 |
| Orchestrator | Domain Agent | 涉及 DB / UI / 技术方案专业判断 | TD / CR 范围、领域问题和约束 |
| PM Agent | QA Agent | FT 已确认且有验收标准 | QA 验证范围、验收标准和人类确认记录 |
| QA Agent | PM Agent | QA 有条件通过或不通过 | 必须修复项、建议优化项、退回对象包 |
| PM Agent | Dev Agent | QA 通过且 IMP 已创建 | 实现任务清单、代码文件索引、验收标准 |
| Dev Agent | PM Agent | 发现需求、设计、方案或验收标准不一致 | 偏差说明、影响范围、是否需要 FB / CR |
| Domain Agent | PM Agent | 专业建议影响产品意图或功能边界 | TD / CR 影响分析、候选方案和推荐 |

---

## 同步维护机制

### 1. 同步基准

以下文件是 docs 与 protocols 的同步基准：

1. [../README.md](../README.md)
2. [../SKILL.md](../SKILL.md)
3. [../智能体提示词.md](../智能体提示词.md)
4. [../技能使用场景介绍.md](../技能使用场景介绍.md)
5. [../agents/orchestrator/SKILL.md](../agents/orchestrator/SKILL.md)
6. [../agents/pm-agent/SKILL.md](../agents/pm-agent/SKILL.md)
7. [../agents/qa-agent/SKILL.md](../agents/qa-agent/SKILL.md)
8. [../agents/dev-agent/SKILL.md](../agents/dev-agent/SKILL.md)
9. [../agents/domain-agent/SKILL.md](../agents/domain-agent/SKILL.md)
10. [../skills/](../skills/)

### 2. 每次变更后的检查项

当新增 Agent、Skill、对象包类型、调度阶段或项目结构时，必须同步检查：

1. [README.md](./README.md) 是否更新文档入口、Agent / Skill 索引和同步检查重点。
2. [操作手册.md](./操作手册.md) 是否更新用户可见流程、硬规则和推荐对话方式。
3. [INTEGRATION-GUIDE.md](./INTEGRATION-GUIDE.md) 是否更新目录结构、模块职责、交接关系和同步基准。
4. [../protocols/communication-protocol.md](../protocols/communication-protocol.md) 是否更新核心链路、标准交接字段、调度规则和人类确认点。
5. [../protocols/error-handling-protocol.md](../protocols/error-handling-protocol.md) 是否更新错误分类、自动处理规则、标准响应和禁止事项。

### 3. 不同步高风险信号

出现以下内容时，说明文档可能已落后：

- 链路仍写成 `RQ → VR → PD → TD → FT → IMP → TEST`，缺少 QA。
- Base 结构缺少 `10-QA验证/QA-*`。
- Dev 实现前置条件缺少 `QA-*` 或人类确认通过。
- 文档仍描述当前目录不存在的 `platform-adapters/`、`cli/`。
- 文档仍要求 `npm install`、`npm link` 或 `ai-agents-cli`。
- Skill 索引仍写为 12 个 Code 子技能，而当前为 21 个子技能。
- Agent 列表缺少 QA Agent，或仍用 Code Agent 替代 Dev Agent。

### 4. 同步原则

- docs 与 protocols 只做说明、索引、协议和维护规则，不承载具体需求或实现细节。
- 运行时项目事实以 `Base/` 为准；模板库结构事实以根目录 `README.md`、`SKILL.md`、`智能体提示词.md` 和 `agents/` 为准。
- 新增或调整产品链路阶段时，必须同时更新 Base 结构、调度规则、协议交接字段和用户操作手册。
- 不描述不存在的 CLI、平台适配器、安装流程或运行服务。
