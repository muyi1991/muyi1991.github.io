---
layout: post
title: Django中drf类视图之viewset视图集
categories: [后端]
tags: [drf]
date: 2020-1-12 13:48:07
comments: true
---


#### 概述

ViewSet视图集，相当于是之前我们学习视图的一个集合。在视图集中，不再有get和post，取而代之的是list和create。以下分别进行讲解。

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

from rest_framework import viewsets
from rest_framework.decorators import action

class MerchantViewSet(viewsets.ModelViewSet):
    queryset = Merchant.objects.all()
    serializer_class = MerchantSerializer

    @action(['GET'],detail=False)
    def cs(self,request,*args,**kwargs):
        queryset = self.get_queryset()
        queryset = queryset.filter(name__contains="长沙")
        serializer = MerchantSerializer(queryset,many=True)
        return Response(serializer.data)
```


#### urls.py


```python
from django.urls import path
from .views import MerchantView,MerchantListView,MerchantViewSet
from rest_framework.routers import DefaultRouter

app_name = 'classview'

urlpatterns = [
    path('merchant/',MerchantView.as_view()),
    path('merchant/<str:name>/',MerchantView.as_view()),
    path('merchants/',MerchantListView.as_view())
]

router = DefaultRouter()
router.register('shangjia',MerchantViewSet,basename='shangjia')
urlpatterns += router.urls
```

urls.py路由部分不需要修改。以后直接可以通过/merchant/cs/可以访问到name中包含了"长沙"两个字的所有商家。

主urls.py

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('classview.urls')),
]
```

















