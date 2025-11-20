---
title: 个人网站的组织和管理计划
date: 2025-08-20
tags:
  - site
  - hugo
  - cloudflare
categories: plan
draft: true
---

# 个人网站的组织和管理计划

## 域名

### 买域名

### 在域名下的内容组织

- example.com
- blog.example.com
  - blog.example.com/diary
  - blog.example.com/posts
- math.example.com
  - math.example.com/notes
  - math.example.com/profile
- dev.example.com
  - dev.example.com/docs
  - dev.example.com/projects
  - dev.example.com/kanban,todo
  - rss.example.com
  - git.example.com
- servers.example.com
  - links.example.com
  - imap.example.com
  - smtp.example.com
  - caldav.example.com
  - carddav.example.com
  - mail.example.com
- clouds.example.com
  - clouds.example.com/openlist
  - bucket-name.example.com
  - another-bucket-name.example.com
- gallery.example.com
  - gallery.example.com/art
  - gallery.example.com/book
  - gallery.example.com/music
  - gallery.example.com/film
  - gallery.example.com/poem
  - gallery.example.com/meme
  - gallery.example.com/photo

### 托管域名

dns记录

重定向

## 个人网站内容

### 主页

### blog

### 存储

### 各类服务

## 服务的部署

### github

### cloudflare

### netlify

### vercel

### leancloud

## deepseek建议

### 整体架构设计

#### 主域名与子域名分配

- **主域名**: yourdomain.com (作为门户/重定向页)
- **邮箱服务**: mail.yourdomain.com
- **个人博客**: blog.yourdomain.com
- **学习笔记**: notes.yourdomain.com
- **资源展示**: library.yourdomain.com
- **个人主页**: cv.yourdomain.com
- **技术文档**: docs.yourdomain.com
- **云同步服务**: cloud.yourdomain.com

### 各版块内容组织与工具推荐

#### 1. 邮箱服务 (mail.yourdomain.com)

**包含内容**:

- 个人域名邮箱（如: name@yourdomain.com）
- IMAP/SMTP邮件服务
- CalDAV日历同步
- CardDAV联系人同步
- Webmail网页界面

**推荐工具**:

- **Mailcow**: 功能完整的邮件服务器套件
- **Mail-in-a-Box**: 一体化自建邮件解决方案
- **MXRoute**: 专业邮件托管服务（免自维护）

#### 2. 个人博客 (blog.yourdomain.com)

**包含内容**:

- 日记与个人生活记录
- 文化批判与社会思考
- 日常感想与哲学思考
- 按时间顺序排列的文章流
- 分类和标签系统

**推荐工具**:

- **Hugo**: 快速静态网站生成器，适合技术型用户
- **Gatsby**: React基础的静态站点生成器
- **WordPress**: 传统动态博客系统（需数据库）
- **Ghost**: 专注于内容的现代发布平台

#### 3. 学习笔记 (notes.yourdomain.com)

**包含内容**:

- 数学研究笔记（作为博士生）
- 读书笔记与书摘
- 学术论文摘要
- 课程学习材料
- 知识管理系统

**推荐工具**:

- **Obsidian Publish**: 本地笔记的发布服务
- **Jekyll**: 支持Markdown的静态站点
- **Logseq**: 大纲式笔记发布功能
- **Wiki.js**: 知识库系统，支持协作

#### 4. 资源展示墙 (library.yourdomain.com)

**包含内容**:

- 书籍收藏与评论
- 电影观看记录与评分
- 音乐专辑推荐
- 精彩摘抄集合
- 艺术作品展示

**推荐工具**:

- **Static CMS**: 无头CMS+GitHub工作流
- **Front Matter**: VS Code扩展+静态站点
- **Tina CMS**: 可视化编辑的静态CMS
- **自制HTML/CSS**: 完全自定义展示页面

#### 5. 个人主页 (cv.yourdomain.com)

**包含内容**:

- 学术简历（CV）
- 教育背景与学历证明
- 数学研究方向介绍
- 出版物列表
- 项目经历展示
- 联系方式

**推荐工具**:

- **HTML/CSS/JS**: 手工编写，完全控制
- **React/Vue**: 现代前端框架
- **JSON Resume**: 基于JSON的标准简历格式
- **TeX-based**: 学术风格的简历模板

#### 6. 技术文档 (docs.yourdomain.com)

**包含内容**:

- Linux使用指南
- 软件开发笔记
- 网站建设教程
- 数学工具使用文档
- 技术问题解决方案

**推荐工具**:

- **Docsify**: 动态文档生成器
- **Docusaurus**: React文档网站框架
- **MkDocs**: Python驱动的文档站点
- **GitBook**: 专业的文档平台

#### 7. 云同步服务 (cloud.yourdomain.com)

**包含内容**:

- WebDAV文件同步
- CalDAV日历同步
- CardDAV联系人同步
- 文件存储与分享

**推荐工具**:

- **Nextcloud**: 全功能自托管云平台
- **OwnCloud**: Nextcloud的前身
- **Syncthing**: 去中心化文件同步（无需WebDAV）

### 技术栈整合方案

#### 静态内容托管

- **GitHub Pages**: 免费、简单的静态托管
- **Cloudflare Pages**: 更快的全球分发
- **Netlify**: 优秀的开发者体验

#### 动态服务暴露

- **Cloudflare Tunnel**: 安全暴露本地服务，无需公网IP
- **Cloudflare Zero Trust**: 添加额外安全层
- **Cloudflare Workers**: 处理轻量级API需求

#### 部署策略

1. **静态站点**: GitHub Pages + Cloudflare CDN
2. **动态服务**: 自建服务器 + Cloudflare Tunnel
3. **邮件服务**: 专用VPS + 专业邮件软件
4. **文件同步**: 家庭服务器 + Cloudflare Tunnel暴露

## resource

- [latexcss](https://github.com/davidrzs/latexcss)
- [pages cms](https://pagescms.org/)
