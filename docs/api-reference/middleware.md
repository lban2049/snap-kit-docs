# Middleware API 参考

## 前置条件

阅读本文档前,建议先了解:
- [API 参考概述](../api-reference.md) - API 概览和通用说明
- [SEO Middleware](../modules/middleware.md) - SEO Middleware 模块介绍

## 安装

```bash
npm install @arcblock/crawler-middleware
```

## API

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

**参数详解**:

#### endpoint (必需)

- **类型**: `string`
- **说明**: SnapKit 端点 URL
- **示例**: `'https://snap.example.com'`

#### accessKey (必需)

- **类型**: `string`
- **说明**: SnapKit 访问密钥
- **获取方式**: 在 SnapKit Web 界面中生成

#### cacheMax (可选)

- **类型**: `number`
- **默认值**: `0` (无限制)
- **说明**: LRU 内存缓存最大条目数
- **建议值**:
  - 小型站点 (< 50 页): 50
  - 中型站点 (50-200 页): 100
  - 大型站点 (> 200 页): 200+

#### updateInterval (可选)

- **类型**: `number`
- **默认值**: `86400000` (24 小时)
- **说明**: 成功缓存的过期时间（毫秒）
- **建议值**:
  - 静态内容: 604800000 (7天)
  - 每日更新: 86400000 (1天)
  - 实时内容: 3600000 (1小时)

#### failedUpdateInterval (可选)

- **类型**: `number`
- **默认值**: `86400000` (24 小时)
- **说明**: 失败缓存的重试间隔（毫秒）
- **建议值**: 与 updateInterval 相同或稍短

#### updatedConcurrency (可选)

- **类型**: `number`
- **默认值**: `10`
- **说明**: 后台更新队列的并发数
- **建议值**:
  - 低流量站点: 5
  - 中流量站点: 10
  - 高流量站点: 20

#### autoReturnHtml (可选)

- **类型**: `boolean`
- **默认值**: `true`
- **说明**: 缓存命中时是否自动调用 `res.send(html)`
- **使用场景**:
  - `true`: 直接返回缓存（推荐）
  - `false`: 需要自定义响应头或处理

#### allowCrawler (可选)

- **类型**: `(req: Request) => boolean`
- **默认值**: `() => true` (允许所有请求)
- **说明**: 自定义函数，判断是否为请求返回缓存内容
- **参数**: Express Request 对象
- **返回值**: `true` 表示允许返回缓存

**返回值**: Express 中间件函数

## 使用示例

### 基本使用

最简单的配置：

```typescript
import express from 'express';
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const app = express();

const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY
});

app.use(middleware);
```

### 应用到特定路由

只为特定路由启用 SEO 优化：

```typescript
// 只为首页启用
app.use('/', middleware);

// 为文档路由启用
app.use('/docs', middleware);

// 为博客路由启用
app.use('/blog', middleware);
```

### 自定义缓存配置

根据站点特性配置缓存：

```typescript
const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  
  // 缓存最多 100 个页面
  cacheMax: 100,
  
  // 12 小时更新一次
  updateInterval: 43200000,
  
  // 失败后 6 小时重试
  failedUpdateInterval: 21600000,
  
  // 5 个并发更新
  updatedConcurrency: 5
});
```

### 自定义爬虫检测

只为特定路径启用 SEO：

```typescript
const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  
  allowCrawler: (req) => {
    // 只为以下路径启用
    const allowedPaths = ['/', '/docs', '/blog'];
    return allowedPaths.some(path => req.path.startsWith(path));
  }
});
```

### 排除特定路径

不为某些路径启用 SEO：

```typescript
const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  
  allowCrawler: (req) => {
    // 排除 API 和管理路由
    const excludedPaths = ['/api', '/admin', '/auth'];
    return !excludedPaths.some(path => req.path.startsWith(path));
  }
});
```

### 手动处理缓存

不自动返回，自定义响应：

```typescript
const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  autoReturnHtml: false  // 不自动返回
});

app.use(middleware, (req, res, next) => {
  // @ts-ignore
  if (req.cachedHtml) {
    // 添加自定义响应头
    res.setHeader('X-Cached', 'true');
    res.setHeader('X-Cache-Time', new Date().toISOString());
    
    // @ts-ignore
    if (req.cachedLastmod) {
      // @ts-ignore
      res.setHeader('Last-Modified', req.cachedLastmod);
    }
    
    // 返回缓存的 HTML
    // @ts-ignore
    res.send(req.cachedHtml);
  } else {
    // 返回原始响应
    next();
  }
});
```

### 条件性返回缓存

根据查询参数决定是否返回缓存：

```typescript
const middleware = createSnapshotMiddleware({
  endpoint: process.env.SNAP_KIT_ENDPOINT,
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  
  allowCrawler: (req) => {
    // 如果有 nocache 参数，不返回缓存
    if (req.query.nocache) {
      return false;
    }
    
    // 默认为爬虫返回缓存
    return true;
  }
});
```

## Request 对象扩展

当中间件检测到缓存时，会在 Request 对象上添加以下属性：

### req.cachedHtml

- **类型**: `string | undefined`
- **说明**: 缓存的 HTML 内容
- **示例**:
```typescript
app.use(middleware, (req, res) => {
  // @ts-ignore
  if (req.cachedHtml) {
    console.log('Using cached HTML');
  }
});
```

### req.cachedLastmod

- **类型**: `string | undefined`
- **说明**: 缓存的最后修改时间（UTC 字符串）
- **格式**: `'Tue, 30 Dec 2025 12:00:00 GMT'`
- **示例**:
```typescript
app.use(middleware, (req, res) => {
  // @ts-ignore
  if (req.cachedLastmod) {
    // @ts-ignore
    res.setHeader('Last-Modified', req.cachedLastmod);
  }
});
```

## 环境变量

在非 Blocklet 环境中使用时，需要配置以下环境变量：

### BLOCKLET_DATA_DIR (必需)

存储 SQLite 数据库的目录。

```bash
export BLOCKLET_DATA_DIR="/var/lib/app/data"
```

### BLOCKLET_LOG_DIR (必需)

存储日志文件的目录。

```bash
export BLOCKLET_LOG_DIR="/var/log/app"
```

### BLOCKLET_APP_URL (可选)

应用部署的域名。

```bash
export BLOCKLET_APP_URL="https://your-app.com"
```

## 缓存机制

### 缓存层级

```
请求 → 内存 LRU → SQLite → SnapKit API
         ↑         ↑          ↑
       最快      持久化      源数据
```

### 缓存命中流程

```
1. 检查内存 LRU 缓存
   ├─ 命中 → 返回
   └─ 未命中 → 下一步

2. 检查 SQLite 数据库
   ├─ 找到 → 加载到内存 → 返回
   └─ 未找到 → 下一步

3. 异步请求 SnapKit
   ├─ 成功 → 保存到数据库和内存
   └─ 失败 → 记录失败状态
```

### 缓存更新流程

```
1. 检查缓存是否过期
   ├─ 未过期 → 直接使用
   └─ 已过期 → 下一步

2. 加入更新队列（异步）
   
3. 当前请求返回旧缓存

4. 后台执行更新
   ├─ 请求 SnapKit API
   ├─ 更新 SQLite
   └─ 更新内存缓存

5. 下次请求命中新缓存
```

## 类型定义

### Middleware

```typescript
type Middleware = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<void>
```

### Request 扩展

```typescript
interface Request {
  cachedHtml?: string;
  cachedLastmod?: string;
}
```

## 最佳实践

### 1. 选择合适的缓存大小

```typescript
// 小型站点
cacheMax: 50

// 中型站点
cacheMax: 100

// 大型站点
cacheMax: 200
```

### 2. 根据更新频率配置过期时间

```typescript
// 静态内容（很少更新）
updateInterval: 7 * 24 * 60 * 60 * 1000  // 7天

// 每日更新
updateInterval: 24 * 60 * 60 * 1000      // 1天

// 频繁更新
updateInterval: 60 * 60 * 1000           // 1小时
```

### 3. 选择性启用

不要对所有路由启用：

```typescript
// ✅ 只为需要 SEO 的路由启用
app.use('/', middleware);
app.use('/docs', middleware);

// ❌ 不要为 API 路由启用
// app.use('/api', middleware);
```

### 4. 监控缓存效果

记录缓存命中率：

```typescript
let hits = 0;
let misses = 0;

app.use(middleware, (req, res, next) => {
  // @ts-ignore
  if (req.cachedHtml) {
    hits++;
    console.log(`Cache hit rate: ${(hits / (hits + misses) * 100).toFixed(2)}%`);
  } else {
    misses++;
  }
  next();
});
```

## 故障排查

### 缓存一直未命中

**可能原因**:
1. 页面未被 SnapKit 爬取
2. endpoint 或 accessKey 错误
3. allowCrawler 返回 false

**解决方法**:
1. 在 SnapKit 中手动爬取页面
2. 检查配置参数
3. 检查 allowCrawler 逻辑

### 返回旧内容

**可能原因**:
1. updateInterval 设置过长
2. 更新队列拥堵

**解决方法**:
1. 减小 updateInterval
2. 增加 updatedConcurrency
3. 手动清除数据库

### SQLite 错误

**可能原因**:
1. BLOCKLET_DATA_DIR 不存在或无权限
2. 磁盘空间不足

**解决方法**:
1. 检查目录权限
2. 确保磁盘空间充足
3. 检查 SQLite 数据库文件

## 相关主题

- [API 参考](../api-reference.md) - 返回 API 概述
- [SEO Middleware](../modules/middleware.md) - SEO Middleware 模块文档
- [配置说明](../configuration.md) - 环境变量配置

## 下一步

- 查看 [Crawler API](crawler.md) 了解爬虫 API
- 阅读 [应用场景](../use-cases.md) 查看实际示例
- 参考 [配置说明](../configuration.md) 进行高级配置
