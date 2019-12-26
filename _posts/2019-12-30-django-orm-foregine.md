---
layout: post
title: django中orm的外健和外健删除详解
categories: [后端]
tags: [数据库]
date: 2019-12-30 13:48:07
comments: true
---


#### 外健的使用

外健的使用是两个有关联的模型之间的引用，如：地区和员工，一个员工只有一个地区，但是一个地区却可以有很多员工，这样就产生了一对多的关系，就需要在员工里面添加一个字段区域，对应一个地区，这就是外健的使用了。

```
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
    area = models.ForeignKey('Area',on_delete=models.PROTECT,null=False)//员工的区域使用外健和区域进行关联

    """设置手机号为验证方式"""
    USERNAME_FIELD = 'telephone'
    REQUIRED_FIELDS = ['username']

    """把重写objects模型赋值到重写的user模型中"""
    objects = UserManager()

    def get_full_name(self):
        return self.username

    def get_short_name(self):
        return self.username


class Area(models.Model):
    """定义区域模型"""
    name = models.CharField(max_length=100,unique=True,null=False)

    class Meta:
        db_table = 'area'

```


#### 引用的外健不再同一个app下面

在同一个app下面。

```
area = models.ForeignKey('Area',on_delete=models.PROTECT,null=False)//员工的区域使用外健和区域进行关联。
```

不在同一个app下面。需要用“app名.模型名”，这样写完整的路径才能引用到。加入区域在product这个app下面。就需要像下面这样引用:product.Area

```
area = models.ForeignKey('product.Area',on_delete=models.PROTECT,null=False)//员工的区域使用外健和区域进行关联。
```


#### 引用的外健是这个模型本身

如：评价是针对的哪一个评价进行的评价，门店的父门店是哪一个。可以使用self，模型名，或者完整路径都可以。


```
class Comment(models.Model):
    content = models.TextField()
    origin_comment = models.ForeignKey("self",on_delete=models.CASCADE)//标准写法
    origin_comment = models.ForeignKey("Comment",on_delete=models.CASCADE)//这样写也可以
    origin_comment = models.ForeignKey("product.Comment",on_delete=models.CASCADE)//写完整的路径也可以

```

```
class Store(models.Model):
    name = models.CharField(max_length=100)
    parent_store = models.ForeignKey("self",on_delete=models.CASCADE)//标准写法
    parent_store = models.ForeignKey("Store",on_delete=models.CASCADE)//这样写也可以
    parent_store = models.ForeignKey("product.Store",on_delete=models.CASCADE)//写完整的路径也可以

```


#### 外健的删除

拿区域和员工距离，员工表中引用了区域，如果区域删除了，那么员工这条数据如何处理？？？

CASCADE，及联删除，就是区域如果删除了，那么引用的员工也删除。
```
area = models.ForeignKey('Area',on_delete=models.CASCADE,null=False)
```

PROTECT，保护，员工引用了区域，那么区域就不能删除。
```
area = models.ForeignKey('Area',on_delete=models.PROTECT,null=False)
```

set_null，设置为空，员工引用了区域，区域删除那么员工的区域为空。注：使用set_null，则必须该字段的null=True，必须可以为空。
```
area = models.ForeignKey('Area',on_delete=models.set_null,null=True)
```

set_default，设置默认，员工引用了区域，区域删除那么员工的区域为设置的默认值。注：使用set_default，则必须该字段要设置默认值。
```
area = models.ForeignKey('Area',on_delete=models.set_default,null=True,default=Area.objects.get(pk=4))//默认使用主键为4的区域为默认值
```

set()，设置，员工引用了区域，区域删除那么员工的区域为设置的区域。注：使用set()，唯一的区别是可以不设置默认值。set()中还可以中间放一个函数。
```
area = models.ForeignKey('Area',on_delete=models.set(Area.objects.get(pk=4)),null=True)//默认使用主键为4的区域为默认值
```

DO_NOTHING，orm层面什么都不做一切看数据库的约束。
```
area = models.ForeignKey('Area',on_delete=models.DO_NOTHING,null=True)
```


















