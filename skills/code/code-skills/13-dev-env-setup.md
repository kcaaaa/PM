---
name: dev-env-setup
description: 开发环境搭建——从零基础到可开发的完整环境配置流程
when_to_use: 当用户需要搭建开发环境、配置工具链、或遇到环境相关问题时阅读
source: 项目实战经验总结，适配 Windows/macOS/Linux 多平台
---

# 开发环境搭建 Development Environment Setup

> 当前项目适配：默认运行环境为 Windows，所有命令均提供 PowerShell 语法；同时支持 macOS/Linux 终端命令。

---

## 1. 环境搭建流程

### 1.1 检查现有环境

在开始前，先检查已安装的工具：

```powershell
node --version
npm --version
git --version
```

### 1.2 安装 Node.js

**Windows（推荐使用 nvm-windows）：**

```powershell
choco install nvm -y
nvm install 20
nvm use 20
```

**macOS/Linux（推荐使用 fnm）：**

```bash
curl -fsSL https://fnm.vercel.app/install | bash
fnm install 20
fnm use 20
```

### 1.3 安装 Git

**Windows：**

```powershell
choco install git -y
```

**macOS：**

```bash
brew install git
```

**Linux：**

```bash
sudo apt install git -y
```

### 1.4 安装编辑器

推荐使用 Visual Studio Code：

**Windows：**

```powershell
choco install vscode -y
```

**macOS：**

```bash
brew install --cask visual-studio-code
```

**Linux：**

```bash
sudo snap install --classic code
```

### 1.5 配置 VS Code 插件

```powershell
code --install-extension dbaeumer.vscode-eslint
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode-remote.remote-wsl
code --install-extension bradlc.vscode-tailwindcss
code --install-extension dsznajder.es7-react-js-snippets
```

---

## 2. 项目环境配置

### 2.1 初始化项目

```powershell
mkdir my-project
cd my-project
npm init -y
```

### 2.2 安装基础依赖

```powershell
npm install react react-dom typescript @types/react @types/react-dom
npm install -D tailwindcss@3 postcss autoprefixer eslint prettier
```

### 2.3 初始化配置文件

```powershell
npx tailwindcss init -p
npx tsc --init
npx eslint --init
```

### 2.4 配置 Tailwind CSS

修改 `tailwind.config.js`：

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### 2.5 配置 TypeScript

修改 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

---

## 3. 移动端开发环境

### 3.1 React Native + Expo

```powershell
npm install -g expo-cli
npx create-expo-app@6.5.0 my-app
cd my-app
npm run ios    # 或 npm run android
```

**iOS 开发要求：**
- 必须使用 macOS
- 安装 Xcode（通过 App Store）
- 配置 Apple 开发者账号

**Android 开发要求：**
- 安装 Android Studio
- 配置 ANDROID_HOME 环境变量
- 创建虚拟设备

### 3.2 Flutter

**Windows：**

```powershell
choco install flutter -y
flutter doctor
```

**macOS：**

```bash
brew install flutter
flutter doctor
```

---

## 4. 环境验证

完成配置后，运行以下命令验证：

```powershell
node --version    # 应显示 v20.x.x
npm --version     # 应显示 10.x.x
git --version     # 应显示 2.x.x
npx tsc --version # 应显示 5.x.x
```

---

## 关键红线

- [ ] 必须使用 nvm/fnm 管理 Node.js 版本
- [ ] 项目必须使用 TypeScript
- [ ] 必须配置 ESLint 和 Prettier
- [ ] 移动端开发前必须运行 `flutter doctor` 或 `expo doctor`
- [ ] 环境变量必须写入 `.env.example`，不得提交真实 `.env`