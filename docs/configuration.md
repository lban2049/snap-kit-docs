# 配置说明

本文档详细说明 SnapKit 的环境变量配置、定时任务配置和数据库配置。

## 环境变量

### Crawler Engine 环境变量

#### PUPPETEER_EXECUTABLE_PATH

Puppeteer 的执行路径。

- **必需**: 否（Docker 镜像中自动配置）
- **类型**: 字符串
- **用途**: 指定 Chrome/Chromium 的可执行文件路径

**平台配置**:

```bash
# macOS
export PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"

# Linux
export PUPPETEER_EXECUTABLE_PATH="/usr/bin/google-chrome"

# Docker (已预配置)
# PUPPETEER_EXECUTABLE_PATH="/usr/bin/chromium"
```

#### BLOCKLET_DATA_DIR

数据存储目录。

- **必需**: 是
- **类型**: 字符串
- **用途**: 
  - 存储 SQLite 数据库
  - 保存截图文件
  - 保存 HTML 文件

**示例**:
```bash
export BLOCKLET_DATA_DIR="/var/lib/blocklet/data"
```

**目录结构**:
```
${BLOCKLET_DATA_DIR}/
├── crawler.db              # SQLite 数据库
├── screenshots/            # 截图文件
│   ├── xxx.png
│   └── yyy.webp
└── html/                   # HTML 文件
    └── zzz.html
```

#### BLOCKLET_LOG_DIR

日志文件目录。

- **必需**: 是
- **类型**: 字符串
- **用途**: 存储 @blocklet/logger 日志

**示例**:
```bash
export BLOCKLET_LOG_DIR="/var/log/blocklet"
```

#### BLOCKLET_CACHE_DIR

Puppeteer 自动安装目录。

- **必需**: 否
- **类型**: 字符串
- **默认值**: `process.cwd()`
- **用途**: Puppeteer 自动下载 Chrome 的缓存目录

**示例**:
```bash
export BLOCKLET_CACHE_DIR="/var/cache/blocklet"
```

#### BLOCKLET_APP_URL

应用的域名前缀。

- **必需**: 否
- **类型**: 字符串
- **默认值**: `/`
- **用途**: 构建完整 URL，用于截图和爬取

**示例**:
```bash
export BLOCKLET_APP_URL="https://snap.example.com"
```

#### BLOCKLET_PORT

服务端口。

- **必需**: 否（Blocklet 环境中自动配置）
- **类型**: 数字
- **默认值**: `3000`
- **用途**: HTTP 服务监听端口

**示例**:
```bash
export BLOCKLET_PORT=3001
```

#### LOG_LEVEL

日志级别。

- **必需**: 否
- **类型**: 字符串
- **默认值**: `info`
- **可选值**: `debug`, `info`, `warn`, `error`

**示例**:
```bash
export LOG_LEVEL=debug
```

#### NODE_ENV

运行环境。

- **必需**: 否
- **类型**: 字符串
- **默认值**: `development`
- **可选值**: `development`, `production`

**示例**:
```bash
export NODE_ENV=production
```

### SEO Middleware 环境变量

SEO Middleware 需要以下环境变量（仅在非 Blocklet 环境）:

#### BLOCKLET_DATA_DIR

同 Crawler Engine 的 BLOCKLET_DATA_DIR。

#### BLOCKLET_LOG_DIR

同 Crawler Engine 的 BLOCKLET_LOG_DIR。

#### BLOCKLET_APP_URL

同 Crawler Engine 的 BLOCKLET_APP_URL。

## 定时任务配置

### 配置结构

```typescript
type SiteCronConfig = {
  enabled: boolean;        // 是否启用定时任务
  immediate: boolean;      // 是否立即执行一次
  sites: Site[];           // 站点列表
  time: string;            // Cron 表达式
  concurrency: number;     // 并发数
}

type Site = {
  url: string;            // Sitemap XML 的 URL
  pathname: string;       // 路径过滤器
  interval?: number;      // 最小爬取间隔（毫秒）
}
```

### 基本配置

```typescript
await initCrawler({
  siteCron: {
    enabled: true,
    immediate: false,
    sites: [
      {
        url: 'https://example.com/sitemap.xml',
        pathname: '/',
        interval: 86400000  // 24小时
      }
    ],
    time: '0 2 * * *',  // 每天凌晨2点
    concurrency: 5
  }
});
```

### Cron 表达式

使用标准的 Cron 表达式格式：

```
┌───────────── 分钟 (0 - 59)
│ ┌───────────── 小时 (0 - 23)
│ │ ┌───────────── 日期 (1 - 31)
│ │ │ ┌───────────── 月份 (1 - 12)
│ │ │ │ ┌───────────── 星期 (0 - 7) (0 和 7 都代表星期日)
│ │ │ │ │
* * * * *
```

**常用示例**:

```typescript
// 每小时执行
time: '0 * * * *'

// 每天凌晨 2 点执行
time: '0 2 * * *'

// 每 6 小时执行
time: '0 */6 * * *'

// 每周一凌晨 3 点执行
time: '0 3 * * 1'

// 每月 1 号凌晨 4 点执行
time: '0 4 1 * *'
```

### 多站点配置

```typescript
await initCrawler({
  siteCron: {
    enabled: true,
    sites: [
      {
        url: 'https://blog.example.com/sitemap.xml',
        pathname: '/blog',
        interval: 86400000  // 每天最多爬一次
      },
      {
        url: 'https://docs.example.com/sitemap.xml',
        pathname: '/docs',
        interval: 604800000  // 每周最多爬一次
      },
      {
        url: 'https://news.example.com/sitemap.xml',
        pathname: '/news',
        interval: 3600000  // 每小时最多爬一次
      }
    ],
    time: '0 */2 * * *',  // 每2小时检查一次
    concurrency: 3
  }
});
```

### 路径过滤

`pathname` 用于过滤 Sitemap 中的 URL：

```typescript
{
  url: 'https://example.com/sitemap.xml',
  pathname: '/blog',  // 只爬取 /blog 开头的 URL
  interval: 86400000
}
```

**过滤逻辑**:
- `/blog` → 匹配 `/blog`, `/blog/post1`, `/blog/post2` 等
- `/` → 匹配所有 URL
- `/docs/api` → 只匹配 `/docs/api` 及其子路径

### 爬取间隔

`interval` 控制最小爬取间隔，避免过于频繁：

```typescript
{
  url: 'https://example.com/sitemap.xml',
  pathname: '/',
  interval: 86400000  // 24小时内不会重复爬取同一 URL
}
```

**建议值**:
- 静态内容: 604800000 (7天)
- 每日更新: 86400000 (1天)
- 频繁更新: 3600000 (1小时)

### 立即执行

`immediate` 控制是否在启动时立即执行一次：

```typescript
{
  enabled: true,
  immediate: true,  // 启动后立即执行一次
  sites: [...],
  time: '0 2 * * *'
}
```

**使用场景**:
- 首次部署时立即抓取全站
- 测试定时任务配置
- 更新 Sitemap 后立即同步

### 并发控制

`concurrency` 控制同时执行的爬取任务数：

```typescript
{
  enabled: true,
  sites: [...],
  time: '0 2 * * *',
  concurrency: 5  // 最多5个并发任务
}
```

**建议值**:
- 小型服务器: 2-3
- 中型服务器: 5-10
- 大型服务器: 10+

## 数据库配置

### SQLite 配置

SnapKit 使用 SQLite 作为默认数据库，自动在 `BLOCKLET_DATA_DIR` 创建。

#### 数据库文件

```
${BLOCKLET_DATA_DIR}/crawler.db
```

#### 数据表

1. **jobs** - 任务信息
   - jobId: 任务 ID
   - url: 目标 URL
   - status: 状态 (pending/success/failed)
   - createdAt: 创建时间
   - updatedAt: 更新时间

2. **snapshots** - 爬取结果
   - jobId: 关联的任务 ID
   - url: 目标 URL
   - html: HTML 内容路径
   - screenshot: 截图路径
   - status: 状态
   - meta: 元信息 (JSON)

3. **sites** - 站点配置
   - url: Sitemap URL
   - pathname: 路径过滤
   - lastCrawledAt: 最后爬取时间

#### 迁移

数据库 schema 变更会自动迁移：

```typescript
import { migrate } from '@arcblock/crawler';

await migrate();
```

### 数据清理

定期清理旧数据：

```typescript
import { deleteSnapshots } from '@arcblock/crawler';

// 删除指定 URL 的所有快照
await deleteSnapshots({
  url: 'https://example.com',
  replace: true
});
```

**手动清理**:
```bash
# 删除数据库文件（谨慎操作）
rm ${BLOCKLET_DATA_DIR}/crawler.db

# 删除截图文件
rm -rf ${BLOCKLET_DATA_DIR}/screenshots/*

# 删除 HTML 文件
rm -rf ${BLOCKLET_DATA_DIR}/html/*
```

## 性能优化配置

### 并发数优化

根据服务器性能调整：

```typescript
await initCrawler({
  concurrency: 10,  // 主队列并发数
  siteCron: {
    concurrency: 5  // 定时任务并发数
  }
});
```

### 缓存配置

SEO Middleware 缓存配置：

```typescript
createSnapshotMiddleware({
  cacheMax: 100,           // LRU 缓存大小
  updateInterval: 3600000  // 1小时更新
});
```

### 超时配置

根据目标网站调整：

```typescript
await crawlUrl({
  url: 'https://slow-site.com',
  timeout: 180,   // 3分钟超时
  waitTime: 10    // 最少等待10秒
});
```

## 配置文件示例

### 开发环境

`.env.development`:
```bash
NODE_ENV=development
LOG_LEVEL=debug
BLOCKLET_PORT=3000
BLOCKLET_DATA_DIR=./data
BLOCKLET_LOG_DIR=./logs
PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
```

### 生产环境

`.env.production`:
```bash
NODE_ENV=production
LOG_LEVEL=info
BLOCKLET_PORT=3000
BLOCKLET_DATA_DIR=/var/lib/blocklet/data
BLOCKLET_LOG_DIR=/var/log/blocklet
BLOCKLET_APP_URL=https://snap.example.com
```

### Docker 环境

`docker-compose.yml`:
```yaml
version: '3'
services:
  snap-kit:
    image: arcblock/snap-kit
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
      BLOCKLET_APP_URL: https://snap.example.com
    volumes:
      - ./data:/var/lib/blocklet/data
      - ./logs:/var/log/blocklet
```

## 配置验证

### 检查环境变量

```typescript
import { config } from '@arcblock/crawler';

console.log('Data dir:', config.dataDir);
console.log('Log dir:', process.env.BLOCKLET_LOG_DIR);
console.log('Puppeteer path:', config.puppeteerPath);
console.log('Concurrency:', config.concurrency);
```

### 测试配置

```typescript
// 测试爬虫初始化
try {
  await initCrawler({
    concurrency: 5,
    siteCron: {
      enabled: true,
      sites: [{
        url: 'https://example.com/sitemap.xml',
        pathname: '/'
      }],
      time: '0 2 * * *',
      concurrency: 3
    }
  });
  console.log('✅ 配置正确');
} catch (error) {
  console.error('❌ 配置错误:', error);
}
```

## 常见问题

### Q: 如何修改默认并发数？

A: 在 `initCrawler` 时设置：
```typescript
await initCrawler({ concurrency: 10 });
```

### Q: 如何更改数据存储位置？

A: 设置环境变量：
```bash
export BLOCKLET_DATA_DIR="/path/to/data"
```

### Q: 定时任务不执行怎么办？

A: 检查以下配置：
1. `enabled` 是否为 `true`
2. Cron 表达式是否正确
3. Sitemap URL 是否可访问
4. 查看日志文件中的错误信息

### Q: 如何禁用定时任务？

A: 设置 `enabled: false`：
```typescript
await initCrawler({
  siteCron: {
    enabled: false
  }
});
```

## 相关主题

- [快速开始](getting-started.md) - 基础配置
- [Crawler Engine](modules/crawler.md) - Crawler 配置详情
- [SEO Middleware](modules/middleware.md) - Middleware 配置详情

## 下一步

- 查看 [应用场景](use-cases.md) 了解实际使用
- 阅读 [开发指南](development.md) 进行本地开发
