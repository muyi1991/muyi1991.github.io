---
layout: post
title: 权限在前端和后端视图中的使用
categories: [前端,后端]
tags: [权限]
date: 2019-12-26 13:48:07
comments: true
---


#### 步骤

超级管理员：在前端可以使用user.is_superuser来判断是否是超级管理员来判断是否显示相应的内容，在后端可以自定义一个装饰器来给视图函数装饰。auth中间件会返回员工是否是超级管理员。

普通员工：在前端可以通过中间件的权限来判断是否是有相应的权限，在后端可以使用装饰器来判断是否是有相应的权限。auth中间件会返回员工是否是有相应的权限。


#### 超级管理员

在前端的使用。使用user.is_superuser来判断返回值，来判断是否是超级用户。

```
user.is_superuser 
                <li class="nav-item dropdown">
                    <a id="dropdownSubMenu1" href="#" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" class="nav-link dropdown-toggle">员工</a>
                    <ul aria-labelledby="dropdownSubMenu1" class="dropdown-menu border-0 shadow">
                      <li><a href="{% url 'staff:staff_list' %}" class="dropdown-item">员工列表</a></li>
                    </ul>
                </li>

```

后端定义装饰器超级管理员装饰器


```
from . import restful
from django.shortcuts import redirect, reverse
from functools import wraps
from django.http import Http404


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


def xfz_superuser_required(viewfunc):
    @wraps(viewfunc)
    def decorator(request,*args,**kwargs):
        if request.user.is_superuser:
            return viewfunc(request,*args,**kwargs)
        else:
            raise Http404//不是超级管理员返回404错误
    return decorator
```

视图函数中使用


```
from django.shortcuts import render
from django.views.generic import View
from apps.cmsauth.models import User
from django.contrib.auth.models import Group
from utils import restful
from .forms import AddStaffForm, EditStaffForm
from utils.decorators import xfz_superuser_required,xfz_login_required//导入装饰器
from django.utils.decorators import method_decorator//导入类视图装饰器需要用到的装饰器


@method_decorator(xfz_superuser_required,name='dispatch')//使用超级管理员装饰器
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


#### 普通员工，普通权限

在前端使用。使用perms.product.change_product来判断是否拥有某个模型的权限，其中perms是中间件自带的，会返回给html页面perms，可以用来判断用户是否拥有权限，product是指模型的名称，change_product是指更改模型的权限，可以在auth_permission这张表里看到所有的权限，一般一个模型包含三种权限，add,change,delete。

使用perms.product.change_product来判断登录用户是否有相应的权限。


```
 perms.product.change_product
                <li class="nav-item dropdown">
                <a id="dropdownSubMenu1" href="#" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" class="nav-link dropdown-toggle">商品</a>
                <ul aria-labelledby="dropdownSubMenu1" class="dropdown-menu border-0 shadow">
                  <li><a href="{% url 'product:product_list' %}" class="dropdown-item">商品列表</a></li>
                  <li><a href="{% url 'product:add_product' %}" class="dropdown-item">新增商品</a></li>
                  <li><a href="{% url 'product:product_category' %}" class="dropdown-item">商品分类</a></li>
                  <li><a href="#" class="dropdown-item">商品回收站</a></li>
                  <li><a href="#" class="dropdown-item">商品评价</a></li>
                </ul>
                </li>

```

在后端视图函数中判断权限。
先导入permission_required，然后添加装饰器“@method_decorator(permission_required(perm="product.change_product",login_url='/'),name='dispatch')”，其中product是指某个模型，login_url是指如果不满足权限跳转哪个界面，/表示跳转首页。就是模型+模型权限就可以实现控制了。


```django
from django.shortcuts import render
from utils.decorators import xfz_login_required
from django.views.generic import View
from django.utils.decorators import method_decorator
from .models import Product
from .models import ProductCategory
from utils import restful
from .forms import EditCategoryForm,AddProductForm,EditProductForm
import os
import qiniu
from django.conf import settings
from django.core.paginator import Paginator
from datetime import datetime
from django.utils.timezone import make_aware
from urllib import parse
from django.contrib.auth.decorators import permission_required//导入权限装饰


@method_decorator(xfz_login_required,name='dispatch')
class Category(View):
    def get(self,request,*args,**kwargs):
        categories = ProductCategory.objects.all()
        context={
            'categories':categories
        }
        return render(request,'product/product_category.html',context=context)

    def post(self,request,*args,**kwargs):
        name = request.POST.get('name')
        exists = ProductCategory.objects.filter(name=name).exists()
        if not exists:
            ProductCategory.objects.create(name=name)
            return restful.ok()
        else:
            return restful.params_error(message='该分类已经存在')


class EditCategory(View):
    def post(self,request,*args,**kwargs):
        form = EditCategoryForm(request.POST)
        if form.is_valid():
            pk = request.POST.get('pk')
            name = request.POST.get('name')
            try:
                ProductCategory.objects.filter(pk=pk).update(name=name)
                return restful.ok()
            except:
                return restful.params_error(message='该分类不存在')
        else:
            return restful.params_error(message=form.get_errors())


class DeleteCategory(View):
    def post(self,request,*args,**kwargs):
        pk = request.POST.get('pk')
        try:
            ProductCategory.objects.get(pk=pk).delete()
            return restful.ok()
        except:
            return restful.params_error(message='该分类不存在')


@method_decorator(xfz_login_required,name='dispatch')
class DeleteProduct(View):
    def post(self,request,*args,**kwargs):
        product_id = request.POST.get('product_id')
        Product.objects.filter(pk=product_id).delete()
        return restful.ok()


class EditProduct(View):
    def get(self,request,*args,**kwargs):
        product_id = request.GET.get('product_id')
        product = Product.objects.get(pk=product_id)
        context = {
            'product':product,
            'categories': ProductCategory.objects.all()
        }
        return render(request,'product/add_product.html',context=context)

    def post(self,request,*args,**kwargs):
        form = EditProductForm(request.POST)
        if form.is_valid():
            title = form.cleaned_data.get('title')
            price = form.cleaned_data.get('price')
            thumbnail = form.cleaned_data.get('thumbnail')
            detail = form.cleaned_data.get('detail')
            category_id = form.cleaned_data.get('category')
            category = ProductCategory.objects.get(pk=category_id)
            pk = form.cleaned_data.get('pk')
            Product.objects.filter(pk=pk).update(title=title,price=price,thumbnail=thumbnail,detail=detail,category=category)
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


@method_decorator(permission_required(perm="product.change_product",login_url='/'),name='dispatch')//装饰这个类视图函数“模型+模型权限”就可以实现装饰了。
@method_decorator(xfz_login_required,name='dispatch')
class ProductList(View):
    def get(self,request,*args,**kwargs):
        page = int(request.GET.get('p',1))
        start = request.GET.get('start')
        end = request.GET.get('end')
        title = request.GET.get('title')
        category_id = int(request.GET.get('category',0) or 0)
        products = Product.objects.select_related('category').order_by('-pub_time')

        if start or end:
            if start:
                start_date = datetime.strptime(start, '%Y/%m/%d')
            else:
                start_date = datetime(year=2019,month=11,day=18)

            if end:
                end_date = datetime.strptime(end, '%Y/%m/%d')
            else:
                end_date = datetime.today()
            products = products.filter(pub_time__range=(make_aware(start_date),make_aware(end_date)))

        if title:
            products = products.filter(title__icontains=title)

        if category_id:
            products = products.filter(category=category_id)

        paginator = Paginator(products,20)
        page_obj = paginator.page(page)
        context_data = self.get_pagination_data(paginator,page_obj)
        context = {
            'categories' : ProductCategory.objects.all(),
            'products': page_obj.object_list,
            'paginator': paginator,
            'page_obj': page_obj,
            'start':start,
            'end':end,
            'title':title,
            'category_id':category_id,
            'url_query': '&'+parse.urlencode({
                'start': start or '',
                'end': end or '',
                'title': title or '',
                'category': category_id or '',
            })
        }
        # print(context['url_query'])
        context.update(context_data)
        return render(request, 'product/product_list.html',context=context)

    def get_pagination_data(self,paginator,page_obj,around_count=2):
        current_page = page_obj.number
        num_pages = paginator.num_pages

        left_has_more = False
        right_has_more = False

        if current_page <= around_count +2:
            left_pages = range(1,current_page)
        else:
            left_has_more = True
            left_pages = range(current_page-around_count,current_page)

        if current_page >= num_pages - around_count -1:
            right_pages = range(current_page+1, num_pages+1)
        else:
            right_has_more = True
            right_pages = range(current_page+1,current_page+around_count+1)

        return {
                'left_pages': left_pages,
                'right_pages': right_pages,
                'current_page': current_page,
                'left_has_more': left_has_more,
                'right_has_more': right_has_more,
                'num_pages': num_pages
        }


@method_decorator(xfz_login_required,name='dispatch')
class AddProduct(View):
    def get(self,request,*args,**kwargs):
        categories = ProductCategory.objects.all()
        context = {
            'categories':categories
        }
        return render(request, 'product/add_product.html',context=context)

    def post(self,request,*args,**kwargs):
        form = AddProductForm(request.POST)
        if form.is_valid():
            title = form.cleaned_data.get('title')
            price = form.cleaned_data.get('price')
            thumbnail = form.cleaned_data.get('thumbnail')
            detail = form.cleaned_data.get('detail')
            category_id = form.cleaned_data.get('category')
            category = ProductCategory.objects.get(pk=category_id)
            Product.objects.create(title=title,price=price,thumbnail=thumbnail,category=category,detail=detail)
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


@method_decorator(xfz_login_required,name='dispatch')
class UploadFile(View):
    def post(self,request,*args,**kwargs):
        file = request.FILES.get('file')
        name = file.name
        with open(os.path.join(settings.MEDIA_ROOT,name),'wb') as fp:
            for chunk in file.chunks():
                fp.write(chunk)
        url = request.build_absolute_uri(settings.MEDIA_URL+name)
        print(url)
        return restful.result(data={'url':url})


@method_decorator(xfz_login_required,name='dispatch')
class QnToken(View):
    def get(self,request,*args,**kwargs):
        access_key = settings.QINIU_ACCESS_KEY
        secret_key = settings.QINIU_SECRET_KEY
        bucket = settings.QINIU_BUCKET_NAME
        q = qiniu.Auth(access_key,secret_key)
        token = q.upload_token(bucket)
        # print(token)
        return restful.result(data={'token':token})



```




