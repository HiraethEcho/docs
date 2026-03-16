---
title: 我的网站计划
summary: 建站资料，技术和服务汇总
date: 2025-08-17
tags:
  - site
categories: plan
---

# 我的网站们

想要建站，各种方式建了很多站，试了很多技术功能。  
截至 2025-08-29前 汇总一下。

## 服务器上的

在starship上的服务器跑的服务，用zero trust-network-tunnel的方式绑定到877675.xyz域名下的子域名。

- [memos](https://memos.877675.xyz) 一个类似flomo的轻量级笔记服务
- [blinko](https://blinko.877675.xyz) 一个结合AI的memo服务
- ~~[copilot-api](https://copilot.877675.xyz)~~ 把github的copilot加上OpenAI的接口。找不到怎么加密，先关掉了。项目的[github](https://github.com/ericc-ch/copilot-api)有本地部署方法。
- [openlist](https://openlist.877675.xyz) 网盘合集
- [caldav](https://cal.877675.xyz) 用radicale做的日历和联系人服务

## 一些赛博善人

### netlify

- twikoo
- waline
- 代替 obsidian publish的 [quartz](https://quartz-hiraeth.netlify.app/)，要把`github-repo/content`作为obsidian repo，但是里面还有很多别的东西
- digitalgarden 同上，但是可以选择哪些发布，并且不用放在github仓库里，简单但同时自定义差一点。

### vercel

- memos.top 一个memos的展示[页面](https://memos-vercel.877675.xyz/)，需要有memos
- neno 类似flomo的[页面](https://neno.877675.xyz)，功能很少。
- digitalgarden

### claw-cloud

- 企图host一个[memos](https://rttwqxsfwixv.ap-northeast-1.clawcloudrun.com/)，但自定义域名未遂。

### github pages

- [生活](https://hiraethecho.github.io/hugo) [思考](https://hiraethecho.github.io/hexo) [知识](https://hiraethecho.github.io/wiki) 这三个博客暂时都是
- memos.top 同vercel的上，但是用github pages部署的[页面](https://hiraethecho.github.io/memospage)

## cloudflare 属于赛博菩萨了

### pages

- [flaredrive](https://flaredrive-asset.877675.xyz) 把桶存储变成webdav。目前是把图床和memos的桶放在这。

### workers

用cloudflare works创建的服务。

- [cloud mail](https://cloud-mail.877675.xyz) 用resend服务发邮件
  - admin@cloud-mail.877675.xyz
  - hiraeth@cloud-mail.877675.xyz
- [llm chat](https://cfchat.877675.xyz) 用cloudflare的ai

### mails

cloudflare自己转发的邮件服务。有点乱，看图吧。

![](https://asset.877675.xyz/202508171454341.webp)

![](https://asset.877675.xyz/202508171455969.webp)

### bucket

- 图床
- rclone连接，当备份的网盘。

## 为博客添加的功能

### 闲言碎语

- artitalk 一个说说或即刻页面，无后端，leancloud存储。可以用js插入到html里。Github pages上有一个[页面](https://hiraethecho.github.io/hexo/dynamic/moment)
- nonsense 无后端，leancloud存储。可以单独页面，也可以插入到博客里。这个写入有点费劲。
- memos 似乎也可以插入到博客里。[参考](https://memos.top/archives/50.html)

### 评论

- valine 无后端，leancloud存储
- waline 有后端，在netlify。leancloud存储。 [这里](https://hexowaline.netlify.app/.netlify/functions/comment)试用。
- twikoo 有后端，在netlify。MongDB存储。

### 图床

cloudflare的桶存储，和picgo上传

merge了picgo和memos的桶，把`pic`和`memos`的删掉，合并到`asset`，然后似乎出了点问题，openlist有点bug似乎。删掉重新添加就行，只改bucket不行，好神秘啊。  
以及`picgo`的url全错了，看不到预览，不好找图片了。
