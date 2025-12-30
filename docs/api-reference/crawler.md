# Crawler API 参考

## 前置条件

阅读本文档前，建议先了解：
- [API 参考概述](../api-reference.md) - API 概览和通用说明
- [Crawler Engine](../modules/crawler.md) - Crawler Engine 模块介绍

## Node.js API

### 安装

```bash
npm install @arcblock/crawler
```

### initCrawler

初始化爬虫引擎。必须在使用其他 API 前调用。

**函数签名**:
```typescript
async function initCrawler(params?: {
  puppeteerPath?: string;
  concurrency?: number;
  cookies?: CookieParam[];
  localStorage?: { key: string; value: string }[];
  siteCron?: {
    enabled: boolean;
    immediate?: boolean;
    sites: Site[];
    time: string;
    concurrency: number;
  };
}): Promise<void>
```

**参数**:
- `puppeteerPath`: Puppeteer 可执行文件路径
- `concurrency`: 并发数，默认 2
- `cookies`: 全局 Cookie 配置
- `localStorage`: 全局 localStorage 配置
- `siteCron`: 定时任务配置

**示例**:
```typescript
import { initCrawler } from '@arcblock/crawler';

await initCrawler({
  concurrency: 5,
  cookies: [
    {
      name: 'session',
      value: 'xxx',
      domain: 'example.com'
    }
  ],
  siteCron: {
    enabled: true,
    sites: [{
      url: 'https://example.com/sitemap.xml',
      pathname: '/',
      interval: 86400000
    }],
    time: '0 2 * * *',
    concurrency: 3
  }
});
```

### crawlUrl

爬取指定 URL。

**函数签名**:
```typescript
async function crawlUrl(params: {
  url: string;
  includeScreenshot?: boolean;
  includeHtml?: boolean;
  timeout?: number;
  waitTime?: number;
  header?: Record<string, string>;
  cookies?: CookieParam[];
  localStorage?: { key: string; value: string }[];
  replace?: boolean;
  ignoreRobots?: boolean;
  format?: 'png' | 'jpeg' | 'webp';
  quality?: number;
  width?: number;
  height?: number;
  fullPage?: boolean;
}): Promise<string>
```

**参数**:
- `url` (必需): 目标 URL
- `includeScreenshot`: 是否生成截图，默认 false
- `includeHtml`: 是否提取 HTML，默认 false
- `timeout`: 超时时间（秒），默认 120
- `waitTime`: 最小等待时间（秒），默认 0
- `header`: 自定义请求头
- `cookies`: 自定义 Cookie
- `localStorage`: 设置 localStorage
- `replace`: 是否替换旧快照，默认 false
- `ignoreRobots`: 是否忽略 robots.txt，默认 false
- `format`: 截图格式，默认 'webp'
- `quality`: 截图质量 (1-100)，默认 80
- `width`: 视口宽度，默认 1440
- `height`: 视口高度，默认 900
- `fullPage`: 全屏截图，默认 false

**返回值**: Promise<string> - jobId

**示例**:
```typescript
import { crawlUrl } from '@arcblock/crawler';

// 只爬取 HTML
const jobId1 = await crawlUrl({
  url: 'https://example.com',
  includeHtml: true
});

// 爬取 HTML 和截图
const jobId2 = await crawlUrl({
  url: 'https://example.com',
  includeHtml: true,
  includeScreenshot: true,
  format: 'png',
  width: 1920,
  height: 1080
});

// 使用自定义 Cookie
const jobId3 = await crawlUrl({
  url: 'https://example.com',
  includeHtml: true,
  cookies: [{
    name: 'auth_token',
    value: 'your_token',
    domain: 'example.com'
  }]
});
```

### getSnapshot

通过 jobId 获取爬取结果。

**函数签名**:
```typescript
async function getSnapshot(jobId: string): Promise<Snapshot | null>
```

**返回值**:
```typescript
type Snapshot = {
  jobId: string;
  url: string;
  html?: string;
  screenshot?: string;
  status: 'success' | 'failed' | 'pending';
  error?: string;
  meta?: {
    title: string;
    description: string;
  };
  createdAt: string;
  updatedAt: string;
}
```

**示例**:
```typescript
import { getSnapshot } from '@arcblock/crawler';

const snapshot = await getSnapshot(jobId);

if (snapshot?.status === 'success') {
  console.log('Title:', snapshot.meta?.title);
  console.log('Description:', snapshot.meta?.description);
  console.log('HTML length:', snapshot.html?.length);
  console.log('Screenshot path:', snapshot.screenshot);
} else if (snapshot?.status === 'failed') {
  console.error('Crawl failed:', snapshot.error);
} else {
  console.log('Still pending...');
}
```

### getLatestSnapshot

获取指定 URL 的最新爬取结果。

**函数签名**:
```typescript
async function getLatestSnapshot(url: string): Promise<Snapshot | null>
```

**示例**:
```typescript
import { getLatestSnapshot } from '@arcblock/crawler';

const snapshot = await getLatestSnapshot('https://example.com');
if (snapshot) {
  console.log('Last crawled:', snapshot.updatedAt);
  console.log('HTML:', snapshot.html);
}
```

### deleteSnapshots

删除快照记录。

**函数签名**:
```typescript
async function deleteSnapshots(
  params: {
    url: string;
    replace?: boolean;
  },
  options?: {
    txn?: Transaction;
  }
): Promise<string[]>
```

**参数**:
- `url`: 目标 URL
- `replace`: 是否替换模式
- `options.txn`: Sequelize 事务对象

**返回值**: Promise<string[]> - 被删除的 jobId 数组

**示例**:
```typescript
import { deleteSnapshots } from '@arcblock/crawler';

const deletedIds = await deleteSnapshots({
  url: 'https://example.com',
  replace: true
});

console.log('Deleted snapshots:', deletedIds);
```

## HTTP REST API

### POST /api/crawl

爬取网页内容。

**请求参数**:
| 参数 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| url | string | 是 | - | 目标 URL |
| timeout | number | 否 | 120 | 超时时间 (10-120秒) |
| waitTime | number | 否 | 0 | 最小等待时间 (0-120秒) |
| sync | boolean | 否 | false | 同步模式 |
| header | object | 否 | - | 自定义请求头 |
| cookies | array | 否 | - | 自定义 Cookie |
| localStorage | array | 否 | - | 设置 localStorage |

**响应（异步模式）**:
```json
{
  "code": "ok",
  "data": {
    "jobId": "job_id"
  }
}
```

**响应（同步模式）**:
```json
{
  "code": "ok",
  "data": {
    "jobId": "job_id",
    "url": "https://example.com",
    "html": "<!DOCTYPE html>...",
    "status": "success",
    "meta": {
      "title": "Example Domain",
      "description": "Example site"
    }
  }
}
```

**示例**:
```bash
# 异步模式
curl --request POST \
  --url 'https://snap.example.com/api/crawl' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com"
  }'

# 同步模式
curl --request POST \
  --url 'https://snap.example.com/api/crawl' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true,
    "timeout": 60
  }'

# 带 Cookie
curl --request POST \
  --url 'https://snap.example.com/api/crawl' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "sync": true,
    "cookies": [
      {
        "name": "session",
        "value": "xxx",
        "domain": "example.com"
      }
    ]
  }'
```

### GET /api/crawl

查询爬取结果。

**请求参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| jobId | string | 是 | 任务 ID |

**响应**:
```json
{
  "code": "ok",
  "data": {
    "jobId": "job_id",
    "url": "https://example.com",
    "html": "<!DOCTYPE html>...",
    "status": "success",
    "meta": {
      "title": "Example Domain",
      "description": "Example site"
    }
  }
}
```

**示例**:
```bash
curl --request GET \
  --url 'https://snap.example.com/api/crawl?jobId=xxx' \
  --header 'Authorization: Bearer YOUR_KEY'
```

### POST /api/snap

捕获网页截图。

**请求参数**:
| 参数 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| url | string | 是 | - | 目标 URL |
| width | number | 否 | 1440 | 视口宽度（最小375） |
| height | number | 否 | 900 | 视口高度（最小500） |
| quality | number | 否 | 80 | 质量 (1-100) |
| format | string | 否 | webp | 格式 (png/jpeg/webp) |
| timeout | number | 否 | 120 | 超时时间 (10-120秒) |
| waitTime | number | 否 | 0 | 最小等待时间 (0-120秒) |
| fullPage | boolean | 否 | false | 全屏截图 |
| sync | boolean | 否 | false | 同步模式 |
| header | object | 否 | - | 自定义请求头 |
| cookies | array | 否 | - | 自定义 Cookie |

**响应（同步模式）**:
```json
{
  "code": "ok",
  "data": {
    "jobId": "job_id",
    "url": "https://example.com",
    "screenshot": "/screenshots/xxx.png",
    "status": "success",
    "options": {
      "width": 1920,
      "height": 1080,
      "format": "png"
    },
    "meta": {
      "title": "Example Domain",
      "description": "Example site"
    }
  }
}
```

**示例**:
```bash
# PNG 截图
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

# JPEG 截图（可控质量）
curl --request POST \
  --url 'https://snap.example.com/api/snap' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "format": "jpeg",
    "quality": 90,
    "sync": true
  }'

# 全屏截图
curl --request POST \
  --url 'https://snap.example.com/api/snap' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "fullPage": true,
    "sync": true
  }'
```

### GET /api/snap

查询截图结果。

**请求参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| jobId | string | 是 | 任务 ID |

**响应**: 同 POST /api/snap

**示例**:
```bash
curl --request GET \
  --url 'https://snap.example.com/api/snap?jobId=xxx' \
  --header 'Authorization: Bearer YOUR_KEY'
```

### POST /api/carbon

生成代码截图。

**请求参数**:
| 参数 | 类型 | 必需 | 默认值 | 说明 |
|------|------|------|--------|------|
| code | string | 是 | - | 代码内容 |
| timeout | number | 否 | 120 | 超时时间 (10-120秒) |
| sync | boolean | 否 | false | 同步模式 |
| format | string | 否 | png | 格式 (png/jpeg) |
| bg | string | 否 | rgba(171,184,195,1) | 背景色 |
| t | string | 否 | one-dark | 主题 |
| l | string | 否 | auto | 语言 |
| width | number | 否 | 680 | 宽度 |
| fm | string | 否 | Hack | 字体 |
| fs | string | 否 | 14px | 字体大小 |
| ln | string | 否 | false | 显示行号 |

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
    "ln": "true",
    "sync": true
  }'
```

### GET /api/carbon

查询代码截图结果。

**请求参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| jobId | string | 是 | 任务 ID |

**示例**:
```bash
curl --request GET \
  --url 'https://snap.example.com/api/carbon?jobId=xxx' \
  --header 'Authorization: Bearer YOUR_KEY'
```

## 类型定义

### CookieParam

```typescript
type CookieParam = {
  name: string;
  value: string;
  domain?: string;
  expires?: string;
  path?: string;
}
```

### Site

```typescript
type Site = {
  url: string;           // Sitemap URL
  pathname: string;      // 路径过滤
  interval?: number;     // 最小爬取间隔（毫秒）
}
```

### Snapshot

```typescript
type Snapshot = {
  jobId: string;
  url: string;
  html?: string;
  screenshot?: string;
  status: 'success' | 'failed' | 'pending';
  error?: string;
  meta?: {
    title: string;
    description: string;
  };
  createdAt: string;
  updatedAt: string;
}
```

## 相关主题

- [API 参考](../api-reference.md) - 返回 API 概述
- [Crawler Engine](../modules/crawler.md) - Crawler Engine 模块文档
- [配置说明](../configuration.md) - 环境变量配置

## 下一步

- 查看 [Middleware API](middleware.md) 了解中间件 API
- 阅读 [应用场景](../use-cases.md) 查看实际示例
- 参考 [配置说明](../configuration.md) 进行高级配置
