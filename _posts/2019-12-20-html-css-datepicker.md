---
layout: post
title: 初始化datepicker日期控件
categories: [前端]
tags: []
date: 2019-12-20 14:48:07
comments: true
---


#### 步骤

1. 引用datepicker的js文件
2. 完成html
3. 在js中初始化datepicker控件


#### 引用js


```
<link rel="stylesheet" href="{% 'adminlte/bower_components/bootstrap-datepicker/dist/css/bootstrap-datepicker.min.css' %}">
    <script src="{% 'adminlte/bower_components/bootstrap-datepicker/dist/js/bootstrap-datepicker.min.js' %}"></script>
    <script src="{% 'adminlte/bower_components/bootstrap-datepicker/dist/locales/bootstrap-datepicker.zh-CN.min.js' %}"></script>
    <link rel="stylesheet" href="{% 'css/product_list.min.css' %}">
    <script src="{% static 'js/product_list.min.js' %}"></script>
```

#### 完成html


```
<label>
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
```

#### 初始化datepicker日期控件


```
ProductList.prototype.initDatePicker = function(){
    $.noConflict();
   var startPicker = $("#start-picker");
    var endPicker = $("#end-picker");

    var todayDate = new Date();
    var todayStr = todayDate.getFullYear() + '/' + (todayDate.getMonth()+1) + '/' + todayDate.getDate();//获取今天的日期
    var options = {
        'showButtonPanel': true,
        'format': 'yyyy/mm/dd',
        'startDate': '2019/11/18',//设置开始日期
        'endDate': todayStr,
        'language': 'zh-CN',
        'todayBtn': 'linked',
        // 'todayHighlight': true,
        'clearBtn': true,
        'autoclose': true
    };
    startPicker.datepicker(options);
    endPicker.datepicker(options);
};
```




