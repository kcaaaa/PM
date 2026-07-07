---
name: performance-optimization
description: 性能优化——前端/后端/数据库性能优化的系统化方法
when_to_use: 当用户遇到页面加载慢、API响应慢、数据库查询慢或资源占用高等性能问题时阅读
source: 项目实战经验总结，适配多技术栈性能优化场景
---

# 性能优化 Performance Optimization

> 当前项目适配：性能优化必须先做影响分析，创建 CR-* 变更包，经确认后再执行优化。

---

## 1. 性能优化原则

### 1.1 优化流程

```
测量 → 定位瓶颈 → 优化 → 验证 → 监控
```

### 1.2 核心策略

- **减少请求**：合并请求、缓存静态资源
- **减少传输**：压缩、懒加载、CDN
- **减少计算**：优化算法、异步处理
- **减少渲染**：虚拟列表、CSS 优化

---

## 2. 前端性能优化

### 2.1 加载性能

**资源优化：**

```html
<!-- 压缩 CSS/JS -->
<link rel="stylesheet" href="styles.min.css">
<script src="app.min.js"></script>

<!-- 图片优化 -->
<img src="image.webp" alt="描述" loading="lazy">

<!-- 字体优化 -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter&display=swap" rel="stylesheet">
```

**代码分割：**

```javascript
// React 懒加载
const LazyComponent = React.lazy(() => import('./Component'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

### 2.2 运行时性能

**React 优化：**

```javascript
// 使用 useMemo 缓存计算结果
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// 使用 useCallback 缓存函数引用
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);

// 使用 React.memo 避免不必要重渲染
const MemoComponent = React.memo(Component);
```

**虚拟列表：**

```javascript
// 使用 react-window 或 react-virtualized
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={400}
  itemCount={items.length}
  itemSize={50}
>
  {({ index, style }) => (
    <div style={style}>{items[index]}</div>
  )}
</FixedSizeList>
```

### 2.3 性能指标

| 指标 | 说明 | 目标 |
|---|---|---|
| LCP | 最大内容绘制 | < 2.5s |
| FID | 首次输入延迟 | < 100ms |
| CLS | 累积布局偏移 | < 0.1 |
| FCP | 首次内容绘制 | < 1.8s |

### 2.4 前端检查清单

- [ ] 启用 gzip/brotli 压缩
- [ ] 使用 CDN 分发静态资源
- [ ] 图片使用 WebP/AVIF 格式
- [ ] 实现代码分割和懒加载
- [ ] 使用 React.memo/useMemo/useCallback
- [ ] 长列表使用虚拟滚动
- [ ] 避免不必要的重渲染
- [ ] 使用 Service Worker 缓存

---

## 3. 后端性能优化

### 3.1 API 优化

**响应压缩：**

```javascript
// Express 启用压缩
const compression = require('compression');
app.use(compression());
```

**缓存策略：**

```javascript
// Redis 缓存
const cache = require('./redis');

app.get('/api/data', async (req, res) => {
  const cached = await cache.get('data');
  if (cached) return res.json(JSON.parse(cached));
  
  const data = await fetchData();
  await cache.set('data', JSON.stringify(data), 'EX', 3600);
  res.json(data);
});
```

### 3.2 异步处理

```javascript
// 使用 Worker Threads 处理 CPU 密集任务
const { Worker } = require('worker_threads');

function runWorker(task) {
  return new Promise((resolve) => {
    const worker = new Worker('./worker.js', { workerData: task });
    worker.on('message', resolve);
  });
}
```

### 3.3 连接池配置

```javascript
// PostgreSQL 连接池
const { Pool } = require('pg');

const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});
```

### 3.4 后端检查清单

- [ ] 启用响应压缩
- [ ] 实现缓存策略（Redis/Memory）
- [ ] 使用连接池管理数据库连接
- [ ] CPU 密集任务使用 Worker Threads
- [ ] 优化 API 响应结构，只返回必要数据
- [ ] 实现请求限流和熔断
- [ ] 使用负载均衡
- [ ] 启用日志和监控

---

## 4. 数据库性能优化

### 4.1 索引优化

```sql
-- 创建复合索引
CREATE INDEX idx_users_email_status ON users (email, status);

-- 创建覆盖索引
CREATE INDEX idx_orders_user_id_total ON orders (user_id) INCLUDE (total, created_at);

-- 删除无用索引
DROP INDEX idx_users_name;
```

### 4.2 查询优化

```sql
-- 避免 SELECT *
SELECT id, name, email FROM users WHERE status = 1;

-- 使用 JOIN 替代子查询
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.status = 1;

-- 使用 LIMIT 分页
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 0;
```

### 4.3 表结构优化

```sql
-- 使用合适的数据类型
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL,
  status SMALLINT DEFAULT 1,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 分区表（大数据量）
CREATE TABLE orders (
  id BIGSERIAL,
  order_date DATE,
  total NUMERIC(10,2)
) PARTITION BY RANGE (order_date);
```

### 4.4 数据库检查清单

- [ ] 为 WHERE/JOIN 条件创建索引
- [ ] 定期分析查询计划（EXPLAIN）
- [ ] 避免 SELECT *，只查询需要的列
- [ ] 使用覆盖索引减少回表
- [ ] 合理设置连接池大小
- [ ] 定期清理无用数据
- [ ] 监控慢查询日志
- [ ] 考虑读写分离

---

## 关键红线

- [ ] 性能优化前必须创建 CR-* 变更包
- [ ] 优化前必须测量基线数据
- [ ] 优化后必须验证效果
- [ ] 禁止在生产环境进行大规模优化
- [ ] 优化后必须更新监控配置