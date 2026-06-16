# 知识库 · 09 React / Next.js 性能（44 条）

> **来源**：由 `react-performance.csv` 完整转译。
> **作用**：React / Next.js 项目必读的性能优化 Checklist。按问题分类，含 Do / Don't 与代码示例。
> **搜索字段**：`Category / Issue / Keywords / Description`

---

## 严重性分级

- **Critical**：生产环境致命 → 必须修
- **High**：体验严重受损 → 强烈建议修
- **Medium**：轻量优化 → 有时间就做
- **Low**：细节打磨 → 有余力再做

---

## 1. Async Waterfall（异步瀑布流，5 条，全 Critical）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 1 | **Defer Await** | 把 `await` 放进实际使用它的分支 | 函数顶部 `await` 阻塞所有分支 | Critical |
| 2 | **Promise.all Parallel** | `const [a, b] = await Promise.all([fA(), fB()])` | 顺序 `const a = await fA(); const b = await fB()` | Critical |
| 3 | **Dependency Parallelization** | 用 `better-all`（or 手写 Promise.all 组合）让每个任务在最早时机启动 | 等无关数据回来再发起依赖请求 | Critical |
| 4 | **API Route Optimization** | `const sessionP = auth(); const configP = fetchConfig(); const session = await sessionP` | `const session = await auth(); const config = await fetchConfig()` | Critical |
| 5 | **Suspense Boundaries** | `<Suspense fallback={<Skeleton />}><DataDisplay /></Suspense>` | 整页 await 后才渲染 | High |

## 2. Bundle Size（包体积，5 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 6 | **Barrel Imports** | `import Check from 'lucide-react/dist/esm/icons/check'` | `import { Check } from 'lucide-react'`（barrel） | **Critical** |
| 7 | **Dynamic Imports** | `const Monaco = dynamic(() => import('./monaco'), { ssr: false })` | 顶层 `import { MonacoEditor }` | **Critical** |
| 8 | **Defer Third Party** | `dynamic(() => import('@vercel/analytics'), { ssr: false })` | 分析脚本入主 bundle | Medium |
| 9 | **Conditional Loading** | `useEffect(() => { if (enabled) import('./heavy.js') }, [enabled])` | 无条件 import 大模块 | High |
| 10 | **Preload Intent** | `onMouseEnter={() => import('./editor')}` | 只在 click 时 import | Medium |

## 3. Server（服务端，5 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 11 | **React.cache Dedup** | `export const getUser = cache(async () => await db.user.find())` | 同一请求内多次 fetch 相同数据 | Medium |
| 12 | **LRU Cache Cross-Request** | `const cache = new LRUCache({ max: 1000, ttl: 5*60*1000 })` | 每次请求都打 DB | High |
| 13 | **Minimize Serialization** | `<Profile name={user.name} />` | `<Profile user={user} />`（50 字段全序列化） | High |
| 14 | **Parallel Fetching** | 通过组件组合让 `<Header />` `<Sidebar />` 各自并行 fetch | 在父组件顺序 await | **Critical** |
| 15 | **After Non-blocking** | `after(async () => { await logAction() })` | await 日志/分析阻塞响应 | Medium |

## 4. Client（客户端数据，2 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 16 | **SWR Deduplication** | `const { data } = useSWR('/api/users', fetcher)` | `useEffect(() => { fetch(...) })` 手写 | Medium-High |
| 17 | **Event Listener Dedup** | `useSWRSubscription('global-keydown', ...)` | 每个组件都 `window.addEventListener` | Low |

## 5. Rerender（重渲染，7 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 18 | **Defer State Reads** | 回调里直接 `new URLSearchParams(location.search)` | 顶层 `useSearchParams()` 订阅只在 handler 使用的值 | Medium |
| 19 | **Memoized Components** | 把昂贵计算抽到 `memo(({ user }) => ...)` 并 early return loading | 在 early return 前做昂贵计算 | Medium |
| 20 | **Narrow Dependencies** | `useEffect(() => {}, [user.id])` | `useEffect(() => {}, [user])`（对象引用） | Low |
| 21 | **Derived State** | `const isMobile = useMediaQuery('(max-width: 767px)')` | 订阅连续 `width` 再推导 | Medium |
| 22 | **Functional setState** | `setItems(curr => [...curr, newItem])` | `setItems([...items, newItem])`（需把 items 入依赖） | Medium |
| 23 | **Lazy State Init** | `useState(() => buildSearchIndex(items))` | `useState(buildSearchIndex(items))`（每次 render 都跑） | Medium |
| 24 | **Transitions** | `startTransition(() => setScrollY(window.scrollY))` | 每次滚动都阻塞 UI | Medium |

## 6. Rendering（渲染，6 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 25 | **SVG Animation Wrapper** | `<div class='animate-spin'><svg>...</svg></div>` | 直接动画 `<svg>` | Low |
| 26 | **Content Visibility** | `.item { content-visibility: auto; contain-intrinsic-size: 0 80px }` | 1000 条列表一次性渲染 | High |
| 27 | **Hoist Static JSX** | `const skeleton = <div class='animate-pulse' />` 模块顶层 | 组件内每次重建相同节点 | Low |
| 28 | **Hydration No Flicker** | `<script dangerouslySetInnerHTML={{ __html: 'el.className=localStorage.theme' }} />` | `useEffect` 设置主题 → 闪烁 | Medium |
| 29 | **Conditional Render** | `{count > 0 ? <Badge>{count}</Badge> : null}` | `{count && <Badge>{count}</Badge>}`（0/NaN 会渲染） | Low |
| 30 | **Activity Component** | `<Activity mode={isOpen ? 'visible' : 'hidden'}><Menu /></Activity>` | `{isOpen && <Menu />}`（丢状态） | Medium |

## 7. JS Perf（通用 JS 性能，12 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 31 | **Batch DOM/CSS** | `element.classList.add('highlighted')` 或 `cssText` 批改 | 单属性一个个改（触发 reflow） | Medium |
| 32 | **Index Map Lookup** | `const byId = new Map(users.map(u => [u.id, u])); byId.get(id)` | `users.find(u => u.id === order.userId)` 循环里 | Low-Medium |
| 33 | **Cache Property Access** | `const val = obj.config.settings.value; for (...) process(val)` | 循环里逐次访问深层属性 | Low-Medium |
| 34 | **Cache Function Results** | 模块级 `Map` 缓存；`if (cache.has(x)) return cache.get(x)` | 100 次相同输入都重新计算 | Medium |
| 35 | **Cache Storage API** | `if (!cache.has(key)) cache.set(key, localStorage.getItem(key))` | 每次都 `localStorage.getItem('theme')` | Low-Medium |
| 36 | **Combine Iterations** | 单次 for 循环同时分类（`for (u of users) { if (u.isAdmin) admins.push(u); ... }`） | 连续多个 `.filter()` | Low-Medium |
| 37 | **Length Check First** | `if (a.length !== b.length) return true; // 再比` | 长度不同也跑昂贵对比 | Medium-High |
| 38 | **Early Return** | 首个错误立刻 `return { error: 'Email required' }` | 处理完所有项再判断 | Low-Medium |
| 39 | **Hoist RegExp** | `const EMAIL_RE = /^[^@]+@[^@]+$/;` 模块顶层 | 组件内 `new RegExp(...)` | Low-Medium |
| 40 | **Loop Min/Max** | `let max = arr[0]; for (x of arr) if (x > max) max = x`（O(n)） | `arr.sort((a,b) => b-a)[0]`（O(n log n)） | Low |
| 41 | **Set/Map Lookups** | `const allowed = new Set(['a','b']); allowed.has(id)`（O(1)） | `allowed.includes(id)`（O(n)） | Low-Medium |
| 42 | **toSorted Immutable** | `users.toSorted((a,b) => a.name.localeCompare(b.name))` | `users.sort(...)`（会 mutate） | Medium-High |

## 8. Advanced（进阶，2 条）

| # | Issue | Do | Don't | 严重 |
|---|---|---|---|---|
| 43 | **Event Handler Refs** | `const onEvent = useEffectEvent(handler); useEffect(() => listen(onEvent), [])` | `useEffect(() => listen(handler), [handler])`（每次重订阅） | Low |
| 44 | **useLatest Hook** | `const cbRef = useLatest(cb); setTimeout(() => cbRef.current())` | `useEffect(() => { setTimeout(cb) }, [cb])`（reruns） | Low |

---

## 关键速记（5 条最关键）

1. **瀑布流必断** → 用 `Promise.all` + Suspense 边界。
2. **Barrel imports 必除** → 用深路径 import。
3. **大组件必 dynamic** → `ssr: false` 延迟加载。
4. **Parallel fetching via composition** → Next.js RSC 下让子组件各自 fetch。
5. **toSorted / Set.has** → 现代 API 远比 sort / Array.includes 高效。
