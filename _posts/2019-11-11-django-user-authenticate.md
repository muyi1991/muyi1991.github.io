---
layout: post
title: Django内置的User系统的验证、登录、退出登录
categories: [Django]
tags: []
date: 2019-11-30 08:48:07
comments: true
---


#### User系统验证、登录、退出实现步骤


步骤：
1. 在views.py的login_view视图中实现'GET‘请求
2. 在templates模版中完成登录html界面
3. 在forms.py中完成验证表单
2. 在views.py的login_view视图中实现'POST‘请求，完成验证、登录功能。
3. 利用Django自带的中间件使用user的信息。
3. 在views.py的logout_view视图中实现退出登录功能。


#### login_view视图中返回登录页面


```Python
def login_view(request):
    """GET请求返回登录模版"""
    if request.method == 'GET':
        return render(request, 'login.html')
```

#### 在templates模版中完成登录html界面


```Html
<form action="" method="post">
        <div class="input-group mb-3">
          <input type="text" class="form-control" name="telephone" placeholder="手机号">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-mobile"></span>
            </div>
          </div>
        </div>
        <div class="input-group mb-3">
          <input type="password" class="form-control" name="password" placeholder="密码">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-lock"></span>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-8">
            <div class="icheck-primary">
              <input type="checkbox" id="remember" name="remember" value="1">
              <label for="remember">
                记住我
              </label>
            </div>
          </div>
          <!-- /.col -->

          <div class="col-4">
            <button type="submit" class="btn btn-primary btn-block"> 登录 </button>
          </div>
          <!-- /.col -->
        </div>
      </form>
```

#### 在forms.py中完成验证表单


```Python
from django import forms
from django.core import validators
from apps.forms import FormMixin


class LoginForm(forms.Form, FormMixin):
    remember = forms.IntegerField(required=False)
    telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}', message='手机号格式不正确')], error_messages = {'required':'手机号必填'})
    password = forms.CharField(max_length=16,min_length=6,error_messages={'min_length':'密码最少6位','max_length':'密码不能超过16位', 'required':'密码必填'})


```

#### 完成验证登录

验证User的方法，导入login,logout,autnenticate方法。

验证：调用autnenticate方法，会返回一个user对象。
user = authenticate(request, username=telephone, password=password)

登录：调用login方法，传入已经已经验证的user对象。
login(request, user)

退出登录：调用logout方法即可。
logout(request)


```Python
from django.shortcuts import render, redirect, reverse
from django.contrib.auth import logout, login, authenticate
from .forms import LoginForm
from utils import restful


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
```

#### 利用Django自带的中间件在templates中调用user对象
如果已经完成授权登录，可以利用自带的中间件在所有模版中使用user对象。

首现确保已经开启使用自带的AuthenticationMiddleware。

```Python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

在模版中使用中间件。
{{ user.username }}

```Python
                    <a id="dropdownSubMenu1" href="#" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" class="nav-link dropdown-toggle">{{ user.username }}</a>

```

在视图函数中使用中间件。

```Python
def goods_list(request):
    user = request.user
    print(user.username)
    print(user.telephone)
    print(user.is_active)
    print(user.is_superuser)
    print(user.data_joined)
    return render(request, 'goods_list.html')
    
    杨建华
   13051706922
   True
   True
   2019-11-29 08:44:39.743826+00:00  
```


#### 实现退出登录


定义退出登录的视图函数
```Python
def logout_view(request):
    logout(request)
    return redirect(reverse('cmsauth:login'))
```

在templates模版中，访问一下视图函数即可退出登录
```Html
<li><a href="{% url 'cmsauth:logout' %}" class="dropdown-item">退出登录</a></li>
```




