---
layout: post
title: Django中drf权限认证之JWT认证
categories: [后端]
tags: [drf]
date: 2020-1-13 13:48:07
comments: true
---


#### 概述

认证可以简单的理解为登录和访问需要登录的接口的认证。只要认证通过了，那么在request对象（是drf的request对象）上便有两个属性，一个是request.user，一个是request.auth，前者就是django中的User对象，内置根据不同的认证机制有不同的对象。DRF内置了几个认证的模块。以下进行简单了解。

JSON Web令牌认证机制：
JSON Web Token简短JWT。在前初始分离的项目中，或者是app项目中。推荐使用JWT。JWT是在成功后，把用户的相关信息（选择用户ID）以及过期时间进行加密，然后生成一个token返回给客户端，客户端拿到后可以存储起来，以后每次请求的时候都搬运这个token，服务器在接收到需要登录的API请求候，对这个token进行解密，然后获取过期时间和用户信息（称为用户id），如果过期了或者用户信息不对，那么都是认证失败。

由于认证和解析认证都是依靠user.pk来进行的，所以建议认证的user用户的pk用唯一主键，用shortuuid，不要用自增长的主键。

jwt认证先安装，运行命令“pip install jwt”，安装完成后才能使用，前提是进入项目所在的虚拟环境之中。

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

#### 完成认证代码

在authentications.py中完成认证的代码：


```python
import jwt
import time
from django.conf import settings
from rest_framework.authentication import BaseAuthentication,get_authorization_header
from rest_framework import exceptions
from django.contrib.auth import get_user_model
User = get_user_model() //可以是任意模型 Django中自带的user模型
from meituan.models import FrontUser  //可以是我们自定义的user模型，
//注意 由于验证的是user.pk所以建议模型的pk使用shortuuid


def generate_jwt(user):
    timestamp = int(time.time()) + 60*60*24*7
    # timestamp = int(time.time()) + 2
    # 因为jwt.encode返回的是bytes数据类型，因此需要decode解码成str类型
    return jwt.encode({"userid":user.pk,"exp":timestamp},settings.SECRET_KEY).decode('utf-8')


class JWTAuthentication(BaseAuthentication):
    """
    Authorization: JWT 401f7ac837da42b97f613d789819ff93537bee6a
    """

    keyword = 'JWT'
    model = None

    def authenticate(self, request):
        auth = get_authorization_header(request).split()

        if not auth or auth[0].lower() != self.keyword.lower().encode():
            # return None
            msg = '没有携带可用的JWT请求头'
            raise exceptions.AuthenticationFailed(msg)

        if len(auth) == 1:
            msg = 'Authorization 不可用！'
            raise exceptions.AuthenticationFailed(msg)
        elif len(auth) > 2:
            msg = 'Authorization 不可用！应该提供一个空格！'
            raise exceptions.AuthenticationFailed(msg)

        try:
            jwt_token = auth[1]
            # 如果解码的时候，发现
            jwt_info = jwt.decode(jwt_token,settings.SECRET_KEY)
            userid = jwt_info.get('userid')
            try:
                user = FrontUser.objects.get(pk=userid)
                return (user,jwt_token)
            except:
                msg = "用户不存在！"
                raise exceptions.AuthenticationFailed(msg)
        except jwt.ExpiredSignatureError:
            msg = 'Token已经过期了！'
            raise exceptions.AuthenticationFailed(msg)
```


#### 视图函数


```python

from rest_framework import viewsets
from meituan.models import Merchant
from .serializers import MerchantSerializer
from .authentications import generate_jwt,JWTAuthentication
from django.contrib.auth import get_user_model
from rest_framework.response import Response
from rest_framework.decorators import api_view
from meituan.models import FrontUser

User = get_user_model()


class MerchantViewSet(viewsets.ModelViewSet):
    queryset = Merchant.objects.all()
    serializer_class = MerchantSerializer
    authentication_classes = [JWTAuthentication]
    #设置JWT认证方式，就是这个api需要通过JWT认证通过后才可以使用


@api_view(['GET'])
#获取token接口
def token_view(request):
    token = generate_jwt(FrontUser.objects.first())
    return Response({"token":token})
```


#### urls.py


```python
from rest_framework.routers import DefaultRouter
from .views import MerchantViewSet,token_view
from django.urls import path

router = DefaultRouter()
router.register('merchant',MerchantViewSet,basename='merchant')

urlpatterns = [
    path('token/',token_view,name='token')
] + router.urls
```

配置好了之后就可以通过postman发送请求了。

```
http://127.0.0.1:8000/meituan/token/   //这个接口可以获取token，会返回Frontuser中第一个用户的token

http://127.0.0.1:8000/meituan/merchant/  //这个接口可以获取商家信息，如果没有在请求头重添加 key:Authorization   value: jwt token 那么就不能进行访问

```

这样前端就可以先获取token，然后再把token存放在缓存中，以后每次需要请求登录之后才能访问的接口时就可以在请求头中添加token信息，获取访问接口的权限了。


主urls.py

```python
from django.urls import path,include

urlpatterns = [
    path('meituan/', include('authpermission.urls')),
]
```

















