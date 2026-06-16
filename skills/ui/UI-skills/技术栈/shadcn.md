# 技术栈 · shadcn

> **来源**：`stacks/shadcn.csv`。shadcn/ui = Radix Primitives + Tailwind + 复制粘贴代码。
> **特点**：**不是 npm 包**——通过 CLI 把组件源码复制到你的 repo，完全可定制。

---

## 1. Setup（配合 Next.js / Vite / Remix）

```bash
npx shadcn@latest init
# 回答：
#  - TypeScript? yes
#  - Style: Default / New York
#  - Base color: Slate / Gray / Zinc / Neutral / Stone
#  - CSS variables? yes
```

产物：

- `components/ui/` ← 所有 shadcn 组件源码会落到这里
- `lib/utils.ts` ← `cn()` 工具
- `tailwind.config.ts` + `globals.css` ← 主题变量

## 2. 安装组件

```bash
npx shadcn@latest add button card dialog input select tabs toast
```

每次 `add` 会把组件源码复制到 `components/ui/`，**你可以直接改**。

## 3. 主题（CSS 变量）

`globals.css`：

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --muted: 210 40% 96.1%;
    --border: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
  }
  .dark {
    --background: 222.2 84% 4.9%;
    /* ... */
  }
}
```

把《03-配色方案》的 hex 转成 HSL 替换即可。

## 4. 常用模式

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Dialog, DialogContent, DialogTrigger } from '@/components/ui/dialog'

<Card>
  <CardHeader><CardTitle>标题</CardTitle></CardHeader>
  <CardContent>
    <Dialog>
      <DialogTrigger asChild><Button>打开</Button></DialogTrigger>
      <DialogContent>Hi</DialogContent>
    </Dialog>
  </CardContent>
</Card>
```

## 5. Variants via `class-variance-authority` (cva)

```ts
import { cva } from 'class-variance-authority'

export const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  }
)
```

## 6. 表单（React Hook Form + Zod）

```tsx
import { zodResolver } from '@hookform/resolvers/zod'
import { useForm } from 'react-hook-form'
import { z } from 'zod'
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from '@/components/ui/form'

const schema = z.object({ email: z.string().email() })

const form = useForm({ resolver: zodResolver(schema), defaultValues: { email: '' } })

<Form {...form}>
  <form onSubmit={form.handleSubmit(onSubmit)}>
    <FormField control={form.control} name="email" render={({ field }) => (
      <FormItem>
        <FormLabel>Email</FormLabel>
        <FormControl><Input {...field} /></FormControl>
        <FormMessage />
      </FormItem>
    )} />
  </form>
</Form>
```

## 7. Toast

```tsx
import { toast } from 'sonner'  // 新版推荐
toast.success('Saved!')
toast.error('Failed')
```

`<Toaster />` 放在 `app/layout.tsx`。

## 8. Icon

用 `lucide-react`（shadcn 默认），参考《知识库/08-图标库》。

```tsx
import { Menu, Search } from 'lucide-react'
<Button size="icon" aria-label="Menu"><Menu /></Button>
```

## 9. 暗色模式

```tsx
// 用 next-themes
import { ThemeProvider } from 'next-themes'

<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  {children}
</ThemeProvider>
```

---

## 关键红线

- [ ] **组件即源码**——直接改 `components/ui/` 下的文件，不要把它当黑盒
- [ ] 主题用 HSL CSS 变量驱动，不要硬编码 hex
- [ ] 表单必配 React Hook Form + Zod
- [ ] Toast 用 `sonner`
- [ ] Icon 用 `lucide-react`（已内置）
