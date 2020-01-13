---
layout: post
title: Jquery基础之Json在js和python中的应用
categories: [前端]
tags: [jquery]
date: 2020-1-23 13:48:07
comments: true
---

#### 概述

json是一种数据格式，在前端和后端进行交互的时候用到的就是json格式，其实不同的后端语言在进行交互的时候都适用json格式。

所以就涉及到如何在js中把json转成js能够识别的对象？？？把js的对象转成json？？
如何在django中把json转成django能够识别的字典，把字典转成json？？？

#### JSON.parse() Json.stringify()

从json字符串转换成js对象，使用JSON.parse()方法

```html
var obj = JSON.parse('{"a":"hello","b":"world"}') //结果是 {a: 'Hello', b: 'World'}
```

要实现从js对象转换JSON字符串，使用JSON.stringfy()方法

```html
var json = JSON.stringify({a: 'Hello', b: 'World'}); //结果是 '{"a": "Hello", "b": "World"}'
```

#### json.loads() json.dumps()

将json对象转换成python的字典，使用json.loads(),转换成功后使用['tagName']进行访问。

将python字段转换成json，使用json.dumps()，进行转换。

```python
import json
 
# Python 字典类型转换为 JSON 对象
data1 = {
    'no' : 1,
    'name' : 'Runoob',
    'url' : 'http://www.runoob.com'
}
 
json_str = json.dumps(data1)
print ("Python 原始数据：", repr(data1))
print ("JSON 对象：", json_str)
 
# 将 JSON 对象转换为 Python 字典
data2 = json.loads(json_str)
print ("data2['name']: ", data2['name'])
print ("data2['url']: ", data2['url'])
```

#### python中的编码和解码

str.encode('utf-8') 对字符串进行编码
str.decode('utf-8') 对字符串进行解码

编码和解码必须用同一种格式，如："utf-8"

```python
>>> u = '中文' 

>>> str = u.encode('utf-8')   # 以utf-8编码对u进行编码，获得bytes类型对象
>>> print(str)
b'\xe4\xb8\xad\xe6\x96\x87'

>>> u1 = str.decode('utf-8') # 以utf-8编码对字符串str进行解码，获得字符串类型对象
>>> print(u1)
'中文'
```

#### 使用ajax发送数组数据到后台

使用ajax发送数组给到后台需要设置traditional：true，设置这个属性，不让后台接受不到数据。


```html
xfzajax.post({
            'url':url,
            'traditional':true, //必须设置这个属性，不然后台接受不到数据
            'data':{
                'title':title,
                'category1':category1,
                'category2':category2,
                'cost_price':costPrice,
                'price':price,
                'detail':detail,
                'pk':pk,
                'thumbnails':thumbnails,
                'colors':colors,
                'skus':skus

            },
            'success':function (result) {
                if (result['code'] === 200){
                    if(pk){
                        window.location.replace('/product/product_list/')
                    }else {
                            window.messageBox.showSuccess('发布成功')
                        setTimeout(function () {
                            window.location.replace('/product/product_list/')
                        },1500)
                    }
                    console.log(result);
                }
            }
        })
```













 





















