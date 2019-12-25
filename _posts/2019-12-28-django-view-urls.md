---
layout: post
title: 优化views和urls的小技巧
categories: [后端]
tags: []
date: 2019-12-28 13:48:07
comments: true
---


#### 步骤

随着views中的视图函数代码越写越多，会出现代码过长，不容易找到内容的问题，同时urls中的代码也越来越多，不太方便寻找。所以需要优化一些内容。

优化views，新建其他views，如一个app中有product，category。就可以分开写两个views，如：product_views.py和category_views.py。分成两个写。

优化urls，如果urls中有太多，就可以分开写。如相关的放一起，其他不相关的用+=来链接。


#### 优化views

新建一个category_views.py，全部用来处理和产品分类相关的视图函数。里面内容如下：


```django
from django.shortcuts import render
from utils.decorators import xfz_login_required
from django.views.generic import View
from django.utils.decorators import method_decorator
from .models import ProductCategory
from utils import restful
from .forms import EditCategoryForm


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
```


#### 优化urls


```
from django.urls import path
from . import views
from . import category_views

app_name = 'product'

# 和产品相关的url
urlpatterns = [
    path('product_list/', views.ProductList.as_view(), name='product_list'),
    path('add_product/', views.AddProduct.as_view(), name='add_product'),
    path('edit_product/', views.EditProduct.as_view(), name='edit_product'),
    path('delete_product/', views.DeleteProduct.as_view(), name='delete_product'),
    path('upload_file/', views.UploadFile.as_view(), name='upload_file'),
    path('qntoken/', views.QnToken.as_view(), name='qntoken'),
]

# 和产品分类相关的url
urlpatterns += [
    path('product_category/', category_views.Category.as_view(), name='product_category'),
    path('edit_category/', category_views.EditCategory.as_view(), name='edit_category'),
    path('delete_category/', category_views.DeleteCategory.as_view(), name='delete_category'),
]

```

这样优化之后会分工更明确，更清晰了。也更方便去寻找相应的内容了。


