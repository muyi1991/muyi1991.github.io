---
layout: post
title: Jquery基础之基本使用
categories: [前端]
tags: [jquery]
date: 2020-1-16 13:48:07
comments: true
---


#### 概述

jquery是js的一个库，用jquery可以简化我们的js代码，让我们写的更少，做的更多。query是查询的意思，所以jquery主要是来查询js中的dom元素。要使用jquery首现需要下载，下载地址如下：

```html
https://code.jquery.com/jquery-3.3.1.js
```
改变后面3.3.1可以下载不同版本的jquery，1开头的版本支持ie6,7,8，2和3版本的不支持。同时可以把3.3.1.js改成3.3.1.min.js即可下载生产环境下的压缩jquery。改变版本号可以下载不同版本jquery。

按ctl+s即可保存jquery文件，保存之后就可以用来学习和生产环境的使用了。

#### 如何使用

首现将保存好的jquery文件在html页面中引用，然后在引入我们自己的js代码，然后就可以使用了。
注意：我们自己写的js代码必须放在引入的jquery文件后面，jquery必须要先引入，在前面。

#### jquery和js的区别

1. jquery会等到dom元素加载完后就执行，不会等到网络资源如网络请求的img的src加载完再执行，而js会等到网络资源加载完后执行；
2. js中的入口函数如果写了两个alert，那么后写的会覆盖先写的，而jquery则是先写的先执行，后写的后执行，不会被覆盖掉。


#### jquery入口函数的编写方式

方式一

```html
    jQuery(function () {
        alert('aaa')
    });
```

方式二(推荐)

```
$(function () {
    alert('aaa')
});
```

其他方式

```
$(document).ready(function () {
    alert('aaa')
});

jQuery(document).ready(function () {
    alert('aaa')
});
```

推荐使用第二种方式，因为更简洁方便。

#### jquery的冲突问题

有时候当我们用多个js框架是，别的框架中也使用到了$符号来写的更少，做的更多。那么这时候就需要来释放掉$的使用权。或者用上面的其他入口函数的方式。

使用“jQuery.noConflict()”可以释放掉$的使用权，然后使用jQuery代替。

```html
$(function () {
    jQuery.noConflict();
    alert('aa')
});

jQuery.noConflict();
jQuery(function () {
    alert('aa')
});
```

如果冲突了不想用jQuery这个来访问，那么可以自定义我们的jQuery访问符号，方法如下：

```html
var my = jQuery.noConflict();
my(function () {
    alert('aa')
});
```

















