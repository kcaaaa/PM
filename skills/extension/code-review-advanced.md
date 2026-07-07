---
name: code-review-advanced
description: 代码审查高级技能——深度代码审查、架构审查、性能审查
when_to_use: 当需要进行深度代码审查、架构评估或性能分析时阅读
source: 项目实战经验总结，适配复杂代码审查场景
---

# 代码审查高级技能 Advanced Code Review

> 当前项目适配：代码审查结果必须记录到 `Base/09-测试验收/TEST-*/测试记录.md`，问题必须进入问题清单。

---

## 1. 深度代码审查

### 1.1 复杂度分析

```typescript
// 圈复杂度计算
function calculateCyclomaticComplexity(code: string): number {
  const decisionPoints = [
    /if\s*\(/g,
    /else/g,
    /else if\s*\(/g,
    /for\s*\(/g,
    /while\s*\(/g,
    /do\s*\{/g,
    /case\s+\w+/g,
    /&&/g,
    /\|\|/g,
    /\?/g
  ];
  
  let complexity = 1;
  for (const pattern of decisionPoints) {
    const matches = code.match(pattern);
    if (matches) complexity += matches.length;
  }
  
  return complexity;
}

// 审查阈值
const COMPLEXITY_THRESHOLD = 10;
const LINES_THRESHOLD = 50;
```

### 1.2 代码异味检测

| 代码异味 | 描述 | 示例 |
|---|---|---|
| 重复代码 | 相同或相似的代码出现多次 | 多个函数中有相同的验证逻辑 |
| 过长函数 | 函数超过 50 行 | 一个函数处理多个职责 |
| 过大类 | 类超过 200 行 | 一个类包含太多方法 |
| 魔幻数字 | 未命名的常量值 | `if (status === 2)` |
| 上帝对象 | 一个对象了解太多或做太多 | 包含所有逻辑的 Service 类 |
| 紧耦合 | 模块间依赖过多 | 直接引用具体实现而非接口 |

### 1.3 依赖分析

```typescript
interface DependencyGraph {
  nodes: string[];
  edges: { from: string; to: string }[];
}

function analyzeDependencies(codebase: string[]): DependencyGraph {
  const nodes = new Set<string>();
  const edges: { from: string; to: string }[] = [];
  
  for (const file of codebase) {
    const filename = file.split('/').pop()!;
    nodes.add(filename);
    
    const importPattern = /import.*from\s+['"]([^'"]+)['"]/g;
    let match;
    while ((match = importPattern.exec(file)) !== null) {
      edges.push({ from: filename, to: match[1] });
    }
  }
  
  return { nodes: Array.from(nodes), edges };
}
```

---

## 2. 架构审查

### 2.1 SOLID 原则检查

| 原则 | 检查项 | 问题示例 |
|---|---|---|
| SRP | 类是否只有一个职责 | 用户类同时处理认证和数据存储 |
| OCP | 是否对扩展开放、对修改关闭 | 添加新功能需要修改现有代码 |
| LSP | 子类是否可以替换父类 | 子类改变了父类的契约 |
| ISP | 是否有不必要的接口依赖 | 实现了不需要的接口方法 |
| DIP | 是否依赖抽象而非具体 | 直接依赖具体类而非接口 |

### 2.2 设计模式识别

```typescript
// 设计模式检测器
const designPatterns = [
  { name: 'Singleton', pattern: /class\s+\w+\s*\{[^}]*instance[^}]*private\s+constructor/ },
  { name: 'Factory', pattern: /create\w+\s*\(/ },
  { name: 'Observer', pattern: /subscribe|notify|listener/ },
  { name: 'Strategy', pattern: /interface.*Strategy/ },
  { name: 'Decorator', pattern: /extends.*Decorator/ }
];

function detectPatterns(code: string): string[] {
  return designPatterns
    .filter(p => p.pattern.test(code))
    .map(p => p.name);
}
```

### 2.3 模块化评估

```typescript
interface ModuleMetrics {
  name: string;
  size: number;
  dependencies: number;
  cyclomaticComplexity: number;
  cohesion: number;
  coupling: number;
}

class ModuleAnalyzer {
  analyze(module: string): ModuleMetrics {
    return {
      name: module,
      size: this.getSize(module),
      dependencies: this.getDependencies(module),
      cyclomaticComplexity: this.getComplexity(module),
      cohesion: this.getCohesion(module),
      coupling: this.getCoupling(module)
    };
  }
  
  private getCohesion(module: string): number {
    const functions = module.match(/function\s+\w+/g) || [];
    const sharedVariables = module.match(/const\s+\w+/g) || [];
    return functions.length > 0 ? sharedVariables.length / functions.length : 0;
  }
  
  private getCoupling(module: string): number {
    const imports = module.match(/import.*from/g) || [];
    return imports.length;
  }
}
```

---

## 3. 性能审查

### 3.1 性能问题检测

| 问题类型 | 检测方法 | 修复建议 |
|---|---|---|
| N+1 查询 | 检查循环中的数据库查询 | 使用批量查询或 JOIN |
| 不必要的重渲染 | 检查 React 组件依赖 | 使用 useMemo/useCallback/React.memo |
| 内存泄漏 | 检查 useEffect 清理 | 确保清理函数正确 |
| 同步阻塞 | 检查 CPU 密集操作 | 使用 Web Workers 或异步处理 |
| 过大的状态 | 检查状态管理 | 拆分状态或使用选择器 |

### 3.2 性能审查清单

- [ ] 数据库查询是否有 N+1 问题
- [ ] 是否使用了索引优化查询
- [ ] React 组件是否有不必要的重渲染
- [ ] 是否使用了虚拟列表处理长列表
- [ ] 是否有内存泄漏风险
- [ ] 是否有同步阻塞操作
- [ ] 是否使用了缓存策略
- [ ] 是否有性能监控

---

## 4. 安全审查

### 4.1 安全漏洞检测

| 漏洞类型 | 检测方法 | 修复建议 |
|---|---|---|
| SQL 注入 | 检查动态 SQL 拼接 | 使用参数化查询 |
| XSS | 检查用户输入渲染 | 使用 HTML 转义 |
| CSRF | 检查表单提交 | 添加 CSRF token |
| 敏感信息泄露 | 检查日志和响应 | 过滤敏感字段 |
| 认证绕过 | 检查权限验证 | 使用中间件验证 |

### 4.2 安全审查清单

- [ ] 是否使用参数化查询
- [ ] 是否对用户输入进行验证和转义
- [ ] 是否有 CSRF 防护
- [ ] 是否正确处理认证和授权
- [ ] 是否有敏感信息泄露风险
- [ ] 是否使用 HTTPS
- [ ] 是否设置了安全的 CORS 策略
- [ ] 是否正确处理错误信息

---

## 关键红线

- [ ] 代码审查结果必须记录到 TEST-*/测试记录.md
- [ ] 发现的问题必须进入问题清单
- [ ] 必须检查代码复杂度
- [ ] 必须检查架构设计
- [ ] 必须检查性能问题
- [ ] 必须检查安全漏洞