# Snap Kit Blocklet

## 前置条件

阅读本文档前，建议先了解：
- [核心模块概述](../modules.md) - 了解三个核心模块的关系

## 概述

Snap Kit Blocklet 是主应用程序，提供完整的 Web 界面和 RESTful API 服务。它整合了 Crawler Engine 和前端界面，为用户提供一站式的网页自动化解决方案。

## 项目信息

- **包名**: `@blocklet/snap-kit`
- **版本**: 1.5.1
- **DID**: `z2qaEE3vhcouzhZVntfX3WbcvtL1uQhjXKr72`
- **许可证**: MIT

## 核心功能

### 1. Web 界面

现代化的 React 前端，提供：

- **任务管理**
  - 创建截图任务
  - 创建爬取任务
  - 查看任务状态
  - 下载结果

- **实时监控**
  - 任务队列状态
  - 执行进度跟踪
  - 错误日志查看
  - 性能指标展示

- **配置管理**
  - 访问密钥管理
  - 定时任务配置
  - 系统参数设置

### 2. API 服务

提供四个主要 API 端点：

#### POST /api/snap - 网页截图

捕获网页截图，支持多种格式和配置。

**参数**:
- `url` (string, 必需): 目标 URL
- `width` (number, 可选): 视口宽度，默认 1440
- `height` (number, 可选): 视口高度，默认 900
- `quality` (number, 可选): 质量 (1-100)，默认 80
- `format` (string, 可选): 格式 (png/jpeg/webp)，默认 webp
- `fullPage` (boolean, 可选): 全屏截图，默认 false
- `sync` (boolean, 可选): 同步模式，默认 false

**示例**:
```bash
curl --request POST \
  --url 'https://snap.example.com/api/snap' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "format": "png",
    "width": 1920,
    "height": 1080,
    "sync": true
  }'
```

#### POST /api/crawl - 网页爬取

爬取网页内容，提取 HTML 和 meta 信息。

**参数**:
- `url` (string, 必需): 目标 URL
- `timeout` (number, 可选): 超时时间 (10-120秒)，默认 120
- `sync` (boolean, 可选): 同步模式，默认 false
- `header` (object, 可选): 自定义请求头
- `cookies` (array, 可选): 自定义 Cookie

**示例**:
```bash
curl --request POST \
  --url 'https://snap.example.com/api/crawl' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true
  }'
```

#### POST /api/carbon - 代码截图

使用 Carbon 生成代码截图。

**参数**:
- `code` (string, 必需): 代码内容
- `format` (string, 可选): 格式 (png/jpeg)，默认 png
- `t` (string, 可选): 主题，默认 one-dark
- `sync` (boolean, 可选): 同步模式，默认 false

**Carbon 样式参数**:
- `bg`: 背景色
- `width`: 宽度
- `fm`: 字体
- `fs`: 字体大小
- `ln`: 显示行号

**示例**:
```bash
curl --request POST \
  --url 'https://snap.example.com/api/carbon' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "code": "console.log(\"Hello World\");",
    "t": "one-dark",
    "format": "png",
    "sync": true
  }'
```

#### GET /api/snap 和 /api/crawl

通过 jobId 查询任务结果。

**示例**:
```bash
curl --request GET \
  --url 'https://snap.example.com/api/crawl?jobId=xxx' \
  --header 'Authorization: Bearer YOUR_KEY'
```

### 3. 身份认证

所有 API 都需要访问密钥认证：

```
Authorization: Bearer YOUR_ACCESS_KEY
```

访问密钥可以在 Web 界面中生成和管理。详细认证方法参考：[Blocklet SDK 文档](https://www.arcblock.io/docs/blocklet-developer/en/access-key)

## 技术架构

### 前端技术栈

- **React 19.1**: 最新的 React 特性
- **TypeScript**: 类型安全
- **Vite 7.0**: 快速的开发和构建
- **Material-UI**: UI 组件库

### 后端技术栈

- **Express 4.21**: Web 框架
- **@arcblock/crawler**: 爬虫引擎
- **Joi**: 请求验证
- **DID Auth**: 身份认证

### 项目结构

```
blocklets/snap-kit/
├── src/                    # React 前端
│   ├── app.tsx             # 应用主组件
│   ├── pages/              # 页面组件
│   ├── assets/             # 静态资源
│   └── libs/               # 工具库
├── api/                    # Express API
│   ├── src/
│   │   ├── index.ts        # API 入口
│   │   └── routes/         # API 路由
│   └── hooks/
│       └── pre-start.js    # 启动钩子
├── public/                 # 公共资源
├── blocklet.yml            # Blocklet 配置
└── package.json
```

## 部署方式

### Docker 部署

使用官方 Docker 镜像：

```bash
docker run -p 3000:3000 arcblock/snap-kit
```

Blocklet 配置了专门的 Docker 镜像，包含：
- 预安装的 Chromium
- 优化的运行环境
- 自动初始化脚本

### Blocklet Server 部署

```bash
# 构建
npm run bundle

# 部署
npm run deploy
```

### 环境变量

虽然 Blocklet 会自动配置大部分环境变量，但您也可以自定义：

- `BLOCKLET_PORT`: 服务端口
- `BLOCKLET_DATA_DIR`: 数据目录
- `BLOCKLET_LOG_DIR`: 日志目录
- `PUPPETEER_EXECUTABLE_PATH`: Chrome 路径（Docker 中自动配置）

## 开发指南

### 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 前端: http://localhost:3000
# API: http://localhost:3000/api
```

### 构建

```bash
# 构建前端和后端
npm run bundle

# 只构建前端
npm run build

# 只构建后端
npm run build:api
```

### 版本更新

```bash
# 自动更新版本号
npm run bump-version
```

## 配置文件

### blocklet.yml

主要配置项：

```yaml
title: Snap Kit
version: 1.5.1
group: dapp
interfaces:
  - type: web
    port: BLOCKLET_PORT
    protocol: http
timeout:
  start: 90
requirements:
  server: '>=1.16.28'
docker:
  image: arcblock/snap-kit
```

### 启动钩子

`api/hooks/pre-start.js` 在 Blocklet 启动前执行，用于：
- 初始化数据库
- 初始化爬虫引擎
- 配置定时任务

## 最佳实践

### 1. 访问密钥管理

- 定期轮换访问密钥
- 为不同的客户端使用不同的密钥
- 妥善保管密钥，不要提交到代码仓库

### 2. 性能优化

- 使用异步模式（`sync: false`）处理大量任务
- 合理配置并发数
- 启用缓存减少重复爬取

### 3. 错误处理

- 检查 API 返回的状态码
- 处理超时情况
- 实现重试机制

### 4. 安全建议

- 使用 HTTPS 部署
- 启用 Rate Limiting
- 验证输入 URL
- 限制访问来源

## 故障排查

### API 返回 401 错误

检查访问密钥是否正确配置：
```bash
# 确保 Authorization 头正确
Authorization: Bearer YOUR_ACCESS_KEY
```

### 截图失败

1. 检查 Chrome 是否正确安装
2. 查看日志文件中的错误信息
3. 验证目标 URL 是否可访问

### 任务一直处于 pending 状态

1. 检查队列是否正常运行
2. 查看并发数配置
3. 重启 Blocklet

## 相关主题

- [核心模块](../modules.md) - 返回模块概述
- [Crawler Engine](crawler.md) - 了解底层爬虫引擎
- [API 参考](../api-reference.md) - 完整的 API 文档
- [配置说明](../configuration.md) - 高级配置选项

## 下一步

- 了解 [Crawler Engine](crawler.md) 的工作原理
- 查看 [API 参考](../api-reference.md) 学习更多 API 用法
- 阅读 [配置说明](../configuration.md) 进行高级配置
