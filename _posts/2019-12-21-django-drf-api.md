---
layout: post
title: django中使用restframework进行序列化
categories: [后端]
tags: []
date: 2019-12-21 13:48:07
comments: true
---


#### 步骤

序列化就像form表单验证一样，定义一个序列化文件，然后选择需要序列化的内容。

1. 安装restframework包并在settings.py文件中注册
2. 在serialisers.py中编写序列化文件
3. 在views.py中使用


#### 安装包

运行以下命令安装包。
```
pip install djangorestframework
```

在settings.py中进行注册。
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'apps.cmsauth',
    'apps.index',
    'apps.product',
    'utils',
    'apps.ueditor',
    'apps.frontuser',
    'rest_framework'//安装完包后在这里注册，会自动提示
]
```

#### 编写序列化文件

因为序列化都是正对模型的，所以我们用类似于模型表单的方式，使用模型序列化。
在需要用到的序列化的app中创建serializers.py文件，在这个文件中编写所有我们需要用到的序列化文件。

```
from rest_framework import serializers//导入序列化
from apps.product.models import Product, Category//导入模型


class CategorySerialiser(serializers.ModelSerializer)://定义分类的序列化
    class Meta:
        model =Category//对应的模型
        fields = ('id','name')//需要用到的字段


class ProductListSerialiser(serializers.ModelSerializer):
    category = CategorySerialiser()//分类因为是外健，所有还需要定义一个分类的序列化
    class Meta:
        model = Product//对应的模型
        fields = ('id','title','price','thumbnail','category')//需要用到的字段
```

#### 在views.py中使用


```
from django.shortcuts import render
from django.views.generic import View
from utils import restful
from apps.product.models import Product//导入模型
from .serializers import ProductListSerialiser//导入序列化，就像导入验证表单一样

# Create your views here.


class ProductList(View):
    def get(self,request,*args,**kwargs):
        page = int(request.GET.get('p',1))
        start = (page-1)*20
        end = start + 20
        products = Product.objects.order_by('-pub_time')[start:end]
        serializer = ProductListSerialiser(products,many=True)//序列化products，不要少了many=True
        data = serializer.data//获取序列化后的数据
        print(data)
        return restful.result(data=data)//通过restfuly传过去
```






