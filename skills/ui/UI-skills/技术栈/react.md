# 技术栈 · react

> **来源**：`stacks/react.csv`（53 条）。
> 配合《知识库/09-React性能.md》一起看——本文侧重 **React 最佳实践**（State / Hooks / Components / TypeScript / Patterns），性能篇在 09。

---

## 1. State

| # | 规则 | Do | Don't | 严重 |
|---|---|---|---|---|
| 1 | 局部状态 | `const [count, setCount] = useState(0)` | `this.state = { count: 0 }` | Medium |
| 2 | 状态上提 | 把共享状态提到公共祖先 | 5 层 prop drilling | Medium |
| 3 | 复杂状态 | `useReducer` + action types | 5+ 关联 `useState` | Medium |
| 4 | 避免可推导状态 | 渲染中直接计算 `const total = items.reduce(...)` | 存入 state | **High** |
| 5 | 惰性初始化 | `useState(() => computeExpensive())` | `useState(computeExpensive())` | Medium |

## 2. Effects

| # | 规则 | Do | Don't | 严重 |
|---|---|---|---|---|
| 6 | 清理 | `useEffect(() => { sub(); return unsub })` | 无清理 | **High** |
| 7 | 依赖齐全 | 所有引用值入依赖 | 空依赖仍引用外部 | **High** |
| 8 | 避免无意义 effect | 派生值在 render 直接算；事件直接处理 | 用 effect 做数据 transform | **High** |
| 9 | 非响应值用 ref | `useRef(null)` 存 interval id / DOM | `useState` 存不需要触发渲染的值 | Medium |

## 3. Rendering

| # | 规则 | Do | Don't |
|---|---|---|---|
| 10 | 稳定 key | `key={item.id}` | `key={index}`（动态列表） |
| 11 | `useMemo` | 昂贵计算用 | 每次 render 都重算 |
| 12 | `useCallback` | 传给 memoized 子组件的函数用 | 每次 render 都新建函数引用 |
| 13 | `React.memo` | 频繁渲染且 props 稳定的纯组件 | 全局乱用或完全不用 |
| 14 | 避免 JSX 内联对象 | 外部或 memo `styles.container` | `style={{ margin: 10 }}` |

## 4. Components

| 规则 | Do |
|---|---|
| 15 小而单一 | 每个组件一个关注点；`<UserAvatar /> <UserName />` |
| 16 组合 > 继承 | `<Card>{content}</Card>`；用 children 传 |
| 17 同主题文件同目录 | `components/User/UserCard.tsx` + `hooks/useUser.ts` |
| 18 Fragment | `<>{items.map(...)}</>` 避免无意义 div |

## 5. Props

| 规则 | Do | Don't |
|---|---|---|
| 19 解构 | `function User({ name, age })` | `props.name` 反复 |
| 20 默认值 | `function Button({ size = 'md' })` | 条件判断 undefined |
| 21 避免 prop drilling | `Context` 传全局数据；composition 传 UI | 5 层传 |
| 22 TypeScript prop 类型 | `interface ButtonProps { onClick: () => void }` | PropTypes 或无 |

## 6. Events

| 规则 | Do |
|---|---|
| 23 合成事件 | `e.preventDefault()` / `e.stopPropagation()` |
| 24 避免 render 内 bind | 使用箭头函数 |
| 25 传函数引用而非调用 | `onClick={handleClick}`（**不是** `handleClick()`） |

## 7. Forms

| 规则 | Do | Don't |
|---|---|---|
| 26 受控组件 | `<input value={val} onChange={setVal}>` | 全用 ref |
| 27 表单提交 | `<form onSubmit={handleSubmit}>` + `preventDefault()` | 仅在 button `onClick` |
| 28 Debounce | `useDeferredValue(searchTerm)` 或自行防抖 | 每次 keystroke 都过滤 |

## 8. Hooks Rules

| 规则 | Do | Don't |
|---|---|---|
| 29 顶层调用 | 组件顶层 | 条件/循环/回调里 |
| 30 自定义 hook | `useFetch` / `useForm` 提取 | 重复 effect/state |
| 31 `use` 前缀 | `function useFetch(url)` | `fetchData` |

## 9. Context

| 规则 | Do |
|---|---|
| 32 全局数据 | 主题 / Auth / Locale |
| 33 按关注点拆分 | `ThemeContext` + `AuthContext`，不要一个大 AppContext |
| 34 Memo context value | `value={useMemo(() => ({ ... }), [])}` |

## 10. Performance（见《09-React 性能.md》）

35-38 核心要点：

- **Profile** → 用 React DevTools Profiler
- **Lazy load** → `const Page = lazy(() => import('./Page'))`
- **Virtualize** → `react-window` / `react-virtual`（> 100 项）
- **Batch updates** → React 18 自动批处理

## 11. Error Handling

| 规则 | Do |
|---|---|
| 39 Error Boundary | `<ErrorBoundary><App /></ErrorBoundary>` |
| 40 Async error | `try { await fetch() } catch(e) {}` |

## 12. Testing

| 规则 | Do |
|---|---|
| 41 测试行为 | `expect(screen.getByText('Hello'))` |
| 42 可达性查询 | `getByRole('button')` / `getByLabelText` 优先于 `getByTestId` |

## 13. Accessibility

| 规则 | Do |
|---|---|
| 43 语义化 HTML | `<button>` / `<nav>` / `<main>` |
| 44 Focus 管理 | 模态聚焦陷阱 + 关闭时返还 |
| 45 Live Regions | `<div aria-live="polite">{msg}</div>` |
| 46 Label 关联 | `<label htmlFor="email">Email</label>` |

## 14. TypeScript

| 规则 | Do |
|---|---|
| 47 Props 类型 | `interface Props { name: string }` |
| 48 State 类型 | `useState<User \| null>(null)` |
| 49 事件类型 | `React.ChangeEvent<HTMLInputElement>` |
| 50 泛型组件 | `<List<T> items={T[]} />` |

## 15. Patterns

| 规则 | Do |
|---|---|
| 51 容器 / 展示分离 | `<UserContainer><UserView /></UserContainer>` |
| 52 Render props | `<DataFetcher render={data => ...} />` |
| 53 复合组件 | `<Tabs><Tab /><TabPanel /></Tabs>` 共享 context |
