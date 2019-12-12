---
layout: post
title: Django后端开发的准备工作
categories: [后端]
tags: [后端]
date: 2019-12-02 13:48:07
comments: true
---


#### 实现步骤

步骤：
1. 配置好数据库
2. 配置好模版文件的路径
3. 配置好静态文件的路径
4. 配置好时区
5. 配置模版的static标签


#### 配置数据库

在Django的settings.py文件中配置好数据库相关信息。

```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', //数据库引擎
        'NAME': 'xfz', //数据库名称
        'USER': 'root', //用户名
        'PASSWORD': '123', //密码
        'HOST': '127.0.0.1', //host
        'PORT': '3306' //port
    }
}
```

#### 配置模版文件的路径

现在templates文件夹被放在cms/templates下面，所有需要重新配置。
在Django的setting.py文件中配置模版路径。

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'cms', 'templates')] //现在templates文件被放在cms文件下面，所以需要添加"cms"
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            'builtins':['django.templatetags.static']
        },
    },
]
```


#### 配置静态文件路径

现在静态文件夹被放在cms/dist下面，所有需要重新配置。
在Django的setting.py文件中配置静态文件路径。



```Python
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'cms', 'dist')//当前dist文件夹所在位置
]
```


#### 配置时区

在Django的setting.py文件中配置好时区。



```Python
LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = True
```
    
    
#### 配置模版的static标签    

为了每次在模版中使用静态文件都需要load一下，我们配置一下static的标签。我们把它作为内置的标签。



```Python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'cms', 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            'builtins':[
                'django.templatetags.static' //配置static标签
            ]
        },
    },
]
```

配置好了之后以后就可以直接用，不用每次都在模版中load一次了。
    










