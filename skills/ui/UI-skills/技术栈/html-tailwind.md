# 技术栈 · html-tailwind（默认栈）⭐

> **来源**：`stacks/html-tailwind.csv`（55 条）。
> **定位**：当用户未指定任何框架时，**这就是默认栈**。适合纯网页 / 静态站点 / 原型 / Marketing Page。
> **基于**：Tailwind CSS v3/v4。

---

## 1. Setup

```html
<!-- 最小化单文件方案（CDN，仅原型用）-->
<script src="https://cdn.tailwindcss.com"></script>
```

生产项目：

```bash
npm init -y
npm i -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```js
// tailwind.config.js（Tailwind v3）
export default {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary:    '#2563EB',   // 从《03-配色方案》拷贝
        secondary:  '#3B82F6',
        cta:        '#F97316',
        background: '#F8FAFC',
        text:       '#1E293B',
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],  // 从《04-字体搭配》拷贝
      },
    },
  },
}
```

---

## 2. Animation

| 规则 | Do | Don't | 严重 |
|---|---|---|---|
| 使用内置动画工具类 | `animate-pulse` `animate-spin` `animate-ping` | 为简单效果写 `@keyframes` | Medium |
| 限制 bounce | `animate-bounce` 仅偶尔用在 CTA | 满屏多个 bounce | **High** |
| 过渡时长 | `duration-150 ~ 300` | `duration-1000` | Medium |
| Hover 过渡 | `hover:bg-gray-100 transition-colors` | 无过渡瞬时切换 | Low |

## 3. Z-Index

| 规则 | Do | Don't | 严重 |
|---|---|---|---|
| 使用 scale | `z-0 z-10 z-20 z-30 z-40 z-50` | `z-[9999]` | Medium |
| 固定元素 | `z-50` nav · `z-40` dropdowns | 靠 DOM 顺序 | **High** |
| 负值做装饰背景 | `-z-10` | 背景反而是正值 | Low |

## 4. Layout

| 规则 | Do | Don't |
|---|---|---|
| 容器宽度 | `max-w-7xl mx-auto` + `px-4` | `w-full` 不限宽 |
| 响应内边距 | `px-4 md:px-6 lg:px-8` | 固定 `px-8` |
| 网格间距 | `grid gap-6` | 每个 item 加 `mb-*` |
| Flex 对齐 | `flex items-center justify-between` | 嵌套多层 wrapper |
| Container Queries | `@container @lg:grid-cols-2` | 组件内用 @media |

## 5. Images

| 规则 | Do |
|---|---|
| 保持比例 | `aspect-video` / `aspect-square` |
| 裁剪方式 | `object-cover` / `object-contain w-full h-full` |
| 懒加载 | `<img loading="lazy" src="...">` |
| 响应式 | `srcset` + `sizes` 多尺寸 |
| SVG 显式尺寸 | `<svg class="size-6" width="24" height="24">`（防 CLS） |

## 6. Typography

| 规则 | Do |
|---|---|
| 富文本 | `prose prose-lg max-w-none`（Tailwind Typography 插件） |
| 行高 | `leading-relaxed`（1.625） |
| 字号 scale | `text-sm text-base text-lg text-xl` |
| 文字省略 | `line-clamp-2` / `truncate` |

## 7. Colors

| 规则 | Do | Don't |
|---|---|---|
| 透明度 | `bg-black/50 text-white/80` | `bg-black opacity-50` |
| 暗色模式 | `dark:bg-gray-900 dark:text-white` | 不支持 |
| 语义命名 | `bg-primary bg-cta` | 到处 `bg-blue-500` |
| 梯度（v4） | `bg-linear-to-r from-blue-500 to-purple-500` | `bg-gradient-to-r`（v4 弃用） |
| 直接用主题色 | `bg-primary text-success` | `bg-[var(--color-primary)]`（别 `var()` 套壳） |

## 8. Spacing

| 规则 | Do | Don't |
|---|---|---|
| 一致 scale | `p-4 m-6 gap-8` | `p-[15px]` |
| 负 margin | 重叠特效 `-mt-4` | 修 bug 用负值 |
| 垂直列表间距 | `space-y-4` | 每个子元素 `mb-4` |
| 方形尺寸 | `size-6`（= `h-6 w-6`） | `h-6 w-6` 两条 |
| `shrink-0` 简写 | `shrink-0` | `flex-shrink-0` |

## 9. Forms

| 规则 | Do | Don't |
|---|---|---|
| 焦点态 | `focus:ring-2 focus:ring-blue-500 focus:ring-offset-2` | `focus:outline-none` 无替代 |
| 输入尺寸 | `h-10 w-full px-3` | 不一致高度 |
| 禁用态 | `disabled:opacity-50 disabled:cursor-not-allowed` | 与正常态同样 |
| Placeholder 样式 | `placeholder:text-gray-400` | 默认黑色 |

## 10. Responsive

| 规则 | Do |
|---|---|
| 移动优先 | 默认 mobile → `md:` `lg:` `xl:` |
| 断点测试 | 320 / 375 / 768 / 1024 / 1280 / 1536 |
| 显隐 | `hidden md:block` / `hidden md:flex` |

## 11. Buttons

| 规则 | Do |
|---|---|
| 尺寸 | `px-4 py-2` 或 `px-6 py-3` |
| 触控目标 | `min-h-[44px] min-w-[44px]`（移动端） |
| 加载态 | `<button disabled><Spinner /></button>` |
| 图标按钮 | `<button aria-label="Close"><XIcon /></button>` |

## 12. Cards

| 规则 | Do |
|---|---|
| 基础样式 | `rounded-lg shadow-md p-6` |
| Hover | `hover:shadow-lg transition-shadow` |
| 内部间距 | `space-y-4` 或 `p-6` |

## 13. Accessibility

| 规则 | Do |
|---|---|
| 屏幕阅读器文字 | `<span class="sr-only">Close menu</span>` |
| Focus visible（仅键盘） | `focus-visible:ring-2` |
| Reduced motion | `motion-reduce:animate-none motion-reduce:transition-none` |

## 14. Performance

| 规则 | Do | Don't |
|---|---|---|
| 配置 content 路径 | `content: ['./src/**/*.{js,ts,jsx,tsx}']` | `purge: [...]`（v2 老 API） |
| JIT | v3 默认启用 | 关闭 JIT |
| 少用 `@apply` | 直接写工具类 | 到处 `@apply` |

## 15. Plugins

- `@tailwindcss/forms` — 表单基础样式
- `@tailwindcss/typography` — `prose` 富文本
- `@tailwindcss/aspect-ratio` — 比例工具
- `@tailwindcss/container-queries` — `@container` 支持

## 16. Interactivity

| 规则 | Do |
|---|---|
| Group / Peer | `group-hover:text-blue-500` / `peer-checked:` |
| Arbitrary values | 一次性值 `w-[350px]`，避免 inline style |

---

## 17. 关键红线（同于《检查清单/通用UI规范.md》）

- [ ] SVG 图标（Heroicons/Lucide），无 Emoji 当 icon
- [ ] 所有可点击 `cursor-pointer`
- [ ] Hover 用 `transition-colors`，不用 `scale` 引起 layout shift
- [ ] 浅色模式正文对比度 ≥ 4.5:1
- [ ] 背景透明/glass 元素在浅色下必须有足够不透明度（`bg-white/80+`）
- [ ] Navbar 浮动时给 `top-4 left-4 right-4`，不要 `top-0 left-0`
- [ ] 响应断点 375 / 768 / 1024 / 1440 都要过
- [ ] `prefers-reduced-motion` 必须尊重
