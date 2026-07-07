---
name: system-optimization
description: 系统级优化——上下文压缩、迭代优化、性能监控
when_to_use: 当需要优化系统性能、减少上下文消耗或实现迭代改进时阅读
source: 项目实战经验总结，适配 AI Agent 系统优化
---

# 系统级优化 System Optimization

> 当前项目适配：系统优化必须先创建 CR-* 变更包，经确认后再执行优化。

---

## 1. 上下文管理优化

### 1.1 上下文压缩策略

```typescript
// 智能分段
function splitContext(context: string, maxTokens: number): string[] {
  const chunks: string[] = [];
  let currentChunk = '';
  
  for (const line of context.split('\n')) {
    if (currentChunk.length + line.length > maxTokens) {
      chunks.push(currentChunk);
      currentChunk = '';
    }
    currentChunk += line + '\n';
  }
  
  chunks.push(currentChunk);
  return chunks;
}

// 关键信息提取
function extractKeyInfo(context: string): string {
  const keyPatterns = [
    /TODO:/g,
    /FIXME:/g,
    /@param/g,
    /@returns/g,
    /export/g
  ];
  
  let result = '';
  for (const pattern of keyPatterns) {
    const matches = context.match(pattern);
    if (matches) result += matches.join('\n');
  }
  
  return result;
}
```

### 1.2 状态保存机制

```typescript
interface SessionState {
  sessionId: string;
  context: string;
  keyInfo: string;
  lastAccess: Date;
  version: number;
}

class StateManager {
  private states: Map<string, SessionState> = new Map();
  
  saveState(sessionId: string, context: string): void {
    const state: SessionState = {
      sessionId,
      context,
      keyInfo: this.extractKeyInfo(context),
      lastAccess: new Date(),
      version: this.getNextVersion(sessionId)
    };
    this.states.set(sessionId, state);
  }
  
  loadState(sessionId: string): SessionState | undefined {
    return this.states.get(sessionId);
  }
  
  cleanupOldStates(maxAgeHours: number): void {
    const cutoff = new Date(Date.now() - maxAgeHours * 3600000);
    for (const [id, state] of this.states) {
      if (state.lastAccess < cutoff) {
        this.states.delete(id);
      }
    }
  }
}
```

---

## 2. 迭代优化机制

### 2.1 渐进式改进

```typescript
interface Improvement {
  id: string;
  type: 'bugfix' | 'feature' | 'refactor' | 'optimization';
  description: string;
  impact: 'high' | 'medium' | 'low';
  priority: number;
  status: 'pending' | 'in-progress' | 'completed';
}

class ImprovementTracker {
  private improvements: Improvement[] = [];
  
  addImprovement(improvement: Omit<Improvement, 'id' | 'status'>): Improvement {
    const newImprovement: Improvement = {
      ...improvement,
      id: `IMP-${Date.now()}`,
      status: 'pending'
    };
    this.improvements.push(newImprovement);
    return newImprovement;
  }
  
  getPrioritizedList(): Improvement[] {
    return [...this.improvements]
      .filter(i => i.status !== 'completed')
      .sort((a, b) => {
        const impactOrder = { high: 0, medium: 1, low: 2 };
        if (impactOrder[a.impact] !== impactOrder[b.impact]) {
          return impactOrder[a.impact] - impactOrder[b.impact];
        }
        return a.priority - b.priority;
      });
  }
}
```

### 2.2 A/B 测试框架

```typescript
interface TestVariant {
  id: string;
  name: string;
  weight: number;
  implementation: () => void;
}

class ABTest {
  private variants: TestVariant[] = [];
  private results: Map<string, { success: number; total: number }> = new Map();
  
  addVariant(variant: TestVariant): void {
    this.variants.push(variant);
    this.results.set(variant.id, { success: 0, total: 0 });
  }
  
  async run(): Promise<string> {
    const random = Math.random();
    let cumulativeWeight = 0;
    
    for (const variant of this.variants) {
      cumulativeWeight += variant.weight;
      if (random < cumulativeWeight) {
        try {
          await variant.implementation();
          this.recordResult(variant.id, true);
        } catch {
          this.recordResult(variant.id, false);
        }
        return variant.id;
      }
    }
    
    return this.variants[0].id;
  }
  
  recordResult(variantId: string, success: boolean): void {
    const result = this.results.get(variantId)!;
    result.total++;
    if (success) result.success++;
  }
}
```

---

## 3. 性能监控

### 3.1 指标收集

```typescript
interface PerformanceMetrics {
  responseTime: number;
  memoryUsage: number;
  cpuUsage: number;
  errorRate: number;
  throughput: number;
}

class PerformanceMonitor {
  private metrics: PerformanceMetrics[] = [];
  
  recordMetrics(metrics: PerformanceMetrics): void {
    this.metrics.push(metrics);
    if (this.metrics.length > 100) {
      this.metrics.shift();
    }
  }
  
  getAverageResponseTime(): number {
    if (this.metrics.length === 0) return 0;
    const total = this.metrics.reduce((sum, m) => sum + m.responseTime, 0);
    return total / this.metrics.length;
  }
  
  detectAnomalies(): PerformanceMetrics[] {
    const avgResponseTime = this.getAverageResponseTime();
    return this.metrics.filter(m => m.responseTime > avgResponseTime * 2);
  }
}
```

### 3.2 告警系统

```typescript
interface Alert {
  id: string;
  type: 'warning' | 'critical';
  message: string;
  timestamp: Date;
  resolved: boolean;
}

class AlertSystem {
  private alerts: Alert[] = [];
  
  triggerAlert(type: 'warning' | 'critical', message: string): Alert {
    const alert: Alert = {
      id: `ALERT-${Date.now()}`,
      type,
      message,
      timestamp: new Date(),
      resolved: false
    };
    this.alerts.push(alert);
    return alert;
  }
  
  resolveAlert(alertId: string): void {
    const alert = this.alerts.find(a => a.id === alertId);
    if (alert) alert.resolved = true;
  }
}
```

---

## 4. 缓存策略

### 4.1 多级缓存

```typescript
class MultiLevelCache {
  private l1Cache: Map<string, any> = new Map(); // 内存缓存
  private l2Cache: Map<string, any> = new Map(); // 持久化缓存
  
  get(key: string): any {
    // 先查 L1
    if (this.l1Cache.has(key)) {
      return this.l1Cache.get(key);
    }
    // 再查 L2
    if (this.l2Cache.has(key)) {
      const value = this.l2Cache.get(key);
      this.l1Cache.set(key, value);
      return value;
    }
    return null;
  }
  
  set(key: string, value: any, ttl?: number): void {
    this.l1Cache.set(key, value);
    this.l2Cache.set(key, value);
    
    if (ttl) {
      setTimeout(() => {
        this.l1Cache.delete(key);
        this.l2Cache.delete(key);
      }, ttl);
    }
  }
}
```

---

## 关键红线

- [ ] 系统优化必须先创建 CR-* 变更包
- [ ] 优化前必须测量基线数据
- [ ] 优化后必须验证效果
- [ ] 必须配置监控和告警
- [ ] 必须保留回滚机制