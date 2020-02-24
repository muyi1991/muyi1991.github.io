---
layout: post
title: html一些效果样式css小技巧
categories: [前端]
tags: [html]
date: 2019-11-27 13:48:07
comments: true
---


#### 固定一个元素的位置

固定在顶部
```css
xx{
    position:fiexed;
    top:0;
    left:0;
    right:0;
    z-index:100;
}
```
固定后xx元素就浮动起来了，该元素下面的元素如果想和该元素保持一定距离，需要加上margin-top值。


