---
layout: post
title: django自定义表单验证器
categories: [后端]
tags: [django,forms]
date: 2019-12-04 15:48:07
comments: true
---


#### 概述

定义一个简单的注册表单，输入手机号和密码注册，需要自定义验证的两个条件：
1. 检查手机号是否存在
2. 两次输入密码是否一致


#### forms.py文件


```Python
from django import forms
from apps.forms import FormMixin //表单组件，可以在视图函数中get_errors返回错误消息
from django.core import validators //django内置验证器，用正则来验证手机号
from .models import User //导入User，需要来查询手机号是否存在


class changePasswordForm(forms.Form,FormMixin):
telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}',message='手机号格式不正确')]) //利用自带的正则验证手机号格式是否正确
    password1 = forms.CharField(min_length=6,max_length=16,required=True,error_messages={'min_length':'密码最少6位', 'max_length':'密码不能超过16位','required':'新密码不能为空'})
    password2 = forms.CharField(min_length=6,max_length=16,required=True,error_messages={'min_length':'密码最少6位', 'max_length':'密码不能超过16位','required':'重复新密码不能为空'})

    def clean(self):
        //如果来到了clean方法，说明之前每一个字段都验证成功了
        cleaned_data = super().clean()
        telephone = cleaned_data.get('telephone')
        password1 = cleaned_data.get('password1')
        password2 = cleaned_data.get('password2')
        exists = User.objects.filter(telephone=telephone).exists()
        //通过User模型查询手机号是否已经注册
        if exists:
            raise forms.ValidationError(message='%s已经被注册'%telephone)
        if password1 != password2:
            //判断两次密码输入是否一致
            raise forms.ValidationError(message='两次密码输入不一致')
        //如果验证没有问题，把cleaned_data返回回去
        return cleaned_data

```





