---
layout: post
title: 自定义Django内置的User系统
categories: [Django]
tags: []
date: 2019-11-10 11:48:07
comments: true
---


#### 自定义User系统步骤
Django内置的User系统可以帮我们完成后台用户验证，授权，非常强大。

但内置的User系统验证是以Username和Password，也就是Username是unique，而我们常用的是手机号或者邮箱是来验证。而且内置的User还有一些字段如：first_name,last_name等等也不符合我们的需求。

所以我们需要来改写AbstractBaseUser来自定义我们的User系统，修改验证系统以手机号来验证，另外自定义我们需求的字段。

步骤：
1. 创建一个cmsauth，用来管理用户系统。(不能使用auth,auth已经被系统占用)
2. 继承AbstractBaseUser重写User系统的验证、字段，定义UserManager。
3. 注册app,设置AUTH_USER_MODEL。修改系统的User路径(告诉Django你现在不用内置的User系统了，用自定义的User系统)
4. 在第一次进行migrations和migrate之前必须先完成以上两个步骤，然后映射到数据库中。

#### 继承AbstractBaseUser重写User系统

在创建的cmsauth这个app内的model模型中写模型，代码如下:


```Python
from django.db import models
from django.contrib.auth.models import BaseUserManager
from django.contrib.auth.models import AbstractBaseUser
from django.contrib.auth.models import PermissionsMixin


class UserManager(BaseUserManager):
    """重写user的object模型"""
    def _create_user(self, telephone, username, password, **kwargs):
        if not telephone:
            raise ValueError('手机号不能为空')
        if not username:
            raise ValueError('姓名不能为空')
        if not password:
            raise ValueError('密码不能为空')
        user = self.model(telephone=telephone, username=username, **kwargs)
        user.set_password(password)
        user.save()
        return user
    """创建普通用户"""
    def create_user(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = False
        self._create_user(telephone=telephone, username=username, password=password, **kwargs)
    """创建超级用户"""
    def create_superuser(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = True
        self._create_user(telephone=telephone, username=username, password=password, **kwargs)


class User(AbstractBaseUser, PermissionsMixin):
    """重新定义user模型"""
    telephone = models.CharField(max_length=11, unique=True)
    email = models.CharField(max_length=100, null=True)
    username = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)
    data_joined = models.DateTimeField(auto_now_add=True)

    """设置手机号为验证方式"""
    USERNAME_FIELD = 'telephone'
    REQUIRED_FIELDS = ['username']

    """把重写objects模型赋值到重写的user模型中"""
    objects = UserManager()

    def get_full_name(self):
        return self.username

    def get_short_name(self):
        return self.username

```


#### 注册app,设置AUTH_USER_MODEL

在settings.py中注册cmsauth这个app，不注册无法进行migrations操作。代码如下：

```Python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'apps.cmsauth',
]
```

在settings.py中设置AUTH_USER_MODEL,告诉Django现在用我们自定义的User。代码如下:

```Python
AUTH_USER_MODEL = 'cmsauth.User'
```

不是完整路径，只需要app的名字，和模型的名字就可以了。不需要写成app.cmsauth.forms.User或者app.cmsauth.User这些都是不可以的。


#### 把自定义的User映射到数据库

配置好数据库，运行makemigrations,然后再migrate就可以了。




