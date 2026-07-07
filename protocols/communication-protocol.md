# Agent 通信协议 · 轻量版

> 本协议用于约束 Agent 之间传递信息的最小字段。不要使用复杂类型系统模拟运行时，LLM 只需要清晰、可执行的上下文规则。

---

## 1. 通信目标

所有 Agent 交接必须围绕 `Base/` 对象包进行，确保每一步都能追溯到需求源头、QA 结论、实现记录和测试验收。

核心链路：

```text
RQ → VR → PD → TD → FT → QA → IMP → TEST → FB → CR
```

QA 是 `FT-*` 与 `IMP-*` 之间的强制验证环节。未经 QA Agent 验证并经人类确认，不得创建 `IMP-*` 或进入代码实现。

---

## 2. 标准交接格式

Agent 交接时只传递以下信息：

```text
任务类型：新需求 / 需求验证 / 产品设计 / 技术方案 / 功能定义 / QA验证 / 实现 / 测试 / 反馈 / 变更 / BUG / 专业咨询
关联 Base：RQ / VR / PD / TD / FT / QA / IMP / CR / TEST / FB
当前状态：已确认 / 待确认 / 缺失 / 冲突 / QA通过 / QA有条件通过 / QA不通过
用户意图：一句话描述
候选方案：如影响产品意图，列 3-5 个
推荐方案：当前最优方案与理由
QA结论：无 / 通过 / 有条件通过 / 不通过；如有 Critical 或 High 风险必须标明
约束条件：不可改变的产品、技术、代码、数据或上线限制
下一步：要创建、读取或更新的 Base 对象包
需要人类确认：是 / 否；如是，说明确认点
```

---

## 3. 调度规则

| 来源 | 目标 | 条件 |
|---|---|---|
| Orchestrator | PM Agent | 涉及需求、验证、设计、方案、功能、变更、测试 |
| Orchestrator | QA Agent | `FT-*` 已确认准备进入 `IMP-*`，或 `CR-*` / `TD-*` 风险较高 |
| Orchestrator | Dev Agent | 已有完整 `RQ / FT / QA / IMP`，QA 已通过并经人类确认，且任务是实现、修复或验证 |
| Orchestrator | Domain Agent | 涉及 DB / UI / 技术方案专业判断 |
| PM Agent | QA Agent | `FT-*` 已确认且有验收标准，需要实现前验证 |
| QA Agent | PM Agent | QA 有条件通过 / 不通过，或发现 RQ / PD / TD / FT 不一致 |
| PM Agent | Dev Agent | `QA-*` 已通过且人类确认，`IMP-*` 已创建并确认 |
| PM Agent | Domain Agent | `TD-*` 需要专业评审或补充 |
| Dev Agent | PM Agent | 发现需求、设计、方案、QA 结论或验收标准不一致 |
| Domain Agent | PM Agent | 专业建议影响产品意图、功能边界、数据结构或核心交互 |

---

## 4. 信息最小化

- 只传递当前任务相关 Base 对象包。
- 不全量复制长文档。
- 不把总览文件当作唯一事实来源。
- 具体需求、验证、设计、方案、功能、QA、实现、变更、测试必须进入独立对象包。
- QA 报告只在交接时传递结论、风险等级、必须修复项和人类确认状态；细节保留在 `QA-*` 子文档中。

---

## 5. 人类确认点

以下情况必须请求人类确认：

1. 选择或修改候选需求方案。
2. 改变产品目标或功能范围。
3. 改变用户核心流程。
4. 改变数据结构或核心接口。
5. QA Agent 输出 Critical / High 风险。
6. QA 结论为有条件通过或不通过。
7. QA 通过后是否允许创建 `IMP-*` 并进入实现。
8. 执行破坏性变更、迁移或大规模重构。
9. 接受或拒绝 `CR-*` 变更。

---

## 6. 禁止事项

- 禁止无 `RQ-*` 直接进入功能或实现。
- 禁止无 `PD-* / TD-* / FT-*` 直接进入实现。
- 禁止跳过 `QA-*` 从 `FT-*` 直接进入 `IMP-*`。
- 禁止 QA 未通过或未经人类确认时创建 `IMP-*`。
- 禁止无 `CR-*` 覆盖历史需求。
- 禁止用 Agent 内部判断替代人类产品确认。
- 禁止把真实源码写入 `Base/`。
- 禁止用复杂接口、枚举、类定义替代可执行规则。

---

## 7. 协议同步机制

当以下文件变化时，必须同步检查本协议：

- [../SKILL.md](../SKILL.md)
- [../智能体提示词.md](../智能体提示词.md)
- [../agents/orchestrator/SKILL.md](../agents/orchestrator/SKILL.md)
- [../agents/pm-agent/SKILL.md](../agents/pm-agent/SKILL.md)
- [../agents/qa-agent/SKILL.md](../agents/qa-agent/SKILL.md)
- [../agents/dev-agent/SKILL.md](../agents/dev-agent/SKILL.md)
- [../docs/INTEGRATION-GUIDE.md](../docs/INTEGRATION-GUIDE.md)

同步检查重点：

1. 核心链路是否仍为 `RQ → VR → PD → TD → FT → QA → IMP → TEST → FB → CR`。
2. 标准交接字段是否包含 `QA结论` 与 `QA-*`。
3. 调度规则是否阻止绕过 QA 直接进入 Dev。
4. 人类确认点是否覆盖 QA Critical / High 风险、有条件通过、不通过和进入实现授权。
5. 禁止事项是否覆盖无 QA、QA 未通过、真实源码写入 Base 等情况。
