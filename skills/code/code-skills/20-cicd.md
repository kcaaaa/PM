---
name: cicd
description: CI/CD 流程——持续集成和持续部署的最佳实践
when_to_use: 当需要配置自动化构建、测试、部署流程时阅读
source: 项目实战经验总结，适配 GitHub Actions、GitLab CI 等主流 CI/CD 工具
---

# CI/CD 流程 CI/CD Pipeline

> 当前项目适配：CI/CD 配置必须记录到 `Base/06-实现管理/IMP-*/实现记录.md`，部署结果必须记录到 TEST-*。

---

## 1. CI/CD 概念

### 1.1 持续集成（CI）

```
代码提交 → 自动构建 → 自动测试 → 代码审查
```

### 1.2 持续交付（CD）

```
CI 通过 → 自动部署到测试环境 → 人工验证 → 手动部署到生产
```

### 1.3 持续部署（CD）

```
CI 通过 → 自动部署到生产环境
```

---

## 2. GitHub Actions 配置

### 2.1 基本配置

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install
      - run: npm test
      - run: npm run build
```

### 2.2 部署到 Vercel

```yaml
deploy:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: amondnet/vercel-action@v25
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
        vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
        vercel-args: '--prod'
```

### 2.3 部署到 AWS

```yaml
deploy-aws:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    - run: aws s3 sync build/ s3://my-bucket --delete
```

---

## 3. GitLab CI 配置

### 3.1 基本配置

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: node:20
  script:
    - npm install
    - npm run build

test:
  stage: test
  image: node:20
  script:
    - npm test

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production..."
```

---

## 4. 环境配置

### 4.1 环境变量管理

```yaml
# GitHub Secrets
# 在仓库 Settings → Secrets and variables → Actions 中配置

# 环境变量示例
- name: Build
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
```

### 4.2 多环境配置

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    steps:
      - run: echo "Testing ${{ matrix.environment }}"
```

---

## 5. 测试自动化

### 5.1 单元测试

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
    - run: npm install
    - run: npm test -- --coverage
    - uses: codecov/codecov-action@v4
      with:
        files: ./coverage/lcov.info
```

### 5.2 端到端测试

```yaml
e2e:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
    - run: npm install
    - run: npm run build
    - run: npm run test:e2e
```

---

## 6. 部署策略

### 6.1 蓝绿部署

```
Blue (生产) ←→ Green (新版本)
```

1. 在 Green 环境部署新版本
2. 验证 Green 环境
3. 切换流量到 Green
4. 如果失败，切换回 Blue

### 6.2 滚动部署

```
逐个替换旧版本实例
```

1. 停止一个旧版本实例
2. 启动一个新版本实例
3. 验证新版本
4. 重复直到所有实例更新

### 6.3 金丝雀部署

```
先向一小部分用户发布新版本
```

1. 向 1% 用户发布新版本
2. 监控性能和错误
3. 如果正常，逐步扩大范围
4. 全部用户更新后完成

---

## 7. CI/CD 检查清单

- [ ] 配置文件完整（GitHub Actions/GitLab CI）
- [ ] 环境变量配置正确
- [ ] 构建流程自动化
- [ ] 测试流程自动化
- [ ] 代码质量检查（ESLint/Prettier）
- [ ] 部署流程自动化
- [ ] 多环境支持（dev/staging/prod）
- [ ] 回滚机制配置
- [ ] 监控和告警配置
- [ ] 日志收集配置

---

## 关键红线

- [ ] CI/CD 配置必须记录到 IMP-*/实现记录.md
- [ ] 部署结果必须记录到 TEST-*
- [ ] 敏感信息必须使用 Secrets 管理
- [ ] 生产环境部署前必须通过所有测试
- [ ] 必须配置回滚机制