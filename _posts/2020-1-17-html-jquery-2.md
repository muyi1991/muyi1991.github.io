---
layout: post
title: jquery基础之each,map等静态方法
categories: [前端]
tags: [jquery]
date: 2020-1-17 13:48:07
comments: true
---


#### 概述

$有两种用法,一是在页面加载结束后执行一个函数，二是查找一个dom元素。

在jquery中$()这样一般有两种用法，第一接收一个函数，第二查找某个dom对象。一般情况下就是这两种用法，接下来我们看下两种用法。

利用$查找出来的dom元素本质上是一个伪数组，即：有0到length-1个下标，有length属性。

```html

$(function(){
    alert('aaa');
})

var box =$("box")[0];//查找标签为box的dom元素
var box1 = $(".box1");//查找class的dom元素
var box2 = $("#box");//查找id的dom元素

```

#### $.each(a,fn(i,v){}) 方法

伪数组是有0到length-1下标，并且有length属性，就是伪数组。jQuery可以遍历伪数组和真数组，但是js中的foreach不能便利伪数组。

```html
$(function () {
    arr = [1,3,5,7,9];//真数组
    obj = {0:1, 1:3, 2:5, 3:7, 4:9, length:5}//伪数组
    $.each(arr,function (index, value) {
        console.log(index,value);

    });
    $.each(obj,function (index,value) {
        console.log(index,value);
    })
});
```

jQuery中遍历的伪数组不会把length属性遍历出来。

#### $.map(a,fn(i,v){}) 方法


```html
$(function () {
    arr = [1,3,5,7,9];//真数组
    obj = {0:1, 1:3, 2:5, 3:7, 4:9, length:5}//伪数组
    $.map(arr,function (index, value) {
        console.log(index,value);

    });
    $.map(obj,function (index,value) {
        console.log(index,value);
    })
});
```

注意：map方法和each方法都是传递两个参数：
1. 遍历的数据
2. 回调函数

回调函数中each和map有轻微区别，each遍历出来的是index，value。map遍历出来的是value，index。

#### each和map区别

区别一：

each方法默认是遍历谁就返回谁
map方法默认是返回一个空数组

区别二：

each方法不支持在回调函数中对遍历的数组进行处理
map方法可以在回调函数中通过return对遍历的数组进行处理，然后生成一个新的数组返回

```html
var res = $.map(obj,function (index,value) {
        // console.log(index,value);
        return index+value
    });
    console.log(res);
```
返回值是一个新的数组。


#### $.trim() 

trim方法主要作用是去除字符串左右的空格。

```html
$(function () {
    var str = '   aaa   ';
    console.log('--'+str+'--');
    res = $.trim(str);
    console.log('--'+res+'--');

});
```

#### $.isWindow

判断传入的对象是否是window对象。

```html
var arr =[1,2,3,4];
var w = window;

console.log($.isWindow(arr))  //false
console.log($.isWindow(w)) //true
```

#### $.isArray


```html
$(function () {
    arr = [1,3,5,7,9];//真数组
    obj = {0:1, 1:3, 2:5, 3:7, 4:9, length:5}//伪数组

    console.log($.isArray(arr)) //true
    console.log($.isArray(obj)) //false
});
```


#### $.isFunction()


```html
$(function () {
    arr = [1,3,5,7,9];//真数组
    obj = {0:1, 1:3, 2:5, 3:7, 4:9, length:5}//伪数组
    fn = function(){};
    
    console.log($.isFunction(obj)); //false
    console.log($.isFunction(fn)); //true

});
```

$.isFunction(jQuery) 返回值是true，通过这我门了解到jQuery本质是一个匿名函数。

(function(window,undefined){
    
})(window);


















