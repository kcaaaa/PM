# 检查清单 · 通用 UI 规范（防业余感）

> 这些是**最常被忽略**的 UI 细节。严格遵守可以让任何界面立刻脱离"业余感"。
> 本文对应原 UI-skill.md 中的 *"Common Rules for Professional UI"* 部分。

---

## 1. 图标与视觉元素

| 规则 | Do（正确） | Don't（错误） |
|---|---|---|
| **不用 Emoji 做 icon** | 使用 SVG（Heroicons / Lucide / Simple Icons） | 用 🎨 🚀 ⚙️ 💡 当 UI icon |
| **悬停态稳定** | Hover 用 color / opacity 过渡 | 用 `transform: scale` 引起 layout shift |
| **品牌 Logo 正确** | 从 Simple Icons 取官方 SVG | 凭记忆画 logo 或拿错路径 |
| **图标尺寸一致** | 统一 24×24 viewBox + `w-6 h-6` | 不同图标混用不同尺寸 |

## 2. 交互与光标

| 规则 | Do | Don't |
|---|---|---|
| **cursor-pointer** | 所有可点击 / 悬停的卡片加 `cursor-pointer` | 可交互元素保持默认光标 |
| **Hover 反馈** | 颜色 / 阴影 / 边框变化 | 毫无可交互提示 |
| **过渡平滑** | `transition-colors duration-200` | 瞬时切换或 > 500ms |

## 3. 浅色 / 深色模式对比度

| 规则 | Do | Don't |
|---|---|---|
| **Glass Card 浅色态** | `bg-white/80` 或更高不透明度 | `bg-white/10`（近乎透明） |
| **浅色正文** | `#0F172A`（slate-900） | `#94A3B8`（slate-400） |
| **次要文字** | 至少 `#475569`（slate-600） | gray-400 或更浅 |
| **边框可见** | `border-gray-200` 浅色模式 | `border-white/10`（看不见） |

## 4. 布局与间距

| 规则 | Do | Don't |
|---|---|---|
| **浮动导航栏** | `top-4 left-4 right-4` 留空 | `top-0 left-0 right-0` 贴边 |
| **内容 padding** | 为 fixed navbar 预留高度 | 内容被 fixed 元素遮挡 |
| **一致的最大宽度** | 统一 `max-w-6xl` 或 `max-w-7xl` | 不同页面用不同容器宽 |

## 5. 主题变量使用

| 规则 | Do | Don't |
|---|---|---|
| **直接用主题色** | `bg-primary text-success` | `bg-[var(--color-primary)]` 多此一举 |
| **`dark:` 前缀配对** | `bg-white dark:bg-gray-900` | 只写一模式 |
| **CSS 变量用 HSL** | HSL 变量让渐变更好控 | 一律硬编码 hex |

## 6. 阴影与圆角

| 规则 | 细节 |
|---|---|
| **圆角一致** | Card 用 `rounded-xl`（12px）或 `rounded-2xl`（16px）；Button 同级 |
| **阴影层级** | `shadow-sm → shadow-md → shadow-lg → shadow-xl` 递增；一个页面不超过 3 档 |
| **Hover 阴影变化** | `hover:shadow-lg transition-shadow` |

## 7. 动效与动画

| 规则 | 细节 |
|---|---|
| **Reduced Motion** | `@media (prefers-reduced-motion: reduce)` 必须降级 |
| **持续动画** | 只用于 Loader；禁止装饰性 `animate-bounce` |
| **Transform 优先** | `transform` / `opacity` 动画；避免 `width/height/top/left` |

## 8. Focus 可见性

| 规则 | 细节 |
|---|---|
| **绝不 `outline-none` 独立使用** | 必须给替代：`focus:outline-none focus:ring-2 focus:ring-blue-500` |
| **仅键盘用户可见** | 推荐 `focus-visible:ring-2` 而非 `focus:ring-2` |
| **对比度** | Focus ring 对比背景 ≥ 3:1 |

## 9. 图片与媒体

| 规则 | 细节 |
|---|---|
| **Alt text** | 所有有意义图片必带描述性 alt |
| **Lazy load** | 首屏之下 `loading="lazy"` |
| **响应式** | `<img srcset="..." sizes="...">` 或 `<picture>` |
| **固定尺寸** | `width` / `height` 或 `aspect-ratio` 防 CLS |
| **Video** | 无自动播放（`autoplay loop`）；必须 `muted playsinline preload="none"` |

## 10. 表单

| 规则 | 细节 |
|---|---|
| **Label 永在** | 不允许只用 placeholder |
| **错误靠近字段** | 错误文本在字段下方，不是表单顶部 |
| **提交按钮显示 loading** | 禁点 + spinner |
| **Input 类型正确** | `type="email"` / `"tel"` / `"number"` / `"url"` |
| **autocomplete** | `autocomplete="email"` / `"current-password"` 等 |
| **不阻断粘贴** | 密码 / 验证码可以粘贴 |

## 11. 响应式

| 规则 | 细节 |
|---|---|
| **测四档** | 375 / 768 / 1024 / 1440 |
| **移动字号 ≥ 16px** | `text-base` 起步 |
| **不允许横向滚动** | `overflow-x-hidden` 必加到 `<body>` |
| **触控目标 ≥ 44×44** | `min-h-[44px] min-w-[44px]` |

## 12. 无障碍速记

| 规则 | 细节 |
|---|---|
| **语义 HTML** | `<button>/<a>/<nav>/<main>` 取代 `<div>` |
| **ARIA 仅在需要时加** | 先用原生 HTML，再加 ARIA |
| **图标按钮** | `aria-label="Close"` |
| **装饰图标** | `aria-hidden="true"` |
| **颜色非唯一** | 状态靠颜色 + 图标 / 文字 |
| **跳过导航** | `<a href="#main">Skip to main</a>` |

## 13. 红线：AI 流行色（反模式）

下列产品类目下**禁用** "AI 紫 + 粉渐变"（`#6366F1` → `#EC4899`）——即使用户没明令：

- 政府 / 公共服务
- 医疗（诊所、药店、老年护理）
- 金融（银行、保险）
- 法律 / 咨询 / B2B 服务 / B2B SaaS Enterprise
- 建筑 / 农业 / 物流

> 完整清单见《知识库/01-产品-风格映射》Part B.3 的 "AI purple/pink gradients" 标注行。
