# Agent 整合指南

## 概述

本项目是一套面向非开发者、产品经理与 AI 编程工具的全流程 Agent / Skill 系统。架构以 `Base/` 作为 AI 的项目认知数据库。

默认使用者不需要懂代码，只需要描述需求想法、问题现象和期望效果；AI 负责维护 Base、定位代码工程文件、执行实现和验证。

核心模式：

```text
人类输入方向
→ AI 生成 3-5 个候选方案
→ AI 推荐当前最优方案
→ 人类选择 / 修正 / 否定
→ AI 写入 Base 对象包
→ 按 RQ → VR → PD → TD → FT → IMP → TEST → FB → CR 链路推进
```

---

## 目录结构

```text
Agent/
├── SKILL.md
├── agents/
│   ├── orchestrator/
│   ├── pm-agent/
│   ├── dev-agent/
│   └── domain-agent/
├── skills/
│   ├── pm/
│   ├── code/
│   ├── db/
│   └── ui/
├── platform-adapters/
├── protocols/
├── cli/
└── docs/
```

---

## Base 运行时产出结构

```text
Base/
├── 00-项目总览/
├── 01-需求管理/RQ-*/
├── 02-验证管理/VR-*.md
├── 03-产品设计/PD-*/
├── 04-方案设计/TD-*/
├── 05-功能管理/FT-*/
├── 06-实现管理/IMP-*/
├── 07-反馈管理/FB-*.md
├── 08-变更管理/CR-*/
└── 09-测试验收/TEST-*/
```

---

## 模块职责

| 模块 | 职责 |
|---|---|
| `SKILL.md` | 顶层入口，基于 Base 状态调度 Agent |
| `agents/orchestrator/` | 识别真实意图，生成候选方案，推荐最优路径 |
| `agents/pm-agent/` | 维护 Base 产品链路与关键闸口 |
| `agents/dev-agent/` | 基于 IMP 执行代码任务 |
| `agents/domain-agent/` | 提供 DB / UI / 技术方案专业判断 |
| `skills/pm/` | 产品链路、对象包模板、变更与测试规则 |
| `skills/code/` | 编码行为规约 |
| `skills/db/` | 数据库设计与优化规则 |
| `skills/ui/` | UI/UX 设计与检查规则 |

---

## 调度原则

1. 项目相关请求先扫描 `Base/`。
2. 新需求、新功能、变更、设计取舍必须先输出 3-5 个候选方案。
3. AI 必须给出当前最优推荐，并说明推荐依据。
4. 人类确认后，才能写入或更新 Base 对象包。
5. 无 `RQ` 不得创建 `FT` 或 `IMP`。
6. 代码实现必须绑定 `RQ / FT / IMP`。
7. 变更必须创建 `CR`，不得直接覆盖原需求。

---

## 平台适配

| 平台 | 入口 |
|---|---|
| Cursor | `platform-adapters/cursor-*.md` |
| Claude Code | `platform-adapters/claude-code-CLAUDE.md` |
| Continue | `platform-adapters/continue-config.json` |
| Codex | `platform-adapters/codex-config.yaml` |
| Trae | `platform-adapters/trae-config.ts` |

---

## 维护规则

- 新增产品阶段时，优先新增 Base 对象包或对象内文档。
- 新增领域能力时，优先新增 Skill，不在顶层入口堆叠方法论。
- 总文档只做索引，具体需求、方案、功能、实现、变更、测试必须独立成包。
- 平台适配器必须与 `SKILL.md` 和 `agents/orchestrator/SKILL.md` 保持一致。