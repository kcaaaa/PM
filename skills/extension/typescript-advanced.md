---
name: typescript-advanced
description: TypeScript 高级类型——高级类型系统、泛型、类型体操
when_to_use: 当需要使用高级 TypeScript 类型、泛型编程或类型体操时阅读
source: 项目实战经验总结，适配复杂类型场景
---

# TypeScript 高级类型 TypeScript Advanced Types

> 当前项目适配：TypeScript 配置必须与 `Base/04-方案设计/TD-*/数据结构.md` 保持一致，类型定义必须符合项目编码规范。

---

## 1. 高级类型概念

### 1.1 泛型

```typescript
// 基础泛型
function identity<T>(arg: T): T {
  return arg;
}

// 泛型接口
interface Container<T> {
  value: T;
  getValue(): T;
}

// 泛型类
class DataStore<T> {
  private items: T[] = [];
  add(item: T): void { this.items.push(item); }
  get(index: number): T { return this.items[index]; }
}
```

### 1.2 条件类型

```typescript
type IsString<T> = T extends string ? true : false;
type A = IsString<string>;  // true
type B = IsString<number>;  // false

// infer 关键字
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type Fn = () => string;
type Result = ReturnType<Fn>;  // string
```

### 1.3 映射类型

```typescript
// Readonly
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// Partial
type Partial<T> = { [P in keyof T]?: T[P] };

// Required
type Required<T> = { [P in keyof T]-?: T[P] };

// Pick
type Pick<T, K extends keyof T> = { [P in K]: T[P] };

// Omit
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

---

## 2. 实用类型技巧

### 2.1 类型守卫

```typescript
// typeof 类型守卫
function isString(x: any): x is string {
  return typeof x === 'string';
}

// instanceof 类型守卫
class Animal {}
class Dog extends Animal {}
function isDog(animal: Animal): animal is Dog {
  return animal instanceof Dog;
}

// 自定义类型守卫
interface Bird { fly(): void; }
interface Fish { swim(): void; }
function isBird(pet: Bird | Fish): pet is Bird {
  return (pet as Bird).fly !== undefined;
}
```

### 2.2 模板字面量类型

```typescript
type Color = 'red' | 'green' | 'blue';
type Size = 'small' | 'medium' | 'large';

type ColorSize = `${Color}-${Size}`;
// 'red-small' | 'red-medium' | 'red-large' | ...

type EventName<T extends string> = `${T}Changed`;
type ClickEvent = EventName<'click'>;  // 'clickChanged'
```

### 2.3 递归类型

```typescript
interface TreeNode<T> {
  value: T;
  children?: TreeNode<T>[];
}

type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? DeepReadonly<T[P]> 
    : T[P];
};
```

---

## 3. shadcn 配置

### 3.1 初始化 shadcn

```powershell
npx shadcn@latest init -y -b natural
```

### 3.2 添加组件

```powershell
npx shadcn@latest add button dialog input card
npx shadcn@latest add table form
npx shadcn@latest add dropdown-menu tooltip
```

### 3.3 配置路径别名

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

## 4. TypeScript 配置最佳实践

```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "resolveJsonModule": true
  }
}
```

---

## 关键红线

- [ ] 必须使用严格模式（strict: true）
- [ ] 禁止使用 any 类型
- [ ] 类型定义必须与数据结构保持一致
- [ ] 必须配置路径别名
- [ ] 必须使用 shadcn 组件库