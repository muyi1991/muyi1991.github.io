---
layout: post
title: 自定义login_required装饰器及给普通视图和类视图添加装饰器
categories: [后端]
tags: [后端]
date: 2019-12-04 19:03:07
comments: true
---


#### 步骤

Django自带的login_required装饰器的重定向对于ajax请求是没有用的，所以需要我们自己去定义一个login_required装饰器。

1. 在utils包下面创建一个decoreators.py文件，在里面写login_requred代码
2. 在views.py中导入，然后视图函数中就可以使用了


#### 创建装饰器


```
apps
    ...
utils
    templatetags
        __init__.py
        time_filters.py
   __init__.py
   decorators.py // 在里面定义我们自己的装饰器
   restful.py    
```

decorators.py里面的代码如下：


```Python
from . import restful
from django.shortcuts import redirect, reverse


def xfz_login_required(func):
    # 视图函数可能接受参数用*args和**kwargs表示
    def wrapper(request, *args, **kwargs):
        if request.user.is_authenticated:
            return func(request,*args,**kwargs)
        else:
            if request.is_ajax():
                # 如果是ajax请求返回错误信息
                return restful.unauth(message='请先登录')
            else:
                # 如果不是ajax请求，让用户跳转登录页
                return redirect(reverse('index:login'))
    return wrapper
```


#### 给普通视图添加装饰器


```
from utils.decorators import xfz_login_required // 导入装饰器


@xfz_login_required //使用
def index(request):
    return render(request, 'index/index.html')
```


```
from django.shortcuts import render
from utils.decorators import xfz_login_required
from django.views.decorators.http import require_http_methods
from django.views.decorators.http import require_GET
from django.views.decorators.http import require_POST


@require_http_methods(['GET','POST'])
def profile(request):
    return render(request, 'account/profile.html')
    
@require_GET
def profile(request):
    return render(request, 'account/profile.html')   
    
@require_POST
def profile(request):
    return render(request, 'account/profile.html')     
     
@xfz_login_required
@require_http_methods(['GET'])
def profile(request):
   return render(request, 'account/profile.html')
   
@method_decorator(xfz_login_required,name='dispatch')
class profileView(View):
    def get(self,request,*args,**kwargs):
        return render(request, 'account/profile.html')
   
```


#### 给类视图添加装饰器


给类视图添加装饰可以直接装饰到dispatch这个方法上面，因为类视图最后会走dispatch这个方法。Django还提供了method_decorator这个非常好用的方法用来给类视图添加装饰器。

如果有多个装饰器可以使用列表把装饰器装起来，如:@method_decorator([xfz_login_required,xx_required], name='dispatch')。其中xx_required就是其他装饰器


```Python
from django.shortcuts import render
from django.views.generic import View
from utils import restful
from .forms import changePasswordForm
from utils.decorators import xfz_login_required
from django.utils.decorators import method_decorator //导入给类添加装饰器的方法


@xfz_login_required //给普通视图添加装饰器直接装饰在视图函数上面即可
def profile(request):
    if request.method == 'GET':
        return render(request, 'account/profile.html')


//给类添加装饰器需要装饰到dispatch这个方法上面，这个单词可能没有提示。name='dispatch'
//如果有多个装饰器可以使用列表把装饰器装起来，如：@method_decorator([xfz_login_required,xx_required], name='dispatch')
@method_decorator(xfz_login_required, name='dispatch')
class ChangePasswordView(View):
    def get(self,request,*args,**kwargs):
        return render(request,'account/changePassword.html')

    def post(self,request,*args,**kwargs):
        user = request.user
        password1 = request.POST.get('password1')
        password2 = request.POST.get('password2')
        form =changePasswordForm(request.POST)
        if form.is_valid():
            password1 = form.cleaned_data.get('password1')
            password2 = form.cleaned_data.get('password2')
            user.set_password(password1)
            user.save()
            return restful.ok()
        else:
            errors = form.get_errors()
            return restful.params_error(message=errors)

```

建议：为了维护代码统一，建议后台视图全部用类视图来写，这样更统一，也方便进行检查和维护。

