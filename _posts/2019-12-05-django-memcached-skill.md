---
layout: post
title: memcached的安装和使用
categories: [django]
tags: [后端]
date: 2019-12-05 15:48:07
comments: true
---


#### memchached的安装

参考链接：https://blog.csdn.net/weixin_37696997/article/details/78574683

brew install libevent 安装依赖包
brew install memcached 安装memcached
brew services start memcached 启动服务


#### Django中memecached的使用

1. 首现在setting.py中配置使用memecached的
2. 使用时导入memecached文件，存储和提取数据

第一步：在setting.py中配置
```python
# 缓存配置
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211'
    }
}
```

第二部：导入使用的包并使用


```
from django.core.cache import cache //导入使用的包

cache.set(telephone,code,5*60) //telephone是key，code验证码是value，5*60是有效期五分钟，使用时通过key提取数据

code = cache.get(telephone) //这样就能提取到存储的值

```



    


