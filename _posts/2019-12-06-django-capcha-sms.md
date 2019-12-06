---
layout: post
title: django发送短信验证码详解
categories: [django]
tags: [后端]
date: 2019-12-06 15:48:07
comments: true
---


#### 步骤

以修改密码为例，分为以下几个步骤来实现短信验证码的发送和验证。

1. 配置好第三方发送验证码的文件包和随机验证码的包
2. 定义修改密码的视图函数并映射
3. 完成前端html的代码
4. 在js中利用ajax向后段发送请求并反馈给页面


#### 配置好第三方发送验证码的文件包和随机验证码包

以聚合数据为例，首次可以免费发送10条，我们用它的api，来完成我们的发送验证码功能。
在utilts工具包中定义一个smssender.py文件和一个captcha包文件，在captcha包文件中写获取随机验证码的代码，在smssender中写发送验证码的代码。结构如下：

```
utils
    captcha //获取验证码的文件包
        __init__.py
        verdana.ttf //图形验证码的字体文件
        xfzcaptcha.py //获取验证码和图形验证码的文件
   templatetags //模版过滤器文件
   __init__.py //工具包的初始化包文件
   decorators.py //视图装饰器文件
   restful.py //restful的api文件
   smssender.py //这里是第三方发送短信验证码的包
```

xfzcaptcha.py中的内容如下：


```
# -*- coding: utf-8 -*-
# @Author: huangyong
# @Date:   2016-10-29 22:18:38
# @Last Modified by:   Administrator
# @Last Modified time: 2016-11-21 20:08:31
import random
# pip install Pillow
# Image:是一个画板(context),ImageDraw:是一个画笔, ImageFont:画笔的字体
from PIL import Image,ImageDraw,ImageFont
import time
import os
import string

# Captcha验证码

class Captcha(object):
    # 把一些常量抽取成类属性
    #字体的位置
    font_path = os.path.join(os.path.dirname(__file__),'verdana.ttf')
    # font_path = 'utils/captcha/verdana.ttf'
    #生成几位数的验证码
    number = 6 //定义几位数的验证码，我们以6位数纯数字验证码为例
    #生成验证码图片的宽度和高度
    size = (100,40)
    #背景颜色，默认为白色 RGB(Re,Green,Blue)
    bgcolor = (0,0,0)
    #随机字体颜色
    random.seed(int(time.time()))
    fontcolor = (random.randint(200,255),random.randint(100,255),random.randint(100,255))
    # 验证码字体大小
    fontsize = 20
    #随机干扰线颜色。
    linecolor = (random.randint(0,250),random.randint(0,255),random.randint(0,250))
    # 是否要加入干扰线
    draw_line = True
    # 是否绘制干扰点
    draw_point = True
    # 加入干扰线的条数
    line_number = 3

    # SOURCE = list(string.ascii_letters)//图形验证码加入了字母，短信不用，我们注释掉
    SOURCE = list() //我们给一个空的列表
    for index in range(0, 10):
        SOURCE.append(str(index))//在列表中添加1-9数字

    #用来随机生成一个字符串(包括英文和数字)
    # 定义成类方法,然后是私有的,对象在外面不能直接调用
    @classmethod
    def gene_text(cls)://调用这个方法获得随机6位验证码
        # return ''.join(random.sample(cls.SOURCE,cls.number))#number是生成验证码的位数
        return ''.join(random.sample(cls.SOURCE,cls.number))#number是生成验证码的位数

    #用来绘制干扰线
    @classmethod
    def __gene_line(cls,draw,width,height):
        begin = (random.randint(0, width), random.randint(0, height))
        end = (random.randint(0, width), random.randint(0, height))
        draw.line([begin, end], fill = cls.linecolor)

    # 用来绘制干扰点
    @classmethod
    def __gene_points(cls,draw,point_chance,width,height):
        chance = min(100, max(0, int(point_chance))) # 大小限制在[0, 100]
        for w in range(width):
            for h in range(height):
                tmp = random.randint(0, 100)
                if tmp > 100 - chance:
                    draw.point((w, h), fill=(0, 0, 0))

    #生成验证码
    @classmethod
    def gene_code(cls):
        width,height = cls.size #宽和高
        image = Image.new('RGBA',(width,height),cls.bgcolor) #创建画板
        font = ImageFont.truetype(cls.font_path,cls.fontsize) #验证码的字体
        draw = ImageDraw.Draw(image)  #创建画笔
        text = cls.gene_text() #生成字符串
        font_width, font_height = font.getsize(text)
        draw.text(((width - font_width) / 2, (height - font_height) / 2),text,font= font,fill=cls.fontcolor) #填充字符串
        # 如果需要绘制干扰线
        if cls.draw_line:
            # 遍历line_number次,就是画line_number根线条
            for x in range(0,cls.line_number):
                cls.__gene_line(draw,width,height)
        # 如果需要绘制噪点
        if cls.draw_point:
            cls.__gene_points(draw,10,width,height)

        return (text,image)
```


smssender.py 中的代码如下：

```
#encoding: utf-8

import requests

//模版id需要先申请模版，key在聚合的个人中心获取
def send(mobile,captcha):
    url = "http://v.juhe.cn/sms/send"//第三方的请求地址
    params = {
        "mobile": mobile,//手机号
        "tpl_id": "202060",//发送验证码的模版id
        "tpl_value": "#code#="+captcha,//发送的6位验证码内容
        "key": "9e9f65be15c4d4458299f5425b24cfe0"//我们自己的key
    }
    response = requests.get(url,params=params)
    result = response.json()
    print(result)
    if result['error_code'] == 0:
        return True
    else:
        return False
```


#### 定义修改密码的视图函数并映射


```
from django.shortcuts import render, redirect, reverse
from django.http import HttpResponse
from django.contrib.auth import logout, login, authenticate
from .forms import LoginForm, GetSmsCapchaForm, FindPasswordForm
from utils import restful
from utils.decorators import xfz_login_required
from django.views.generic import View
from django.views.decorators.http import require_POST,require_GET,require_http_methods
from utils import smssender
from utils.captcha.xfzcaptcha import Captcha
from django.core.cache import cache
from apps.cmsauth.models import User
from django.contrib.auth import get_user_model


@require_GET
def sms_capcha_view(request)://发送验证码的视图函数
    telephone = request.GET.get('telephone')
    # print(telephone)
    form = GetSmsCapchaForm(request.GET)
    if form.is_valid():
        capcha = Captcha.gene_text()
        cache.set(telephone, capcha, 5*60)
        print('验证码是:%s' % capcha)
        # result = smssender.send(telephone,capcha)
        # if result:
        #     return restful.ok()
        # else:
        #     return restful.params_error(message='短信验证码发送失败')
        return restful.ok()
    else:
        errors = form.get_errors()
        return restful.params_error(message=errors)


# def capcha_test(request):
#     cache.set('username','muyi',60)
#     result = cache.get('username')
#     print(result)
#     return HttpResponse('success')

//找回密码的视图函数
class FindPassword(View):
    def get(self,request,*args,**kwargs):
        return render(request,'index/findPassword.html')

    def post(self,request,*args,**kwargs):
        telephone = request.POST.get('telephone')
        password1 = request.POST.get('password1')
        sms_capcha = request.POST.get('sms_capcha')
        password2 = request.POST.get('password2')
        # print(cache.get(telephone))
        # print(telephone, password1,password2,sms_capcha)
        form = FindPasswordForm(request.POST)
        if form.is_valid():
            user = User.objects.filter(telephone=telephone).first()
            user.set_password(password1)
            user.save()
            return restful.ok()
        else:
            errors = form.get_errors()
            return restful.params_error(message=errors)


```

发送验证码和修改密码的两个表单验证

```
from django import forms
from django.core import validators
from apps.forms import FormMixin
from apps.cmsauth.models import User
from django.core.cache import cache


class LoginForm(forms.Form, FormMixin):
    remember = forms.IntegerField(required=False)
    telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}', message='手机号格式不正确')], error_messages = {'required':'手机号不能为空'})
    password = forms.CharField(max_length=16,min_length=6,error_messages={'min_length':'密码最少6位','max_length':'密码不能超过16位', 'required':'密码不能为空'})


class GetSmsCapchaForm(forms.Form, FormMixin):
    telephone = forms.CharField()

    def clean(self):
        cleaned_data = super(GetSmsCapchaForm, self).clean()
        telephone = cleaned_data.get('telephone')
        exists = User.objects.filter(telephone=telephone).exists()
        if not telephone:
            raise forms.ValidationError(message='手机号不能为空')
        if exists is False:
            raise forms.ValidationError(message='手机号在系统中不存在')
        return cleaned_data


class FindPasswordForm(forms.Form, FormMixin):
    telephone = forms.CharField()
    sms_capcha = forms.CharField(max_length=6,min_length=6,error_messages={'min_length':'验证码错误','max_length':'验证码错误', 'required':'验证码不能为空'})
    password1 = forms.CharField(min_length=6, max_length=16, required=True,error_messages={'min_length': '密码最少6位', 'max_length': '密码不能超过16位','required': '新密码不能为空'})
    password2 = forms.CharField(min_length=6, max_length=16, required=True,error_messages={'min_length': '密码最少6位', 'max_length': '密码不能超过16位','required': '重复新密码不能为空'})

    def clean(self):
        cleaned_data = super(FindPasswordForm, self).clean()
        telephone = cleaned_data.get('telephone')
        password1 = cleaned_data.get('password1')
        password2 = cleaned_data.get('password2')
        sms_capcha = cleaned_data.get('sms_capcha')
        cached_sms_capcha = cache.get(telephone)
        # print(sms_capcha,cached_sms_capcha)
        exists = User.objects.filter(telephone=telephone).exists()
        if not telephone:
            raise forms.ValidationError(message='手机号不能为空')
        if not exists:
            raise forms.ValidationError(message='手机号在系统中不存在')
        if sms_capcha != cached_sms_capcha:
            raise forms.ValidationError(message='验证码错误')
        if password1 != password2:
            raise forms.ValidationError(message='两次密码输入不一致')
        return cleaned_data

```

映射url


```
from django.urls import path
from . import views

app_name = 'index'

urlpatterns = [
    path('', views.index, name='index'),
    path('login/', views.login_view, name='login'),
    path('logout/', views.logout_view, name='logout'),
    path('getSmscapcha/', views.sms_capcha_view, name='getsmscapcha'),
    path('findPassword/', views.FindPassword.as_view(), name='findPassword'),
    # path('test/', views.capcha_test)
]
```

#### 完成前端html的代码

```
<div class="login-box">
  <div class="login-logo">
    <a href="#"></a>
      酷客管理系统
  </div>
  <!-- /.login-logo -->
  <div class="card">
    <div class="card-body login-card-body">
      <p class="login-box-msg">找回密码</p>

{#      <form action="" class="signin-group"  >#}
          {% csrf_token %}

        <div class="input-group mb-3">
                <input type="text" class="form-control" name="telephone" placeholder="手机号">
              <div class="input-group-append">
                <div class="input-group-text">
                  <span class="fas fa-phone"></span>
                </div>
              </div>
        </div>
        <div class="row">
          <div class="col-7">
            <div class="input-group mb-3">
                <input type="text" class="form-control" name="sms_capcha" placeholder="验证码">
              <div class="input-group-append">
                <div class="input-group-text">
{#                  <span class="fas fa-phone"></span>#}
                </div>
              </div>
            </div>
          </div>
          <!-- /.col -->

          <div class="col-5">
            <button  id="getcapcha-btn" class="btn btn-outline-primary btn-block">发送验证码</button>
          </div>
          <!-- /.col -->
        </div>
        <div class="input-group mb-3">
          <input type="password" class="form-control" name="password1" placeholder="新密码">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-lock"></span>
            </div>
          </div>
        </div>
        <div class="input-group mb-3">
          <input type="password" class="form-control" name="password2" placeholder="重复密码">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-lock"></span>
            </div>
          </div>
        </div>
        <div class="col-12">
            <button id="submit-btn" class="btn btn-primary btn-block">提交</button>
        </div>
{#      </form>#}
        <div class="social-auth-links text-center mb-3">
            <p>- OR -</p>
            <p class="mb-1">
                <a href="{% url 'index:login' %}">登录</a>
            </p>
        </div>

    </div>
    <!-- /.login-card-body -->
  </div>
</div>
```

#### 在js中利用ajax向后段发送请求并反馈给页面


```
function Auth() {
    var self = this;
    self.smsCapcha = $('#getcapcha-btn')//获取发送验证码按钮

}

Auth.prototype.run = function(){
    var self = this;
    self.listnSmsCapchaEvent();//初始化监听发送验证码事件
    self.listenSubmitEvent();//监听提交事件
};

Auth.prototype.smsSuccessEvent = function(){
    var self = this;
    window.messageBox.showSuccess('短信验证码发送成功');
    self.smsCapcha.addClass('disabled');//给发送验证码按钮禁用添加disabled样式
    var count =60;
    self.smsCapcha.unbind('click');//把发送验证码按钮禁用掉点击事件
    var timer = setInterval(function () {//60秒倒计时
        self.smsCapcha.text(count+'S');//添加按钮文字为60s倒计时
        count--;
        if (count <= 0){//如果小于0秒恢复按钮的点击状态
            clearInterval(timer);//清除掉定时器
            self.smsCapcha.removeClass('disabled');//移除禁用样式
            self.smsCapcha.text('发送验证码');//回复按钮文字为发送验证码
            self.listnSmsCapchaEvent();
        }
    },1000)
};

Auth.prototype.listnSmsCapchaEvent =function(){
    var self = this;
    var smsCapcha = $('#getcapcha-btn');//获取发送验证码按钮
    var telephoneInput = $("input[name='telephone']");
    smsCapcha.click(function () {
        var telephone = telephoneInput.val();
        // console.log(telephone);
        xfzajax.get({
            'url':'/getSmscapcha/',
            'data':{
                'telephone': telephone
            },
            'success':function (result) {
                // console.log(result);
                if (result['code'] === 200){
                    self.smsSuccessEvent();
                }
            },
            'fail':function (error) {
                console.log(error)
            }
        })
    })
};

Auth.prototype.listenSubmitEvent = function(){
    var submitBtn = $("#submit-btn");//获取提交按钮
    var telephontInput = $("input[name='telephone']")
    var sms_capchaInput = $("input[name='sms_capcha']")
    var password1Input = $("input[name='password1']")
    var password2Input = $("input[name='password2']")
    submitBtn.click(function (event) {
        event.preventDefault();
        var telephone = telephontInput.val();
        var sms_capcha = sms_capchaInput.val();
        var password1 = password1Input.val();
        var password2 = password2Input.val();
        console.log(sms_capcha)
        xfzajax.post({
            'url':'/findPassword/',
            'data':{
                'telephone':telephone,
                'sms_capcha':sms_capcha,
                'password1':password1,
                'password2':password2
            },
            'success':function (result) {
                console.log(result)
                if (result['code'] === 200){
                    window.messageBox.showSuccess('成功找回密码');//成功找回密码toast弱提示
                    setTimeout(function () {//成功找回密码跳转登录页面
                        window.location.replace(/login/)
                    },1500)
                }
            },
            'fail':function (error) {
                console.log(error)
            }
        })
    })
};


$(function () {
    var auth = new Auth();
    auth.run()
});
```


