# 核心模块

Snap Kit 由三个核心模块组成，每个模块都专注于特定的功能领域。本文档介绍这三个模块的概览和主要功能。

## 模块架构

```
┌─────────────────────────────────────────────────────────┐
│                    Snap Kit                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────┐ │
│  │  Snap Kit       │  │  Crawler        │  │   SEO   │ │
│  │  Blocklet       │  │  Engine         │  │Middleware│ │
│  │                 │  │                 │  │         │ │
│  │  React + API    │  │  Puppeteer     │  │ Cache   │ │
│  └─────────────────┘  └─────────────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

## 1. Snap Kit Blocklet

### 概述

Snap Kit Blocklet 是主应用程序，提供完整的 Web 界面和 API 服务。它是用户与 Snap Kit 交互的主要入口点。

### 主要功能

- **React 前端界面**
  - 现代化的用户界面
  - 实时任务监控仪表板
  - 访问密钥管理
  - 配置管理界面

- **Express API 服务**
  - RESTful API 端点
  - DID 身份认证
  - 请求验证和错误处理
  - API 访问密钥管理

- **核心 API 端点**
  - `/api/snap` - 网页截图
  - `/api/crawl` - 网页爬取
  - `/api/carbon` - 代码截图

### 技术特性

- **React 19.1**: 最新的 React 特性
- **TypeScript**: 类型安全的开发
- **Vite 7.0**: 快速的构建和开发体验
- **DID Auth**: 去中心化身份认证

### 适用场景

- 需要 Web 界面管理爬取任务
- 需要访问密钥管理和权限控制
- 需要实时监控任务状态
- 需要完整的 Blocklet 生态集成

详细信息请参考：[Snap Kit Blocklet 详细文档](modules/blocklet.md)

## 2. Crawler Engine

### 概述

Crawler Engine 是 Snap Kit 的核心自动化引擎，基于 Puppeteer 构建，负责实际的网页访问、截图和内容提取。

### 主要功能

- **网页爬取**
  - 异步和同步爬取模式
  - HTML 内容提取
  - Meta 信息提取（标题、描述等）
  - 支持自定义 Headers 和 Cookies

- **截图生成**
  - 多种图片格式（PNG, JPEG, WebP）
  - 全屏或视口截图
  - 自定义尺寸和质量
  - 代码截图（集成 Carbon）

- **队列管理**
  - 并发控制
  - 任务优先级
  - 失败重试
  - 任务持久化

- **定时任务**
  - 基于 Sitemap 的定时爬取
  - Cron 表达式支持
  - 自动更新缓存

- **数据存储**
  - SQLite 数据库
  - 文件系统存储
  - 自动迁移

### 核心 API

```typescript
// 初始化爬虫
await initCrawler({
  concurrency: 5,
  siteCron: {
    enabled: true,
    sites: ['https://example.com/sitemap.xml']
  }
});

// 爬取 URL
const jobId = await crawlUrl({
  url: 'https://example.com',
  includeScreenshot: true,
  includeHtml: true
});

// 获取结果
const snapshot = await getSnapshot(jobId);
```

### 适用场景

- 需要独立的爬虫服务
- 需要集成到现有 Node.js 应用
- 需要定时爬取功能
- 需要大规模批量处理

详细信息请参考：[Crawler Engine 详细文档](modules/crawler.md)

## 3. SEO Middleware

### 概述

SEO Middleware 是一个 Express 中间件，为单页应用（SPA）提供预渲染能力，确保搜索引擎可以正确索引动态内容。

### 主要功能

- **智能预渲染**
  - 自动检测搜索引擎爬虫
  - 返回预渲染的 HTML
  - 支持自定义爬虫检测规则

- **多层缓存**
  - 内存 LRU 缓存（快速访问）
  - SQLite 持久化缓存
  - 可配置的缓存策略

- **自动更新**
  - 缓存过期自动更新
  - 异步更新不影响当前请求
  - 失败重试机制

### 核心 API

```typescript
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const snapshotMiddleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  allowCrawler: (req) => {
    return req.path === '/';
  }
});

// 应用到所有路由
app.use(snapshotMiddleware);

// 应用到特定路由
app.use('/docs', snapshotMiddleware);
```

### 缓存工作流程

```
1. 请求到达 → 检测是否为爬虫
2. 是爬虫 → 检查内存缓存
3. 缓存命中 → 返回缓存的 HTML
4. 缓存未命中 → 检查 SQLite
5. 数据库有记录 → 返回并更新内存缓存
6. 数据库无记录 → 异步请求 Snap Kit
7. 后台更新缓存，下次请求命中
```

### 适用场景

- SPA 应用需要 SEO 优化
- 需要为搜索引擎提供预渲染内容
- 需要缓存动态页面
- 需要提高首次加载速度

详细信息请参考：[SEO Middleware 详细文档](modules/middleware.md)

## 模块间协作

### Blocklet + Crawler Engine

Snap Kit Blocklet 内部集成了 Crawler Engine，通过以下方式协作：

```typescript
// Blocklet API 内部调用 Crawler Engine
import { initCrawler, crawlUrl } from '@arcblock/crawler';

await initCrawler({ concurrency: 5 });

app.post('/api/snap', async (req, res) => {
  const jobId = await crawlUrl({
    url: req.body.url,
    includeScreenshot: true
  });
  res.json({ jobId });
});
```

### Blocklet + SEO Middleware

其他 Blocklet 应用可以使用 SEO Middleware 连接到 Snap Kit：

```typescript
// 在其他 Blocklet 中使用
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const middleware = createSnapshotMiddleware({
  endpoint: 'https://your-snap-kit.example.com',
  accessKey: 'your-access-key'
});

app.use(middleware);
```

### 完整生态

```
┌─────────────────────────────────────────────────┐
│              用户的 SPA 应用                     │
│  ┌───────────────────────────────────────────┐  │
│  │   SEO Middleware                         │  │
│  │   ↓                                       │  │
│  │   请求 Snap Kit API                       │  │
│  └───────────────────────────────────────────┘  │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│           Snap Kit Blocklet                     │
│  ┌───────────────┐     ┌────────────────────┐   │
│  │   API 服务    │────▶│  Crawler Engine   │   │
│  └───────────────┘     └────────────────────┘   │
└─────────────────────────────────────────────────┘
```

## 模块选择指南

### 使用 Snap Kit Blocklet

✅ 需要完整的 Web 界面和 API 服务  
✅ 需要 Blocklet 生态集成  
✅ 需要访问控制和权限管理  
✅ 需要快速部署的完整解决方案

### 使用 Crawler Engine

✅ 需要集成到现有 Node.js 应用  
✅ 需要定制化的爬虫逻辑  
✅ 需要独立的爬虫服务  
✅ 需要定时爬取功能

### 使用 SEO Middleware

✅ SPA 应用需要 SEO 优化  
✅ 需要为搜索引擎提供预渲染  
✅ 需要简单的集成方式  
✅ 已有 Snap Kit 实例可用

## 相关主题

- [Snap Kit Blocklet 详细文档](modules/blocklet.md)
- [Crawler Engine 详细文档](modules/crawler.md)
- [SEO Middleware 详细文档](modules/middleware.md)
- [API 参考](api-reference.md)

## 下一步

- 深入了解[各个模块的详细功能](modules/blocklet.md)
- 查看[API 参考](api-reference.md)了解如何使用
- 阅读[配置说明](configuration.md)进行高级配置
