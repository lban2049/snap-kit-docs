# SEO Middleware

## 前置条件

阅读本文档前，建议先了解：
- [核心模块概述](../modules.md) - 了解三个核心模块的关系
- [Crawler Engine](crawler.md) - 中间件依赖爬虫引擎

## 概述

SEO Middleware (`@arcblock/crawler-middleware`) 是一个 Express 中间件，为 Blocklet 提供 Snap Kit 生成的预渲染 HTML，确保搜索引擎能够正确索引动态生成的内容。

## 工作原理

中间件采用智能缓存策略，为搜索引擎爬虫提供预渲染的 HTML：

```
1. 拦截请求
2. 检测是否为搜索引擎爬虫
3. 尝试从缓存读取 HTML（内存 LRU + SQLite）
4. 缓存命中 → 直接返回
5. 缓存未命中 → 异步请求 Snap Kit
6. 更新本地缓存
7. 下次访问直接命中缓存
```

## 核心功能

### 1. 智能爬虫检测

自动识别搜索引擎爬虫：
- Google Bot
- Bing Bot
- Baidu Spider
- 其他主流搜索引擎

### 2. 多层缓存

- **内存 LRU 缓存**：快速访问，减少数据库查询
- **SQLite 持久化**：重启后保留缓存
- **自动过期**：定期更新确保内容新鲜

### 3. 异步更新

缓存过期时异步请求 Snap Kit 更新：
- 当前请求不等待更新
- 后台异步更新缓存
- 下次请求命中新缓存

## API 参考

### createSnapshotMiddleware

创建 SEO 中间件实例。

**函数签名**:
```typescript
function createSnapshotMiddleware(options: {
  endpoint: string;
  accessKey: string;
  cacheMax?: number;
  updateInterval?: number;
  failedUpdateInterval?: number;
  updatedConcurrency?: number;
  autoReturnHtml?: boolean;
  allowCrawler?: (req: Request) => boolean;
}): Middleware
```

**参数说明**:

- `endpoint` (必需)
  - 类型: `string`
  - 说明: Snap Kit 端点 URL
  - 示例: `'https://snap.example.com'`

- `accessKey` (必需)
  - 类型: `string`
  - 说明: Snap Kit 访问密钥

- `cacheMax` (可选)
  - 类型: `number`
  - 默认值: `0`（无限制）
  - 说明: LRU 缓存最大条目数

- `updateInterval` (可选)
  - 类型: `number`
  - 默认值: `86400000` (24小时)
  - 说明: 缓存过期时间（毫秒）

- `failedUpdateInterval` (可选)
  - 类型: `number`
  - 默认值: `86400000` (24小时)
  - 说明: 失败缓存重试间隔（毫秒）

- `updatedConcurrency` (可选)
  - 类型: `number`
  - 默认值: `10`
  - 说明: 更新队列并发数

- `autoReturnHtml` (可选)
  - 类型: `boolean`
  - 默认值: `true`
  - 说明: 缓存命中时自动调用 `res.send(html)`

- `allowCrawler` (可选)
  - 类型: `(req: Request) => boolean`
  - 默认值: `() => true`
  - 说明: 自定义函数，判断是否返回缓存内容

## 使用示例

### 基本使用

```typescript
import express from 'express';
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const app = express();

const snapshotMiddleware = createSnapshotMiddleware({
  endpoint: 'https://snap.example.com',
  accessKey: 'your-access-key'
});

// 应用到所有路由
app.use(snapshotMiddleware);

app.get('/', (req, res) => {
  res.send('<html>...</html>');
});
```

### 应用到特定路由

```typescript
// 只为首页启用
app.use('/', snapshotMiddleware, (req, res) => {
  res.send('<html>...</html>');
});

// 为文档路由启用
app.use('/docs', snapshotMiddleware, (req, res) => {
  res.send('<html>...</html>');
});
```

### 自定义爬虫检测

```typescript
const snapshotMiddleware = createSnapshotMiddleware({
  endpoint: 'https://snap.example.com',
  accessKey: 'your-access-key',
  allowCrawler: (req) => {
    // 只为首页和文档页面启用
    return req.path === '/' || req.path.startsWith('/docs');
  }
});
```

### 手动处理缓存

```typescript
const snapshotMiddleware = createSnapshotMiddleware({
  endpoint: 'https://snap.example.com',
  accessKey: 'your-access-key',
  autoReturnHtml: false  // 不自动返回
});

app.use(snapshotMiddleware, (req, res) => {
  // @ts-ignore
  if (req.cachedHtml) {
    // 添加自定义响应头
    res.setHeader('X-Cached', 'true');
    // @ts-ignore
    res.send(req.cachedHtml);
  } else {
    res.send('<html>...</html>');
  }
});
```

## 配置选项

### 环境变量

在非 Blocklet 环境中使用时，需要配置：

#### BLOCKLET_DATA_DIR (必需)

存储 SQLite 数据库的目录。

```bash
export BLOCKLET_DATA_DIR="/path/to/data"
```

#### BLOCKLET_LOG_DIR (必需)

存储日志文件的目录。

```bash
export BLOCKLET_LOG_DIR="/path/to/logs"
```

#### BLOCKLET_APP_URL (可选)

部署的域名。

```bash
export BLOCKLET_APP_URL="https://your-app.com"
```

## 缓存策略

### 缓存层级

```
┌─────────────────────┐
│  1. 内存 LRU 缓存   │  ← 最快
├─────────────────────┤
│  2. SQLite 数据库   │  ← 持久化
├─────────────────────┤
│  3. Snap Kit API    │  ← 源数据
└─────────────────────┘
```

### 缓存流程

1. **检查内存缓存**
   - 命中 → 返回
   - 未命中 → 下一步

2. **检查 SQLite**
   - 有记录 → 加载到内存 → 返回
   - 无记录 → 下一步

3. **请求 Snap Kit**
   - 异步请求
   - 更新数据库
   - 更新内存缓存

### 缓存更新

缓存过期时自动更新：

```typescript
// 成功的缓存：24小时后过期
updateInterval: 86400000

// 失败的缓存：24小时后重试
failedUpdateInterval: 86400000
```

## 验证方法

### 1. 修改浏览器 User Agent

将浏览器的 User Agent 设置为包含 "spider" 的字符串。

**Chrome DevTools**:
1. 打开 DevTools (F12)
2. 点击右上角三个点 → More tools → Network conditions
3. User agent 取消勾选 "Use browser default"
4. 输入: `Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)`

### 2. 首次访问（缓存未命中）

访问已被 Snap Kit 爬取的页面：
- 服务器日志显示 "Cache miss"
- 后台发送请求到 Snap Kit
- 返回原始 HTML

### 3. 第二次访问（缓存命中）

稍等片刻后再次访问：
- 服务器日志显示 "Cache hit"
- 返回的 HTML 包含 `<meta name="arcblock-crawler" content="true">`
- 响应头包含 `Last-Modified`

## 数据存储

### SQLite 数据库

中间件会在 `BLOCKLET_DATA_DIR` 创建 SQLite 数据库：

```
${BLOCKLET_DATA_DIR}/crawler-middleware.db
```

数据表结构：
- `url`: 页面 URL
- `html`: 缓存的 HTML
- `lastModified`: 最后修改时间
- `createdAt`: 创建时间
- `status`: 状态 (success/failed)

### 清理缓存

手动删除数据库文件可清空所有缓存：

```bash
rm ${BLOCKLET_DATA_DIR}/crawler-middleware.db
```

## 性能优化

### 1. 合理配置缓存大小

根据页面数量配置 LRU 缓存：

```typescript
createSnapshotMiddleware({
  endpoint: '...',
  accessKey: '...',
  cacheMax: 100  // 最多缓存100个页面
});
```

### 2. 调整更新间隔

根据内容更新频率配置：

```typescript
createSnapshotMiddleware({
  endpoint: '...',
  accessKey: '...',
  updateInterval: 3600000  // 1小时更新一次
});
```

### 3. 控制并发数

```typescript
createSnapshotMiddleware({
  endpoint: '...',
  accessKey: '...',
  updatedConcurrency: 5  // 最多5个并发更新
});
```

## 故障排查

### 缓存一直未命中

1. 检查页面是否已被 Snap Kit 爬取
2. 验证 endpoint 和 accessKey 是否正确
3. 查看日志中是否有错误信息

### 返回旧内容

1. 检查 `updateInterval` 设置
2. 手动清除缓存数据库
3. 在 Snap Kit 中重新爬取页面

### SQLite 错误

1. 检查 `BLOCKLET_DATA_DIR` 权限
2. 确保目录存在且可写
3. 检查磁盘空间

## 最佳实践

### 1. 选择性启用

不要对所有路由启用，只为需要 SEO 的页面启用：

```typescript
app.use('/', snapshotMiddleware);  // ✅ 首页
app.use('/docs', snapshotMiddleware);  // ✅ 文档
// ❌ 不对 /api 启用
```

### 2. 配置合理的更新间隔

根据内容更新频率：
- 静态内容：7天
- 每日更新：1天
- 实时内容：1小时

### 3. 监控缓存命中率

记录日志统计缓存效果：
```typescript
let hits = 0, misses = 0;
// 在日志中统计 "Cache hit" 和 "Cache miss"
```

### 4. 预先爬取重要页面

使用 Snap Kit 的定时任务预先爬取：
```typescript
// 在 Snap Kit 中配置
siteCron: {
  enabled: true,
  sites: [{
    url: 'https://your-app.com/sitemap.xml',
    pathname: '/'
  }]
}
```

## 相关主题

- [核心模块](../modules.md) - 返回模块概述
- [Crawler Engine](crawler.md) - 了解爬虫引擎
- [API 参考](../api-reference/middleware.md) - 完整的 API 文档
- [配置说明](../configuration.md) - 环境变量配置

## 下一步

- 查看 [API 参考](../api-reference/middleware.md) 获取完整文档
- 了解 [Crawler Engine](crawler.md) 如何生成快照
- 阅读 [配置说明](../configuration.md) 优化性能
