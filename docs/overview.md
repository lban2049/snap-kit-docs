# SnapKit 概述

SnapKit 是基于 Puppeteer 构建的 Web 自动化服务，提供 RESTful API 接口用于网页截图、内容抓取和预渲染。

![Snap Kit Logo](../sources/snap-kit/blocklets/snap-kit/logo.png)

## 使用场景

SnapKit 主要应用于以下技术场景：

- **自动化测试** - 在自动化测试流程中生成页面截图，用于视觉回归测试
- **内容聚合平台** - 从目标网站采集结构化数据，用于内容聚合和商业智能分析
- **SPA 应用的 SEO 优化** - 为单页应用生成预渲染的静态 HTML，便于搜索引擎爬取和索引

## 核心特性

- **一键部署** - 支持 Docker、Kubernetes 和 Blocklet Server 的零配置快速部署
- **企业级性能** - 智能队列系统，轻松处理上万个并发请求
- **高可用架构** - 内置健康检查和故障恢复机制
- **RESTful API** - 现代化的 TypeScript API，完整的文档和类型定义
- **自托管方案** - 无需按请求付费，完全掌控数据和服务

## 主要功能

### 核心能力

- **高保真截图** - 捕获像素级完美的网页快照
- **智能内容提取** - 使用高级解析提取结构化数据
- **批量处理** - 通过队列管理高效处理多个 URL
- **站点地图爬取** - 自动发现和爬取整个网站
- **SEO 预渲染** - 为 SPA 生成搜索引擎友好的 HTML
- **自定义请求头和 Cookie** - 完全控制请求定制

### 技术亮点

- **Puppeteer 集成** - 最新的 Chrome 自动化能力
- **SQLite 数据库** - 使用 Sequelize ORM 实现高效的数据存储
- **React 19.1 UI** - 现代化的响应式 Web 界面
- **Express 4.21 API** - 提供 TypeScript 支持的 RESTful 端点
- **Blocklet 平台** - 一键部署和扩展

## 项目组成

SnapKit 包含三个核心模块：

- **SnapKit Blocklet** - 主应用程序，包含 React 前端界面和 Express API 服务
- **Crawler Engine** - 基于 Puppeteer 的爬虫引擎，负责网页访问、截图和数据提取
- **SEO Middleware** - Express 中间件，为 SPA 应用提供预渲染支持

详细信息请参考[核心模块](modules.md)文档。

## 技术栈

- **前端**: React 19.1, TypeScript, Vite 7.0
- **后端**: Express 4.21, TypeScript, DID Auth
- **数据库**: SQLite with Sequelize ORM
- **自动化**: Puppeteer, @blocklet/puppeteer
- **部署**: Blocklet Platform, Docker

## 许可证

SnapKit 采用 MIT 许可证，可自由用于商业和个人项目。

## 相关主题

- [快速开始](getting-started.md) - 了解如何安装和部署 SnapKit
- [架构说明](architecture.md) - 深入了解项目架构和技术栈
- [核心模块](modules.md) - 探索三个核心模块的详细功能
- [API 参考](api-reference.md) - 查看完整的 API 文档

## 下一步

准备好开始使用 SnapKit 了吗？查看[快速开始](getting-started.md)指南，了解如何在几分钟内启动和运行 SnapKit。
