---
layout: post
title: jquery基础之动效
categories: [前端]
tags: [jquery]
date: 2020-1-20 13:48:07
comments: true
---


#### 基本显示、隐藏动画

无动画效果的显示和隐藏
$("tagName").css('display','block') //显示
$("tagName").css('display','none') //隐藏

有动画效果的显示和隐藏
$("tagName").show(1000) //1000是毫秒参数
$("tagName").hide(1000) //1000是毫秒参数
$("tagName").toggle(1000) //切换隐藏和显示

show还可以接受一个回调参数，用来接受显示完毕后调用

```html
$("tagName").show(1000,function(){
        alert("动画显示完毕")
    })
```

#### 网页滚动动画

网页滚动到指定位置显示对联广告,回滚隐藏对联广告

```html
$(window).scroll(function(){
    var offset = $("html,body").scrollTop();
    if(offset >= 500){
        $("img").show(1000)
    }else{
        $("img").hide(1000)
    }
})
```

#### 展开收起动画

有动画效果的显示和隐藏
$("tagName").slideDown(1000) //1000是毫秒参数
$("tagName").slideUp(1000) //1000是毫秒参数
$("tagName").slideToggle(1000) //切换展开和收起

slideDown还可以接受一个回调参数，用来接受显示完毕后调用

```html
$("tagName").slideDown(1000,function(){
        alert("展开完毕")
    })
```

#### 停止动画 对话队列

在执行动画之前，我们可以先调用stop()，清除掉动画队列。

如果有一个动画移入，移除都需要一秒执行，我们在很短时间做了大量鼠标移入，移出操作，会导致一个动画队列，不停的做动画。$("tagName").stop();

```html
$(".nav>li").mouseenter(function(){
    $(this).stop(); //停止掉当前动画
    $(this).slideDown(1000)
});

$(".nav>li").mouseleave(function(){
    $(this).stop(); //停止掉当前动画
    $(this).slideUp(1000)
});
```

#### 淡入淡出动画

有动画效果的显示和隐藏
$("tagName").fadeIn(1000) //1000是毫秒参数
$("tagName").fadeOut(1000) //1000是毫秒参数
$("tagName").fadeToggle(1000) //切换淡入和淡出

fadeIn还可以接受一个回调参数，用来接受显示完毕后调用

```html
$("tagName").slideDown(1000,function(){
        alert("展开完毕")
    })
```

#### 自定义动画

$("tagName").animate({},1000,function(){})

可以接受四个参数
第一个参数：接受一个对象，可以在对象中修改属性
第二个参数：指定动画时长
第三个参数：指定动画节奏，默认就是swing
第四个参数：动画执行完毕后的回调函数

```html
$("tagName").animate({
    width:500,
    heigth:500
    },1000,function(){
    console.log("自定义动画执行完毕")
})
```

#### stop() delay() 方法

delay()方法是用来链式调用时延迟动画时间的

```html
$("tagName").animate({width:500},1000).delay(2000).animate({height:500},1000)
```

执行完宽度动画 等两秒 再执行高度动画

stop(true or false,true or false)  可以接受两个参数，两个参数都是布尔类型，true或者false


#### 定时器

var timer = setInterval(function(){},100) //设置定时器

clearInterval(timer) //关闭定时器





 





















