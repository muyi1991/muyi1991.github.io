---
layout: post
title: Django数据库表关系之多对多
categories: [后端]
tags: [数据库]
date: 2020-1-1 13:48:07
comments: true
---


#### 概述

文章和标签的关系，一个标签可以对应多个文章，一个文章也可以对应多个标签，因此文章和标签的关系就是典型的多对多的关系。


#### 实现方式

Django为这种多对多的关系提供了专门的field，叫做MangToManyField，拿文章和标签举例，代码如下：

注意：这里没有on_delte这个设置属性。on_delete这个属性只有在外健的时候才有。

```python
class Tag(models.Model):
    name = models.CharField(max_length=100)
    articles = models.ManyToManyField("Article",related_name='tags')//注意没有on_delete这个属性。

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    category = models.ForeignKey("Category",on_delete=models.CASCADE,null=True,related_name='articles')
    author = models.ForeignKey("frontuser.FrontUser",on_delete=models.CASCADE,null=True)
```

#### 多对多的查询


```python
def many_to_many_view(request):
    # article = Article.objects.first()
    # tag = Tag(name='冷门文章')
    # tag.save()
    # article.tag_set.add(tag)

    # tag = Tag.objects.get(pk=1)
    # article = Article.objects.get(pk=3)
    # tag.articles.add(article)

    article = Article.objects.get(pk=1)
    tags = article.tags.all()
    for tag in tags:
        print(tag)

    return HttpResponse("success")
```




