---
layout: post
title: Django中drf类视图之generic
categories: [后端]
tags: [drf]
date: 2020-1-11 13:48:07
comments: true
---


#### 概述

以上我们通过mixin可以非常方便的实现一些CURD操作。实际上针对这些mixin，DRF还进一步的进行了封装，放到generics下。

#### 序列化

```python
from rest_framework import serializers
from meituan.models import Merchant,GoodsCategory,Goods

class MerchantSerializer(serializers.ModelSerializer):
    class Meta:
        model = Merchant
        fields = "__all__"

class GoodsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Goods
        fields = "__all__"

class GoodsCategorySerializer(serializers.ModelSerializer):
    merchant = MerchantSerializer(read_only=True)
    merchant_id = serializers.IntegerField(write_only=True)
    goods_list = GoodsSerializer(many=True,read_only=True)
    class Meta:
        model = GoodsCategory
        fields = "__all__"

    def validate_merchant_id(self,value):
        if not Merchant.objects.filter(pk=value).exists():
            raise serializers.ValidationError("商家不存在！")
        return value

    def create(self,validated_data):
        merchant_id = validated_data.get('merchant_id')
        merchant = Merchant.objects.get(pk=merchant_id)
        category = GoodsCategory.objects.create(**validated_data,merchant=merchant)
        return category
```

#### 视图函数


```python

from meituan.models import Merchant
from .serializers import MerchantSerializer
from rest_framework import generics

class MerchantView(
    # generics.ListAPIView,
    generics.CreateAPIView,
    generics.UpdateAPIView,
    generics.DestroyAPIView,
    generics.RetrieveAPIView
    #列表和检索不能共存
):
    queryset = Merchant.objects.all()
    serializer_class = MerchantSerializer

# 实现列表
class MerchantListView(
    generics.ListAPIView
):
    queryset = Merchant.objects.all()
    serializer_class = MerchantSerializer
```

因为这里retrieve占用了get方法，所以如果想要实现获取列表的功能，那么需要再重新定义一个url和视图。


```python
# views.py
class MerchantListView(generics.ListAPIView,):
    serializer_class = MerchantSerializer
    queryset = Merchant.objects.all()

# urls.py
urlpatterns = [
    path('merchant/',views.MerchantView.as_view()),
    path('merchant/<int:pk>/',views.MerchantView.as_view()),
    path('merchants/',views.MerchantListView.as_view())
    ...
]
```

#### urls.py


```python
from django.urls import path
from .views import MerchantView,MerthantListView,MerchantViewSet
from rest_framework.routers import DefaultRouter

app_name = 'classview'
urlpatterns = [
    # 列表：/merchant/ get
    # 新增：/merchant/ post
    # 详情：/merchant/[pk]/ get
    # 修改：/merchant/[pk]/ put
    # 删除：/merchant/[pk]/ delete
    path('merchant/',MerchantView.as_view()),
    path('merchant/<int:pk>/',MerchantView.as_view()),
    path('merchants/',MerchantListView.as_view())
    #merchants实现列表功能
]
```

主urls.py

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('classview.urls')),
]
```

#### GenericAPIView介绍

如果想要深入学会generic的一些用法。比如如何分页，如何过滤数据等。那么这时候就需要学习GenericAPIView的使用。

queryset：
queryset是用来控制视图返回给前端的数据。如果没什么逻辑，可以直接写在视图的类属性中，如果逻辑比较复杂，也可以重写get_queryset方法用来返回一个queryset对象。如果重写了get_queryset，那么以后获取queryset的时候就需要通过调用get_queryset方法。因为queryset 这个属性只会调用一次，以后所有的请求都是使用他的缓存。

serializer_class:
serializer_class用来验证和序列化数据的。也是可以通过直接设置这个属性，也可以通过重写get_serializer_class来实现。

lookup_field和lookup_url_kwarg：
lookup_field：是在检索的时候，根据什么参数进行检索。默认是pk，也就是主键。
lookup_url_kwarg：在检索的url中的参数名称。默认没有设置，跟lookup_field保持一致。

分页：
分页是通过设置pagination_class来实现的。默认这个属性的值是rest_framework.pagination.PageNumberPagination，也就是通过控制页码，每页的数量来实现的。我们可以通过在settings.REST_FRAMEWORK中设置PAGE_SIZE来控制每页的数量，然后在url中通过传递page参数来获取指定页数的数据。

#### 重写方法

1. get_queryset(self)：
用于动态的返回一个queryset对象。

2. get_object(self)：
用于在数据检索的时候，返回一条数据的。

3. perform_create(self,serializer)：
保存对象的时候调用。

4. perform_update(self,serializer)：
更新对象的时候调用。

5. perform_destroy(self,serializer)：
删除对象的时候调用。


















