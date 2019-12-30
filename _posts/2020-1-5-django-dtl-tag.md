---
layout: post
title: Django中模版介绍和基本标签使用
categories: [后端]
tags: []
date: 2020-1-5 13:48:07
comments: true
---


#### 概述

DTL是一种带有特殊语法的HTML文件，这个HTML文件可以被Django编译，可以传递参数，实现数据动态化，在编译完成后，生成一个普通的HTML文件，然后发送给客户端，相当于一个预编译器。

#### 模版变量使用

1. 在模版中使用变量，需要将变量放到`{{ 变量 }}`中。
2. 如果想要访问对象的属性，那么可以通过`对象.属性名`来进行访问。
    ```python
    class Person(object):
        def __init__(self,username):
            self.username = username

    context = {
        'person': p
    }
    ```
    以后想要访问`person`的`username`，那么就是通过`person.username`来访问。
3. 如果想要访问一个字典的key对应的value，那么只能通过`字典.key`的方式进行访问，不能通过`中括号[]`的形式进行访问。
    ```python
    context = {
        'person': {
            'username':'zhiliao'
        }
    }
    ```
    那么以后在模版中访问`username`。就是以下代码`person.username`
4. 因为在访问字典的`key`时候也是使用`点.`来访问，因此不能在字典中定义字典本身就有的属性名当作`key`，否则字典的那个属性将编程字典中的key了。
    ```python
    context = {
        'person': {
            'username':'zhiliao',
            'keys':'abc'
        }
    }
    ```
    以上因为将`keys`作为`person`这个字典的`key`了。因此以后在模版中访问`person.keys`的时候，返回的不是这个字典的所有key，而是对应的值。
5. 如果想要访问列表或者元组，那么也是通过`点.`的方式进行访问，不能通过`中括号[]`的形式进行访问。这一点和python中是不一样的。示例代码如下：


    ```python
    {{ persons.1 }}
    ```
    
#### 模版标签使用

...

#### 模版过滤器使用

为什么需要过滤器？
因为在DTL中，不支持函数的调用形式`()`，因此不能给函数传递参数，这将有很大的局限性。而过滤器其实就是一个函数，可以对需要处理的参数进行处理，并且还可以额外接收一个参数（也就是说，最多只能有2个参数）。

add过滤器：
将传进来的参数添加到原来的值上面。这个过滤器会尝试将`值`和`参数`转换成整形然后进行相加。如果转换成整形过程中失败了，那么会将`值`和`参数`进行拼接。如果是字符串，那么会拼接成字符串，如果是列表，那么会拼接成一个列表。示例代码如下：

```python
{{ value|add:"2" }}
```

如果`value`是等于4，那么结果将是6。如果`value`是等于一个普通的字符串，比如`abc`，那么结果将是`abc2`。`add`过滤器的源代码如下：

```python
def add(value, arg):
"""Add the arg to the value."""
try:
return int(value) + int(arg)
except (ValueError, TypeError):
try:
return value + arg
except Exception:
return ''
```









