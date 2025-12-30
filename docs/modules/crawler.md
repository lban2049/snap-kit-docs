# Crawler Engine

## 前置条件

阅读本文档前,建议先了解:
- [核心模块概述](../modules.md) - 了解三个核心模块的关系

## 概述

Crawler Engine (`@arcblock/crawler`) 是 SnapKit 的核心自动化引擎,基于 Puppeteer 构建。它负责实际的网页访问、截图生成、内容提取和数据存储。

## 包信息

- **包名**: `@arcblock/crawler`
- **用途**: 为 Blocklet 设计的爬虫模块
- **主要功能**: 批量爬取 HTML、网页截图、标题、描述等

## 核心功能

### 1. 网页爬取

支持基于 URL 或 Sitemap 的批量爬取:

```typescript
import { crawlUrl, initCrawler } from '@arcblock/crawler';

await initCrawler();

// 异步爬取页面
const jobId = await crawlUrl({
  url: 'https://www.arcblock.io',
  includeScreenshot: true,
  includeHtml: true
});
```

### 2. 结果获取

提供两种方式获取爬取结果:

```typescript
import { getSnapshot, getLatestSnapshot } from '@arcblock/crawler';

// 通过 jobId 获取
const snapshot = await getSnapshot(jobId);

// 获取最新结果
const latest = await getLatestSnapshot('https://www.arcblock.io');
```

### 3. 定时任务

支持基于 Sitemap 的定时爬取:

```typescript
await initCrawler({
  siteCron: {
    enabled: true,
    immediate: false,
    sites: [
      {
        url: 'https://example.com/sitemap.xml',
        pathname: '/',
        interval: 86400000 // 24小时
      }
    ],
    time: '0 2 * * *', // 每天凌晨2点
    concurrency: 5
  }
});
```

## API 参考

### initCrawler

初始化爬虫引擎。

**函数签名**:
```typescript
function initCrawler(params?: {
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

**参数说明**:
- `puppeteerPath`: Puppeteer 可执行文件路径
- `concurrency`: 并发数,默认 2
- `cookies`: 全局 Cookie 配置
- `localStorage`: 全局 localStorage 配置
- `siteCron`: 定时任务配置

**示例**:
```typescript
await initCrawler({
  concurrency: 5,
  cookies: [
    { name: 'token', value: 'xxx', domain: 'example.com' }
  ]
});
```

### crawlUrl

爬取指定页面。

**函数签名**:
```typescript
function crawlUrl(params: {
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
}): Promise<string>
```

**参数说明**:
- `url`: 目标 URL
- `includeScreenshot`: 是否包含截图
- `includeHtml`: 是否包含 HTML
- `timeout`: 超时时间(秒)
- `waitTime`: 最小等待时间(秒)
- `header`: 自定义请求头
- `cookies`: 自定义 Cookie
- `localStorage`: 设置 localStorage
- `replace`: 是否替换旧快照
- `ignoreRobots`: 是否忽略 robots.txt

**返回值**: jobId (字符串)

**示例**:
```typescript
const jobId = await crawlUrl({
  url: 'https://example.com',
  includeScreenshot: true,
  includeHtml: true,
  timeout: 60,
  header: {
    'User-Agent': 'Custom Bot'
  }
});
```

### getSnapshot

通过 jobId 获取爬取结果。

**函数签名**:
```typescript
function getSnapshot(jobId: string): Promise<Snapshot | null>
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
}
```

**示例**:
```typescript
const snapshot = await getSnapshot(jobId);
if (snapshot?.status === 'success') {
  console.log('Title:', snapshot.meta?.title);
  console.log('HTML:', snapshot.html);
}
```

### getLatestSnapshot

获取 URL 的最新爬取结果。

**函数签名**:
```typescript
function getLatestSnapshot(url: string): Promise<Snapshot | null>
```

**示例**:
```typescript
const snapshot = await getLatestSnapshot('https://example.com');
```

## 配置选项

### 环境变量

Crawler Engine 使用以下环境变量:

#### PUPPETEER_EXECUTABLE_PATH

Puppeteer 的执行路径。

- **必需**: 否(在 Docker 镜像中自动配置)
- **示例**:
  - macOS: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`
  - Linux: `/usr/bin/google-chrome`

#### BLOCKLET_CACHE_DIR

Puppeteer 自动安装目录。

- **必需**: 否
- **默认值**: `process.cwd()`

#### BLOCKLET_APP_URL

截图的域名前缀。

- **必需**: 否
- **默认值**: `/`

#### BLOCKLET_DATA_DIR

数据存储目录。

- **必需**: 是
- **用途**: 保存截图和 HTML 文件

#### BLOCKLET_LOG_DIR

日志文件目录。

- **必需**: 是
- **用途**: 保存 @blocklet/logger 日志

### 定时任务配置

```typescript
type SiteCronConfig = {
  enabled: boolean;        // 是否启用
  immediate: boolean;      // 是否立即执行
  sites: Site[];           // 站点列表
  time: string;            // Cron 表达式
  concurrency: number;     // 并发数
}

type Site = {
  url: string;            // Sitemap URL
  pathname: string;       // 路径过滤
  interval?: number;      // 最小爬取间隔(毫秒)
}
```

**示例**:
```typescript
{
  enabled: true,
  immediate: false,
  sites: [
    {
      url: 'https://example.com/sitemap.xml',
      pathname: '/blog',
      interval: 3600000  // 1小时
    }
  ],
  time: '0 */6 * * *',  // 每6小时
  concurrency: 3
}
```

## 数据存储

### SQLite 数据库

调用 `initCrawler` 时会在 `BLOCKLET_DATA_DIR` 创建 SQLite 数据库,用于:
- 缓存 HTML 内容
- 存储截图路径
- 管理任务状态

### 文件存储

- **截图**: `${BLOCKLET_DATA_DIR}/screenshots/`
- **HTML**: `${BLOCKLET_DATA_DIR}/html/`

## 队列系统

Crawler Engine 使用 `@abtnode/queue` 管理任务队列:

### 队列类型

1. **urlCrawler**: 异步 URL 爬取
2. **syncCrawler**: 同步 URL 爬取
3. **codeCrawler**: 代码截图
4. **cronJobs**: 定时任务

### 并发控制

通过 `concurrency` 参数控制同时执行的任务数:

```typescript
await initCrawler({
  concurrency: 5  // 最多5个并发任务
});
```

### 任务持久化

任务状态保存在 SQLite 数据库中,重启后可恢复:
- 队列名称
- 任务参数
- 执行状态
- 错误信息

## 工作流程

### 爬取流程

```
1. 调用 crawlUrl()
2. 创建任务并生成 jobId
3. 任务加入队列
4. 队列调度器分配任务
5. 初始化 Puppeteer 页面
6. 访问目标 URL
7. 等待页面加载完成
8. 提取 HTML 和 meta 信息
9. 捕获截图(如果需要)
10. 保存到数据库和文件系统
11. 更新任务状态为 success/failed
```

### 定时任务流程

```
1. 解析 Sitemap XML
2. 提取所有 URL
3. 按 pathname 过滤
4. 检查最后爬取时间
5. 跳过间隔时间内的 URL
6. 为符合条件的 URL 创建任务
7. 加入 cronJobs 队列
8. 按并发数执行
```

## 最佳实践

### 1. 合理配置并发数

根据服务器性能配置:
- 小型服务器: 2-3
- 中型服务器: 5-10
- 大型服务器: 10+

### 2. 使用定时任务

对于需要定期更新的站点:
```typescript
sites: [
  {
    url: 'https://blog.example.com/sitemap.xml',
    pathname: '/blog',
    interval: 86400000  // 每天最多爬一次
  }
]
```

### 3. 错误处理

```typescript
try {
  const snapshot = await getSnapshot(jobId);
  if (snapshot?.status === 'failed') {
    console.error('Crawl failed:', snapshot.error);
  }
} catch (error) {
  console.error('Error:', error);
}
```

### 4. 尊重 robots.txt

默认情况下会检查 robots.txt,除非设置 `ignoreRobots: true`。

## 性能优化

### 1. 复用浏览器实例

Crawler Engine 自动复用 Puppeteer 浏览器实例,无需手动管理。

### 2. 队列优化

- 使用 `replace: true` 避免重复快照
- 合理设置 `timeout` 和 `waitTime`
- 定时任务使用 `interval` 避免频繁爬取

### 3. 资源清理

定期清理旧的截图和 HTML 文件:
```typescript
import { deleteSnapshots } from '@arcblock/crawler';

await deleteSnapshots({
  url: 'https://example.com',
  replace: true
});
```

## 故障排查

### Puppeteer 启动失败

1. 检查 `PUPPETEER_EXECUTABLE_PATH` 是否正确
2. 确保 Chrome/Chromium 已安装
3. 查看日志中的详细错误信息

### 截图为空白

1. 增加 `waitTime` 等待页面加载
2. 检查目标网站是否可访问
3. 查看是否有 JavaScript 错误

### 数据库锁定错误

1. 检查是否有多个进程同时访问
2. 降低并发数
3. 重启应用

## 相关主题

- [核心模块](../modules.md) - 返回模块概述
- [SEO Middleware](middleware.md) - 了解 SEO 中间件
- [API 参考](../api-reference/crawler.md) - 完整的 API 文档
- [配置说明](../configuration.md) - 环境变量配置

## 下一步

- 了解 [SEO Middleware](middleware.md) 如何使用爬虫结果
- 查看 [API 参考](../api-reference/crawler.md) 获取完整 API 文档
- 阅读 [配置说明](../configuration.md) 进行高级配置
