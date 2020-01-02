---
layout: post
title: Django中drf基础之序列化的嵌套
categories: [后端]
tags: [drf]
date: 2020-1-7 13:48:07
comments: true
---


#### 步骤


在创建的meituanapp下面创建models，在模型中创建以下模型。

```python
from django.db import models
from django.contrib.auth.models import User


class Merchant(models.Model):
    """
    商家
    """
    name = models.CharField(max_length=200,verbose_name='商家名称',null=False)
    address = models.CharField(max_length=200,verbose_name='商家',null=False)
    logo = models.CharField(max_length=200,verbose_name='商家logo',null=False)
    notice = models.CharField(max_length=200, verbose_name='商家的公告',null=True,blank=True)
    up_send = models.DecimalField(verbose_name='起送价',default=0,max_digits=6,decimal_places=2)
    lon = models.FloatField(verbose_name='经度')
    lat = models.FloatField(verbose_name='纬度')
    created = models.ForeignKey(User,on_delete=models.SET_NULL,null=True)


class GoodsCategory(models.Model):
    """
    商家商品分类
    """
    name = models.CharField(max_length=20,verbose_name='分类名称')
    merchant = models.ForeignKey(Merchant,on_delete=models.CASCADE,verbose_name='所属商家',related_name='categories')


class Goods(models.Model):
    """
    商品
    """
    name = models.CharField(max_length=200,verbose_name='商品名称')
    picture = models.CharField(max_length=200,verbose_name='商品图片')
    intro = models.CharField(max_length=200)
    price = models.DecimalField(verbose_name='商品价格',max_digits=6,decimal_places=2) # 最多6位数，2位小数。9999.99
    category = models.ForeignKey(GoodsCategory,on_delete=models.CASCADE,related_name='goods_list')

```

#### 序列化的嵌套


```python
from rest_framework import serializers
from meituan.models import Merchant,Goods,GoodsCategory


class MerchantSerializer(serializers.ModelSerializer):
    class Meta:
        model = Merchant
        fields = '__all__'


class GoodsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Goods
        fields = '__all__'


class GoodsCategorySerializer(serializers.ModelSerializer):
    merchant = MerchantSerializer(read_only=True)
    # 只读，序列化出去的时候可以看见
    merchant_id = serializers.IntegerField(write_only=True)
    # 验证提交数据的时候使用，那么需要设置write_only=True,因为提交的时候merchant不能为空
    # 需要自己去检查id是否存在，然后实现保存的方法
    goods_list = GoodsSerializer(many=True,read_only=True)
    # 外健的商品有很多，所以many需要设置为True，然后是只读的
    class Meta:
        model = GoodsCategory
        fields = "__all__"

    def validate_mechant_id(self, value):
        if not Merchant.objects.filter(pk=value).exist():
            raise serializers.ValidationError('商家不存在')
        return value

    def create(self, validated_data):
        merchant_id = validated_data.get('merchant_id')
        merchant = Merchant.objects.get(pk=merchant_id)
        category = GoodsCategory.objects.create(**validated_data,merchant=merchant)
        return category

```


#### 在视图函数中完成视图函数

views.py

```python
from .serializers import MerchantSerializer,GoodsSerializer,GoodsCategorySerializer
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from meituan.models import Merchant,GoodsCategory


@require_http_methods(['GET','POST'])
def category(request):
    if request.method == 'GET':
        queryset = GoodsCategory.objects.all()
        serializer = GoodsCategorySerializer(instance=queryset,many=True)
        return JsonResponse(serializer.data,safe=False)
    else:
        serializer = GoodsCategorySerializer(data=request.POST)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=200)
        else:
            return JsonResponse(serializer.errors, status=400)
```

#### urls.py中完成配置


```python
from django.urls import path
from .views import merchant,category

app_name = 'serialize'

urlpatterns = [
    path('merchant/',merchant,name='merchant'),
    path('category/',category,name='category'),
]
```

主url.py中完成配置

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('serializerapp.urls')),
]
```

#### 关于read_only和write_only

read_only=True：这个分区只可以读，只有在返回数据的时候会使用。
write_only=True：这个主轴只能被写，只有在补充数据或更新数据的时候会用到。

#### 验证

1. 验证在Field中通过参数的形式进行指定。某些required等。
2. 通过重组validate(self,attrs)方法进行验证。attrs中包含了所有分割。如果验证不通过，那么调用raise serializer.ValidationError('error')即可。
3. 重写validate_字段名(self,value)方法进行验证。这个是针对某个字段进行验证的。如果验证不通过，也可以抛出异常。















