---
layout: post
title: Jquery基础之属性操作
categories: [前端]
tags: [jquery]
date: 2020-1-18 13:48:07
comments: true
---


#### 选择器

通过属性可以拿到dom元素，通过$("tagName")可以拿到标签名的元素，通过$(".div")可以拿到class的名称内容，通过$("#div")可以拿到id的名称内容。

还可以通过其他属性查找到我们需要的内容。

#### :empty :parent :contains :has

:empty 找到既没有文本内容也没有子元素的制定元素

```html
var div = $("div:empty")
```

:parent 找到有文本内容或子元素的制定元素

```html
var div = $("div:parent")
```

:contains 找到包含指定文本内容的指定元素

```html
var div = $("div:contains('我是div')")
```

:has 找到包含指定子元素的指定元素

```html
var div = $("div:has('span')")
```

#### 属性、属性节点操作

首现了解什么是属性，属性就是对象身上保存的变量。
操作属性：对象.属性名称=值   对象[“属性名称”] = 值

在标签中添加的属性就是属性节点。html代码中如一个span下面展开的就是属性，其中有一个attributes下面保存的就是所有的属性节点。如：name , type ,id,class等等。

#### 操作属性节点 $.attr()

第一种方法：

$("tagName").setAttribute("name","my")  //设置属性节点的值
$("tagName").getAttribute("name") //获取属性节点的值

第二种方法：

$("tagName").attr(name|pro|key,val|fn)可以接受两个参数，如果传递一个参数代表获取属性节点的值，如果传递两个参数代表设置属性节点的值。

如果是获取：只会获取第一个dom元素的属性节点的值。
如果是设置：找到多少个元素，就会设置多少个元素。若设置的属性不存在，那就会新增一个属性节点。

$("tagName").removeAtrr("class name") 会删除所有找到元素的属性节点的属性。

#### 操作属性 $.prop()

$("tagName").eq(0).prop(name|pro|key,val|fn)可以接受两个参数，如果传递一个参数代表获取属性的值，如果传递两个参数代表设置属性的值。

获取：只会获取找到的第一个元素的属性值。
设置：会设置所有找到元素的属性值。

$("tagName").removeProp("demo") 会删除所有找到元素的属性的属性。

prop不光可以操作属性，还可以操作属性节点。


####  区别 $.attr() $.prop()

在操作属性节点时，具有true和false两个属性节点，如：checked selected 或 disabled 使用prop()

其他使用attr()


#### class操作类相关方法

$("tagName").addClass(class|fn) //添加，可以添加多个

$("tagName").removeClass(class|fn) //删除，可以删除多个

$("tagName").toggleClass(class|fn) //有就删除 没有就新增，切换


#### 操作文本相关方法

代码片段：
设置 $("tagName").html("<p>我是段落</p>")  //添加代码片段 和innerHtml一样
获取 $("tagName").html()  //获取代码片段 和innerHtml一样

text方法：
设置 $("tagName").text("我是段落")  //添加文本 和innerText一样
获取 $("tagName").text()  //获取文本 和innerText一样

操作值，value方法：
设置 $("tagName").value("我是段落")  //添加值
获取 $("tagName").value()  //获取值

#### 操作css样式方法

设置：
```
$("tagName").css("width","100px") //添加样式 单个设置
$("tagName").css({
        width:"100px",
        height:"100px"        
        }) //添加样式 多个
```

获取
$("tagName").css("width") //获取宽度

#### 操作尺寸，位置方法

$("tagName").with() //获取宽度
$("tagName").with(“200px”) //设置元素宽度

$("tagName").offset().left //获取元素距离窗口的偏移位
$("tagName").offset().top //获取元素距离窗口的偏移位


```html
$("tagName").offset({
        left:10px
    }) /设置取元素距离窗口的偏移位
```

#### scrolltop 方法

$("tagName").scrollTop() //获取滚动距离
$("tagName").scrollTop(300) //设置滚动距离















