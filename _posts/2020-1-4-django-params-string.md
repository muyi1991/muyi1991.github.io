---
layout: post
title: Django中普通传参和查询字符串传参
categories: [后端]
tags: [传参]
date: 2020-1-4 13:48:07
comments: true
---


#### 概述

传递参数给视图函数，然后视图函数根据传递进来的参数进行数据处理。如：前端传递一个产品ID，后台根据ID到数据库中查询出该产品的详情，然后在返回给前端进行渲染。

传参有两种方式，第一种，普通传参，第二种字符串传参，两种方式都能实现向视图函数中传递参数。

#### 普通传参

以获取图书详情为例，普通方式传递一个图书ID。

```python
from Django.http import path

urlpatterns  = {
    path('book_detail/<int:book_id>/',book_detail)
}
//path中的<book_id>必须和def中的request,book_id必须一致，如果不一致会导致找不到

def book_detail(request,book_id):
    text = "您获取的图书ID是%s" % book_id
    return HttpRestponse(text)
```

这样通过"book_detail/1"就可以传参给视图函数了

如果想传递多个参数怎么办呢？

分开传递就能传递多个参数了
```python
from Django.http import path

urlpatterns  = {
    path('book_detail/<book_id>/<category_id>',book_detail)
}
//path中的<book_id>必须和def中的request,book_id必须一致，如果不一致会导致找不到

def book_detail(request,book_id,category_id):
    text = "您获取的图书ID是%s,分类ID是%s" %  (book_id,category_id)
    return HttpRestponse(text)
```

注意：如果你在path中传递两个参数，那么你在视图函数中必须来接受两个参数，而且参数的名字还必须要对应正确，不一致也会报错。

如果想要传递的参数是指定的类型可以使用<int:book_id>，这样限制传递的book_id只能是int类型。


#### 查询字符串传参

以获取图书详情为例，普通方式传递一个图书ID。

```python
from Django.http import path

urlpatterns  = {
    path('book_detail/',book_detail)
}
//book_detail/?book_id=1这样就可以传递参数了

def book_detail(request):
    book_id = request.GET.get('book_id')
    //可以指定默认参数，如果没有传递就使用默认参数，常用的是分页，如果没有传递，默认第一页。
    //page = request.GET.get('p',1)，如果没有传递，就是默认参数1.
    //page = int(request.GET.get('p',1)),因为request传递过来都是字符串，所以要转成int类型
    text = "您获取的图书ID是%s" % book_id
    return HttpRestponse(text)
```


