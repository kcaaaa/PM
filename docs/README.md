# 文档索引

> 本目录提供 AI Agents CLI 项目的详细说明入口。  
> 面向实际操作的完整手册在 [操作手册.md](./操作手册.md)。

## 推荐阅读顺序

1. [../README.md](../README.md) — 项目总览（快速了解系统定位与架构）
2. [操作手册.md](./操作手册.md) — 完整使用指南（安装、日常用法、链路说明）
3. [INTEGRATION-GUIDE.md](./INTEGRATION-GUIDE.md) — 架构与集成说明（Agent/Skill/Base 协作关系）

## 核心文档

- [../README.md](../README.md): 项目总览，包含核心理念、系统架构、Base 链路说明
- [操作手册.md](./操作手册.md): 面向人类用户的完整操作手册，覆盖新需求、新功能、变更、代码问题处理等场景
- [INTEGRATION-GUIDE.md](./INTEGRATION-GUIDE.md): Agent/Skill/Base 集成架构与协作流程

## Agent 文档

- [../agents/orchestrator/SKILL.md](../agents/orchestrator/SKILL.md): Orchestrator（协调器）职责与调度规则
- [../agents/pm-agent/SKILL.md](../agents/pm-agent/SKILL.md): PM-Agent（产品流程）工作流与闸口控制
- [../agents/dev-agent/SKILL.md](../agents/dev-agent/SKILL.md): Dev-Agent（开发执行）编码规范与追溯规则

## Skill 文档

- [../skills/BA/BA-skill.md](../skills/BA/BA-skill.md): BA（业务分析）隐性需求挖掘与方案生成
- [../skills/pm/PM-skill.md](../skills/pm/PM-skill.md): PM（产品管理）Base 链路调度规则
- [../skills/code/code-skill.md](../skills/code/code-skill.md): Code（编码规范）12 个子技能索引
- [../skills/db/DB-skill.md](../skills/db/DB-skill.md): DB（数据库设计）建模、SQL、性能调优
- [../skills/ui/UI-skill.md](../skills/ui/UI-skill.md): UI（界面设计）风格系统与技术栈落地

## 协议与规范

- [../protocols/communication-protocol.md](../protocols/communication-protocol.md): Agent 间通信协议
- [../protocols/error-handling-protocol.md](../protocols/error-handling-protocol.md): 错误处理与异常响应规范
