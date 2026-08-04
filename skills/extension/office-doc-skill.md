---
name: office-doc-skill
description: Office 文档梳理技能——基于 officecli CLI 创建、检查、整理、优化 .docx / .xlsx / .pptx 文档内容与格式
when_to_use: 当用户明确提出需要输出、生成、整理、检查或优化 Office 文档（Word / Excel / PowerPoint）时加载本技能并执行
source: 基于 officecli 基础技能包（curl -fsSL https://officecli.ai/SKILL.md）固化为本项目内置工具技能
---

# Office 文档梳理技能 Office Doc Skill

> 当前项目适配：本技能是工具型 Skill，定位为“文档输出与整理的执行器”。当用户明确需要产出 .docx / .xlsx / .pptx 文档时，由 Orchestrator 触发调用；技术类文档（设计说明、接口文档、方案记录等）仍可酌情保留为 Markdown 格式，写入对应 Base 对象包。
>
> **核心定位**：Base 仍是唯一项目认知基准；Office 文档是 Base 内容的导出产物与交付件，不替代 Base 的 Markdown 记录。

---

## 一、触发条件与职责边界

### 1. 何时触发（显式触发）

当用户明确提出以下意图时，AI 必须加载本技能：

| 触发语句示例 | 触发动作 |
|---|---|
| “把这份需求文档导出成 Word” | 创建 .docx 并填入 RQ-* 内容 |
| “生成一份 PPT 汇报” | 创建 .pptx 并按汇报结构组织幻灯片 |
| “把这些数据整理成 Excel” | 创建 .xlsx 并结构化写入 |
| “检查这份 Word 的格式问题” | `view issues` 诊断并输出问题清单 |
| “整理一下这份文档的内容” | 读取 → 重组 → 优化文档结构 |
| “给客户出一份正式的方案文档” | 基于 PD/TD 内容生成 .docx 交付件 |

### 2. 何时不触发

- 用户只需要 Markdown 技术文档、方案说明、接口文档 → 直接写入 Base 对应对象包，不走本技能。
- 用户只是讨论需求、设计、方案，未明确要“输出文档” → 走 PM / BA / Domain 链路。
- 用户要修改代码工程文件 → 走 Dev Agent。

### 3. 与 Base 的关系（导出产物 + 记录索引）

- **导出产物不入 Base**：生成的 .docx / .xlsx / .pptx 文件放在与 `Base/` 平级的交付目录（如 `deliverables/`、`docs-export/` 或用户指定目录），不写入 `Base/`。
- **记录写入 Base**：文档梳理任务的执行记录写入对应对象包：
  - 实现阶段产出的交付文档 → 记录在 `Base/06-实现管理/IMP-*/实现完成记录.md`，附产物路径与校验结果。
  - 测试验收阶段的文档检查 → 记录在 `Base/09-测试验收/TEST-*/测试记录.md` 与 `问题清单.md`。
  - 独立的文档梳理任务（不绑定具体 RQ） → 至少在 `Base/00-项目总览/决策记录.md` 留痕，说明产物路径、来源、用途。
- **Base 优先原则**：当文档内容与 Base 对象包存在冲突时，以 Base 为准；文档为导出快照，不反向覆盖 Base 认知。

---

## 二、工具安装与验证

本技能依赖 `officecli` CLI（单一二进制，无依赖，无需安装 Office）。首次使用前确认已安装：

```bash
officecli --version
```

若未安装，按当前操作系统安装（安装命令固化自基础技能包，仅在缺失时执行一次）：

```bash
# macOS / Linux
curl -fsSL https://d.officecli.ai/install.sh | bash

# Windows (PowerShell)
irm https://d.officecli.ai/install.ps1 | iex
```

安装后如命令未识别，打开新终端再试。本技能后续所有命令默认 `officecli` 已就绪，不再重复安装步骤。

> 说明：本技能文件已固化 officecli 基础技能包的核心能力，AI 在触发文档梳理任务时直接读取本文件执行，无需每次重新 `curl` 获取云端 SKILL.md。仅在需要核对最新命令或遇到本文件未覆盖的高级用法时，可酌情执行 `curl -fsSL https://officecli.ai/SKILL.md` 比对更新。

---

## 三、三层操作策略（核心方法论）

**L1（读取）→ L2（DOM 编辑）→ L3（原始 XML）**，始终优先使用更高层。需要结构化输出时加 `--json`。

| 层级 | 用途 | 典型命令 |
|---|---|---|
| L1 读取 | 创建、查看、检查、提取文本 | `create` / `view` / `get` / `query` / `validate` |
| L2 DOM 编辑 | 修改属性、增删元素、批量操作 | `set` / `add` / `move` / `swap` / `remove` / `batch` |
| L3 原始 XML | L2 无法表达的底层操作 | `raw` / `raw-set` / `add-part` |

**遇到不确定的属性名、值格式、命令语法时，先跑 help，不要猜。** 一次 help 查询胜过猜错重试循环。

```bash
officecli help                              # 所有命令 + 全局选项
officecli help docx                         # 列出所有 docx 元素
officecli help docx paragraph               # 完整 schema：属性、别名、示例
officecli help docx set paragraph           # 按 verb 过滤：仅 set 可用属性
officecli help docx paragraph --json        # 结构化 schema（机器可读）
```

格式别名：`word`→`docx`，`excel`→`xlsx`，`ppt`/`powerpoint`→`pptx`。动词：`add`、`set`、`get`、`query`、`remove`。

---

## 四、性能模式：常驻模式（Resident Mode）

每条命令首次访问文件会自动启动常驻进程（60 秒空闲超时），自动避免文件锁冲突。长会话建议显式 `open` / `close`（12 分钟空闲）：

```bash
officecli open report.docx       # 显式常驻内存
officecli set report.docx ...    # 无文件 I/O 开销
officecli close report.docx      # 保存并释放
```

**刷新边界规则**：仅在非 officecli 程序（python-docx / openpyxl / Word / 渲染器 / 上传交付）读取文件前才需要 `save`（保留常驻）或 `close`（刷新并释放）。officecli 自身的 `get` / `query` / `view` / `dump` 始终能看到最新编辑，无需中途保存。

---

## 五、核心命令速查

### 1. L1 创建与读取

```bash
officecli create <file>               # 按扩展名创建空白 .docx/.xlsx/.pptx
officecli view <file> <mode>          # outline | stats | issues | text | annotated | html
officecli get <file> <path> --depth N # 获取节点及子节点 [--json]
officecli query <file> <selector>     # CSS-like 查询
officecli validate <file>             # 校验 OpenXML schema
```

**view 模式表**：

| Mode | 说明 | 常用 flag |
|---|---|---|
| `outline` | 文档结构 | |
| `stats` | 统计（页数、字数、形状数） | |
| `issues` | 格式 / 内容 / 结构问题 | `--type format\|content\|structure`，`--limit N` |
| `text` | 纯文本提取 | `--start N --end N`，`--max-lines N` |
| `annotated` | 带格式标注的文本 | |
| `html` | 静态 HTML 快照（无需服务） | `--browser`，docx `--page N`，pptx `--start N --end N` |
| `screenshot` / `svg` / `pdf` | PNG / SVG（pptx 幻灯片）/ PDF 导出 | `-o`，`--screenshot-width/-height`，pptx `--grid N` |

### 2. 稳定 ID 寻址（多步工作流优先使用）

带稳定 ID 的元素返回 `@attr=value` 路径，插入 / 删除后不会漂移；位置索引 `[N]` 会随结构变化漂移。多步工作流优先用稳定 ID：

```
/slide[1]/shape[@id=550950021]                    # PPT 形状
/slide[1]/table[@id=1388430425]/tr[1]/tc[2]       # PPT 表格
/body/p[@paraId=1A2B3C4D]                         # Word 段落
/comments/comment[@commentId=1]                    # Word 批注
```

PPT 还支持 `@name=`（如 `shape[@name=Title 1]`）。

### 3. query 选择器

CSS-like：`[attr=value]`、`[attr!=value]`、`[attr~=text]`、`[attr>=value]`、`[attr<=value]`、`:contains("text")`、`:empty`、`:has(formula)`、`:no-alt`。支持布尔 `and` / `or`：

```bash
officecli query report.docx 'paragraph[style=Normal] > run[font!=Arial]'
officecli query slides.pptx 'shape[fill=FF0000]'
officecli query data.xlsx 'cell[value>5000 or value<100]'
officecli query data.xlsx 'Sheet1!row[Salary>5000]'   # 按列名按行查
```

### 4. L2 set —— 修改属性 / 查找替换

```bash
officecli set <file> <path> --prop key=value [--prop ...]
```

**值格式**：

| 类型 | 格式 | 示例 |
|---|---|---|
| 颜色 | Hex / 命名 / RGB / 主题色 | `FF0000`、`#FF0000`、`red`、`rgb(255,0,0)`、`accent1`..`accent6` |
| 间距 | 带单位 | `12pt`、`0.5cm`、`1.5x`、`150%` |
| 尺寸 | EMU 或带后缀 | `914400`、`2.54cm`、`1in`、`72pt`、`96px` |

**点号别名**：`font.<attr>` 可用于 shape / run / paragraph / table / row / cell / section / styles，如 `--prop font.color=red --prop font.bold=true --prop font.size=14pt`。

**查找与替换**（顶层 `--find` / `--replace`）：

```bash
# 格式化匹配文本（自动跨 run 拆分）
officecli set doc.docx '/body/p[1]' --find weather --prop bold=true --prop color=red

# 正则匹配
officecli set doc.docx '/body/p[1]' --find '\d+%' --prop regex=true --prop color=red

# 全文替换（/ 表示整个文档）
officecli set doc.docx / --find draft --replace final

# Word 带修订痕迹的查找替换
officecli set doc.docx / --find draft --replace final --prop revision.author=Alice
```

路径控制搜索范围：`/` = 全文档，`/body/p[1]` 或 `/slide[N]/shape[M]` = 指定元素，`/header[1]` / `/footer[1]` = 页眉页脚。默认大小写敏感；不区分大小写：`--prop 'find=(?i)error' --prop regex=true`。无匹配 = 静默成功，`--json` 含 `"matched": N`。Excel 仅支持 `find` + `replace`，不支持 find + 格式化属性。

### 5. L2 add —— 添加元素 / 克隆

```bash
officecli add <file> <parent> --type <type> [--prop ...]
officecli add <file> <parent> --type <type> --after <path> [--prop ...]    # 锚点后插入
officecli add <file> <parent> --type <type> --before <path> [--prop ...]   # 锚点前插入
officecli add <file> <parent> --type <type> --index N [--prop ...]         # 0-based 位置
officecli add <file> / --from '/slide[1]'                                  # 克隆现有元素
```

`--after` / `--before` / `--index` 互斥，无位置参数 = 末尾追加。

**元素类型速查**：

| 格式 | 主要类型 |
|---|---|
| **pptx** | slide、shape、picture、chart、table、row、connector、group、video、audio、equation、notes、comment、animation、transition、paragraph、run、zoom、ole、placeholder、model3d、smartart、diagram |
| **docx** | paragraph、run、table、row、cell、image、header、footer、section、bookmark、comment、footnote、endnote、formfield、sdt、chart、equation、field、hyperlink、style、toc、watermark、break、ole、num、tab、textbox、shape、diagram |
| **xlsx** | sheet、row、col、cell、chart、image、comment、table、namedrange、pivottable、sparkline、validation、autofilter、shape、textbox、CF、ole、csv |

**文本锚点插入**（`--after find:X` / `--before find:X`）：按文本匹配定位插入点。行内类型（run、picture、hyperlink）插入段落内；块类型（table、paragraph）自动拆分段落。PPT 仅支持行内。

```bash
# Word：在匹配文本后插入行内 run
officecli add doc.docx '/body/p[1]' --type run --after find:weather --prop text=" (sunny)"

# Word：在匹配文本后插入块级 table（自动拆分段落）
officecli add doc.docx '/body/p[1]' --type table --after "find:First sentence." --prop rows=2 --prop cols=2
```

### 6. move / swap / remove

```bash
officecli move <file> <path> [--to <parent>] [--index N] [--after <path>] [--before <path>]
officecli swap <file> <path1> <path2>
officecli remove <file> '/body/p[4]'
```

使用 `--after` / `--before` 时可省略 `--to`，目标容器从锚点推断。

### 7. batch —— 批量原子操作

**默认原子性（v1.0.137+）**：每条仍执行并上报（保持 `N succeeded, M failed` 有意义），但任一条失败则整批回滚，磁盘文件与批处理前字节一致。`--best-effort` 恢复“成功即保留”行为（适用于有损 `dump→batch` 重放）。`--stop-on-error` 只改变停止时机，不影响已执行是否保留。

```bash
echo '[
  {"command":"set","path":"/Sheet1/A1","props":{"value":"Name","bold":"true"}},
  {"command":"set","path":"/Sheet1/B1","props":{"value":"Score","bold":"true"}}
]' | officecli batch data.xlsx --json

officecli batch data.xlsx --commands '[{"op":"set","path":"/Sheet1/A1","props":{"value":"Done"}}]' --json
officecli batch data.xlsx --input updates.json --best-effort --json
```

支持操作：`add`、`set`、`get`、`query`、`remove`、`move`、`swap`、`view`、`raw`、`raw-set`、`validate`。

`officecli dump <file> [<path>]` 导出可回放的 batch JSON，支持 .docx（全覆盖）、.pptx（文本/表格/图片/图表/备注/主题 + OLE/3D/视频/音频/SmartArt/morph 通过 raw-set 透传）、.xlsx（单元格/公式/样式 + 表格、条件格式、验证、批注、图表、迷你图、图片、形状、透视表）。`officecli refresh <file.docx>` 在重放后重算 TOC 页码 / PAGE / 交叉引用。

### 8. L3 原始 XML

L2 无法表达时使用，无需声明 xmlns（前缀自动注册）：

```bash
officecli raw <file> <part>                          # 查看原始 XML
officecli raw-set <file> <part> --xpath "..." --action replace --xml '<w:p>...</w:p>'
officecli add-part <file> <parent>                   # 创建新文档部件（返回 rId）
```

`raw-set` 动作：`append`、`prepend`、`insertbefore`、`insertafter`、`replace`、`remove`、`setattr`。

---

## 六、实时预览与交互选择

```bash
officecli watch <file> [--port N]      # 启动预览服务（默认端口 26315），自动刷新
officecli unwatch <file>               # 停止
officecli goto <file> <path>           # 滚动浏览器到指定元素
officecli get <file> selected [--json] # 读取用户在浏览器中点击选中的元素
```

- 浏览器中点击选中、shift/cmd/ctrl 多选、空白拖拽框选；PPT/Word 蓝框，Excel 原生绿选区（双击单元格内联编辑）。
- 选择状态跨文件编辑保留（路径用稳定 `@id=` 形式）；所有连接的浏览器共享同一选区，后写覆盖先写。
- 同一文件只能有一个 watch 进程。

**mark —— 待审查的编辑提案**：变更需人工审查后再落盘时使用 `mark`，标记仅存在于 watch 进程中，由独立的 `set` 流程应用接受项。一次性变更直接用 `set`；永久文件批注用 `add --type comment`（Word 原生）。

```bash
officecli mark <file> <path> [--prop find=... color=... note=... tofix=... regex=true] [--json]
officecli unmark <file> [--path <p> | --all] [--json]
officecli get-marks <file> [--json]
```

---

## 七、专门技能加载（按文档类型细分）

`officecli load_skill <name>` 输出一个 SKILL.md，按其规则执行。

**加载规则**：
- 在“何时使用”中选最具体的匹配；都不匹配时加载格式默认（`word` / `pptx` / `excel`）。
- 一个产物只加载一个技能，不叠加；场景已包含格式默认规则。
- 已加载规则跨轮次保留，不要每轮重复加载。
- 两个不同产物 → 两次独立加载。

### Word（.docx）

| Name | 何时使用 |
|---|---|
| `word` | 报告、信函、备忘录、提案、通用文档 |
| `academic-paper` | 期刊 / 会议 / 学位论文：APA / Chicago / IEEE / MLA 引用、公式、SEQ + PAGEREF 交叉引用、多栏期刊排版、参考文献。不用于商业报告或信函（走 `word`） |

### PowerPoint（.pptx）

| Name | 何时使用 |
|---|---|
| `pptx` | 通用幻灯片：董事会评审、销售汇报、全员大会、产品发布 |
| `pitch-deck` | **仅融资场景** —— 种子轮 / A-C 轮 / SAFE / 可转债 / 战略融资。不用于销售 / 产品 / 董事会幻灯片（走 `pptx`） |
| `morph-ppt` | 电影级 Morph 动画演示。不用于静态幻灯片（走 `pptx`） |
| `morph-ppt-3d` | 3D Morph：GLB 模型、镜头运动、景深。不用于纯 2D Morph（走 `morph-ppt`） |

### Excel（.xlsx）

| Name | 何时使用 |
|---|---|
| `excel` | 通用工作簿、公式、透视表、跟踪表 |
| `financial-model` | 财务模型、场景、预测。不用于通用数据分析（走 `excel`） |
| `data-dashboard` | CSV / 表格数据 → KPI / 分析 / 高管仪表盘（含图表与迷你图）。不用于原始数据跟踪（走 `excel`） |

---

## 八、典型场景执行流程

### 场景 1：从 Base 对象包导出 Word 方案文档

```bash
# 1. 加载专门技能
officecli load_skill word

# 2. 创建文档
officecli create 方案文档.docx

# 3. 按方案结构添加标题与正文（来源 PD/TD 内容）
officecli add 方案文档.docx /body --type paragraph --prop text="一、方案概述" --prop style=Heading1
officecli add 方案文档.docx /body --type paragraph --prop text="<来自 PD-*/方案说明.md 的概述>"

# 4. 校验
officecli validate 方案文档.docx
officecli view 方案文档.docx issues --type format

# 5. 导出 PDF 交付（可选）
officecli view 方案文档.docx pdf -o 方案文档.pdf
```

执行后：将产物路径与校验结果写入 `Base/06-实现管理/IMP-*/实现完成记录.md` 或 `Base/00-项目总览/决策记录.md`。

### 场景 2：检查并整理既有 Word 文档

```bash
# 1. 诊断格式 / 内容 / 结构问题
officecli view report.docx issues --type format --limit 50
officecli view report.docx outline           # 看结构
officecli view report.docx stats             # 看统计

# 2. 批量修复（原子批处理）
echo '[
  {"command":"set","path":"/","find":"草稿","replace":"正式","props":{}},
  {"command":"set","path":"/body/p[1]","props":{"style":"Heading1"}}
]' | officecli batch report.docx --json

# 3. 复核
officecli validate report.docx
officecli view report.docx issues
```

执行后：问题清单与修复记录写入 `Base/09-测试验收/TEST-*/问题清单.md`。

### 场景 3：生成 Excel 数据表

```bash
officecli create data.xlsx
officecli set data.xlsx /Sheet1/A1 --prop value="名称" --prop bold=true
officecli set data.xlsx /Sheet1/B1 --prop value="数值" --prop bold=true
officecli set data.xlsx /Sheet1/A2 --prop value="指标A"
officecli set data.xlsx /Sheet1/B2 --prop value=100
officecli validate data.xlsx
```

### 场景 4：生成 PPT 汇报

```bash
officecli load_skill pptx
officecli create 汇报.pptx
officecli add 汇报.pptx / --type slide --prop title="Q4 业务汇报" --prop background=1A1A2E
officecli add 汇报.pptx '/slide[1]' --type shape --prop text="收入同比增长 25%" \
  --prop x=2cm --prop y=5cm --prop font=Arial --prop size=24 --prop color=FFFFFF
officecli validate 汇报.pptx
```

---

## 九、常见陷阱与规范

| 陷阱 | 正确做法 |
|---|---|
| `--name "foo"` | 用 `--prop name="foo"`，所有属性走 `--prop` |
| zsh/bash 中未加引号的 `[N]` 路径 | 始终加引号：`'/slide[1]'` 或 `"/slide[1]"`（shell 会展开方括号） |
| PPT 用 `shape[1]` 取内容 | `shape[1]` 通常是标题占位符，内容形状用 `shape[2]+` |
| `/shape[myname]` 名称索引 | 不支持名称索引，用数字索引或 `@name=`（仅 PPT） |
| 猜属性名 | 跑 `officecli help <format> <element>` 查准确名称 |
| 修改正在 Office 中打开的文件 | 先在 PowerPoint / WPS / Word 中关闭文件 |
| shell 字符串中的 `\n` | 用 `\\n` 表示换行 |
| shell 文本中的 `$` | `--prop text="$15M"` 会被剥离，用单引号 `--prop text='$15M'` 或 heredoc batch |

**路径与索引约定**：
- 路径 1-based（XPath 约定）：`'/body/p[3]'` = 第三段
- `--index` 0-based（数组约定）：`--index 0` = 首位
- **Excel 例外**：`add --type row` / `add --type col` 时 `--index N` 为 1-based（匹配 OOXML 行号 / 列字母序号）

**修改后必做**：`validate` 和 / 或 `view issues` 复核。

---

## 十、与项目链路的协作规则

1. **触发判断由 Orchestrator 负责**：Orchestrator 识别“用户明确需要输出 Office 文档”意图后，调度本技能；本技能不自行判断产品意图。
2. **不绕过 Base 链路**：文档梳理任务若涉及需求、设计、方案内容，必须以已确认的 Base 对象包为内容来源；不得在缺 `RQ-*` / `PD-*` / `TD-*` 时凭空生成交付文档。
3. **产物与记录分离**：
   - 产物文件 → 与 `Base/` 平级的交付目录（`deliverables/`、`docs-export/` 或用户指定路径）。
   - 执行记录 → 写入对应 `IMP-*` / `TEST-*` / `决策记录.md`，含产物路径、来源对象包、校验结果。
4. **失败如实汇报**：`validate` 失败、`view issues` 命中严重问题、batch 回滚，必须如实说明，不掩盖。
5. **不替代 Markdown 技术文档**：技术方案、接口设计、数据结构、QA 报告等仍以 Markdown 写入 Base 对象包；Office 文档用于对外交付、汇报、正式输出场景。
6. **破坏性操作前确认**：覆盖已有交付文档、批量替换、删除大量元素前，先列出影响范围，必要时请求人类确认。

---

## 十一、维护约束

- 本技能固化自 officecli 基础技能包（`curl -fsSL https://officecli.ai/SKILL.md`），核心命令与三层策略以基础技能包为准。
- officecli 版本升级带来新命令 / 新属性时，先通过 `officecli help` 核对，再同步更新本文件；不得凭猜测增删命令。
- 本技能只承载文档梳理执行能力，不承载产品需求、设计、方案方法论；后者仍由 PM / BA / Domain Skill 负责。
- 新增细分文档场景（如学位论文、融资 deck、财务模型）时，优先在本文件“专门技能加载”章节补充 `load_skill` 条目，不新增独立 Skill 文件。
