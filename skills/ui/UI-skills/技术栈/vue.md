# 技术栈 · vue

> **来源**：`stacks/vue.csv`。基于 Vue 3 + Composition API。

---

## 1. 组件写法（Composition API）

```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

const count = ref(0)
const double = computed(() => count.value * 2)
watch(count, (v) => console.log(v))
</script>

<template>
  <button @click="count++">{{ double }}</button>
</template>
```

- **新项目用 `<script setup>`**——不要 Options API。
- **TypeScript**：`<script setup lang="ts">`。
- **`ref` vs `reactive`**：优先 `ref`（所有类型一致），`reactive` 仅深对象场景。
- **组件 props / emits**：`defineProps<{ name: string }>()` / `defineEmits<{ (e: 'update', v: string): void }>()`。

## 2. 响应式

| 规则 | Do | Don't |
|---|---|---|
| 解构响应对象需要 `toRefs` | `const { x, y } = toRefs(state)` | 直接解构 `reactive` 丢失响应 |
| 使用 `shallowRef` / `shallowReactive` 大对象 | 局部响应避免深递归开销 | 大对象全深响应 |
| `computed` 派生值 | `const total = computed(() => ...)` | 存入 ref 又手动同步 |
| `watch` / `watchEffect` 副作用 | `watchEffect(() => fetch(url.value))` | 手动 effect-like 逻辑 |

## 3. 组合式函数（Composables）

```ts
// composables/useFetch.ts
export function useFetch(url: Ref<string>) {
  const data = ref(null)
  const loading = ref(false)
  watchEffect(async () => {
    loading.value = true
    data.value = await (await fetch(url.value)).json()
    loading.value = false
  })
  return { data, loading }
}
```

- **以 `use` 前缀**。
- **返回 ref**，不要解构对象值。

## 4. 组件通信

- Props 向下 + emits 向上（简单场景）。
- `provide/inject` 跨层级（带类型：`InjectionKey<T>`）。
- **Pinia** 做全局 state（官方推荐，替代 Vuex）。

## 5. Pinia

```ts
// stores/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', () => {
  const name = ref('')
  const setName = (v: string) => { name.value = v }
  return { name, setName }
})
```

## 6. Vue Router

```ts
const routes = [
  { path: '/', component: Home },
  { path: '/users/:id', component: User, props: true },
]
```

- 路由参数通过 props 传入（`props: true`）。
- `<router-link>` + `<router-view>`。
- 导航守卫：`router.beforeEach` 鉴权。

## 7. 指令

- `v-if` vs `v-show`：频繁切换用 `v-show`。
- `v-for` 必带 `:key`（稳定 ID）。
- `v-for` + `v-if` 不同元素；要过滤先 `computed` 列表。
- `v-model` 自定义组件：`defineModel()`（Vue 3.4+）。

## 8. 性能

- **`v-memo`** 缓存昂贵子树：`v-memo="[item.id]"`。
- **`defineAsyncComponent`** 懒加载大组件。
- **`markRaw`** 避免非必要响应（如第三方实例）。
- **KeepAlive** 缓存路由视图：`<KeepAlive><component :is="view" /></KeepAlive>`。

## 9. TypeScript 重点

```ts
defineProps<{ name: string; age?: number }>()
defineEmits<{ (e: 'save', val: string): void }>()
const inputRef = ref<HTMLInputElement | null>(null)
```

## 10. 无障碍

- `<button>` 替代 `<div @click>`
- 图标按钮：`<button aria-label="Close"><XIcon /></button>`
- 表单：`<label for="email">`

---

## 关键红线

- [ ] 用 `<script setup>` + Composition API
- [ ] `computed` 派生，不存二次真源
- [ ] Pinia 替代 Vuex
- [ ] 组件通信优先 Props/Emits，再 provide/inject，最后 Pinia
- [ ] 列表必 `:key`
