---
layout: post
title: jquery基础之节点操作
categories: [前端]
tags: [jquery]
date: 2020-1-21 13:48:07
comments: true
---


#### 添加节点

$("tagName").append("ele")  //追加ele元素至“tagName”后面
$("tagName").prepend("ele")  //添加ele元素至“tagName”前面

$("tagName").after("ele")  //追加ele元素至“tagName”外部的后面
$("tagName").before("ele")  //追加ele元素至“tagName”外部的前面

#### 删除节点

$("tagName").remove() //删除找到的指定元素
$("tagName").empty() //清空"tagName"内部元素，包括文本和标签,tagName本身不会删除

#### 替换节点

$("tagName").replaceWith(ele) //用ele替换所有找到的tagName节点

#### 复制节点

$("tagName").clone(false or true) //浅复制，深复制节点

```html
var li = $("li:first").clone(false); //浅复制
$("ul").append(li)

```
```html
var li = $("li:first").clone(true); //深复制
$("ul").append(li)

```

浅复制和深复制区别：浅复制仅仅复制元素，深复制不光复制元素还能复制元素的相关事件。

#### 时间相关


```html
var date =new Date(); //创建时间
date.getFullYear(); //获取年
date.getMonth() + 1; //获取月
date.getDate(); //获取年

date.getHours(); //获取年
date.getMinutes(); //获取年
date.getSeconds(); //获取年
```








 





















