---
layout: post
title: Jquery基础之ajax
categories: [前端]
tags: [jquery]
date: 2020-1-22 13:48:07
comments: true
---


#### 什么是ajax

ajax是一种不用刷新页面来更新页面部分数据的技术方式。与传统类似Django中的表单提交不同，可以实现更新部分页面内容。也就是需不需要全新加载页面。

#### json和js对象

什么是json，json是一个字符串，是一种数据格式。

```html
var obj = {a:'hello',b:'world'} //定义了js对象
var json = '{"a":"hello","b":"world"}'; //定义了一个json字符串
```

#### json和js互相转换

从json字符串转换成js对象，使用JSON.parse()方法

```html
var obj = JSON.parse('{"a":"hello","b":"world"}') //结果是 {a: 'Hello', b: 'World'}
```

要实现从js对象转换JSON字符串，使用JSON.stringfy()方法

```html
var json = JSON.stringify({a: 'Hello', b: 'World'}); //结果是 '{"a": "Hello", "b": "World"}'
```















 





















