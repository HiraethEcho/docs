---
title: 用obsidian管理多个博客
date: 2025-04-05
summary: 之前在一个obsidian仓库管理hexo hugo mkdocs三个网站的方法
tags:
    - obsidian
    - github
    - geek
categories: handbook
---

# 用obsidian管理多个博客

一些花里胡哨的技巧。

## 目标

多个博客网站，都用github pages功能发布。多个博客的post文件放在一个obsidian仓库中管理，并且本身作为一个github repository。push 这个obsidian/github仓库后将博客内容同步到其他博客repo。各个repo自己发布。

## 思路

用github action，push之后检测变化的文件，clone对应仓库，同步文件，再分别push。

需要注意的是，GitHub action对各个repo的读写权限，需要配置各种keys。  
先生成ssh keys，将pub放入博客repo的deploy key，记得打开写权限；private key放入obsidian repo的secrets中。  
每个博客仓库都需要一个密钥对。

## 示例

例如将`obsidian/hexo`的内容同步到`hexo`框架下对应的`hexo/source/_posts`文件夹下，`/.github/workflows/sync_hexo`中

```yml
name: Sync ob/hexo to hexo/source_posts

on:
    push:
        paths:
            - "hexo/**" # 监听文件夹内的文件变化，没有变化不会触发action
jobs:
    sync:
        runs-on: ubuntu-latest
        steps:
            # 检出 obsidian 仓库的代码
            - name: Checkout blogs repository
              uses: actions/checkout@v3
              with:
                  repository: username/obsidian_repo
                  path: obsidian
            - name: commit
              run: |
                  cd obsidian
                  git log -1 --pretty=%B > ~/commit

            # 设置 Git 配置（用户名和ssh私钥）
            - name: Set up Git
              env:
                  ACTIONS_KEY: ${{ secrets.OB_PRI }}
              run: |
                  mkdir -p ~/.ssh/
                  echo "$ACTIONS_KEY" > ~/.ssh/id_rsa
                  chmod 700 ~/.ssh
                  chmod 600 ~/.ssh/id_rsa
                  ssh-keyscan github.com >> ~/.ssh/known_hosts
                  git config --global user.name "action"
                  git config --global user.email "email"
                  git config --global core.quotepath false
                  git config --global i18n.commitEncoding utf-8
                  git config --global i18n.logOutputEncoding utf-8

            - name: clone
              run: |
                  git clone git@github.com:username/hexo_repo ~/hexo

            - name: Sync files from obsidian/hexo to hexo/source/_posts
              run: |
                  rsync -av obsidian/hexo/ ~/hexo/source/_posts/
            - name: Commit and push changes to HexoBlog repository
              run: |
                  cd ~/hexo
                  git add .
                  git commit -m "Sync obsidian to hexo $(cat ~/commit)"
                  git push origin main
```

注意其中 `username` `hexo_repo` `obsidian_repo` `secrets.OB_PRI`等根据情况改写。
