# 技术栈 · svelte

> **来源**：`stacks/svelte.csv`。覆盖 Svelte 5 Runes + SvelteKit。

---

## 1. Runes（Svelte 5+）

```svelte
<script>
  let count = $state(0)
  let double = $derived(count * 2)
  let input = $state('')

  $effect(() => {
    console.log('count changed:', count)
    return () => console.log('cleanup')
  })
</script>

<button onclick={() => count++}>{double}</button>
<input bind:value={input} />
```

| Rune | 作用 |
|---|---|
| `$state(v)` | 响应状态 |
| `$derived(expr)` | 派生值（只读） |
| `$effect(fn)` | 副作用（返回 cleanup） |
| `$props()` | 接收 props |
| `$bindable()` | 可被父组件双绑 |

## 2. Stores（旧式，仍可用）

```ts
import { writable, derived, readable } from 'svelte/store'
export const count = writable(0)
```

新代码优先用 runes；已有 store 仍可用（`$store` 自动订阅）。

## 3. SvelteKit 路由

```
src/routes/
├── +layout.svelte        ← 根布局
├── +page.svelte          ← 首页
├── +page.server.ts       ← 服务端 load
├── +page.ts              ← 同构 load
├── users/[id]/+page.svelte
├── +error.svelte         ← 错误边界
└── api/users/+server.ts  ← REST endpoint
```

## 4. Data Loading

```ts
// +page.server.ts (仅服务端运行)
export async function load({ params, fetch }) {
  const user = await fetch(`/api/users/${params.id}`).then(r => r.json())
  return { user }
}
```

```svelte
<!-- +page.svelte -->
<script>
  let { data } = $props()
</script>
<h1>{data.user.name}</h1>
```

## 5. Form Actions

```ts
// +page.server.ts
export const actions = {
  default: async ({ request }) => {
    const form = await request.formData()
    // ...
    return { success: true }
  }
}
```

```svelte
<form method="POST" use:enhance>
  <input name="email" />
  <button>Submit</button>
</form>
```

- `use:enhance` progressive enhancement
- 无需 API route — form 直接提交到 action

## 6. Transitions / Animations

```svelte
<script>
  import { fade, fly, slide } from 'svelte/transition'
  import { flip } from 'svelte/animate'
</script>

{#if visible}
  <div transition:fade={{ duration: 200 }}>Hello</div>
{/if}
```

## 7. Performance

- **`{#key expr}`** 强制重建
- **`{#each items as item (item.id)}`** 稳定 key
- **`bind:this={element}`** 拿到 DOM
- SvelteKit **`prerender = true`** 静态化
- `svelte-virtual-list` 大列表虚拟化

## 8. TypeScript

```svelte
<script lang="ts">
  interface Props {
    name: string
    age?: number
  }
  let { name, age = 0 }: Props = $props()
</script>
```

## 9. 与 Tailwind 搭配

```bash
npx sv create my-app  # 选 Tailwind
```

全部规则参考《技术栈/html-tailwind.md》。

---

## 关键红线

- [ ] Svelte 5 新代码用 Runes（`$state` / `$derived` / `$effect`）
- [ ] SvelteKit 表单用 actions + `use:enhance`
- [ ] Load 函数做数据预取，避免组件内手写 fetch
- [ ] `{#each}` 必带 key
- [ ] 动画用 `svelte/transition` 内置，不要自写 `@keyframes`
