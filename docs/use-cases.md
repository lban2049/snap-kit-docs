# 应用场景

本文档介绍 SnapKit 的实际应用场景和使用示例，帮助您了解如何在不同场景中使用 SnapKit。

## 场景 1: 网站监控

### 需求描述

监控竞争对手网站或自己的网站，自动捕获页面截图和内容变化。

### 解决方案

使用定时任务定期爬取目标网站：

```typescript
import { initCrawler } from '@arcblock/crawler';

await initCrawler({
  concurrency: 5,
  siteCron: {
    enabled: true,
    sites: [
      {
        // 监控竞争对手首页
        url: 'https://competitor.com/sitemap.xml',
        pathname: '/',
        interval: 86400000  // 每天爬一次
      }
    ],
    time: '0 2 * * *',  // 每天凌晨2点
    concurrency: 3
  }
});
```

### 数据获取

查询最新快照并对比变化：

```typescript
import { getLatestSnapshot } from '@arcblock/crawler';

const snapshot = await getLatestSnapshot('https://competitor.com');
if (snapshot) {
  // 保存截图用于对比
  console.log('Screenshot:', snapshot.screenshot);
  
  // 分析内容变化
  console.log('Title:', snapshot.meta?.title);
  console.log('HTML length:', snapshot.html?.length);
}
```

### 实际应用

- 价格监控：自动检测竞品价格变化
- 内容更新：监控博客和新闻更新
- 设计变化：记录 UI/UX 改版
- SEO 追踪：监控竞品 SEO 策略

## 场景 2: SEO 优化

### 需求描述

为 React/Vue 等 SPA 应用提供搜索引擎友好的预渲染 HTML。

### 解决方案

部署 SnapKit 和 SEO Middleware：

**1. 部署 SnapKit**:
```bash
docker run -p 3000:3000 arcblock/snap-kit
```

**2. 在 SPA 应用中集成中间件**:
```typescript
import express from 'express';
import { createSnapshotMiddleware } from '@arcblock/crawler-middleware';

const app = express();

// 创建中间件
const seoMiddleware = createSnapshotMiddleware({
  endpoint: 'https://snap.example.com',
  accessKey: process.env.SNAP_KIT_ACCESS_KEY,
  updateInterval: 86400000  // 24小时更新
});

// 为所有路由启用
app.use(seoMiddleware);

// SPA 路由
app.get('*', (req, res) => {
  res.sendFile('index.html');
});
```

**3. 预爬取重要页面**:
```typescript
// 在 SnapKit 中配置定时任务
siteCron: {
  enabled: true,
  sites: [{
    url: 'https://your-app.com/sitemap.xml',
    pathname: '/',
    interval: 86400000
  }],
  time: '0 3 * * *'
}
```

### 验证效果

使用搜索引擎爬虫 User Agent 测试：

```bash
# 模拟 Google Bot
curl -H "User-Agent: Mozilla/5.0 (compatible; Googlebot/2.1)" \
  https://your-app.com

# 检查响应是否包含完整 HTML
```

### 实际应用

- 提升 Google 搜索排名
- 支持社交媒体预览（Open Graph）
- 加快首次内容渲染
- 改善爬虫索引效率

## 场景 3: 数据分析

### 需求描述

从多个网站提取结构化数据用于商业分析。

### 解决方案

批量爬取并提取数据：

```typescript
import { crawlUrl, getSnapshot } from '@arcblock/crawler';

// 爬取多个 URL
const urls = [
  'https://site1.com/products',
  'https://site2.com/products',
  'https://site3.com/products'
];

const jobIds = await Promise.all(
  urls.map(url => crawlUrl({
    url,
    includeHtml: true
  }))
);

// 等待所有任务完成
await sleep(30000);

// 提取数据
for (const jobId of jobIds) {
  const snapshot = await getSnapshot(jobId);
  if (snapshot?.status === 'success') {
    // 解析 HTML 提取产品信息
    const products = parseProducts(snapshot.html);
    console.log('Products:', products);
  }
}
```

### 数据解析

使用 cheerio 解析 HTML：

```typescript
import cheerio from 'cheerio';

function parseProducts(html) {
  const $ = cheerio.load(html);
  const products = [];
  
  $('.product').each((i, el) => {
    products.push({
      name: $(el).find('.name').text(),
      price: $(el).find('.price').text(),
      image: $(el).find('img').attr('src')
    });
  });
  
  return products;
}
```

### 实际应用

- 价格比较平台
- 市场调研分析
- 产品目录聚合
- 竞品情报收集

## 场景 4: 视觉测试

### 需求描述

在 CI/CD 流程中进行自动化视觉回归测试。

### 解决方案

集成到测试流程：

```typescript
import { crawlUrl, getSnapshot } from '@arcblock/crawler';
import pixelmatch from 'pixelmatch';
import { PNG } from 'pngjs';
import fs from 'fs';

async function visualTest(url: string, baselinePath: string) {
  // 捕获当前截图
  const jobId = await crawlUrl({
    url,
    includeScreenshot: true,
    format: 'png',
    width: 1920,
    height: 1080,
    sync: true
  });
  
  const snapshot = await getSnapshot(jobId);
  
  // 与基准图对比
  const baseline = PNG.sync.read(fs.readFileSync(baselinePath));
  const current = PNG.sync.read(fs.readFileSync(snapshot.screenshot));
  
  const diff = new PNG({ width: baseline.width, height: baseline.height });
  const numDiffPixels = pixelmatch(
    baseline.data,
    current.data,
    diff.data,
    baseline.width,
    baseline.height,
    { threshold: 0.1 }
  );
  
  // 判断是否通过
  const threshold = baseline.width * baseline.height * 0.01; // 1%差异
  return numDiffPixels < threshold;
}

// 在 CI 中使用
const passed = await visualTest(
  'https://staging.example.com',
  './baseline.png'
);

if (!passed) {
  throw new Error('Visual regression detected!');
}
```

### 实际应用

- 前端自动化测试
- UI 组件库测试
- 响应式设计验证
- 跨浏览器兼容性测试

## 场景 5: 社交媒体

### 需求描述

自动生成社交媒体分享图片和预览。

### 解决方案

为每个页面生成 OG 图片：

```typescript
import { crawlUrl, getSnapshot } from '@arcblock/crawler';

async function generateOGImage(url: string) {
  const jobId = await crawlUrl({
    url,
    includeScreenshot: true,
    format: 'jpeg',
    quality: 90,
    width: 1200,
    height: 630,  // Twitter/Facebook 推荐尺寸
    sync: true
  });
  
  const snapshot = await getSnapshot(jobId);
  
  return {
    image: snapshot.screenshot,
    title: snapshot.meta?.title,
    description: snapshot.meta?.description
  };
}

// 生成博客文章的 OG 图片
const og = await generateOGImage('https://blog.example.com/post-1');

// 在 HTML meta 标签中使用
const metaTags = `
  <meta property="og:image" content="${og.image}" />
  <meta property="og:title" content="${og.title}" />
  <meta property="og:description" content="${og.description}" />
`;
```

### 代码截图

生成代码片段截图：

```typescript
import { crawlUrl, getSnapshot } from '@arcblock/crawler';

async function generateCodeImage(code: string, language: string) {
  const jobId = await crawlUrl({
    url: 'carbon',  // 特殊的代码截图端点
    code,
    format: 'png',
    t: 'one-dark',  // 主题
    l: language,    // 语言
    ln: 'true',     // 显示行号
    sync: true
  });
  
  const snapshot = await getSnapshot(jobId);
  return snapshot.screenshot;
}

// 生成 Python 代码截图
const image = await generateCodeImage(
  'def hello():\n    print("Hello World")',
  'python'
);
```

### 实际应用

- 博客文章分享图
- 技术文档代码示例
- 教程配图
- 社交媒体营销素材

## 场景 6: 内容归档

### 需求描述

定期归档网站内容，保存历史版本。

### 解决方案

定时爬取并保存：

```typescript
import { initCrawler } from '@arcblock/crawler';
import fs from 'fs-extra';

await initCrawler({
  siteCron: {
    enabled: true,
    sites: [{
      url: 'https://news.example.com/sitemap.xml',
      pathname: '/articles',
      interval: 86400000  // 每天归档一次
    }],
    time: '0 1 * * *',
    concurrency: 5
  }
});

// 定期导出归档
async function exportArchive() {
  const snapshots = await Snapshot.findAll({
    where: {
      createdAt: {
        [Op.gte]: new Date(Date.now() - 86400000)
      }
    }
  });
  
  // 保存到归档目录
  const archiveDir = `./archive/${new Date().toISOString().split('T')[0]}`;
  await fs.ensureDir(archiveDir);
  
  for (const snapshot of snapshots) {
    await fs.writeFile(
      `${archiveDir}/${md5(snapshot.url)}.json`,
      JSON.stringify(snapshot, null, 2)
    );
  }
}
```

### 实际应用

- 新闻网站内容归档
- 法律文件保存
- 学术研究资料
- 企业知识库

## 场景 7: API 文档截图

### 需求描述

自动为 API 文档生成截图用于展示。

### 解决方案

批量生成文档截图：

```typescript
async function generateAPIDocs(endpoints: string[]) {
  const results = [];
  
  for (const endpoint of endpoints) {
    const jobId = await crawlUrl({
      url: `https://docs.example.com/api/${endpoint}`,
      includeScreenshot: true,
      format: 'png',
      width: 1440,
      height: 900,
      sync: true
    });
    
    const snapshot = await getSnapshot(jobId);
    results.push({
      endpoint,
      screenshot: snapshot.screenshot,
      title: snapshot.meta?.title
    });
  }
  
  return results;
}

// 生成所有 API 端点的截图
const docs = await generateAPIDocs([
  'authentication',
  'users',
  'orders',
  'payments'
]);
```

### 实际应用

- API 文档配图
- 技术演示材料
- 培训教材
- 产品演示

## 最佳实践

### 1. 尊重 robots.txt

默认情况下 SnapKit 会检查 robots.txt：

```typescript
await crawlUrl({
  url: 'https://example.com',
  ignoreRobots: false  // 尊重 robots.txt
});
```

### 2. 控制爬取频率

使用 interval 避免过于频繁：

```typescript
{
  url: 'https://example.com/sitemap.xml',
  pathname: '/',
  interval: 3600000  // 最少1小时间隔
}
```

### 3. 错误处理

始终检查任务状态：

```typescript
const snapshot = await getSnapshot(jobId);
if (snapshot?.status === 'failed') {
  console.error('Failed:', snapshot.error);
  // 实现重试逻辑
}
```

### 4. 资源清理

定期清理旧数据：

```typescript
// 清理7天前的快照
const oldDate = new Date(Date.now() - 7 * 86400000);
await Snapshot.destroy({
  where: {
    createdAt: { [Op.lt]: oldDate }
  }
});
```

## 相关主题

- [API 参考](api-reference.md) - 查看完整 API
- [配置说明](configuration.md) - 配置定时任务
- [核心模块](modules.md) - 了解各个模块

## 下一步

- 查看 [开发指南](development.md) 学习本地开发
- 阅读 [API 参考](api-reference.md) 深入了解 API
