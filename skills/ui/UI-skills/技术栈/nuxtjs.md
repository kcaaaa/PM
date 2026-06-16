# 技术栈 · nuxtjs

> **来源**：`stacks/nuxtjs.csv`。基于 Nuxt 3 + Nitro 引擎。

---

## 1. 目录约定

```
app.vue                  ← 根
pages/                   ← 文件路由
├── index.vue
├── users/[id].vue
layouts/                 ← 布局
components/              ← 自动导入
composables/             ← 自动导入（需 use 前缀）
middleware/              ← 路由中间件
server/
├── api/hello.ts         ← /api/hello
└── middleware/
plugins/
stores/                  ← Pinia（需 @pinia/nuxt）
```

## 2. Data Fetching

```vue
<script setup>
// SSR 友好、自动去重、返回 reactive
const { data: users } = await useFetch('/api/users')

// 仅客户端（大数据 / 交互）
const { data } = await useLazyFetch('/api/heavy')

// useAsyncData 更通用
const { data } = await useAsyncData('users', () => $fetch('/api/users'))
</script>
```

| API | 使用场景 |
|---|---|
| `useFetch` | SSR + 客户端；自动关联 key |
| `useLazyFetch` | 不阻塞渲染，客户端拉 |
| `useAsyncData` | 更通用，可自定义 transform |
| `$fetch` | 低级底层调用，服务端/客户端通用 |

- **避免 `onMounted` 中 fetch**——SSR 失效。
- Next 15 类似坑：**显式 `{ server: true/false }`** 决定执行位置。

## 3. Server Routes（API）

```ts
// server/api/users.get.ts
export default defineEventHandler(async (event) => {
  const users = await db.user.findMany()
  return users
})
```

- 文件名即方法：`.get.ts` / `.post.ts` / `.put.ts` / `.delete.ts`
- 动态 `[id].get.ts`
- Nitro 自动生成 OpenAPI、支持 Edge 部署。

## 4. SEO / Meta

```vue
<script setup>
useSeoMeta({
  title: '主页',
  ogImage: 'https://example.com/og.jpg',
  twitterCard: 'summary_large_image',
})
</script>
```

- `useHead` 更底层；`useSeoMeta` 更声明。
- `definePageMeta({ layout: 'default', middleware: ['auth'] })`

## 5. Image

```vue
<script setup>
// 模块：@nuxt/image
</script>
<template>
  <NuxtImg src="/hero.jpg" width="1200" height="800" format="webp" />
  <NuxtPicture src="/hero.jpg" :img-attrs="{ class: 'rounded-lg' }" />
</template>
```

## 6. 路由

- 文件路由：`pages/users/[id].vue` → `/users/:id`
- `<NuxtLink>` 替代 `<router-link>`
- 嵌套路由：`users.vue` + `users/[id].vue`
- `definePageMeta({ middleware: 'auth' })` 守卫

## 7. 状态

- **Pinia 推荐**：`npx nuxi module add pinia`
- **`useState(key, initFn)`** 跨组件共享 SSR-safe 状态

## 8. 性能

- **`<NuxtLazyHydrate>`** 可视区内再激活组件
- **Islands**（`<NuxtIsland>`）部分水合
- **`payloadExtraction: true`** 静态化
- **`nitro.prerender.routes`** 预渲染静态页

---

## 关键红线

- [ ] 不要在 `onMounted` 做初次数据 fetch
- [ ] 用 `useFetch/useAsyncData` 取代 axios
- [ ] Server API 放 `server/api/`，客户端直接 `/api/...`
- [ ] SEO 用 `useSeoMeta`
- [ ] 图片用 `@nuxt/image` 的 `<NuxtImg>`
