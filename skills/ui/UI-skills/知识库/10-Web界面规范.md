# 知识库 · 10 Web 界面规范（30 条 ARIA / Focus / Forms / Perf / State）

> **来源**：由 `web-interface.csv` 完整转译。
> **作用**：专注**浏览器/Web 场景**的接口级规范：语义化 / ARIA / 键盘 / 焦点 / 表单 / 性能 / 状态 / 反模式。
> **搜索字段**：`Category / Issue / Keywords / Description`

---

## 1. Accessibility（无障碍，6 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 1 | **Icon Button Labels** | `<button aria-label="Close"><XIcon /></button>` | 裸 `<button><XIcon /></button>` | **Critical** |
| 2 | **Form Control Labels** | `<label for="email">Email</label><input id="email" />` | `<input placeholder="Email" />` | **Critical** |
| 3 | **Keyboard Handlers** | `<div onClick={fn} onKeyDown={fn} tabIndex={0}>` | `<div onClick={fn}>`（鼠标限定） | High |
| 4 | **Semantic HTML** | `<button onClick={fn}>Submit</button>` | `<div role="button" onClick={fn}>Submit</div>` | High |
| 5 | **Aria Live** | `<div aria-live="polite">{status}</div>` 异步更新 | `<div>{status}</div>` 静默更新 | Medium |
| 6 | **Decorative Icons** | `<Icon aria-hidden="true" />` 装饰用 | 装饰图标被屏幕阅读器朗读 | Medium |

## 2. Focus（焦点，3 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 7 | **Visible Focus States** | `focus-visible:ring-2 focus-visible:ring-blue-500` | `outline-none` 无替代 | **Critical** |
| 8 | **Never Remove Outline** | `focus:outline-none focus:ring-2` | `focus:outline-none` 只这一句 | **Critical** |
| 9 | **Checkbox/Radio Hit Target** | `<label class="flex gap-2"><input type="checkbox" /><span>Option</span></label>` | `<input id="x" />` + 单独 `<label for="x">` | Medium |

## 3. Forms（表单，6 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 10 | **Autocomplete Attribute** | `<input autocomplete="email" type="email" />` | 缺 autocomplete | High |
| 11 | **Semantic Input Types** | `<input type="email" />` `type="tel"` `type="url"` `type="number"` | 全用 `type="text"` | Medium |
| 12 | **Never Block Paste** | 允许密码 / 验证码粘贴 | `onPaste={e => e.preventDefault()}` | High |
| 13 | **Spellcheck Disable** | `<input spellCheck="false" type="email" />` | 邮箱/代码出现红色拼写波浪 | Low |
| 14 | **Submit Button Enabled** | `<button>{loading ? <Spinner /> : 'Submit'}</button>` 保持启用 | `<button disabled={loading}>Submit</button>` | Medium |
| 15 | **Inline Errors** | 错误在相应字段下 + 聚焦首错 | 表单顶部堆一坨错误 | High |

## 4. Performance（性能，5 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 16 | **Virtualize Lists** | 列表 > 50 项用 `<VirtualList items={items} />` | `items.map(item => <Item />)` 一次性渲染 | High |
| 17 | **Avoid Layout Reads** | `useEffect(() => { el.getBoundingClientRect() })` | 在 render 里 `getBoundingClientRect()` | Medium |
| 18 | **Batch DOM Operations** | 先写后读（`writes.forEach(); reads.forEach()`） | 写读交错（layout thrashing） | Medium |
| 19 | **Preconnect CDN** | `<link rel="preconnect" href="https://cdn.example.com" />` | 无 preconnect 提示 | Low |
| 20 | **Lazy Load Images** | `<img loading="lazy" src="..." />` 首屏之下 | 全部 eager 加载 | Medium |

## 5. State（状态管理，3 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 21 | **URL Reflects State** | `?tab=settings&page=2` 把 filter/tab/分页同步到 URL | 纯内存状态（刷新就丢） | High |
| 22 | **Deep Linking** | `router.push({ query: { ...filters } })` 可分享当前视图 | `setFilters(f)` 只在内存 | Medium |
| 23 | **Confirm Destructive Actions** | `if (confirm('Delete?')) delete()` | 点击即删除 | High |

## 6. Typography（排印，3 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 24 | **Proper Unicode** | `…` `'' ""` `—` 真正的 Unicode 字符 | `...` 三个点 / 直引号 `'"` | Low |
| 25 | **Text Overflow** | `truncate` / `line-clamp-2` / `break-words` | 文字溢出容器 | Medium |
| 26 | **Non-Breaking Spaces** | `10&nbsp;kg` / `Next.js&nbsp;14` | `10 kg` 可能换行分离 | Low |

## 7. Anti-Pattern（反模式，4 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 27 | **No Zoom Disable** | `<meta name="viewport" content="width=device-width">` | `<meta ... content="maximum-scale=1">`（禁缩放） | **Critical** |
| 28 | **No Transition All** | `transition-colors duration-200` 指定具体属性 | `transition: all` | Medium |
| 29 | **Outline Replacement**（与 #7 / #8 呼应） | `focus:outline-none focus:ring-2 focus:ring-blue-500` | `focus:outline-none` 单独一行 | **Critical** |
| 30 | **No Hardcoded Dates** | `new Intl.DateTimeFormat('en').format(date)` | `date.toLocaleDateString()` 硬编码或手写 | Medium |

---

## 关键速记（Critical 级必须全过）

1. **图标按钮必须 `aria-label`**（#1）
2. **表单输入必须有 label**（#2）
3. **`focus-visible` ring 必须可见**（#7）
4. **不许关闭 outline 不给替代**（#8 / #29）
5. **不许禁用 zoom**（#27）

## 与其他知识库的协同

- 与《07-UX 规范》的 Accessibility / Forms 部分有重合但视角不同——**《07》关注体验规则**，**本文关注 Web/HTML 级具体属性**。
- 与《09-React 性能》的部分条款（virtualize、layout reads）呼应——Web 角度看是 "DOM 操作效率"，React 角度看是 "render 策略"。
