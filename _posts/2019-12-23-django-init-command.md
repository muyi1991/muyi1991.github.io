---
layout: post
title: 自定义django的command命令
categories: [后端]
tags: [command]
date: 2019-12-23 13:48:07
comments: true
---


#### 步骤

我们通常会使用"startapp","makemigrations","migrate"等等这些django自带的命令，但是有时候我们需要自定义一些命令来做一些事情如：初始化一些不经常改变的数据，分组权限，等等...
以初始化分组为例，以下就是自定义命令的步骤。

1. 在需要自定义命令的app下面创建一个management的python包
2. 在management这个包下面再创建一个commands的python包
3. 在commands这个包下面创建initgroup的python文件
4. 在文件中写分组权限的内容
5. 在run manage.py task下面运行"initgroup"这个命令，就可以运行写的内容了

完成后的目录如下：

```
apps
    cmsauth
        management
            commands
                __init__.py
                initgroup.py
            __init__.py
        migrations
        __init__.py
        admin.py
        apps.py
        forms.py
        models.py
        tests.py
        urls.py
        views.py  
```

#### 自定义django命令创建用户


```
from django.core.management.base import BaseCommand//必须导入
from apps.cmsauth.models import User,Area


class Command(BaseCommand)://必须是Command，定义一个类视图
    def handle(self, *args, **options)://定义handle函数去写自己需要处理的事情
        area = Area.objects.filter(name='长三角区域').first()
        # area = Area.objects.create(name='珠三角区域')
        User.objects.create_superuser(telephone='13344445555',username='王五',password=123456,area=area)
        self.stdout.write(self.style.SUCCESS('create superuser success'))//不能使用print()，必须使用这样才能向控制台输入提示语句
```

定义完成后在run manage.py task 中运行"initgroup",也就是commands下面文件名称，然后就可以看到输出内容了。

同时也可以在终端运行"python manage.py initgroup",也可以运行自定义的命令了。当然需要进入到apps下面进入到这个项目的目录下面才可以。

#### 自定义django命令创建分组权限


```
from django.core.management.base import BaseCommand//导入基础command命令
from django.contrib.auth.models import Group,Permission,ContentType//导入分组，权限等
from apps.product.models import Product,ProductCategory,ProductComment//导入模型
from apps.cmsauth.models import User//导入模型


class Command(BaseCommand):
    def handle(self, *args, **options):
        # 1,产品组，产品、分类、评论权限
        product_content_type = [//根据模型找到产品组的content_type
            ContentType.objects.get_for_model(Product),//利用get_for_model找到content_type
            ContentType.objects.get_for_model(ProductCategory),
            ContentType.objects.get_for_model(ProductComment),
        ]
        product_permissions = Permission.objects.filter(content_type__in=product_content_type)//利用content_type找到产品组所有权限
        productGroup = Group.objects.create(name='产品组')//创建产品组
        productGroup.permissions.set(product_permissions)//给产品组添加权限
        productGroup.save()//保存产品组
        self.stdout.write(self.style.SUCCESS('产品组创建成功'))//向终端输出创建成功信息
        # 2,人事组， 前台用户、后台用户权限
        human_content_type = [
            ContentType.objects.get_for_model(User),
        ]
        human_permissions = Permission.objects.filter(content_type__in=human_content_type)
        humanGroup = Group.objects.create(name='人事组')
        humanGroup.permissions.set(human_permissions)
        humanGroup.save()
        self.stdout.write(self.style.SUCCESS('人事组创建成功'))
        # 3,管理员组，产品组、人事组权限
        admin_permissions = product_permissions.union(human_permissions)
        adminGroup = Group.objects.create(name='管理员组')
        adminGroup.permissions.set(admin_permissions)
        adminGroup.save()
        # 4,超级管理员
        self.stdout.write(self.style.SUCCESS('管理组创建成功'))
```









