---
layout: post
title: django数据库表关系之一对一
categories: [后端]
tags: [数据库]
date: 2019-11-27 13:48:07
comments: true
---


#### 概述

在Django中一对一是通过“models.OneToOneField”来实现的，这个“OneToOneField”其实本质上就是一个外健，只不过这个外健有一个唯一约束(unique key),来实现一对一。

以前端用户和签到表为例：

```
from django.db import models

class FrontUser(models.Model):
    """前台小程序用户表"""
    open_id = models.CharField(max_length=200)
    avatar = models.URLField()
    nikename = models.CharField(max_length=30)
    city = models.CharField(max_length=30)
    gender = models.BooleanField()

    class Meta:
        db_table = 'frontuser'


class FrontUserCheckIN(models.Model):
    """前台用户签到表 和用户表一对一"""
    is_checkin_yesterday = models.BooleanField(default=False)
    is_checkin_today = models.BooleanField(default=False)
    continuous_checkin = models.IntegerField()
    total_checkin = models.IntegerField()
    user = models.OneToOneField('FrontUser',on_delete=models.CASCADE)//如果user删除了那么签到表的数据及联删除

    class Meta:
        db_table = 'frontuser_checkin'
```

#### 监听FrontUser创建信号

因为是一对一关联，所以创建一条前台用户数据，就必须创建一张前台用户签到数据，所有需要监听前台用户创建信号。


```
from django.db import models
from django.dispatch import receiver//导入接收
from django.db.models.signals import post_save//导入监听保存方法


class FrontUser(models.Model):
    """前台小程序用户表"""
    open_id = models.CharField(max_length=200)
    avatar = models.URLField()
    nikename = models.CharField(max_length=30)
    city = models.CharField(max_length=30)
    gender = models.BooleanField()

    class Meta:
        db_table = 'frontuser'


class FrontUserCheckIN(models.Model):
    """前台用户签到表 和用户表一对一"""
    is_checkin_yesterday = models.BooleanField(default=False)
    is_checkin_today = models.BooleanField(default=False)
    continuous_checkin = models.IntegerField(null=True)
    total_checkin = models.IntegerField(null=True)
    user = models.OneToOneField('FrontUser',on_delete=models.CASCADE,related_name='checkin')//注意：使用了related_name，用FrontUser访问签到就可以直接使用FrontUser.checkin了。如果没有使用related_name，就需要使用FrontUser.FrontUserCheckIN来访问了。并且两种访问机制智能允许一种。

    class Meta:
        db_table = 'frontuser_checkin'


@receiver(post_save,sender=FrontUser)//定义监听frontuser方法
def handle_user_checkin(sender,instance,created,**kwargs):
    if created://如果frontuser创建一条数据，那么frontusercheckin也创建一条数据相对应
        FrontUserCheckIN.objects.create(user=instance)
    else://如果不是创建是保存，那么frontusercheckin也保存，因为有可能修改了签到的某些数据
        instance.checkin.save()//这里注意因为使用了related_name='checkin'所以可以直接使用FrontUser.checkin.save().如果没有使用related_name，则需要使用FrontUser.FrontUserCheck.save()了。

```


#### 一对一的访问

FrontUserCheckIN的对象，可以通过FrontUser来访问到对应的FrontUser对象，并且FrontUser对象可以使用FrontUserCheckIN来访问对应的FrontUser对象。

如果不想使用Django默认的引用属性名字，那么可以在“OneToOneField”中添加一个"related_name=‘checkin’"参数，那么以后FrontUser的对象就可以通过checkin属性来访问对应的FrontUserCheckIN对象。



