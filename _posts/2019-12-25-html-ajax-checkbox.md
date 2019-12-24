---
layout: post
title: 使用ajax将复选框的值提交到后台
categories: [前端]
tags: []
date: 2019-12-25 13:48:07
comments: true
---


#### 步骤

现在html页面构建一个复选框，然后给复选框一个name属性，通过ajax发送给后台视图函数，注意发送的时候除了data还需要发送一个属性“'traditional':true”（这个属性可以让ajax传数组或者对象），不然后台接受不到，然后后台用getlist就可以接收到值。

1. 构建复选框
2. 通过ajax发送给后台
3. 后台接受数据


#### html页面


```html
<div class="form-group">
                <label for="">分组</label>
                <div class="icheck-primary">
                  {% for group in groups %}
                      {% if staff %}
                          {% if group in user_groups %}
                              <input checked type="checkbox" id="{{ group.pk }}" name="groups" value="{{ group.pk }}">
                              <label for="{{ group.pk }}">
                                    {{ group.name }}
                              </label>
                          {% elif group not in user_groups %}
                              <input  type="checkbox" id="{{ group.pk }}" name="groups" value="{{ group.pk }}">
                              <label for="{{ group.pk }}">
                                    {{ group.name }}
                              </label>
                          {% endif %}
                      {% else %}
                          <input type="checkbox" id="{{ group.pk }}" name="groups" value="{{ group.pk }}">
                          <label for="{{ group.pk }}">
                                {{ group.name }}
                          </label>
                      {% endif %}
                  {% endfor %}
                </div>
            </div>
```

#### 通过ajax向后台发送请求


```js
function Staff() {

}

Staff.prototype.ListenSubmitEvent = function(){
    var submitBtn = $("#submit-btn");
    var user_id = submitBtn.attr('data-user-id');
    submitBtn.click(function () {
        var username = $("input[name ='username' ]").val();
        var telephone = $("input[name ='telephone' ]").val();
        var email = $("input[name ='email' ]").val();
        var groups = [];//定义一个空数据
        $("input[name='groups']:checked").each(function(i){
            groups.push($(this).val());//把选中的值保存到数组
        });
        if(user_id){
            url = '/staff/edit_staff/'
        }else{
            url = '/staff/add_staff/'
        }
        console.log(username,telephone,email,groups);
        xfzajax.post({
            'url': url,
            'traditional':true,//切记，一定要配置，不然后台接受不到数据
            'data':{
                'username': username,
                'telephone': telephone,
                'email': email,
                'groups': groups,//把数组传递过去
                'user_id':user_id
            },
            'success':function (result) {
                if (result['code'] === 200){
                    window.location.replace("/staff/staff_list/")
                }
            },
            'fail':function (error) {
                console.log(error)
            }
        })
    })

};

Staff.prototype.run = function(){
    var self = this;
    self.ListenSubmitEvent();
};

$(function () {
   var staff = new Staff();
    staff.run()
});
```

#### 后台接受数据，处理请求


```django
from django.shortcuts import render
from django.views.generic import View
from apps.cmsauth.models import User
from django.contrib.auth.models import Group
from utils import restful
from .forms import AddStaffForm, EditStaffForm


class StaffListView(View):
    def get(self,request,*args,**kwargs):
        user_area = request.user.area
        staffs = User.objects.filter(area=user_area)
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
            groups_ids = request.POST.getlist('groups')//通过getlist获取groups数据
            if not groups_ids:
                return restful.params_error(message='分组不能为空')
            # print(groups_ids)
            # print(username,telephone,email)

            password = '123456'
            user = User.objects.create(username=username,telephone=telephone,email=email,area=area)
            user.set_password(password)//设置密码
            groups = Group.objects.filter(pk__in=groups_ids)//通过groups的id找到用户的所有分组
            # print(groups)
            user.groups.set(groups)//把所有分组设置到用户上
            user.save()//保存用户
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


class EditStaffView(View):
    def get(self,request,*args,**kwargs):
        user_id = request.GET.get('user_id')
        user = User.objects.filter(pk=user_id).first()
        user_groups = Group.objects.filter(user=user)//找到该用户的所有分组
        # print(user_groups)
        groups = Group.objects.all()//找到系统中所有分组
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
            user_id = form.cleaned_data.get('user_id')//获取用户id
            username = form.cleaned_data.get('username')
            telephone = form.cleaned_data.get('telephone')
            email = form.cleaned_data.get('email')
            groups_ids = request.POST.getlist('groups')
            if not groups_ids:
                return restful.params_error(message='分组不能为空')
            # print(groups_ids)
            # print(username, telephone, email)

            User.objects.filter(pk=user_id).update(username=username,telephone=telephone,email=email)//更新用户数据
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


#### 添加和编辑员工的表单


```django
from django import forms
from apps.forms import FormMixin
from django.core import validators
from apps.cmsauth.models import User


class AddStaffForm(forms.Form,FormMixin):
    email = forms.EmailField(max_length=100,required=False)
    telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}', message='手机号格式不正确')],
                                error_messages={'required': '手机号不能为空'})
    username = forms.CharField(max_length=4,min_length=2,error_messages={'min_length':'姓名最少2位','max_length':'姓名不能超过4位', 'required':'姓名不能为空'})

    def clean(self):
        cleaned_data = super(AddStaffForm,self).clean()
        telephone = cleaned_data.get('telephone')
        exists = User.objects.filter(telephone=telephone).exists()
        if exists:
            raise forms.ValidationError(message='手机号已经存在')
        return cleaned_data


class EditStaffForm(forms.Form,FormMixin):
    user_id = forms.CharField(error_messages={'required':'user_id不能为空'})
    email = forms.EmailField(max_length=100,required=False)
    telephone = forms.CharField(validators=[validators.RegexValidator(r'1[345678]\d{9}', message='手机号格式不正确')],
                                error_messages={'required': '手机号不能为空'})
    username = forms.CharField(max_length=4,min_length=2,error_messages={'min_length':'姓名最少2位','max_length':'姓名不能超过4位', 'required':'姓名不能为空'})

```




