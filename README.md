# AI Agents CLI · 全流程产品协作系统

> **版本**：2.0.0  
> **定位**：AI 原生产品协作工具链，让非开发者也能用自然语言驱动完整的产品开发流程。  
> **支持平台**：Continue、Cursor、Trae、Codex  

## 一句话说明

人类只需描述业务想法、问题现象和期望效果；AI 负责把自然语言转化为候选方案、Base 对象包、代码实现任务和测试验证，并按标准链路推进到可交付状态。

## 核心理念

- **Base 为唯一基准**：以 `Base/` 作为 AI 的项目认知数据库，所有需求、设计、方案、功能、实现、变更、测试都以独立对象包管理。
- **候选方案优先**：AI 不直接写代码或改需求，而是先输出 3-5 个候选方案并推荐最优解，由人类选择/修正/否定。
- **链路可追溯**：任何代码修改都必须能追溯到对应的 RQ（需求）→ FT（功能）→ IMP（实现）。
- **非开发者友好**：默认使用者不需要懂代码、路径或技术细节；AI 自行定位文件、解释影响、完成验证。

## 快速开始

```bash
npm install
npm link
cd /path/to/your-project
ai-agents-cli install -p cursor --global
```

安装后重启对应 AI 编程工具即可使用。

## 系统架构

### Agent（智能体）

| Agent | 职责 |
|---|---|
| Orchestrator | 意图识别、Base 扫描、候选方案生成、Agent 调度 |
| PM-Agent | 维护 RQ→VR→PD→TD→FT→IMP→TEST→FB→CR 链路 |
| Dev-Agent | 基于 IMP-* 执行代码任务，保证可追溯 |
| Domain-Agent | 数据库/SQL/UI/性能等垂直领域专家 |

### Skill（技能包）

| Skill | 用途 |
|---|---|
| BA-skill | 业务分析、隐性需求挖掘、痛点推断 |
| PM-skill | 产品流程调度与 Base 对象包管理 |
| Code-skill | 编码规范、工具使用、Git 工作流、验证纪律 |
| DB-skill | 数据库设计、SQL 模板、性能调优 |
| UI-skill | UI/UX 设计系统、交互规则、技术栈落地 |

### Base 对象包链路

```text
RQ（需求） → VR（验证） → PD（产品设计） → TD（技术方案）
→ FT（功能定义） → IMP（实现任务） → TEST（测试验收）
→ FB（反馈） → CR（变更） → IMP（增量实现） → TEST（回归测试）
```

## Base 工作方式

```text
人类输入方向
→ AI 生成 3-5 个候选方案
→ AI 推荐当前最优方案
→ 人类选择 / 修正 / 否定
→ AI 写入 Base 对象包
→ 按 RQ → VR → PD → TD → FT → IMP → TEST → FB → CR 链路推进
```

## Base 目录结构

```text
Base/
├── 00-项目总览/          # 项目目标、状态看板、阶段进度、决策记录
├── 01-需求管理/RQ-*/     # 每个需求一个独立对象包
├── 02-验证管理/VR-*.md   # 需求验证记录
├── 03-产品设计/PD-*/     # 页面流程、用户路径、交互规则
├── 04-方案设计/TD-*/     # 接口设计、数据结构、安全策略
├── 05-功能管理/FT-*/     # 功能边界、异常场景、验收标准
├── 06-实现管理/IMP-*/    # 实现任务清单、代码文件索引、变更说明
├── 07-反馈管理/FB-*.md   # 用户反馈与偏差记录
├── 08-变更管理/CR-*/     # 影响分析、处理策略、执行记录
└── 09-测试验收/TEST-*/   # 测试用例、问题清单、验收结论
```

## 主要入口

- [SKILL.md](./SKILL.md): Agent-Skill 总入口（AI 调度规则）
- [docs/README.md](./docs/README.md): 文档索引
- [docs/操作手册.md](./docs/操作手册.md): 完整操作手册（推荐优先阅读）
- [agents/orchestrator/SKILL.md](./agents/orchestrator/SKILL.md): Orchestrator Agent 说明

## 常用命令

```bash
ai-agents-cli install
ai-agents-cli install -p continue
ai-agents-cli migrate
ai-agents-cli doctor
ai-agents-cli check
ai-agents-cli status
ai-agents-cli update
ai-agents-cli uninstall
```

## 开发命令

```bash
npm test
npm run lint
npm run format
```