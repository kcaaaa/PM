# 文档索引

> 本目录提供全流程产品 Agent / Skill 模板库的说明入口。
> 当前项目是纯 Markdown 文件型规则库，无需安装依赖、无需 CLI、无需运行服务。

## 推荐阅读顺序

1. [../README.md](../README.md) — 项目总览：系统定位、目录结构、Agent / Skill 架构、Base 链路
2. [../SKILL.md](../SKILL.md) — 顶层调度入口：Base、候选方案、QA、追溯和 Skill 挂载规则
3. [../智能体提示词.md](../智能体提示词.md) — 可挂载到 AI 工具的完整基础智能体提示词
4. [操作手册.md](./操作手册.md) — 面向非开发者的人类使用指南
5. [INTEGRATION-GUIDE.md](./INTEGRATION-GUIDE.md) — Agent / Skill / Base 集成关系与同步维护规则

## 核心文档

- [../README.md](../README.md): 项目总览，包含当前项目结构、Agent 角色、Skill 能力库、Base 对象包结构和维护原则
- [../SKILL.md](../SKILL.md): AI 顶层调度入口，是运行时调度的首要规则文件
- [../智能体提示词.md](../智能体提示词.md): 完整系统提示词版本，用于同步自定义指令或 Skill 描述
- [../技能使用场景介绍.md](../技能使用场景介绍.md): 说明本技能适用场景、触发方式和各类任务的执行链路
- [操作手册.md](./操作手册.md): 面向人类用户的完整操作手册，覆盖新需求、QA、实现、测试、反馈和变更
- [INTEGRATION-GUIDE.md](./INTEGRATION-GUIDE.md): Agent / Skill / Base 集成架构、依赖关系和文档同步机制

## Agent 文档

- [../agents/orchestrator/SKILL.md](../agents/orchestrator/SKILL.md): Orchestrator，负责意图识别、Base 扫描、候选方案、推荐和调度
- [../agents/pm-agent/SKILL.md](../agents/pm-agent/SKILL.md): PM Agent，负责 RQ → VR → PD → TD → FT → QA → IMP → TEST → FB → CR 链路与闸口
- [../agents/qa-agent/SKILL.md](../agents/qa-agent/SKILL.md): QA Agent，负责实现前强制验证、用户模拟、风险评估、运营分析和 QA 报告
- [../agents/dev-agent/SKILL.md](../agents/dev-agent/SKILL.md): Dev Agent，负责 QA 通过后基于 IMP 的代码实现、验证和记录
- [../agents/domain-agent/SKILL.md](../agents/domain-agent/SKILL.md): Domain Agent，负责 DB / UI / 技术方案等领域专业判断

## Skill 文档

- [../skills/BA/BA-skill.md](../skills/BA/BA-skill.md): BA，负责业务分析、隐性需求挖掘、访谈和候选方案生成
- [../skills/council/advisor-matching-skill.md](../skills/council/advisor-matching-skill.md): Council Advisor 匹配能力，用于复杂判断的多视角辅助
- [../skills/pm/PM-skill.md](../skills/pm/PM-skill.md): PM，负责 Base 对象包、产品链路和阶段调度
- [../skills/code/code-skill.md](../skills/code/code-skill.md): Code，负责 21 个编码行为、验证、调试、架构与工程子技能索引
- [../skills/db/DB-skill.md](../skills/db/DB-skill.md): DB，负责数据库设计、SQL、索引和性能调优
- [../skills/ui/UI-skill.md](../skills/ui/UI-skill.md): UI，负责 UI/UX、设计系统、Web / 多端落地和检查清单
- [../skills/extension/](../skills/extension/): TypeScript、移动端、系统优化、代码审查等扩展能力
- [../skills/prompts/phase-prompts.md](../skills/prompts/phase-prompts.md): 阶段提示词
- [../skills/prompts/joint-node-prompts.md](../skills/prompts/joint-node-prompts.md): 关键节点提示词

## 协议与规范

- [../protocols/communication-protocol.md](../protocols/communication-protocol.md): Agent 间通信协议，定义最小交接字段、调度规则和确认点
- [../protocols/error-handling-protocol.md](../protocols/error-handling-protocol.md): 错误处理协议，定义上下文缺失、QA 风险、实现偏差、测试失败等处理规则

## 同步维护机制

当以下文件发生变化时，必须同步检查本目录文档：

- [../README.md](../README.md)
- [../SKILL.md](../SKILL.md)
- [../智能体提示词.md](../智能体提示词.md)
- [../技能使用场景介绍.md](../技能使用场景介绍.md)
- [../agents/](../agents/)
- [../skills/](../skills/)
- [../protocols/](../protocols/)

同步检查重点：

1. 项目定位是否仍为纯 Markdown 文件型 Agent / Skill 模板库。
2. 是否仍明确“无需安装依赖、无需 CLI、无需运行服务”。
3. Base 链路是否包含 QA：`RQ → VR → PD → TD → FT → QA → IMP → TEST → FB → CR`。
4. 是否包含 `Base/10-QA验证/QA-*` 对象包。
5. Agent 列表是否包含 Orchestrator、PM、QA、Dev、Domain。
6. Skill 列表是否包含 BA、Council、PM、Code、DB、UI、Extension、Prompt。
7. 文档中不得描述当前目录不存在的 `platform-adapters/`、`cli/` 或安装命令。
