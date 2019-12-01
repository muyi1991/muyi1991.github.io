---
layout: post
title: 在Django中定义全局restful的api
categories: [Django]
tags: []
date: 2019-11-30 11:48:07
comments: true
---


#### 实现步骤


步骤：
1. 在apps的同级创建utils包，然后把restful.py文件放里面。
2. 在视图函数中调用restful


#### 创建restful.py工具包

在apps的同级目录上创建一个Python包，取名叫utils，存放工具包。结构如下。
在utils包下面创建一个restful.py文件。

```Python
apps
utils
    __init__.py
    restful.py
```

在restful.py文件里面写方法。


```Python
#encoding: utf-8
from django.http import JsonResponse


class HttpCode(object):
    ok = 200
    paramserror = 400
    unauth = 401
    methoderror = 405
    servererror = 500


# {"code":400,"message":"","data":{}}
def result(code=HttpCode.ok,message="",data=None,kwargs=None):
    json_dict = {"code":code,"message":message,"data":data}

    if kwargs and isinstance(kwargs,dict) and kwargs.keys():
        json_dict.update(kwargs)

    return JsonResponse(json_dict)


def ok():
    return result()


def params_error(message="",data=None):
    return result(code=HttpCode.paramserror,message=message,data=data)


def unauth(message="", data=None):
    return result(code=HttpCode.unauth,message=message,data=data)


def method_error(message='',data=None):
    return result(code=HttpCode.methoderror,message=message,data=data)


def server_error(message='',data=None):
    return result(code=HttpCode.servererror,message=message,data=data)


```


#### 在视图函数调用restful


```Python
from django.shortcuts import render, redirect, reverse
from django.contrib.auth import logout, login, authenticate
from .forms import LoginForm
from utils import restful


def index(request):
    return render(request, 'index.html')


def login_view(request):
    """GET请求返回登录模版"""
    if request.method == 'GET':
        return render(request, 'login.html')
    else:
        form = LoginForm(request.POST)
        if form.is_valid():
            telephone = form.cleaned_data.get('telephone')
            password = form.cleaned_data.get('password')
            remember = form.cleaned_data.get('remember')
            user = authenticate(request, username=telephone, password=password)
            if user:
                if user.is_active:
                    login(request, user)
                    if remember:
                        request.session.set_expiry(None)
                        return redirect(reverse('cmsauth:index'))
                    else:
                        request.session.set_expiry(0)
                        next_url = request.GET.get('next')
                        if next_url:
                            return redirect(next_url)
                        else:
                            return redirect(reverse('cmsauth:index'))
                else:
                    return restful.unauth(message='账号已经被冻结')

            else:
                return restful.params_error(message='手机号或密码错误')
        else:
            errors=form.get_errors()
            return restful.params_error(message=errors)


def logout_view(request):
    logout(request)
    return redirect(reverse('cmsauth:login'))

```




