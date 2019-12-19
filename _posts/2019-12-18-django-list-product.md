---
layout: post
title: 商品列表的布局和查询条件的布局
categories: [后端]
tags: [分页]
date: 2019-12-18 13:48:07
comments: true
---


#### 结构


```
    <div class="card ">
        <div class="card-header">
            <div class="card-title">
                导航的布局
            </div>
        </div>
        <div class="card-body">
            <div class="dataTables_wrapper dt-bootstrap4">
                <div class="row">
                    查询条件布局
                </div>
                <div class="row">
                    列表内容部分布局
                </div>
                <div class="row">
                    <div class="col-sm-12 col-md-5">
                        列表信息布局                    </div>
                    <div class="col-sm-12 col-md-7">
                        分页内容布局
                    </div>
                </div>
            </div>
        </div>
    </div>
```

#### 完整代码

```
    <div class="card ">
        <div class="card-header">
            <div class="card-title">
                <ul class="nav nav-pills">
                    <li class="nav-item">
                        <a href="#" class="nav-link active" data-toggle="tab">已发布</a>
                    </li>
                    <li class="nav-itme">
                        <a href="#" class="nav-link" data-toggle="tab">草稿箱</a>
                    </li>
                </ul>
            </div>
        </div>
        <div class="card-body">
            <div class="dataTables_wrapper dt-bootstrap4">
                <div class="row">
                    <form action="" method="get" class="form-inline col-12">
                        <label>//查询功能使用传统表单提交
                            时间：
                            {% if start %}
                                <input value="{{ start }}" type="text" id="start-picker" class="form-control" placeholder="起始时间" name="start" readonly>
                            {% else %}
                                <input type="text" id="start-picker" class="form-control" placeholder="起始时间" name="start" readonly>
                            {% endif %}
                            <span>-</span>
                            {% if end %}
                                <input value="{{ end }}" type="text" id="end-picker" class="form-control" placeholder="结束时间" name="end" readonly>
                            {% else %}
                                <input type="text" id="end-picker" class="form-control" placeholder="结束时间" name="end" readonly>
                            {% endif %}
                        </label>
                        <label>
                            标题：
                            {% if title %}
                                <input value="{{ title }}" type="text" class="form-control" placeholder="关键字" name="title">
                            {% else %}
                                <input type="text" class="form-control" placeholder="关键字" name="title">
                            {% endif %}
                        </label>
                        <label>
                            分类：
                            <select name="category" class="custom-select">
                                {% if category_id == 0 %}
                                    <option value="0" selected>全部分类</option>
                                {% else %}
                                    <option value="0">全部分类</option>
                                {% endif %}
                                {% for category in categories %}
                                    {% if category_id == category.pk %}
                                        <option selected value="{{ category.pk }}">{{ category.name }}</option>
                                    {% else %}
                                        <option value="{{ category.pk }}">{{ category.name }}</option>
                                    {% endif %}
                                {% endfor %}
                            </select>
                        </label>
                        <label>
                            <button class="btn btn-primary">查询</button>
                        </label>
                        <label>
                            <a href="{%  'product:product_list' %}">清除查询</a>
                        </label>
                    </form>
                </div>
                <div class="row">
                    <div class="col-12">
                        <table class="table table-bordered table-hover dataTable" role="grid" aria-describedby="example1_info">
                            <thead>
                                <tr>
                                    <th>缩略图</th>
                                    <th>标题</th>
                                    <th>分类</th>
                                    <th>发布时间</th>
                                    <th>操作</th>
                                </tr>
                            </thead>
                            <tbody>
                                {% for product in products %}
                                    <tr>
                                    <td><img class="img-rounded img-size-32 mr-2" src="{{ product.thumbnail }}" alt=""></td>
                                    <td>{{ product.title|truncatechars:18 }}</td>
                                    <td>{{ product.category.name }}</td>
                                    <td>{{ product.pub_time|time_format }}</td>
                                    <td>
                                        <a href="{% url 'product:edit_product' %}?product_id={{ product.pk }}" class="btn btn-primary btn-xs edit-btn">编辑</a>
                                        <button class="btn btn-danger btn-xs delete-btn delete-btn" data-product-id="{{ product.pk }}">删除</button>
                                    </td>
                                    </tr>
                                {% endfor %}
                            </tbody>
                            <tfoot>
                                <tr>
                                    <th>缩略图</th>
                                    <th>标题</th>
                                    <th>分类</th>
                                    <th>发布时间</th>
                                    <th>操作</th>
                                </tr>
                            </tfoot>
                        </table>
                    </div>
                </div>
                <div class="row">
                    <div class="col-sm-12 col-md-5">
                        <div class="dataTables_info">{{ paginator.count }}条数据</div>
                    </div>
                    <div class="col-sm-12 col-md-7">
                        <div class="dataTables_paginate page_simple_numbers" >
                            <ul class="pagination">
                                {#上一页#}
                                {% if page_obj.has_previous %}
                                    <li class="paginate_button page-item"><a class="page-link" href="?p={{ page_obj.previous_page_number }}{{ url_query }}">上一页</a></li>
                                {% else %}
                                    <li class="disabled"><a class="page-link" href="javascript:void(0);">上一页</a></li>
                                {% endif %}
                                {# 判断left_has_more #}
                                {% if left_has_more %}
                                    <li><a class="page-link" href="?p=1">1</a></li>
                                    <li><a class="page-link" class="page-link" href="javascript:void(0)">...</a></li>
                                {% endif %}
                                {#左边的页码#}
                                {% for left_page in left_pages %}
                                    <li><a class="page-link" href="?p={{ left_page }}{{ url_query }}">{{ left_page }}</a></li>
                                {% endfor %}
                                {# 当前的页码 #}
                                <li class="active paginate_button page-item"><a class="page-link" href="javascript:void(0)">{{ current_page }}</a></li>
                                {# 右边的页码 #}
                                {% for right_page in right_pages %}
                                    <li><a class="page-link" href="?p={{ right_page }}{{ url_query }}">{{ right_page }}</a></li>
                                {% endfor %}
                                {# 判断right_has_more #}
                                {% if right_has_more %}
                                    <li><a class="page-link" href="javascript:void(0)">...</a></li>
                                    <li><a class="page-link" href="?p={{ num_pages }}{{ url_query }}">{{ num_pages }}</a></li>
                                {% endif %}
                                {# 下一页 #}
                                {% if page_obj.has_next %}
                                    <li><a class="page-link" href="?p={{ page_obj.next_page_number }}{{ url_query }}">下一页</a></li>
                                {% else %}
                                    <li class="disabled"><a class="page-link" href="javascript:void(0);">下一页</a></li>
                                {% endif %}
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
```






