---
layout: post
title: 数据类型的转换
categories: [Python]
tags: [运算符]
date: 2019-06-10 14:00:06
comments: true
---


#### 数据类型
字符串必须用单引号或者双引号‘’、“”
布尔型必须首字母大写

```
>>> 8
8
>>> 8.0
8.0
>>> true
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    true
NameError: name 'true' is not defined
>>> True
True
>>> Hello
Traceback (most recent call last):
  File "<pyshell#4>", line 1, in <module>
    Hello
NameError: name 'Hello' is not defined
>>> 'hello'
'hello'
```

#### 查看数据类型用type()

```
>>> type(8)
<class 'int'>
>>> type(8.0)
<class 'float'>
>>> type(True)
<class 'bool'>
>>> type('Hello')
<class 'str'>
```

#### 整数、浮点和布尔类型的转换
整数和浮点、布尔可以互相转换
布尔True是1或1.0,False是0或0.0

```
>>> float(8)
8.0
>>> int(8.0)
8
>>> int(True)
1
>>> float(True)
1.0
>>> int(False)
0
>>> float(False)
0.0
```

#### 数字字符串可以转换整型、浮点型

```
>>> int('I am learning Python')
Traceback (most recent call last):
  File "<pyshell#16>", line 1, in <module>
    int('I am learning Python')
ValueError: invalid literal for int() with base 10: 'I am learning Python'
>>> int('8')
8
>>> type(int('8'))
<class 'int'>
>>> float('8')
8.0
>>> type(float(8))
<class 'float'>
```




