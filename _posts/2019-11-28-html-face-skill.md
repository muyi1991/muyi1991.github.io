---
layout: post
title: js面向对象详细解
categories: [前端]
tags: [html]
date: 2019-11-28 13:48:07
comments: true
---


#### 步骤

1. 导入jqury文件
2. 理解概念
3. 固定模版


#### 导入jquery-3.3.1.min.js文件
在需要用到的html文件中导入jquery文件


#### 理解概念


```

//面向对象
//1,添加属性
//通过this关键字，绑定属性，并且指定它的值
//2,绑定方法
//在Banner.prototype上绑定方法就可以了

function Banner(){
    //这个里面写的代码
    //相当于是Python中的__init__方法的代码
    console.log('aaa');
    // 通过this关键字绑定属性
    this.person = 'zhiliao';
}

//原型链
Banner.prototype.greet = function (canshu) {
    console.log("hello" + canshu)
}

//创建一个Banner对象就会执行Banner里面的方法
var banner = new Banner();
console.log(banner.person);
banner.greet("zhiliao" )

```


#### 固定模版


```
function Banner() {

};

Banner.prototype.run = function(){
    //id选择器在jquery中用#号来选择
    var bannerul = $("#banner-ul")
    var index = 0;
    setInterval(function () {
        if (index >=3){
            index = 0;
        }else{
            index++;
        }
        bannerul.animate({"left":-798*index},
            500);
    },2000)
};


//$符号里面所有内容会在页面的所有元素都加载完成后执行
$(function () {
    var banner = new Banner();
    banner.run()
});
```

