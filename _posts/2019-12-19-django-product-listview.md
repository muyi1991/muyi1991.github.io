---
layout: post
title: 商品分页功能的实现
categories: [后端]
tags: []
date: 2019-12-19 13:48:07
comments: true
---


#### 步骤

1. 导入Paginator包
2. 构建paginator对象
3. 获取page_obj
4. 获取其他分页需要数据

Paginator代表整个分页的对象，page_obj代表当前这一页的相关数据。


#### 完整代码

```
from django.shortcuts import render
from utils.decorators import xfz_login_required
from django.views.generic import View
from django.utils.decorators import method_decorator
from .models import Category,Product
from utils import restful
from .forms import EditCategoryForm,AddProductForm,EditProductForm
import os
import qiniu
from django.conf import settings
from django.core.paginator import Paginator //导入Paginator包
from datetime import datetime//导入datetime文件
from django.utils.timezone import make_aware//把输入查询的幼稚时间变成清醒时间
from urllib import parse//通过parse可以将一些查询字段变成一个url_query


@method_decorator(xfz_login_required,name='dispatch')
class ProductCategory(View):
    def get(self,request,*args,**kwargs):
        categories=Category.objects.all()
        context={
            'categories':categories
        }
        return render(request,'product/product_category.html',context=context)

    def post(self,request,*args,**kwargs):
        name = request.POST.get('name')
        exists = Category.objects.filter(name=name).exists()
        if not exists:
            Category.objects.create(name=name)
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
                Category.objects.filter(pk=pk).update(name=name)
                return restful.ok()
            except:
                return restful.params_error(message='该分类不存在')
        else:
            return restful.params_error(message=form.get_errors())


class DeleteCategory(View):
    def post(self,request,*args,**kwargs):
        pk = request.POST.get('pk')
        try:
            Category.objects.get(pk=pk).delete()
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
            'categories': Category.objects.all()
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
            category = Category.objects.get(pk=category_id)
            pk = form.cleaned_data.get('pk')
            Product.objects.filter(pk=pk).update(title=title,price=price,thumbnail=thumbnail,detail=detail,category=category)
            return restful.ok()
        else:
            return restful.params_error(message=form.get_errors())


@method_decorator(xfz_login_required,name='dispatch')
class ProductList(View):
    def get(self,request,*args,**kwargs):
        page = int(request.GET.get('p',1))//获取page页码，如果没有传递，默认1，通过get获取到的值都是字符串，需要用int()方法变成int类型
        start = request.GET.get('start')
        end = request.GET.get('end')
        title = request.GET.get('title')
        category_id = int(request.GET.get('category',0) or 0)
        products = Product.objects.select_related('category')

        if start or end:
            if start:
                start_date = datetime.strptime(start, '%Y/%m/%d')
            else:
                start_date = datetime(year=2019,month=11,day=18)

            if end:
                end_date = datetime.strptime(end, '%Y/%m/%d')
            else:
                end_date = datetime.today()
            products = products.filter(pub_time__range=(make_aware(start_date),make_aware(end_date)))//把products重新赋值通过查询的字符串，使用make_aware()把幼稚时间变成清醒时间

        if title://如果有关键字，通过icontains查询包含的字符串
            products = products.filter(title__icontains=title)

        if category_id://通过category_id查询商品分类
            products = products.filter(category=category_id)

        paginator = Paginator(products,8)//构建一个Paginator对象，8是一页显示的数量，可以根据需求修改
        page_obj = paginator.page(page)//获取到当前页的对象
        context_data = self.get_pagination_data(paginator,page_obj)//传入paginator和page_obj获取分页需要的数据
        context = {
            'categories': Category.objects.all(),
            'products': page_obj.object_list,//当前页的每条数据
            'paginator': paginator,
            'page_obj': page_obj,
            'start':start,
            'end':end,
            'title':title,
            'category_id':category_id,
            'url_query': '&'+parse.urlencode({//查询的url_query
                'start': start or '',
                'end': end or '',
                'title': title or '',
                'category': category_id or '',
            })
        }
        # print(context['url_query'])
        context.update(context_data)//把context_data放到context里面
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
        categories = Category.objects.all()
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
            category = Category.objects.get(pk=category_id)
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




