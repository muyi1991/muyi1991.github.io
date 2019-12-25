---
layout: post
title: django数据库表关系之一对多
categories: [后端]
tags: [数据库]
date: 2019-12-29 13:48:07
comments: true
---


#### 概述

一对多关系是使用的比较多的情况，通过ForeignKey来实现。举例：文章和分类的关系就是一对多，员工和地区的关系也是一对多。

```
查看文章的分类：文章.分类.name  
查看分类的文章：分类.文章_set.all()   
注意分类访问文章需要文章的小写形式+“_set”

查看员工的地区：员工.地区.name
查看地区的文章：地区.员工_set.all()   
注意地区访问员工需要员工的小写形式+"_set"
```

#### 一对多模型(地区和员工)

```
from django.db import models
from django.contrib.auth.models import BaseUserManager
from django.contrib.auth.models import AbstractBaseUser
from django.contrib.auth.models import PermissionsMixin


class UserManager(BaseUserManager):
    """重写user的object模型"""
    def _create_user(self, telephone, username, password, **kwargs):
        if not telephone:
            raise ValueError('手机号不能为空')
        if not username:
            raise ValueError('姓名不能为空')
        if not password:
            raise ValueError('密码不能为空')
        user = self.model(telephone=telephone, username=username, **kwargs)
        user.set_password(password)
        user.save()
        return user
    """创建普通用户"""
    def create_user(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = False
        self._create_user(telephone=telephone, username=username, password=password, **kwargs)
    """创建超级用户"""
    def create_superuser(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = True
        self._create_user(telephone=telephone, username=username, password=password, **kwargs)


class User(AbstractBaseUser, PermissionsMixin):
    """重新定义user模型"""
    telephone = models.CharField(max_length=11, unique=True)
    email = models.CharField(max_length=100, null=True)
    username = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)
    data_joined = models.DateTimeField(auto_now_add=True)
    area = models.ForeignKey('Area',on_delete=models.PROTECT,null=False)//如果设置了"related_name='users'"。那么就可以不用area.user_set.all()，可以用area.users.all()来查询某个分类下的所有用户了

    """设置手机号为验证方式"""
    USERNAME_FIELD = 'telephone'
    REQUIRED_FIELDS = ['username']

    """把重写objects模型赋值到重写的user模型中"""
    objects = UserManager()

    def get_full_name(self):
        return self.username

    def get_short_name(self):
        return self.username


class Area(models.Model):
    """定义区域模型"""
    name = models.CharField(max_length=100,unique=True,null=False)

    class Meta:
        db_table = 'area'
        //ording = '-pub_time' 可以指定模型的排序方式，如发布时间排序
        
    def __str__(self):
        reture "<Area:(id:%s,name:%s)>" % (self.id,self.name) 
    //这样可以让视图函数中的print函数可以输出area，可以输出区域的id和名称

```


#### 视图函数中使用


```
from django.shortcuts import render
from django.views.generic import View
from apps.cmsauth.models import User,Area
from django.contrib.auth.models import Group
from utils import restful
from .forms import AddStaffForm, EditStaffForm
from utils.decorators import xfz_superuser_required,xfz_login_required
from django.utils.decorators import method_decorator


@method_decorator(xfz_superuser_required,name='dispatch')
class StaffListView(View):
    def get(self,request,*args,**kwargs):
        area = request.user.area
        # staffs = User.objects.filter(area=area)//也可以通过查询user对象里面所有地区登录该地区的员工，结果是一样的
        staffs = area.user_set.all()//推荐用法"地区+用户_set"查询某个地区所有员工
        //如果设置了"related_name='users'"。那么就可以用staffs = area.users.all()来访问某区域下的所有用户，效果是一样的
        context = {
            'staffs':staffs
        }
        return render(request,'staff/staff_list.html',context=context)


class AddStaffView(View):
    def get(self,request,*args,**kwargs):
        groups = Group.objects.all()
        context = {
            'groups':groups
        }
        return render(request,'staff/add_staff.html',context=context)

    def post(self,request,*args,**kwargs):
        form = AddStaffForm(request.POST)
        if form.is_valid():
            area = request.user.area
            username = form.cleaned_data.get('username')
            telephone = form.cleaned_data.get('telephone')
            email = form.cleaned_data.get('email')
            groups_ids = request.POST.getlist('groups')
            if not groups_ids:
                return restful.params_error(message='分组不能为空')
            # print(groups_ids)
            # print(username,telephone,email)

            password = '123456'
            user = User.objects.create(username=username,telephone=telephone,email=email,area=area)
            user.set_password(password)
            groups = Group.objects.filter(pk__in=groups_ids)
            # print(groups)
            user.groups.set(groups)
            user.save()
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


class EditStaffView(View):
    def get(self,request,*args,**kwargs):
        user_id = request.GET.get('user_id')
        user = User.objects.filter(pk=user_id).first()
        user_groups = Group.objects.filter(user=user)
        # print(user_groups)
        groups = Group.objects.all()
        # print(groups)
        context = {
            'staff': user,
            'user_groups': user_groups,
            'groups' : groups
        }
        return render(request,'staff/add_staff.html',context=context)

    def post(self,request,*args,**kwargs):
        form = EditStaffForm(request.POST)
        if form.is_valid():
            user_id = form.cleaned_data.get('user_id')
            username = form.cleaned_data.get('username')
            telephone = form.cleaned_data.get('telephone')
            email = form.cleaned_data.get('email')
            groups_ids = request.POST.getlist('groups')
            if not groups_ids:
                return restful.params_error(message='分组不能为空')
            # print(groups_ids)
            # print(username, telephone, email)

            User.objects.filter(pk=user_id).update(username=username,telephone=telephone,email=email)
            user = User.objects.filter(pk=user_id).first()
            groups = Group.objects.filter(pk__in=groups_ids)
            # print(groups)
            user.groups.set(groups)
            user.save()
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


class DeleteStaffView(View):
    def post(self,request,*args,**kwargs):
        user_id = request.POST.get('user_id')
        User.objects.filter(pk=user_id).delete()
        return restful.ok()


```




