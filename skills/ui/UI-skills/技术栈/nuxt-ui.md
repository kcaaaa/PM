# 技术栈 · nuxt-ui

> **来源**：`stacks/nuxt-ui.csv`。Nuxt UI（基于 Tailwind + Headless UI 的组件库）。

---

## 1. Setup

```bash
npx nuxi module add ui
```

`nuxt.config.ts`：

```ts
export default defineNuxtConfig({
  modules: ['@nuxt/ui'],
  ui: {
    primary: 'blue',    // 主题主色
    gray: 'cool',       // 中性色
  }
})
```

## 2. 常用组件

| 组件 | 用途 |
|---|---|
| `<UButton>` | 按钮，支持 `color / variant / size / icon / loading` |
| `<UInput>` / `<UTextarea>` | 输入框 |
| `<USelect>` / `<USelectMenu>` | 下拉 |
| `<UModal>` / `<USlideover>` | 模态 / 抽屉 |
| `<UTable>` | 数据表 |
| `<UCard>` | 卡片 |
| `<UAvatar>` | 头像 |
| `<UBadge>` | 徽章 |
| `<UTabs>` | Tab |
| `<UDropdown>` | 下拉菜单 |
| `<UTooltip>` | 提示 |
| `<UToast>` | 吐司（全局 `useToast()`） |
| `<UForm>` | 表单 + 校验（Zod / Yup / Joi） |

## 3. 表单示例（最常用）

```vue
<script setup lang="ts">
import { z } from 'zod'
const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
})
const state = reactive({ email: '', password: '' })
async function onSubmit(event) {
  console.log(event.data)
}
</script>

<template>
  <UForm :schema="schema" :state="state" @submit="onSubmit">
    <UFormGroup label="Email" name="email">
      <UInput v-model="state.email" />
    </UFormGroup>
    <UFormGroup label="Password" name="password">
      <UInput v-model="state.password" type="password" />
    </UFormGroup>
    <UButton type="submit">Submit</UButton>
  </UForm>
</template>
```

## 4. 主题定制

```ts
// app.config.ts
export default defineAppConfig({
  ui: {
    primary: 'emerald',
    button: {
      default: { size: 'md', color: 'primary' }
    }
  }
})
```

颜色来自《03-配色方案》→ 把 Primary hex 映射到 `primary`，CTA 色可做自定义 color。

## 5. 暗色模式

```vue
<UColorMode />  <!-- 自动切换 -->
```

- 默认走 Tailwind `dark:` 类
- 所有 Nuxt UI 组件都原生支持

## 6. Icon

内置 **Iconify**（`i-heroicons-*`、`i-lucide-*`、`i-simple-icons-*`）：

```vue
<UButton icon="i-heroicons-arrow-right">Next</UButton>
<UIcon name="i-lucide-menu" class="w-6 h-6" />
```

配合《知识库/08-图标库》优先使用 Lucide 集合。

---

## 关键红线

- [ ] 用 `<UButton>` / `<UInput>` 等内置组件；不要手写样式一致的
- [ ] 主题色改 `app.config.ts` 的 `ui.primary`
- [ ] 表单用 `<UForm>` + Zod schema
- [ ] Toast 用全局 `useToast()`
- [ ] Icon 用 Iconify（`i-lucide-*`）
