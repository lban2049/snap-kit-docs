# 开发指南

本文档介绍如何在本地开发、构建、测试和发布 SnapKit。

## 开发环境准备

### 系统要求

- **Node.js**: 18.x 或更高版本
- **包管理器**: pnpm（推荐）或 npm
- **Git**: 用于版本控制
- **Chrome/Chromium**: 用于 Puppeteer

### 安装依赖

#### 安装 pnpm

```bash
# 使用 npm 安装
npm install -g pnpm

# 或使用 corepack
corepack enable
corepack prepare pnpm@latest --activate
```

#### 安装 Chrome

**macOS**:
```bash
brew install --cask google-chrome
```

**Linux (Ubuntu/Debian)**:
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

**配置 Chrome 路径**:
```bash
# macOS
export PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"

# Linux
export PUPPETEER_EXECUTABLE_PATH="/usr/bin/google-chrome"
```

## 项目结构

```
snap-kit/
├── blocklets/
│   └── snap-kit/              # 主 Blocklet 应用
│       ├── src/               # React 前端
│       ├── api/               # Express API
│       └── package.json
├── packages/
│   ├── crawler/               # 爬虫引擎
│   │   ├── src/
│   │   └── package.json
│   └── middleware/            # SEO 中间件
│       ├── src/
│       └── package.json
├── scripts/                   # 构建脚本
├── pnpm-workspace.yaml        # Workspace 配置
├── package.json               # 根配置
└── tsconfig.json              # TypeScript 配置
```

## 开发命令

### 克隆仓库

```bash
git clone https://github.com/blocklet/snap-kit.git
cd snap-kit
```

### 安装依赖

```bash
# 安装所有包的依赖
pnpm install
```

### 启动开发服务器

```bash
# 启动所有服务（Blocklet + Packages）
pnpm dev
```

这会同时启动：
- SnapKit Blocklet (前端 + API)
- Crawler Engine (开发模式)
- SEO Middleware (开发模式)

**访问地址**:
- 前端: http://localhost:3000
- API: http://localhost:3000/api

### 单独开发模块

#### 开发 Blocklet

```bash
cd blocklets/snap-kit
npm run dev
```

#### 开发 Crawler Engine

```bash
cd packages/crawler
npm run dev
```

#### 开发 SEO Middleware

```bash
cd packages/middleware
npm run dev
```

## 构建

### 构建所有包

```bash
# 从根目录构建
pnpm build:packages
```

这会构建：
- @arcblock/crawler (CJS + ESM)
- @arcblock/crawler-middleware (CJS + ESM)

### 构建 Blocklet

```bash
cd blocklets/snap-kit
npm run bundle
```

这会：
1. 构建 React 前端
2. 构建 Express API
3. 打包成 Blocklet

## 代码规范

### Lint 检查

```bash
# 检查所有包
pnpm lint

# 修复 lint 错误
pnpm lint:fix
```

### 格式化代码

```bash
# 使用 Prettier 格式化
npx prettier --write "**/*.{js,jsx,ts,tsx,json,css,md}"
```

### Git Hooks

项目使用 `simple-git-hooks` 和 `lint-staged` 在提交前自动检查：

```json
{
  "simple-git-hooks": {
    "pre-commit": "npx lint-staged"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "prettier --write",
      "eslint"
    ]
  }
}
```

## 测试

### 单元测试

```bash
# 运行所有测试
pnpm test

# 运行特定包的测试
cd packages/crawler
npm test
```

### 集成测试

```bash
# 启动开发服务器
pnpm dev

# 在另一个终端运行测试
npm run test:integration
```

### 手动测试

#### 测试爬虫 API

```bash
# 创建爬取任务
curl --request POST \
  --url 'http://localhost:3000/api/crawl' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true
  }'

# 测试截图 API
curl --request POST \
  --url 'http://localhost:3000/api/snap' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "format": "png",
    "sync": true
  }'
```

## 调试

### 调试前端

使用 Chrome DevTools：
1. 打开 http://localhost:3000
2. 按 F12 打开 DevTools
3. 在 Sources 面板设置断点

### 调试后端

使用 VS Code：

**.vscode/launch.json**:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug API",
      "program": "${workspaceFolder}/blocklets/snap-kit/api/dev.ts",
      "preLaunchTask": "npm: build:api",
      "outFiles": ["${workspaceFolder}/blocklets/snap-kit/api/dist/**/*.js"],
      "env": {
        "LOG_LEVEL": "debug"
      }
    }
  ]
}
```

### 查看日志

```bash
# 实时查看日志
tail -f ${BLOCKLET_LOG_DIR}/app.log

# 查看错误日志
grep ERROR ${BLOCKLET_LOG_DIR}/app.log
```

### 调试 Puppeteer

启用 Puppeteer 调试模式：

```typescript
import { initPage } from './puppeteer';

const page = await initPage({
  headless: false,  // 显示浏览器窗口
  devtools: true    // 自动打开 DevTools
});
```

## 版本管理

### 更新版本号

```bash
# 自动更新所有包的版本号
npm run bump-version
```

这会：
1. 更新 `version` 文件
2. 更新所有 `package.json`
3. 更新 `blocklet.yml`

### 查看变更日志

```bash
# 编辑 CHANGELOG.md
vim CHANGELOG.md
```

### Git 提交

```bash
git add .
git commit -m "chore: bump version to 1.5.2"
git push
```

## 依赖管理

### 更新依赖

```bash
# 使用 taze 更新 ArcBlock 相关依赖
npm run update:deps
```

这会更新：
- @abtnode/*
- @aigne/*
- @arcblock/*
- @blocklet/*
- @did-connect/*
- 其他相关包

### 添加依赖

```bash
# 为 root 添加依赖
pnpm add <package> -w

# 为特定包添加依赖
pnpm add <package> --filter @arcblock/crawler

# 为 Blocklet 添加依赖
cd blocklets/snap-kit
pnpm add <package>
```

### 删除依赖

```bash
pnpm remove <package> --filter <workspace>
```

## 发布

### 发布到 npm

```bash
# 构建所有包
pnpm build:packages

# 发布 crawler
cd packages/crawler
npm publish

# 发布 middleware
cd packages/middleware
npm publish
```

### 发布 Blocklet

```bash
cd blocklets/snap-kit

# 构建
npm run bundle

# 部署到 Blocklet Server
npm run deploy
```

### Docker 镜像

```bash
# 构建 Docker 镜像
docker build -t arcblock/snap-kit:latest .

# 推送到 Docker Hub
docker push arcblock/snap-kit:latest
```

## 常见问题

### Q: pnpm install 失败

**解决方法**:
```bash
# 清除缓存
pnpm store prune

# 重新安装
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Q: TypeScript 编译错误

**解决方法**:
```bash
# 清除构建缓存
pnpm clean

# 重新构建
pnpm build:packages
```

### Q: Puppeteer 启动失败

**解决方法**:
```bash
# 检查 Chrome 路径
which google-chrome

# 设置环境变量
export PUPPETEER_EXECUTABLE_PATH="/usr/bin/google-chrome"

# 或安装 Chromium
pnpm add puppeteer
```

### Q: 端口被占用

**解决方法**:
```bash
# 查找占用端口的进程
lsof -i :3000

# 杀死进程
kill -9 <PID>

# 或使用其他端口
export BLOCKLET_PORT=3001
pnpm dev
```

## 开发工作流

### 功能开发

1. 创建功能分支
```bash
git checkout -b feature/new-feature
```

2. 开发和测试
```bash
pnpm dev
# 编写代码和测试
```

3. Lint 和格式化
```bash
pnpm lint:fix
```

4. 提交代码
```bash
git add .
git commit -m "feat: add new feature"
git push origin feature/new-feature
```

5. 创建 Pull Request

### Bug 修复

1. 创建修复分支
```bash
git checkout -b fix/bug-description
```

2. 修复和测试
```bash
pnpm dev
# 修复 bug 和验证
```

3. 提交修复
```bash
git add .
git commit -m "fix: fix bug description"
git push origin fix/bug-description
```

## 贡献指南

### 代码风格

- 使用 TypeScript
- 遵循 ESLint 规则
- 使用 Prettier 格式化
- 编写有意义的提交信息

### 提交信息格式

使用 Conventional Commits：

```
feat: 新功能
fix: Bug 修复
docs: 文档更新
style: 代码格式化
refactor: 重构
test: 测试
chore: 构建/工具链
```

**示例**:
```bash
git commit -m "feat: add screenshot quality control"
git commit -m "fix: resolve memory leak in crawler"
git commit -m "docs: update API documentation"
```

## 相关主题

- [快速开始](getting-started.md) - 基础安装和配置
- [架构说明](architecture.md) - 了解项目架构
- [配置说明](configuration.md) - 环境配置

## 下一步

- 查看 [API 参考](api-reference.md) 了解 API 细节
- 阅读 [应用场景](use-cases.md) 获取实践经验
