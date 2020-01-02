---
layout: post
title: Django中drf基础之请求和响应
categories: [后端]
tags: [drf]
date: 2020-1-8 13:48:07
comments: true
---


#### 请求和响应对象

在drf中，可以使用Request和Response对象来替代django内置的HttpRequest和HttpResponse。替代django的对象有很多好处。以下进行简单讲解。

#### 请求对象

DRF的Request对象的英文从HttpRequest。中拓展出来的，但是增加了一些其他的属性其中最核心的用得最多的属性便是request.data。request.data比request.POST更加灵活：

1. request.POST：只能处理表单数据，获取通过POST方式上传上来的数据。
2. request.data：可以处理任意的数据可以获取通过POST，PUT，PATCH等方式上传上来的数据。
3. request.query_params：查询参数。比request.GET更用起来更直白。

#### 响应对象

Response可以自动的根据返回的数据类型来决定返回的格式。和会自动的监听如果是浏览器访问，那么会返回该路由的信息。

#### 状态码

在Restful API中，响应的状态码是很重要的一部分。某些请求成功是200，参数错误是400等。但是具体某个状态码是干什么的，django是没有做过多的解释（这也不是django所需要解决的问题，因为他只是一个web框架），对于一些初学者而言用起来会有点迷糊。这时候我们可以使用DRF提供的状态码。


```python
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET','POST','PUT','DELETE'])
def merchant(request):
    return Response({"username":"zhiliao"},status=status.HTTP_200_OK)
```

#### 实现APIView

的以上Respone状语从句：Request对象都只能在DRF的APIView。中才能使用如果是视图函数，可以那么装饰使用器rest_framework.decorators.api_view进行装饰，这个装饰器中可以传递本视图函数可以使用什么method。进行请求示例代码如下：

```
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    """
    Retrieve, update or delete a code snippet.
    """
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```


如果是类视图，那么可以让你的类继承自rest_framework.views.APIView。示例代码如下：


```python
from rest_framework.views import APIView

class MerchantView(APIView):
    def get(self,request):
        return Response("你好")
```















