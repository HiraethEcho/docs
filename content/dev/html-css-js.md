---
title: 网页三件套
date: 2025-10-19
summary: 一个网页需要的三件套的简单介绍
tags:
  - site
  - web
  - code
status: draft
draft: true
categories: 规范协议技术
ai: false
---

# 网页三件套

## Intro

众所周知，一个简单的网页通常需要三种语言：HTML、CSS 和 JavaScript。简而言之，

- HTML 声明网页内容的结构和语义，例如一坨是标题，那一坨是正文；
- CSS 为网页添加外观样式，如颜色大小间距等；
- JS 添加一些交互式功能，例如点某个按钮会发生什么。

一个综合的例子：

```html
<button id="myButton">点击</button>
```

表示这里有一个按钮；

```css
#myButton {
  background-color: blue;
  color: white;
  padding: 10px;
}
```

来设定按钮的背景颜色、文字颜色和内边距；

```js
document.getElementById("myButton").addEventListener("click", () => {
  if (document.body.className.includes("dark")) {
    document.body.classList.remove("dark");
  } else {
    document.body.classList.add("dark");
  }
});
```

点击按钮时切换网页的深色模式。

## html

### 标签

### layout

## css

### 元素属性

### layout

#### grid

#### flex

## js

还没学
