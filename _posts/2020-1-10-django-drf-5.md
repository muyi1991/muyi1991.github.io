---
layout: post
title: Django中drf类视图之mixin
categories: [后端]
tags: [drf]
date: 2020-1-10 13:48:07
comments: true
---


#### 概述

mixins翻译成中文是混入，组件的意思。在DRF中，针对获取列表，检索，创建等操作，都有相应的mixin。示例代码如下：

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
from .models import Merchant
from .serializers import MerchantSerializer
from rest_framework import mixins
from rest_framework import generics

class MerchantView(
    generics.GenericAPIView,
    mixins.ListModelMixin,
    mixins.RetrieveModelMixin,
    mixins.CreateModelMixin,
    mixins.UpdateModelMixin,
    mixins.DestroyModelMixin
):
    queryset = Merchant.objects.all()
    serializer_class = MerchantSerializer

    def get(self,request,pk=None):
        if pk:
            return self.retrieve(request)
        else:
            return self.list(request)

    def post(self,request):
        return self.create(request)
        
    # def perform_create(self, serializer):
    #     serializer.save(created=self.request.user)
    # 如果要实现自己的保存方法，可以重写perform_create

    def put(self,request,pk=None):
        return self.update(request)

    def delete(self,request,pk=None):
        return self.destroy(request)
```

以上我们通过继承generics.GenericAPIView，可以设置queryset以及serializer_class，那么视图函数就知道你是要针对哪个模型做处理，你的序列化的类是什么了。接着我们继承mixins.ListModelMixin/CreateModelMixin类，这样MerchantList就拥有了获取列表，以及创建数据的功能。下面我们通过写get和post方法，调用self.list和self.create方法，就可以轻松的实现获取商家列表和创建商家的功能。

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















