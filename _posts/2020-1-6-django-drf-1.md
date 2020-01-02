---
layout: post
title: Django中drf基础之序列化定义
categories: [后端]
tags: [drf]
date: 2020-1-6 13:48:07
comments: true
---


#### 步骤

先安装drf，运行命令“pip install djangorestframework”，当然drf需要两个依赖，“Python (3.5, 3.6, 3.7)”和“Django (1.11, 2.0, 2.1, 2.2)”。

安装完成后需要到setting.py中注册。
```
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

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

#### 基础序列化

drf除了Json序列化，还具有表单验证功能，数据存储和更新功能。

创建一个serializer类：
```
from rest_framework import serializers
from meituan.models import Merchant,Goods,GoodsCategory

#1,用来序列化数据，ORM-JSON
#2,用来验证表单数据，Form
#3,可以创建数据，更新数据等

#Serializer的构造函数有两个参数
#instance:需要传递一个orm对象，或者一个queryset对象，用来将orm模型序列化成JSON的
#data:把需要验证的数据传递给data，用来验证这些数据是否符合要求的
#many:如果instance是一个queryset对象，那么需要设置为True，否则False

class MerchantSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=200,required=True)
    logo = serializers.CharField(max_length=200,required=True)
    address = serializers.CharField(max_length=200,required=True)
    notice = serializers.CharField(max_length=200,required=False)
    up_send = serializers.DecimalField(default=0,max_digits=6,decimal_places=2,required=False)
    lon = serializers.FloatField(required=True)
    lat = serializers.FloatField(required=True)

    def update(self, instance, validated_data):
        instance.name = validated_data.get('name',instance.name)
        instance.logo = validated_data.get('logo',instance.logo)
        instance.address = validated_data.get('address',instance.address)
        instance.notice = validated_data.get('notice',instance.notice)
        instance.up_send = validated_data.get('up_send',instance.up_send)
        instance.lon = validated_data.get('lon',instance.lon)
        instance.lat = validated_data.get('lat',instance.lat)
        instance.save()
        return instance

    def create(self, validated_data):
        return Merchant.objects.create(**validated_data)
   //重写验证方法等等...
```

在视图函数中，对数据进行序列化，也可以对数据进行校验，然后存储数据。


```python
from .serializers import MerchantSerializer
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods
from meituan.models import Merchant


@require_http_methods(['GET','POST'])
def merchant(request):
    if request.method == 'GET':
        queryset = Merchant.objects.all()
        serializer = MerchantSerializer(instance=queryset,many=True)
        # 默认只能序列化一个数据，所以需要many=True，才能序列化多个数据
        return JsonResponse(serializer.data,status=200,safe=False)
    else:
        serializer = MerchantSerializer(data=request.POST)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data,status=200)
        else:
            return JsonResponse(serializer.errors,status=400)
```

urls.py中完成路由


```python
from django.urls import path
from .views import merchant 

app_name = 'serialize'

urlpatterns = [
    path('merchant/',merchant,name='merchant'),
]
```

主urls.py

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('serializerapp.urls')),
]
```


```
http://127.0.0.1:8000/meituan/merchant/ //get获取列表
http://127.0.0.1:8000/meituan/merchant/ //post提交数据
```

自定义错误提示：

```
name = serializers.CharField(max_length=20,required=True,error_messages={'required':'name不能为空','max_length':'name不能超过20字'})
```

#### 模型序列化


```python
class MerchantSerializer(serializers.ModelSerializer):
    class Meta:
        model = Merchant
        fields = '__all__'
```

使用模型序列化就完成了基础序列化中的所有serializer中的代码。










