---
name: troubleshooting
description: 常见问题解决方案库——开发过程中常见问题的快速排查指南
when_to_use: 当用户遇到开发过程中的常见问题、错误提示或环境配置问题时阅读
source: 项目实战经验总结，涵盖前端、后端、数据库、环境等常见问题
---

# 常见问题解决方案库 Troubleshooting Guide

> 当前项目适配：解决问题后必须记录到 `Base/09-测试验收/TEST-*/问题清单.md`。

---

## 1. 环境配置问题

### 1.1 Node.js 版本问题

**问题：** 项目依赖要求特定 Node.js 版本

**解决方案：**

```powershell
# 使用 nvm 管理版本
nvm list
nvm install 20
nvm use 20
```

### 1.2 端口占用

**问题：** `EADDRINUSE: address already in use`

**解决方案：**

```powershell
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9
```

### 1.3 npm 安装失败

**问题：** `npm ERR! code ENOENT` 或权限问题

**解决方案：**

```powershell
# 清除缓存
npm cache clean --force

# 重新安装
rm -rf node_modules package-lock.json
npm install

# 权限问题（macOS/Linux）
sudo npm install -g <package>
```

---

## 2. 前端常见问题

### 2.1 React 组件不渲染

**问题：** 组件无报错但不显示

**检查清单：**

- [ ] 检查条件渲染逻辑
- [ ] 检查组件是否正确导出
- [ ] 检查父组件是否正确传递 props
- [ ] 检查 CSS 是否隐藏了元素
- [ ] 检查 React DevTools 组件树

### 2.2 状态不更新

**问题：** `setState` 后状态没有变化

**解决方案：**

```javascript
// 错误：直接修改状态
this.state.count++;

// 正确：使用 setState
this.setState({ count: this.state.count + 1 });

// 使用函数式 setState（处理异步更新）
this.setState(prevState => ({ count: prevState.count + 1 }));

// React Hooks
setCount(prev => prev + 1);
```

### 2.3 样式不生效

**问题：** CSS/Tailwind 样式不显示

**检查清单：**

- [ ] 检查 CSS 文件是否正确引入
- [ ] 检查类名拼写是否正确
- [ ] 检查 Tailwind 配置文件 `content` 路径
- [ ] 检查 CSS 选择器优先级
- [ ] 检查浏览器是否缓存了旧样式

### 2.4 跨域问题

**问题：** `Access-Control-Allow-Origin` 错误

**解决方案：**

```javascript
// Express 配置 CORS
const cors = require('cors');
app.use(cors());

// 或配置具体域名
app.use(cors({
  origin: 'https://example.com',
  credentials: true
}));
```

### 2.5 打包失败

**问题：** `npm run build` 失败

**检查清单：**

- [ ] 检查 TypeScript 类型错误
- [ ] 检查 ESLint 错误
- [ ] 检查路径别名配置
- [ ] 检查环境变量是否正确
- [ ] 检查 Node.js 版本是否兼容

---

## 3. 后端常见问题

### 3.1 API 404 错误

**问题：** 请求返回 404

**检查清单：**

- [ ] 检查路由路径是否正确
- [ ] 检查 HTTP 方法是否匹配（GET/POST）
- [ ] 检查中间件顺序
- [ ] 检查静态文件配置

### 3.2 数据库连接失败

**问题：** `Error: connect ECONNREFUSED`

**检查清单：**

- [ ] 检查数据库服务是否启动
- [ ] 检查端口是否正确
- [ ] 检查连接字符串是否正确
- [ ] 检查用户名和密码
- [ ] 检查防火墙设置

### 3.3 认证失败

**问题：** `401 Unauthorized`

**检查清单：**

- [ ] 检查 Token 是否有效
- [ ] 检查 Token 是否过期
- [ ] 检查 Token 是否正确传递（Header/Body）
- [ ] 检查认证中间件配置

### 3.4 请求超时

**问题：** 请求超过 30 秒无响应

**检查清单：**

- [ ] 检查后端是否有死循环
- [ ] 检查数据库查询是否过慢
- [ ] 检查外部 API 调用是否超时
- [ ] 检查连接超时配置

---

## 4. 数据库常见问题

### 4.1 查询返回空结果

**问题：** SQL 查询无数据返回

**检查清单：**

- [ ] 检查 WHERE 条件是否正确
- [ ] 检查数据是否存在
- [ ] 检查大小写敏感性
- [ ] 检查 NULL 值处理

### 4.2 数据插入失败

**问题：** `INSERT` 语句报错

**检查清单：**

- [ ] 检查字段是否允许 NULL
- [ ] 检查外键约束
- [ ] 检查数据类型是否匹配
- [ ] 检查唯一约束冲突
- [ ] 检查字段长度限制

### 4.3 事务回滚

**问题：** 事务执行失败后数据未回滚

**解决方案：**

```sql
BEGIN;
-- 执行操作
INSERT INTO orders (...);
UPDATE inventory SET quantity = quantity - 1 WHERE id = 1;
-- 如有错误则回滚
ROLLBACK;
-- 否则提交
COMMIT;
```

---

## 5. 移动端常见问题

### 5.1 React Native 构建失败

**问题：** `npm run ios` 或 `npm run android` 失败

**检查清单：**

- [ ] 检查 Xcode/Android Studio 是否安装
- [ ] 检查 SDK 版本是否正确
- [ ] 运行 `npx expo doctor` 检查依赖
- [ ] 清除构建缓存：`rm -rf ios/build android/build`

### 5.2 真机调试问题

**问题：** 无法在真机上调试

**检查清单：**

- [ ] 检查设备是否连接
- [ ] 检查 USB 调试是否开启
- [ ] 检查开发者模式是否开启
- [ ] 检查设备是否信任电脑

---

## 6. 问题排查模板

当遇到新问题时，按以下步骤排查：

### 步骤 1：记录错误信息

```
错误类型：[错误名称]
错误信息：[完整错误消息]
发生场景：[何时发生]
复现步骤：[如何复现]
```

### 步骤 2：定位问题范围

- [ ] 前端问题（浏览器控制台）
- [ ] 后端问题（服务端日志）
- [ ] 网络问题（Network 面板）
- [ ] 数据库问题（SQL 查询）

### 步骤 3：尝试解决方案

1. [方案一]
2. [方案二]
3. [方案三]

### 步骤 4：验证修复

- [ ] 问题是否解决
- [ ] 是否有回归问题
- [ ] 是否需要更新文档

---

## 关键红线

- [ ] 解决问题后必须记录到 TEST-*/问题清单.md
- [ ] 复杂问题必须创建 CR-* 变更包
- [ ] 禁止在生产环境直接调试
- [ ] 修复后必须编写测试用例