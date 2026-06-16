# 技术栈 · nextjs

> **来源**：`stacks/nextjs.csv`。基于 Next.js 14/15 App Router。
> 配合《技术栈/react.md》+《知识库/09-React 性能.md》一起看。

---

## 1. Routing（App Router）

- 新项目用 `app/` 目录（不要 `pages/`）。
- 文件约定：`page.tsx` / `layout.tsx` / `loading.tsx` / `error.tsx` / `not-found.tsx`。
- **相关文件同目录**：`app/dashboard/_components/`（`_` 前缀私有）。
- **Route Groups**：`(marketing)/about/page.tsx`（括号不影响 URL）。
- **Loading UI**：`app/dashboard/loading.tsx` 而非手写 `useState(loading)`。
- **Error Handling**：`app/dashboard/error.tsx` with `reset()` 函数。

## 2. Rendering

| 规则 | Do | 严重 |
|---|---|---|
| **默认 Server Component** | 服务端组件减少 client bundle | **High** |
| **显式 `'use client'`** | 仅在用到 hooks / 事件 / 浏览器 API 时 | High |
| **Client 组件下推为叶节点** | `<InteractiveButton />` 包在 Server Page 里 | High |
| **流式渲染** | `<Suspense><SlowComponent /></Suspense>` | Medium |
| **选对渲染策略** | SSG 静态 · SSR 动态 · ISR 半静态；`export const revalidate = 3600` | Medium |

## 3. Data Fetching

```tsx
// ✅ 在 Server Component 直接 fetch
export default async function Page() {
  const data = await fetch(url, { cache: 'force-cache' })  // Next 15 必须显式
  return <UI data={data} />
}
```

| 规则 | Do | Don't |
|---|---|---|
| Server 组件直接 fetch | `async function Page() { const data = await fetch() }` | `useEffect` 做初次 fetch |
| **显式 cache（Next 15+）** | `fetch(url, { cache: 'force-cache' })` | 默认不缓存却期望缓存 |
| 去重 fetch | React / Next.js 自动去重相同 URL+headers | 手写缓存层 |
| Server Actions 处理突变 | `<form action={createPost}>` | 每个操作一个 API route |
| 重新校验 | `revalidatePath('/posts')` / `revalidateTag('posts')` | 到处 `router.refresh()` |

## 4. Images

```tsx
import Image from 'next/image'

<Image src="/hero.jpg" alt="Hero" width={1200} height={800} priority />
// 响应式铺满：
<Image src={url} alt="" fill className="object-cover" />
```

- 永远用 `<Image>`；`<img>` 禁。
- 必给 `width/height`（或 `fill` + 相对父级）。
- 首屏大图加 `priority`（优化 LCP）。

## 5. Fonts

```ts
// app/layout.tsx
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap' })
```

## 6. Metadata & SEO

```ts
export const metadata = {
  title: 'Page Title',
  description: '...',
  openGraph: { images: ['/og.jpg'] }
}
```

- 动态：`generateMetadata(params)`
- robots / sitemap：用 `app/robots.ts` / `app/sitemap.ts`

## 7. 性能

- **Parallel Routes** `@modal` 做并行渲染
- **`next/dynamic`** 做客户端动态加载 + `{ ssr: false }`
- **`after()`** 调度非阻塞后处理（日志 / 分析）
- **Partial Prerendering**（v15+）

## 8. 部署

- Vercel 最顺；Node 自托管 `next start`。
- Edge Runtime：`export const runtime = 'edge'` for low-latency routes。

---

## 9. 关键红线

- [ ] 不要把大页面标为 `'use client'`；把交互部分下推为叶组件
- [ ] Next 15+ 的 `fetch()` 不再默认缓存——显式 `cache` 或 `revalidate`
- [ ] `<Image>` 必须带 `width/height`
- [ ] Server Actions 用于所有突变——替代 REST API route
- [ ] `loading.tsx` / `error.tsx` / `not-found.tsx` 齐全
