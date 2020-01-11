---
layout: post
title: jquery基础之事件详解
categories: [前端]
tags: [jquery]
date: 2020-1-19 13:48:07
comments: true
---


#### 事件绑定

事件绑定有两种方式，第一种是$.事件（）,第二种是$.on('click',function(){})，一般情况下我们推荐使用第一种，因为更简单方便。

区别：部分事件$.事件（）不能实现。

可以添加多个相同，不同事件，不会覆盖掉。

```html
$("tagName").click(function(){

        })
```

#### 事件移除

$("tagName").off() 不传递参数，可移除所有事件。

$("tagName").off(“click”) 传递一个参数，会移除所有指定类型事件。

$("tagName").off(“click”,test1) 传递两个参数，移除指定类型的指定事件。

#### 事件冒泡

事件就是从下面传递到上面就是事件冒泡。
reture false //阻止事件冒泡
event.stopPropagation(); //也可以阻止事件冒泡

```html
$(".son").click(function(event){
        alert("son")
        reture false //阻止事件冒泡
        event.stopPropagation();
        })
        
$(".father").click(function(){
        alert("father")
})
```

#### 阻止默认事件

在我们进行ajax提交表单的时候，需要组织默认的表单提交事件。
reture false //阻止默认事件
event.preventDefault(); //也可以阻止默认事件


```html
$("tagName").click(function(event){
        reture false //阻止事件冒泡
        event.preventDefault();
        alert("haha")
        })

```

#### 自动触发事件

$("tagName").trigger("事件名") //即可进入页面自动触发事件，会触发事件冒泡
$("tagName").triggerHandler("事件名") //即可进入页面自动触发事件，不会触发事件冒泡


$("input[type='submit']").trigger("事件名") //即可进入页面自动触发事件，会默认行为
$("input[type=''submit]").triggerHandler("事件名") //即可进入页面自动触发事件，不会触发默认行为

```html       
$("tagName").click(function(){
        alert("haha")
})

$("tagName").trigger("click") //即可进入页面自动触发事件
```

#### 事件委托

事件委托就是请别人帮忙做事情，然后将返回结果反馈给我们。

一般情况下，新增的dom元素是无法响应我们的事件的，因为入口函数里面的js代码会在dom元素加载完后执行，但是新增的dom元素却没有加载完毕，因为新增的元素是在dom加载完成后添加的，所以如果给新增的元素添加事件需要用到事件委托。

首现需要找到新增的dom元素的父级元素，因为父级元素是在入口函数加载完成后就有的，然后通过给父级元素添加点击委托事件，从而给新增的dom元素添加点击事件。

$("parentTagName").delegate("childrenTagName","click",function(){})

```html
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>我是新增的</li>
</ul>

$("ul").delegate("li","click",function(){
    console.log($(this).html())
    })
```

#### 移入移出事件

mouseover/mouseout子元素移入移出也会触发事件。

```html       
$("tagName").mouseover(function(){
        alert("移入事件")
})

$("tagName").mouseout(function(){
        alert("移出事件")
})

```

mouseenter/mouseleave子元素移入移出不会触发事件。

```html       
$("tagName").mouseenter(function(){
        alert("移入事件")
})

$("tagName").mouseleave(function(){
        alert("移出事件")
})

```

jQuery中还提供一个函数可以同时监听移入和移出事件。

$("tagName").hover(fn(){},fn(){})，接收两个参数，一个参数移入回调，一个参数是移出回调。

```html       
$("tagName").hover(function(){
        alert("移入事件")
},function(){
        alert("移出事件")
});
```

hover还可以只接受一个参数。

```html       
$("tagName").hover(function(){
        alert("移入事件")
        alert("移出事件")
});
```

#### index()  eq()  方法tab选项卡

index() 可以打印出来自己在兄弟元素中排第几
eq() 可以找到第几个兄弟元素





















