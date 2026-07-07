---
name: debugging
description: 调试方法论——前端/后端/数据库问题排查的系统化方法
when_to_use: 当用户遇到代码错误、运行时异常、数据不一致或性能问题时阅读
source: 项目实战经验总结，适配多技术栈调试场景
---

# 调试方法论 Debugging Methodology

> 当前项目适配：调试过程必须记录到 `Base/09-测试验收/TEST-*/问题清单.md`，修复后更新 `IMP-*/变更记录.md`。

---

## 1. 调试基本原则

### 1.1 科学调试流程

```
观察现象 → 提出假设 → 验证假设 → 定位根因 → 修复 → 验证修复
```

### 1.2 关键技巧

- **二分法**：逐步缩小问题范围
- **最小复现**：创建最小可复现案例
- **版本对比**：对比工作版本和故障版本
- **日志追踪**：增加详细日志定位问题

---

## 2. 前端调试

### 2.1 Chrome DevTools

**常用面板：**

| 面板 | 用途 |
|---|---|
| Elements | 查看和修改 DOM |
| Console | 输出日志和执行命令 |
| Sources | 断点调试和代码查看 |
| Network | 查看网络请求 |
| Performance | 性能分析 |
| Application | 查看存储和缓存 |

### 2.2 断点调试

```javascript
debugger;  // 在代码中添加断点

// 条件断点
// 在 Sources 面板右键断点 → Edit condition
// 例如：x > 100
```

### 2.3 React 调试

```powershell
npm install -D @redux-devtools/extension react-devtools
```

**React DevTools 使用：**
- 安装浏览器扩展
- 使用 `React Developer Tools` 查看组件树
- 使用 `useDebugValue` 自定义 Hook 显示

### 2.4 常见前端问题

| 问题类型 | 排查方法 |
|---|---|
| 白屏 | 检查 Console 错误、网络请求、JS 语法 |
| 样式错乱 | 检查 CSS 选择器优先级、布局属性 |
| 状态不更新 | 检查 useState/useReducer、闭包问题 |
| 内存泄漏 | 使用 Memory 面板、检查 useEffect 清理 |
| 性能卡顿 | 使用 Performance 面板、检查长任务 |

---

## 3. 后端调试

### 3.1 Node.js 调试

```powershell
node --inspect server.js
# 或
npx nodemon --inspect server.js
```

**VS Code 配置：**

```json
{
  "type": "node",
  "request": "attach",
  "name": "Attach to Node",
  "port": 9229
}
```

### 3.2 日志调试

```javascript
const logger = {
  debug: (...args) => console.log('[DEBUG]', ...args),
  info: (...args) => console.log('[INFO]', ...args),
  error: (...args) => console.error('[ERROR]', ...args)
};

logger.debug('User login attempt:', user.id);
```

### 3.3 API 调试

```powershell
# 使用 curl
curl -v http://localhost:3000/api/users

# 使用 Postman / Insomnia
# 或浏览器 Network 面板
```

### 3.4 常见后端问题

| 问题类型 | 排查方法 |
|---|---|
| 服务器启动失败 | 检查端口占用、依赖缺失、配置错误 |
| API 返回错误 | 检查路由匹配、中间件顺序、错误处理 |
| 数据库连接失败 | 检查连接字符串、端口、权限、防火墙 |
| 认证失败 | 检查 token、session、cookie |

---

## 4. 数据库调试

### 4.1 SQL 查询调试

```sql
-- EXPLAIN 分析查询计划
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 查看慢查询日志（MySQL）
SHOW VARIABLES LIKE 'slow_query_log';
```

### 4.2 数据一致性检查

```sql
-- 检查外键约束
SELECT * FROM information_schema.KEY_COLUMN_USAGE 
WHERE REFERENCED_TABLE_NAME = 'users';

-- 检查重复数据
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;
```

### 4.3 常见数据库问题

| 问题类型 | 排查方法 |
|---|---|
| 查询慢 | 使用 EXPLAIN、检查索引、优化 SQL |
| 死锁 | 检查 SHOW PROCESSLIST、事务隔离级别 |
| 数据丢失 | 检查事务提交、备份恢复 |
| 连接池耗尽 | 检查连接配置、连接泄露 |

---

## 5. 调试检查清单

### 5.1 初步检查

- [ ] 查看错误信息（Console、日志）
- [ ] 确认问题可复现
- [ ] 检查版本差异
- [ ] 确认环境一致性

### 5.2 深入排查

- [ ] 使用断点调试
- [ ] 添加详细日志
- [ ] 检查网络请求
- [ ] 验证数据一致性
- [ ] 排查依赖问题

### 5.3 修复验证

- [ ] 编写测试用例
- [ ] 验证修复效果
- [ ] 检查回归问题
- [ ] 更新文档和注释

---

## 关键红线

- [ ] 调试必须记录到 TEST-*/问题清单.md
- [ ] 修复后必须更新 IMP-*/变更记录.md
- [ ] 禁止在生产环境直接修改代码调试
- [ ] 敏感信息不得写入日志
- [ ] 调试完成后移除临时日志代码