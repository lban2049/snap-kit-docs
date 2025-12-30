# 架构说明

本文档详细介绍 Snap Kit 的整体架构、技术栈和项目结构。

## 架构概览

Snap Kit 采用经典的三层架构设计，将前端展示、API 服务和核心引擎进行解耦。

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│                     │    │                     │    │                     │
│   React Frontend    │───▶│   Express API       │───▶│   Crawler Engine    │
│                     │    │                     │    │                     │
│  • Modern UI        │    │  • RESTful API      │    │  • Puppeteer        │
│  • Real-time        │    │  • Authentication   │    │  • Queue System     │
│  • Dashboard        │    │  • Rate Limiting    │    │  • SQLite Storage   │
│                     │    │                     │    │                     │
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

### 架构层次

1. **表示层（React Frontend）**
   - 提供用户友好的 Web 界面
   - 实时显示爬取任务状态
   - 管理访问密钥和配置

2. **服务层（Express API）**
   - 提供 RESTful API 端点
   - 处理身份认证和授权
   - 管理任务队列和调度

3. **引擎层（Crawler Engine）**
   - 执行实际的网页爬取和截图
   - 管理 Puppeteer 实例
   - 处理数据存储和缓存

## 技术栈

### 前端技术

| 技术 | 版本 | 用途 |
|------|------|------|
| React | 19.1 | UI 框架 |
| TypeScript | 5.7+ | 类型安全 |
| Vite | 7.0 | 构建工具 |

### 后端技术

| 技术 | 版本 | 用途 |
|------|------|------|
| Express | 4.21 | Web 框架 |
| TypeScript | 5.7+ | 类型安全 |
| DID Auth | - | 身份认证 |

### 核心依赖

| 技术 | 版本 | 用途 |
|------|------|------|
| Puppeteer | Latest | 浏览器自动化 |
| @blocklet/puppeteer | - | Blocklet 集成 |
| Sequelize | - | ORM 框架 |
| SQLite | - | 数据存储 |

## 项目结构

Snap Kit 采用 Monorepo 架构，使用 pnpm workspace 管理多个包：

```
snap-kit/
├── blocklets/snap-kit/         # 主 Blocklet 应用
│   ├── src/                    # React 前端源码
│   │   ├── app.tsx             # 应用主组件
│   │   ├── pages/              # 页面组件
│   │   └── assets/             # 静态资源
│   ├── api/                    # Express API 源码
│   │   ├── src/
│   │   │   ├── index.ts        # API 入口
│   │   │   └── routes/         # API 路由
│   │   └── hooks/              # Blocklet 钩子
│   ├── public/                 # 公共资源
│   ├── blocklet.yml            # Blocklet 配置
│   └── package.json
├── packages/
│   ├── crawler/                # 爬虫引擎包
│   │   ├── src/
│   │   │   ├── index.ts        # 导出接口
│   │   │   ├── crawler.ts      # 爬虫核心
│   │   │   ├── puppeteer.ts    # Puppeteer 封装
│   │   │   ├── cron.ts         # 定时任务
│   │   │   ├── services/       # 服务层
│   │   │   └── store/          # 数据存储
│   │   └── package.json
│   └── middleware/             # SEO 中间件包
│       ├── src/
│       │   ├── index.ts        # 中间件入口
│       │   ├── cache.ts        # 缓存管理
│       │   └── store/          # 数据存储
│       └── package.json
├── scripts/                    # 构建和工具脚本
├── pnpm-workspace.yaml         # pnpm workspace 配置
├── package.json                # 根 package.json
└── tsconfig.json               # TypeScript 配置
```

### 目录说明

#### blocklets/snap-kit

主应用程序目录，包含：
- **src/**: React 前端代码
- **api/**: Express API 代码
- **public/**: 静态资源
- **blocklet.yml**: Blocklet 配置文件

#### packages/crawler

爬虫引擎核心包，提供：
- 网页爬取功能
- 截图生成
- 队列管理
- 定时任务
- 数据存储

#### packages/middleware

SEO 中间件包，提供：
- 预渲染功能
- 缓存管理
- 搜索引擎优化

## 数据流

### 截图流程

```
1. 用户请求 → POST /api/snap
2. API 验证请求参数
3. 创建爬取任务
4. 任务加入队列
5. 队列调度任务
6. Crawler Engine 执行:
   - 启动 Puppeteer
   - 访问目标 URL
   - 捕获截图
   - 保存到本地
7. 更新任务状态
8. 返回结果
```

### 爬取流程

```
1. 用户请求 → POST /api/crawl
2. API 验证请求参数
3. 创建爬取任务
4. 任务加入队列
5. 队列调度任务
6. Crawler Engine 执行:
   - 启动 Puppeteer
   - 访问目标 URL
   - 提取 HTML 内容
   - 提取 meta 信息
   - 保存到数据库
7. 更新任务状态
8. 返回结果
```

## 队列系统

Snap Kit 使用 `@abtnode/queue` 实现任务队列管理，支持：

- **并发控制**: 限制同时执行的任务数量
- **任务调度**: 根据优先级调度任务
- **持久化**: 任务状态保存到 SQLite
- **定时任务**: 支持 cron 表达式的定时任务

### 队列类型

1. **urlCrawler**: 异步 URL 爬取队列
2. **syncCrawler**: 同步 URL 爬取队列
3. **codeCrawler**: 代码截图队列
4. **cronJobs**: 定时任务队列

## 数据存储

### SQLite 数据库

Snap Kit 使用 SQLite 作为主要数据存储，包含以下表：

1. **jobs**: 存储任务信息
2. **snapshots**: 存储爬取结果
3. **sites**: 存储站点配置

### 文件存储

- **截图文件**: 保存在 `BLOCKLET_DATA_DIR/screenshots/`
- **HTML 文件**: 保存在 `BLOCKLET_DATA_DIR/html/`
- **日志文件**: 保存在 `BLOCKLET_LOG_DIR/`

## 部署架构

### Docker 部署

```
┌──────────────────────────────────┐
│   Docker Container               │
│                                  │
│  ┌────────────────────────────┐  │
│  │   Snap Kit Application     │  │
│  │   + Chromium               │  │
│  └────────────────────────────┘  │
│                                  │
│  Volumes:                        │
│  • /var/lib/blocklet/data        │
│  • /var/lib/blocklet/logs        │
└──────────────────────────────────┘
```

### Blocklet Server 部署

```
┌────────────────────────────────────┐
│   Blocklet Server                  │
│                                    │
│  ┌──────────────────────────────┐  │
│  │   Snap Kit Blocklet          │  │
│  │   • Frontend                 │  │
│  │   • API                      │  │
│  │   • Crawler Engine           │  │
│  └──────────────────────────────┘  │
│                                    │
│  ┌──────────────────────────────┐  │
│  │   Other Blocklets            │  │
│  └──────────────────────────────┘  │
└────────────────────────────────────┘
```

## 扩展性

::: PATCH
# Delete
### 水平扩展

通过部署多个 Snap Kit 实例，配合负载均衡器实现水平扩展：

```
                ┌─────────────┐
                │Load Balancer│
                └──────┬──────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  ┌─────────┐    ┌─────────┐    ┌─────────┐
  │Instance1│    │Instance2│    │Instance3│
  └─────────┘    └─────────┘    └─────────┘
```
:::

### 性能优化

- **并发控制**: 通过 `concurrency` 参数控制并发数
- **队列管理**: 分离同步和异步任务队列
- **缓存策略**: 使用 LRU 缓存减少重复爬取
- **资源池**: 复用 Puppeteer 浏览器实例

## 相关主题

- [核心模块](modules.md) - 详细了解各个核心模块
- [配置说明](configuration.md) - 配置系统参数
- [开发指南](development.md) - 本地开发和构建

## 下一步

- 查看[核心模块](modules.md)了解各个模块的详细功能
- 阅读[API 参考](api-reference.md)学习如何使用 API
