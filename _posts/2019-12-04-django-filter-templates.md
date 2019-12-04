---
layout: post
title: 自定义模版过滤器格式化时间
categories: [django]
tags: [后端]
date: 2019-12-04 15:48:07
comments: true
---


#### 步骤
1. 在utils中，创建一个python包，叫做“templatetags”,必须叫这个名字，不然找不到。
2. 在这个“templatetags”包下面，创建一个python文件用来存储过滤器。
3. 在新建的python文件中，定义过滤器，写完后进行注册。
4. 把这个过滤器所在的这个app添加到"settings.INSTALLED_APPS"中，不然django也找不到这个过滤器。
5. 在模版中使用load标签加载过滤器所在的python包，在html中就可以使用了。


#### 创建templatetags包，创建过滤器文件


```
utils
    templatetages
        __init__.py
        time_filters.py
   __init__.py
   restful.py
```

#### 在time_filters.py中写过滤器代码


```python

#encoding: utf-8
from django import template
from datetime import datetime
from django.utils.timezone import now as now_func,localtime

register = template.Library()


@register.filter
def time_since(value):
    """
    time距离现在的时间间隔
    1.如果时间间隔小于1分钟以内，那么就显示“刚刚”
    2.如果是大于1分钟小于1小时，那么就显示“xx分钟前”
    3.如果是大于1小时小于24小时，那么就显示“xx小时前”
    4.如果是大于24小时小于30天以内，那么就显示“xx天前”
    5.否则就是显示具体的时间
    2017/10/20 16:15
    """
    if not isinstance(value,datetime):
        return value
    now = now_func()
    # timedelay.total_seconds
    timestamp = (now - value).total_seconds()
    if timestamp < 60:
        return '刚刚'
    elif timestamp >= 60 and timestamp < 60*60:
        minutes = int(timestamp/60)
        return '%s分钟前' % minutes
    elif timestamp >= 60*60 and timestamp < 60*60*24:
        hours = int(timestamp/60/60)
        return '%s小时前' % hours
    elif timestamp >= 60*60*24 and timestamp < 60*60*24*30:
        days = int(timestamp/60/60/24)
        return '%s天前' % days
    else:
        return value.strftime("%Y/%m/%d %H:%M")

@register.filter
def time_format(value):
    if not isinstance(value,datetime):
        return value

    return localtime(value).strftime("%Y/%m/%d %H:%M:%S")

```


#### 把这个过滤器所在的这个app添加到"settings.INSTALLED_APPS"中


```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'apps.cmsauth',
    'apps.index',
    'utils'//过滤器所在的app名字叫utils，也就是我们的工具包
]
```


#### load过滤器，在页面中使用


```
load time_filters 

<div class="form-group">
     <label for="">加入日期</label>
     <p>{{ user.data_joined|time_format }}</p> //格式化成年月日 时分秒
</div>
<div class="form-group">
     <label for="">最近登录</label>
     <p>{{ user.last_login|time_since }}</p>//格式化成刚刚，几分钟前，几小时前，几天前，年月日时分秒
</div>
```































