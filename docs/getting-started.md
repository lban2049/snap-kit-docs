# 快速开始

本指南将帮助您快速安装、配置和运行 Snap Kit。

## 前置条件

在开始之前，请确保您的系统满足以下要求：

- **Node.js**: 18 或更高版本
- **包管理器**: pnpm（推荐）或 npm
- **Docker**（可选）: 用于容器化部署

## 安装方式

Snap Kit 提供多种安装和部署方式，您可以根据需求选择：

### 方式一：本地开发

适用于开发和测试环境。

#### 1. 克隆仓库

```bash
git clone https://github.com/blocklet/snap-kit.git
cd snap-kit
```

#### 2. 安装依赖

```bash
# 使用 pnpm（推荐）
pnpm install

# 或使用 npm
npm install
```

#### 3. 启动开发服务器

```bash
pnpm dev
```

开发服务器将启动在 `http://localhost:3000`。

### 方式二：Docker 部署

适用于生产环境的快速部署。

#### 使用 Docker 镜像

```bash
# 拉取并运行官方镜像
docker run -p 3000:3000 arcblock/snap-kit
```

应用将在 `http://localhost:3000` 可用。

#### 自定义 Docker 配置

如果需要自定义配置，可以使用环境变量：

```bash
docker run -p 3000:3000 \
  -e PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium \
  -v /path/to/data:/var/lib/blocklet/data \
  arcblock/snap-kit
```

### 方式三：Blocklet Server 部署

适用于 Blocklet 生态系统。

#### 1. 构建 Blocklet

```bash
cd blocklets/snap-kit
npm run bundle
```

#### 2. 部署到 Blocklet Server

```bash
npm run deploy
```

部署完成后，Blocklet Server 会为您提供访问 URL。

## 验证安装

### 检查服务状态

访问以下 URL 验证服务是否正常运行：

```
http://localhost:3000
```

您应该看到 Snap Kit 的 Web 界面。

![Snap Kit 界面截图](../sources/snap-kit/blocklets/snap-kit/screenshots/1.jpeg)

### 测试 API

使用以下命令测试 API 是否可用（需要先配置访问密钥）：

```bash
curl --request POST \
  --url 'http://localhost:3000/api/snap' \
  --header 'Authorization: Bearer YOUR_ACCESS_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true
  }'
```

## 配置访问密钥

Snap Kit 的所有 API 都需要访问密钥进行身份验证。

### 生成访问密钥

1. 访问 Snap Kit 的 Web 界面
2. 进入设置页面
3. 点击"生成新的访问密钥"
4. 保存生成的密钥（密钥只会显示一次）

### 使用访问密钥

在 API 请求中添加 Authorization 头：

```bash
Authorization: Bearer YOUR_ACCESS_KEY
```

详细的认证方法请参考 [Blocklet SDK 文档](https://www.arcblock.io/docs/blocklet-developer/en/access-key)。

## 基本使用示例

### 1. 捕获网页截图

```bash
curl --request POST \
  --url 'http://localhost:3000/api/snap' \
  --header 'Authorization: Bearer YOUR_ACCESS_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "format": "png",
    "width": 1920,
    "height": 1080,
    "sync": true
  }'
```

### 2. 抓取网页内容

```bash
curl --request POST \
  --url 'http://localhost:3000/api/crawl' \
  --header 'Authorization: Bearer YOUR_ACCESS_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true
  }'
```

### 3. 生成代码截图

```bash
curl --request POST \
  --url 'http://localhost:3000/api/carbon' \
  --header 'Authorization: Bearer YOUR_ACCESS_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "code": "console.log(\"Hello World\");",
    "format": "png",
    "t": "one-dark",
    "sync": true
  }'
```

## 常见问题

### 本地开发时 Chrome 路径问题

如果在本地开发时遇到 Puppeteer 找不到 Chrome 的问题，可以设置环境变量：

```bash
# macOS
export PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"

# Linux
export PUPPETEER_EXECUTABLE_PATH="/usr/bin/google-chrome"

# Windows
set PUPPETEER_EXECUTABLE_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe"
```

### 端口冲突

如果默认端口 3000 已被占用，可以通过环境变量修改：

```bash
export BLOCKLET_PORT=3001
pnpm dev
```

## 下一步

现在您已经成功安装和运行 Snap Kit，可以：

- 查看[架构说明](architecture.md)了解项目结构
- 阅读[核心模块](modules.md)深入了解各个模块
- 参考[API 参考](api-reference.md)查看完整的 API 文档
- 查看[配置说明](configuration.md)了解高级配置选项

## 相关主题

- [架构说明](architecture.md) - 了解项目架构和技术栈
- [配置说明](configuration.md) - 配置环境变量和定时任务
- [API 参考](api-reference.md) - 查看完整的 API 文档
