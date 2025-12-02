---
title: 薅羊毛cloudflare
date: 2025-07-20
tags:
  - site
  - web
  - hack
categories: handbook
status: in-progress
---

# cloudflare的功能

## 安全防护

Cloudflare 提供全球领先的安全解决方案，保护网站和用户免受攻击：

1. **Web 应用防火墙 (WAF)**：实时阻止恶意流量，防止常见攻击如 SQL 注入和跨站脚本攻击。
2. **DDoS 防护**：自动检测并缓解大规模分布式拒绝服务攻击。
3. **Bot 管理**：区分合法用户和恶意机器人，防止数据抓取或资源滥用。
4. **SSL/TLS 加密**：通过一键启用 HTTPS 提供网站加密，确保数据传输安全。

## 流量管理

Cloudflare 的流量管理服务帮助用户优化请求路径，提高访问速度：

1. **负载均衡**：分配流量到不同的服务器或数据中心，确保高可用性和可靠性。
2. **Argo 智能路由**：利用 Cloudflare 网络的实时数据，为请求选择最快的路径。
3. **内容分发网络 (CDN)**：缓存静态资源，提高全球访问速度并减少服务器负载。

## 分析与监控

Cloudflare 提供详细的分析工具，帮助用户了解网站流量和性能：

1. **实时分析**：显示实时流量、威胁拦截和性能数据。
2. **日志访问**：通过 API 提供详细的请求日志，支持高级分析和故障排查。
3. **用户洞察**：了解访客的地理位置、设备类型和行为数据。

## DNS

添加 DNS 记录是 Cloudflare 的核心功能之一，可以使其他人访问某个 URL 时解析到特定位置。

### 如何添加 DNS 记录

1. 登录到 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 选择需要管理的域名。
3. 导航到 **DNS** 页面。
4. 点击 **添加记录 (Add Record)** 按钮。
5. 填写以下信息：
   - **记录类型 (Type)**：如 A、CNAME、MX 等。
   - **名称 (Name)**：子域名或主域名。
   - **内容 (Content)**：目标地址，例如 IP 或 URL。
   - **TTL (生存时间)**：可选，通常默认即可。
6. 保存更改后，记录会立即生效。

### 支持的服务

- **Cloudflare 内服务**：如桶存储、Zero Trust Tunnel、Workers and Pages、Email 服务等，通常自动添加记录。
- **Cloudflare 外服务**：如服务器、Netlify、Vercel 等服务，用户可以手动配置。

通过精准和高效的 DNS 管理，Cloudflare 提升了域名解析的速度和可靠性。

## zero trust

Cloudflare Zero Trust 提供了一种现代化的网络安全解决方案，可以保护用户、设备和应用程序的安全访问。其主要特性包括：

1. **身份验证**：通过集成身份提供商（如 Okta、Google Workspace），可以确保只有授权用户才能访问资源。
2. **应用程序访问**：无需 VPN，用户可以通过 Cloudflare Access 安全地访问内部应用程序。
3. **威胁防护**：利用 DNS 过滤和浏览器隔离技术，阻止恶意软件、钓鱼攻击等威胁。
4. **设备管理**：通过 Cloudflare Gateway 支持设备的安全策略和流量监控。

### 使用场景

- **远程办公**：为分布式团队提供安全的资源访问。
- **零信任架构迁移**：逐步替代传统的网络边界安全模型。
- **第三方访问控制**：为合作伙伴或供应商提供安全的访问权限。

### 配置方法

1. 登录到 [Cloudflare Zero Trust Dashboard](https://dash.teams.cloudflare.com/)。
2. 设置身份验证规则，选择身份提供商。
3. 配置应用程序访问策略，指定允许访问的用户和条件。
4. 启用威胁防护功能，根据需要调整策略。

通过 Zero Trust，企业可以显著提升安全性，同时简化资源访问的流程。

## 存储

### Bucket

Cloudflare 提供了对象存储服务，用户可以通过 **R2 Bucket** 以低成本存储大量非结构化数据，兼容 S3 API，便于与其他工具整合。

**使用案例：**

- 为 Web 应用提供静态文件存储。
- 存储日志文件或大数据集。
- 备份和归档重要数据。

### KV

Cloudflare Key-Value（KV）存储是一种分布式的键值数据库，适用于缓存和快速读取小型数据。

**使用案例：**

- 缓存 API 响应数据以减少服务器负载。
- 存储用户偏好设置、会话信息等轻量级数据。

### D1

Cloudflare D1 是新推出的分布式 SQL 数据库，提供强大的查询功能和一致性，适合需要结构化数据存储的应用。

**使用案例：**

- 构建基于数据库的动态 Web 应用。
- 存储用户表单输入、订单信息等结构化数据。
- 实现实时分析和报告。

## 计算

### 网页 pages

Cloudflare Pages 是一种无服务器的平台，支持快速部署静态和动态网站。通过与 GitHub 集成，可以自动构建和部署网站。

部署步骤

1. 登录到 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 转到 **Pages** 部分，点击 **创建项目 (Create a Project)**。
3. 选择绑定的 GitHub 仓库，授权 Pages 访问代码。
4. 设置构建配置，包括：
   - **构建命令 (Build Command)**：如 `npm run build`。
   - **输出目录 (Output Directory)**：如 `dist` 文件夹。
5. 点击 **保存并部署 (Save and Deploy)**，系统将开始自动构建和部署。

使用场景

- **个人博客和作品集**：快速搭建和部署个人网站。
- **文档站点**：为开源项目或企业创建文档页面。
- **电商网站**：托管小型在线商店的前端页面。

通过 Pages，开发者可以专注于代码，而无需担心服务器配置和运维。

### workers

Cloudflare Workers 是一种无服务器的计算平台，可以让开发者在边缘运行代码，从而减少延迟并提升性能。

1. **性能优越**：运行代码的延迟极低，因为它直接在用户最近的边缘节点执行。
2. **语言灵活**：支持 JavaScript、TypeScript 和 WebAssembly。
3. **集成生态**：可以与 Cloudflare 的其他服务（如 KV 和 R2）无缝集成。

使用场景

- **API 网关**：快速处理 API 请求，并将其转发到后端服务。
- **动态内容生成**：根据用户请求生成个性化的网页。
- **安全增强**：在边缘执行请求过滤或内容安全策略。

以下是一个简单的示例，展示如何使用 Workers 来响应 HTTP 请求：

```javascript
addEventListener("fetch", (event) => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  return new Response("Hello, World!", {
    headers: { "content-type": "text/plain" },
  });
}
```

通过这个示例代码，开发者可以快速开始使用 Cloudflare Workers 构建和部署无服务器应用。

## AI
