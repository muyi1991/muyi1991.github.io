---
layout: post
title: 定义全局表单错误提取方法
categories: [Django]
tags: []
date: 2019-11-30 11:42:07
comments: true
---


#### 实现步骤


步骤：
1. 在apps下面的forms.py中写错误提取的组件
2. 在cmsauth的app下面调用FormMixin组件
3. 在视图函数中调用form.get_errors()方法


#### 在forms.py中写错误提取组件

注意不是在单个的app的forms中写，而是写一个全局的错误提取组件。
在apps下面写，所有的app都可以用。

```Python

class FormMixin(object):
    def get_errors(self):
        if hasattr(self,'errors'):
            errors = self.errors.get_json_data()
            new_errors = {}
            for key, message_dicts in errors.items():
                messages = []
                for message in message_dicts:
                    messages.append(message['message'])
                new_errors[key] = messages
            return new_errors
        else:
            return {}
```

#### 在cmsauth的app下面调用FormMixin组件

在apps下面的cmsauth这个app的forms.py中 调用写好的组件。

```Python
from django import forms
from django.core import validators
from apps.forms import FormMixin


class LoginForm(forms.Form, FormMixin):
    remember = forms.IntegerField(required=False)
    telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}', message='手机号格式不正确')], error_messages = {'required':'手机号必填'})
    password = forms.CharField(max_length=16,min_length=6,error_messages={'min_length':'密码最少6位','max_length':'密码不能超过16位', 'required':'密码必填'})


```


#### 在cmsauth的视图函数中调用form.get_errors()方法



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


