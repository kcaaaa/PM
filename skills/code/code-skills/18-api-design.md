---
name: api-design
description: API 设计规范——RESTful API 设计原则和最佳实践
when_to_use: 当需要设计或评审 API 接口时阅读，适用于后端接口设计、前后端协作
source: 项目实战经验总结，适配 RESTful API 设计标准
---

# API 设计规范 API Design Guidelines

> 当前项目适配：API 设计必须记录到 `Base/04-方案设计/TD-*/接口设计.md`，并与 `FT-*/功能说明.md` 保持一致。

---

## 1. RESTful API 设计原则

### 1.1 资源命名

| 规则 | 正例 | 反例 |
|---|---|---|
| 使用复数名词 | `/users` | `/user` |
| 使用连字符分隔 | `/user-profiles` | `/userProfiles` |
| 避免动词 | `/orders` | `/getOrders` |
| 使用嵌套表示关系 | `/users/1/orders` | `/userOrders?userId=1` |

### 1.2 HTTP 方法

| 方法 | 用途 | 幂等性 |
|---|---|---|
| GET | 查询资源 | 是 |
| POST | 创建资源 | 否 |
| PUT | 完整更新资源 | 是 |
| PATCH | 部分更新资源 | 是 |
| DELETE | 删除资源 | 是 |

### 1.3 状态码

| 状态码 | 含义 | 使用场景 |
|---|---|---|
| 200 | OK | 请求成功 |
| 201 | Created | 创建成功 |
| 204 | No Content | 删除成功，无返回内容 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突 |
| 500 | Internal Server Error | 服务器内部错误 |

---

## 2. 请求设计

### 2.1 查询参数

```javascript
// 分页
GET /api/users?page=1&limit=20

// 过滤
GET /api/users?status=active&role=admin

// 排序
GET /api/users?sort=createdAt&order=desc

// 搜索
GET /api/users?q=john
```

### 2.2 请求体

```json
{
  "email": "user@example.com",
  "password": "secure123",
  "name": "John Doe"
}
```

**规则：**
- 使用 JSON 格式
- 字段名使用驼峰命名
- 必填字段必须验证
- 提供默认值（如适用）

### 2.3 路径参数

```javascript
GET /api/users/:id
DELETE /api/orders/:orderId
```

---

## 3. 响应设计

### 3.1 成功响应

```json
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

### 3.2 列表响应

```json
{
  "status": "success",
  "data": [
    { "id": 1, "name": "John" },
    { "id": 2, "name": "Jane" }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  }
}
```

### 3.3 错误响应

```json
{
  "status": "error",
  "code": 400,
  "message": "Invalid email format",
  "details": [
    { "field": "email", "message": "Must be a valid email address" }
  ]
}
```

---

## 4. 认证与授权

### 4.1 Token 认证

```javascript
// 请求头
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

// 刷新 Token
POST /api/auth/refresh
{
  "refreshToken": "refresh-token-value"
}
```

### 4.2 权限控制

```javascript
// 基于角色的访问控制
app.get('/api/admin', requireRole('admin'));
app.get('/api/users', requireRole('user', 'admin'));
```

---

## 5. 版本控制

```javascript
// URL 版本控制（推荐）
GET /api/v1/users
GET /api/v2/users

// Header 版本控制
Accept: application/vnd.api.v1+json
```

---

## 6. API 文档

### 6.1 使用 Swagger/OpenAPI

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /api/users:
    get:
      summary: 获取用户列表
      responses:
        '200':
          description: 成功
```

### 6.2 文档生成

```powershell
npx swagger-jsdoc -d swagger.js -o swagger.json
npx swagger-ui-express
```

---

## 7. API 设计检查清单

- [ ] 使用 RESTful 风格设计
- [ ] 资源命名符合规范（复数、连字符）
- [ ] HTTP 方法使用正确
- [ ] 状态码使用合理
- [ ] 请求参数验证完善
- [ ] 响应格式统一
- [ ] 错误处理清晰
- [ ] 认证授权完整
- [ ] 版本控制考虑
- [ ] API 文档完整

---

## 关键红线

- [ ] API 设计必须记录到 TD-*/接口设计.md
- [ ] 接口变更必须创建 CR-* 变更包
- [ ] 敏感信息不得出现在响应中
- [ ] 必须实现请求限流
- [ ] 必须记录 API 调用日志