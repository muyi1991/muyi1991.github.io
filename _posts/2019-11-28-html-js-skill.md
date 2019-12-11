---
layout: post
title: js一些使用小技巧
categories: [前端]
tags: [html,js]
date: 2019-11-28 15:48:07
comments: true
---


#### js中获取元素和改变元素的样式、文字

获取元素

```
var smsCapcha = $('#getcapcha-btn'); //id选择器用#
var telephoneInput = $("input[name='telephone']"); //input选择器用name
var telephone = telephoneInput.val();
```

改变元素样式，文字

```
self.smsCapcha.addClass('disabled'); //添加css样式，设置禁用掉
self.smsCapcha.removeClass('disabled'); //移除css样式，设置启用
self.smsCapcha.text('发送验证码'); //设置元素上的文字
self.smsCapcha.text(count+'S'); //设置元素上的文字
self.smsCapcha.unbind('click'); //设置不能点击，unbind，把点击关掉
```

查找元素

```
parent(selector)	查找父元素，可传入selector进行过滤(下同)
parents(selector)	查找所有的祖先节点
children(selector)	返回所有的子节点，不过该方法只会返回直接的子节点，不会返回所有的子孙节点
prev()	返回该节点的上一个兄弟节点
prevAll()	返回该节点之前所有的节点
next()	返回该节点的下一个兄弟节点
nextAll()	返回该节点之后所有的节点
siblings()	返回该节点所有的兄弟节点，不分前后
find(selector)	返回该节点所有的子孙节点
```

添加属性

```
  //给元素添加属性
  $node.addClass('active'); //给元素添加激活事件
  $node.removeClass('active');//移除某元素的激活事件
  
  //显示隐藏元素
  $node.show();
  $node.hide();
  
  //获取元素 $node 的属性：id、src、title，修改以上属性
  $node.attr('id);
  $node.attr('src);
  $node.attr('title1', 'title2');
  
  // 给 $node 添加自定义属性 data-src
  $('.node').attr('data-src', 'xxx');
  
  //给元素头部尾部添加节点
  $ct.prepend($node);
  $ct.append($node);
  
  //删除某个元素
  $node.remove();
  
  //获取、设置 $node 的宽度、高度（分别不包括内边距、包括内边距、包括边框、包括外边距）
  $node.width()  // content
  $node.height()
  $node.innerWidth()  // content + padding
  $node.innerHeight()
  $node.outerWidth()  // content + padding + border
  $node.outerHeight()
  $node.outerWidth(true)  // content + padding + border + margin
  $node.outerHeight(true)
  
  //修改 $node 的样式，字体颜色设置红色，字体大小设置 14px
  $node.css( {
    'color' : 'red',
    'font-size' : '14px'
   });
   
   //遍历节点，把每个节点里面的文本内容重复一遍
  $node.each(function() {
    $(this).text();
    });
    
  //从 $ct 里面查找 class 为 .item 的子元素    
  $ct.find('.item')
  
  //获取 $ct 里面的所有孩子
  $ct.children();
  
  //对于 $node，向上找到 class 为 '.ct' 的父亲，在从该父亲找到 '.panel' 的孩子
    $node.parents('.ct').find('.panel')
    
  //获取选择元素的数量
   $node.length
      
  //获取当前元素在兄弟中的排行
   $node.index()

```


#### js中“==”和“===”的区别


```
code[0]==200表示 ==200或者==‘200’等于整形和字符串都可以
if(code[0]==200){

}

code[0]===200表示 ===200 只能严格等于整形200
if(code[0]==200){

}

```

#### 定时去处理一件事情setTimeout(...,1500)


setTimeout(...,1000)，隔1秒去处理某件事情。可以放一个函数如：1.5秒后跳转登录页面。

```
 setTimeout(function () {
                        window.location.replace('/login/')
                    },1500)
```


#### js中打开，跳转页面

重新加载当前页面
```
window.location.reload();
```


在当前页打开指定页面

```
window.location.replace('/login/') //在当前页面打开登录页面
```


在新窗口打开制定页面

```
window.location.href="/login/"; 
window.open("/login/");   

```

返回上一页

```
window.history.back(-1);
```

