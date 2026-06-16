# 技术栈 · astro

> **来源**：`stacks/astro.csv`。Astro 4+ 内容优先 / Islands 架构。

---

## 1. 何时用 Astro

- 内容站（博客 / 文档 / Marketing / Portfolio）
- SEO 重要 + 交互少 → Astro 是最优解
- 不适合 App / Dashboard（用 Next.js / Nuxt / SvelteKit）

## 2. 目录约定

```
src/
├── pages/            ← 文件路由（.astro / .md / .mdx）
├── layouts/
├── components/       ← .astro / .jsx / .vue / .svelte
├── content/          ← Content Collections
public/               ← 静态资源
```

## 3. `.astro` 文件

```astro
---
// 服务端代码（构建时运行）
import Layout from '../layouts/Base.astro'
const posts = await Astro.glob('./posts/*.md')
const { title } = Astro.props
---
<Layout title={title}>
  {posts.map(p => <a href={p.url}>{p.frontmatter.title}</a>)}
</Layout>

<style>
  /* scoped */
  h1 { color: var(--primary); }
</style>
```

## 4. Islands 架构

```astro
---
import Counter from '../components/Counter.jsx'
---
<!-- 全静态（默认） -->
<Counter />
<!-- 客户端水合（仅在此岛内注入 JS） -->
<Counter client:load />
<Counter client:idle />
<Counter client:visible />
<Counter client:media="(max-width: 768px)" />
<Counter client:only="react" />
```

| 指令 | 语义 |
|---|---|
| `client:load` | 页面加载即水合 |
| `client:idle` | 空闲时水合 |
| `client:visible` | 进入视口再水合（最常用） |
| `client:media` | 匹配媒体查询才水合 |
| `client:only` | 完全客户端，跳过 SSR |

## 5. Content Collections

```ts
// src/content/config.ts
import { defineCollection, z } from 'astro:content'

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    date: z.date(),
    tags: z.array(z.string()),
  }),
})

export const collections = { blog }
```

```astro
---
import { getCollection } from 'astro:content'
const posts = await getCollection('blog')
---
```

## 6. Multi-Framework

同一项目可混用 React / Vue / Svelte / Solid：

```bash
npx astro add react vue svelte
```

```astro
---
import MyReact from './MyReact.jsx'
import MyVue from './MyVue.vue'
import MySvelte from './MySvelte.svelte'
---
<MyReact client:visible />
<MyVue client:load />
<MySvelte client:idle />
```

## 7. Image & Asset

```astro
---
import { Image } from 'astro:assets'
import hero from '../assets/hero.jpg'
---
<Image src={hero} alt="Hero" width={1200} height={800} />
```

- Astro 构建时优化；支持 WebP / AVIF。

## 8. SEO / Sitemap

```bash
npx astro add sitemap
```

```js
// astro.config.mjs
import sitemap from '@astrojs/sitemap'
export default defineConfig({
  site: 'https://example.com',
  integrations: [sitemap()],
})
```

## 9. 性能

- 默认零 JS 输出 → 极快 LCP
- Islands 只水合必要部分
- View Transitions：`<ViewTransitions />` 无 JS 的 SPA 体验
- `astro:transitions` 处理跨页过渡

---

## 关键红线

- [ ] 默认全静态；只在**真需要交互**时加 `client:*`
- [ ] 首选 `client:visible`，最少水合
- [ ] 内容用 Content Collections 而非手写 markdown glob
- [ ] 图片用 `<Image from 'astro:assets'>`
- [ ] 避免把整站写成 React SPA —— 丢失 Astro 价值
