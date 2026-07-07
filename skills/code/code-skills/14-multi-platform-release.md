---
name: multi-platform-release
description: 多平台发布流程——Web、iOS、Android、小程序的完整发布指南
when_to_use: 当用户需要发布应用到生产环境、配置各平台账号、或了解发布流程时阅读
source: 项目实战经验总结，适配各主流平台发布规范
---

# 多平台发布 Multi-Platform Release

> 当前项目适配：发布流程必须与 Base 对象包绑定，每次发布前必须检查 TEST-* 验证结果。

---

## 1. Web 平台发布

### 1.1 Vercel 发布

**步骤：**

```powershell
npm install -g vercel
vercel login
vercel init
vercel --prod
```

**配置要求：**
- 项目根目录必须有 `package.json`
- 必须配置正确的 `build` 命令
- 环境变量通过 Vercel 控制台配置

**输出：**
- 预览环境 URL
- 生产环境 URL

### 1.2 Netlify 发布

```powershell
npm install -g netlify-cli
netlify login
netlify init
netlify deploy --prod
```

### 1.3 GitHub Pages 发布

```powershell
npm run build
npm install -g gh-pages
gh-pages -d build
```

---

## 2. iOS 平台发布

### 2.1 准备工作

**Apple Developer 账号：**
1. 访问 https://developer.apple.com/
2. 注册开发者账号（年费 99 美元）
3. 创建 App ID
4. 创建签名证书

**Xcode 配置：**
1. 打开项目 `.xcworkspace` 文件
2. 配置 Bundle Identifier
3. 配置签名证书
4. 配置 App Store Connect

### 2.2 使用 EAS Build（Expo）

```powershell
npm install -g eas-cli
eas login
eas build:configure
eas build --platform ios
eas submit --platform ios
```

### 2.3 手动发布步骤

1. 在 Xcode 中选择 `Generic iOS Device`
2. 执行 `Product > Archive`
3. 在 Organizer 中选择 Archive
4. 点击 `Distribute App`
5. 选择 `App Store Connect`
6. 上传并提交审核

---

## 3. Android 平台发布

### 3.1 准备工作

**Google Play 账号：**
1. 访问 https://play.google.com/console
2. 注册开发者账号（一次性费用 25 美元）
3. 创建应用
4. 配置应用签名

**签名密钥生成：**

```powershell
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

### 3.2 使用 EAS Build（Expo）

```powershell
eas build --platform android
eas submit --platform android
```

### 3.3 手动发布步骤

1. 生成 APK/AAB 文件
2. 登录 Google Play Console
3. 创建应用版本
4. 上传 APK/AAB
5. 填写发布信息
6. 提交审核

---

## 4. 微信小程序发布

### 4.1 准备工作

**微信公众平台：**
1. 访问 https://mp.weixin.qq.com
2. 注册小程序账号
3. 配置 AppID
4. 下载微信开发者工具

### 4.2 发布步骤

1. 在微信开发者工具中打开项目
2. 配置 `project.config.json`
3. 点击 `上传`
4. 在微信公众平台提交审核
5. 审核通过后发布

---

## 5. 发布检查清单

### 5.1 代码检查

- [ ] 所有测试用例通过
- [ ] 代码已提交到 Git
- [ ] 版本号已更新
- [ ] 环境变量配置正确

### 5.2 功能检查

- [ ] 核心功能正常
- [ ] 响应式布局适配
- [ ] 错误处理完善
- [ ] 性能达标

### 5.3 安全检查

- [ ] 敏感信息已移除
- [ ] API 密钥已加密
- [ ] HTTPS 已配置
- [ ] CORS 策略正确

### 5.4 平台特定检查

**iOS：**
- [ ] Bundle Identifier 正确
- [ ] 签名证书有效
- [ ] 隐私政策已添加
- [ ] 屏幕截图已准备

**Android：**
- [ ] 签名密钥已配置
- [ ] 权限声明正确
- [ ] 应用图标已准备
- [ ] 启动画面已配置

---

## 6. 版本管理

### 6.1 版本号规范

```
主版本号.次版本号.修订号
例：1.0.0
```

- **主版本号**：重大功能变更，可能不兼容
- **次版本号**：新增功能，向后兼容
- **修订号**：Bug 修复，向后兼容

### 6.2 更新日志

每次发布必须包含更新日志：

```markdown
## v1.0.0 (2024-01-01)

### 新增
- 新增用户登录功能
- 新增数据展示页面

### 修复
- 修复登录页面布局问题
- 修复数据加载错误

### 优化
- 优化页面加载速度
- 优化用户体验
```

---

## 关键红线

- [ ] 发布前必须完成 TEST-* 验证
- [ ] 版本号必须递增，不得重复
- [ ] 敏感信息不得硬编码在代码中
- [ ] iOS 必须使用有效的开发者账号和证书
- [ ] Android 必须配置正确的签名密钥
- [ ] 发布后必须记录到 Base/09-测试验收