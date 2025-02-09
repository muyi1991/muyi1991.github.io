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

cut过滤器：
移除值中所有指定的字符串。类似于`python`中的`replace(args,"")`。示例代码如下：

```python
{{ value|cut:" " }}
```

以上示例将会移除`value`中所有的空格字符。`cut`过滤器的源代码如下：

```python
def cut(value, arg):
"""Remove all values of arg from the given string."""
safe = isinstance(value, SafeData)
value = value.replace(arg, '')
if safe and arg != ';':
return mark_safe(value)
return value
```

 date过滤器：
将
一个日期按照指定的格式，格式化成字符串。示例代码如下：

```python
context = {
"birthday": datetime.now()
}

{{ birthday|date:"Y/m/d" }}
```

那么将会输出`2018/02/01`。其中`Y`代表的是四位数字的年份，`m`代表的是两位数字的月份，`d`代表的是两位数字的日。  
还有更多时间格式化的方式。见下表。

| 格式字符 | 描述 | 示例 |
| --- | --- | --- |
| Y | 四位数字的年份 | 2018 |
| m | 两位数字的月份 | 01-12 |
| n | 月份，1-9前面没有0前缀 | 1-12 |
| d | 两位数字的天 | 01-31 |
| j | 天，但是1-9前面没有0前缀 | 1-31 |
| g | 小时，12小时格式的，1-9前面没有0前缀 | 1-12 |
| h | 小时，12小时格式的，1-9前面有0前缀 | 01-12 |
| G | 小时，24小时格式的，1-9前面没有0前缀 | 1-23 |
| H | 小时，24小时格式的，1-9前面有0前缀 | 01-23 |
| i | 分钟，1-9前面有0前缀 | 00-59 |
| s | 秒，1-9前面有0前缀 | 00-59 |


default过滤器

如果值被评估为`False`。比如`[]`，`""`，`None`，`{}`等这些在`if`判断中为`False`的值，都会使用`default`过滤器提供的默认值。示例代码如下：

```python
{{ value|default:"nothing" }}
```

如果`value`是等于一个空的字符串。比如`""`，那么以上代码将会输出`nothing`。


first

返回列表/元组/字符串中的第一个元素。示例代码如下：

```python
{{ value|first }}
```

如果value是等于['a','b','c']，那么输出将会是a。

last

返回列表/元组/字符串中的最后一个元素。示例代码如下：

```python
{{ value|last }}
```

如果value是等于['a','b','c']，那么输出将会是c。


floatformat

使用四舍五入的方式格式化一个浮点类型。如果这个过滤器没有传递任何参数。那么只会在小数点后保留一个小数，如果小数后面全是0，那么只会保留整数。当然也可以传递一个参数，标识具体要保留几个小数。

1. 如果没有传递参数：

| value | 模版代码 | 输出 |
| --- | --- | --- |
| 34.23234 | `{{ value\|floatformat }}` | 34.2 |
| 34.000 | `{{ value\|floatformat }}` | 34 |
| 34.260 | `{{ value\|floatformat }}` | 34.3 |

2. 如果传递参数：

| value | 模版代码 | 输出 |
| --- | --- | --- |
| 34.23234 | `{{value\|floatformat:3}}` | 34.232 |
| 34.0000 | `{{value\|floatformat:3}}` | 34.000 |
| 34.26000 | `{{value\|floatformat:3}}` | 34.260 |


join

类似与Python中的join，将列表/元组/字符串用指定的字符进行拼接。示例代码如下：

```python
{{ value|join:"/" }}
```

如果`value`是等于`['a','b','c']`，那么以上代码将输出`a/b/c`。

length

获取一个列表/元组/字符串/字典的长度。示例代码如下：

```python
{{ value|length }}
```

如果value是等于['a','b','c']，那么以上代码将输出3。如果value为None，那么以上将返回`0`。

lower

将值中所有的字符全部转换成小写。示例代码如下：

```python
{{ value|lower }}
```

如果`value`是等于`Hello World`。那么以上代码将输出`hello world`。

upper

类似于`lower`，只不过是将指定的字符串全部转换成大写。

random

在被给的列表/字符串/元组中随机的选择一个值。示例代码如下：

```python
{{ value|random }}
```

如果`value`是等于`['a','b','c']`，那么以上代码会在列表中随机选择一个。

safe

标记一个字符串是安全的。也即会关掉这个字符串的自动转义。示例代码如下：

```python
{{value|safe}}
```

如果`value`是一个不包含任何特殊字符的字符串，比如`<a>`这种，那么以上代码就会把字符串正常的输入。如果`value`是一串`html`代码，那么以上代码将会把这个`html`代码渲染到浏览器中。

striptags

删除字符串中所有的`html`标签。示例代码如下：

```python
{{ value|striptags }}
```


truncatechars

如果给定的字符串长度超过了过滤器指定的长度。那么就会进行切割，并且会拼接三个点来作为省略号。示例代码如下：

```python
{{ value|truncatechars:5 }}
```

如果`value`是等于`北京欢迎您~`，那么输出的结果是`北京...`。可能你会想，为什么不会`北京欢迎您...`呢。因为三个点也占了三个字符，所以`北京`+三个点的字符长度就是5。

truncatechars\_html

类似于`truncatechars`，只不过是不会切割`html`标签。示例代码如下：

```python
{{ value|truncatechars:5 }}
```

如果`value`是等于`<p>北京欢迎您~</p>`，那么输出将是`<p>北京...</p>`。








