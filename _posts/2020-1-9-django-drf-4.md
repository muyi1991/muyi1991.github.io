---
layout: post
title: Django中drf类视图之apiview
categories: [后端]
tags: [drf]
date: 2020-1-9 13:48:07
comments: true
---


#### 概述

在DRF中，推荐使用类视图，因为类视图可以通过继承的方式把一些重复性的工作抽取出来，而使得代码更加简洁。当然如果你不想使用类视图，那么就用@api_view装饰器包裹一下就可以。

APIView是DRF中类视图最基本的父类。基本用法跟Django中自带的View类是一样的。也是自己分别实现get、post等方法。示例代码如下：

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
from rest_framework.views import APIView
from meituan.models import Merchant
from django.http import Http404
from .serializers import MerchantSerializer
from rest_framework.response import Response
from rest_framework import status


class MerchantView(APIView):
    """
    检索, 更新和删除一个merchant实例对象.
    """
    def get_object(self, pk):
        try:
            return Merchant.objects.get(pk=pk)
        except Merchant.DoesNotExist:
            raise Http404

    def get(self, request, pk=None):
        if pk:
            merchant= self.get_object(pk)
            serializer = MerchantSerializer(merchant)
            return Response(serializer.data)
        else:
            queryset = Merchant.objects.all()
            serializer = MerchantSerializer(instance=queryset,many=True)
            return Response(serializer.data)

    def put(self, request, pk):
        merchant = self.get_object(pk)
        serializer = MerchantSerializer(merchant, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        merchant= self.get_object(pk)
        merchant.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
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
    path('merchant/<int:pk>/',MerchantView.as_view())
]
```

主urls.py

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('classview.urls')),
]
```

当然APIView中还继承了一些常用的属性，比如authentication_classes、permission_classes、throttle_classes等。















