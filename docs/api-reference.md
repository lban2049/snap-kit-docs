# API 参考

本文档提供 Snap Kit 所有 API 的概览和快速参考。详细的 API 说明请参考各模块的独立文档。

## API 概览

Snap Kit 提供两类 API：

1. **HTTP REST API** - Snap Kit Blocklet 提供的 Web API
2. **Node.js API** - Crawler Engine 和 SEO Middleware 提供的编程接口

## HTTP REST API

### 认证方式

所有 HTTP API 都需要访问密钥认证：

```bash
Authorization: Bearer YOUR_ACCESS_KEY
```

访问密钥可在 Snap Kit Web 界面中生成和管理。

### 端点列表

| 端点 | 方法 | 功能 | 文档 |
|------|------|------|------|
| `/api/snap` | POST | 捕获网页截图 | [详细文档](api-reference/crawler.md#post-apisnap) |
| `/api/snap` | GET | 查询截图结果 | [详细文档](api-reference/crawler.md#get-apisnap) |
| `/api/crawl` | POST | 爬取网页内容 | [详细文档](api-reference/crawler.md#post-apicrawl) |
| `/api/crawl` | GET | 查询爬取结果 | [详细文档](api-reference/crawler.md#get-apicrawl) |
| `/api/carbon` | POST | 生成代码截图 | [详细文档](api-reference/crawler.md#post-apicarbon) |
| `/api/carbon` | GET | 查询代码截图结果 | [详细文档](api-reference/crawler.md#get-apicarbon) |

### 快速示例

#### 截图

```bash
curl --request POST \
  --url 'https://snap.example.com/api/snap' \
  --header 'Authorization: Bearer YOUR_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "url": "https://example.com",
    "format": "png",
    "sync": true
  }'
```

#### 爬取

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

## Node.js API

### Crawler Engine

核心爬虫引擎提供的编程接口。

**安装**:
```bash
npm install @arcblock/crawler
```

**主要函数**:
- `initCrawler()` - 初始化爬虫
- `crawlUrl()` - 爬取 URL
- `getSnapshot()` - 获取快照
- `getLatestSnapshot()` - 获取最新快照

**快速示例**:
```typescript
import { initCrawler, crawlUrl, getSnapshot } from '@arcblock/crawler';

await initCrawler({ concurrency: 5 });

const jobId = await crawlUrl({
  url: 'https://example.com',
  includeScreenshot: true,
  includeHtml: true
});

const snapshot = await getSnapshot(jobId);
console.log(snapshot.html);
```

**详细文档**: [Crawler API 参考](api-reference/crawler.md)

### SEO Middleware

Express 中间件，用于 SEO 优化。

**安装**:
```bash
npm install @arcblock/crawler-middleware
```

**主要函数**:
- `createSnapshotMiddleware()` - 创建中间件

**快速示例**:
```typescript
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const middleware = createSnapshotMiddleware({
  endpoint: 'https://snap.example.com',
  accessKey: 'your-access-key'
});

app.use(middleware);
```

**详细文档**: [Middleware API 参考](api-reference/middleware.md)

## 响应格式

### 成功响应

```json
{
  "code": "ok",
  "data": {
    // 响应数据
  }
}
```

### 错误响应

```json
{
  "code": "error",
  "message": "错误信息"
}
```

## 常用参数

### URL 参数

- **url** (string, 必需): 目标 URL，必须是有效的 URI
- 示例: `https://example.com`

### 截图参数

- **width** (number): 视口宽度，最小 375px
- **height** (number): 视口高度，最小 500px
- **quality** (number): 质量 1-100，仅 JPEG 有效
- **format** (string): 格式 png/jpeg/webp
- **fullPage** (boolean): 是否全屏截图

### 爬取参数

- **timeout** (number): 超时时间 10-120 秒
- **waitTime** (number): 最小等待时间 0-120 秒
- **sync** (boolean): 同步模式
- **header** (object): 自定义请求头
- **cookies** (array): 自定义 Cookie

### Cookie 格式

```typescript
type Cookie = {
  name: string;
  value: string;
  domain?: string;
  expires?: string;
  path?: string;
}
```

### LocalStorage 格式

```typescript
type LocalStorage = {
  key: string;
  value: string;
}
```

## 图片格式支持

### /api/snap

支持三种格式：
- **PNG**: 无损压缩，适合文字和图形
- **JPEG**: 有损压缩，质量可控 (1-100)，文件更小
- **WebP**: 现代格式，压缩率更高，默认格式

### /api/carbon

支持两种格式：
- **PNG**: 无损压缩，默认格式
- **JPEG**: 有损压缩，质量可控
- ❌ **WebP**: 不支持（会回退到 PNG）

## 任务状态

爬取任务有三种状态：

- **pending**: 等待执行
- **success**: 执行成功
- **failed**: 执行失败

查询结果时检查状态：

```typescript
const snapshot = await getSnapshot(jobId);
if (snapshot?.status === 'success') {
  // 处理成功结果
} else if (snapshot?.status === 'failed') {
  console.error('Error:', snapshot.error);
}
```

## 同步 vs 异步模式

### 异步模式 (sync: false)

- 立即返回 jobId
- 任务在后台执行
- 需要轮询查询结果
- 适合大量任务

```typescript
// 创建任务
const { jobId } = await createTask({ url, sync: false });

// 轮询结果
while (true) {
  const result = await getResult(jobId);
  if (result.status !== 'pending') break;
  await sleep(1000);
}
```

### 同步模式 (sync: true)

- 等待任务完成后返回
- 直接返回结果
- 可能超时
- 适合单个任务

```typescript
// 直接获取结果
const result = await createTask({ url, sync: true });
console.log(result.html);
```

## 速率限制

建议的 API 调用频率：

- **异步模式**: 无限制（受队列大小限制）
- **同步模式**: 建议 < 10 次/秒
- **定时任务**: 根据 interval 配置

## 错误处理

### 常见错误码

- **401 Unauthorized**: 访问密钥无效
- **400 Bad Request**: 参数错误
- **504 Gateway Timeout**: 任务超时
- **500 Internal Server Error**: 服务器错误

### 错误处理示例

```typescript
try {
  const result = await crawlUrl({ url });
} catch (error) {
  if (error.response?.status === 401) {
    console.error('Invalid access key');
  } else if (error.response?.status === 504) {
    console.error('Timeout');
  } else {
    console.error('Error:', error.message);
  }
}
```

## 子文档

详细的 API 文档请参考：

- [Crawler API](api-reference/crawler.md) - Crawler Engine 的完整 API
- [Middleware API](api-reference/middleware.md) - SEO Middleware 的完整 API

## 相关主题

- [核心模块](modules.md) - 了解各个模块的功能
- [配置说明](configuration.md) - 配置环境变量
- [应用场景](use-cases.md) - 实际使用示例

## 下一步

- 查看 [Crawler API](api-reference/crawler.md) 了解爬虫引擎 API
- 查看 [Middleware API](api-reference/middleware.md) 了解中间件 API
- 阅读 [应用场景](use-cases.md) 查看实际示例
