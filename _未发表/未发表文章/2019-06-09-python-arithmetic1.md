---
layout: post
title: Python赋值运算符
categories: [Python]
tags: [运算符]
date: 2019-06-09 14:00:02
comments: true
---


#### 变量赋值

```
>>> a=3
>>> a
3
>>> print(a)
3
```

```
>>> a=3
>>> b=a
>>> a
3
>>> b
3

```

#### 多个变量同时赋值

```
>>> x,y,z = 1,2,3
>>> x
1
>>> y
2
>>> z
3
```

#### 汽车销售量计算

```
>>> revenue=1000
>>> price=20
>>> car=revenue/price
>>> car
50.0
>>> print(car)
50.0
>>> revenue=revenue+500
>>> car=revenue/price
>>> print(car)
75.0
```

#### +=、-=、*=、/=

```
>>> revenue=1000
>>> price=20
>>> car = revenue/price
>>> print(car)
50.0
>>> revenue+=500
>>> car = revenue/price
>>> print(car)
75.0
>>> 
```

#### 整数型和浮点型

```
>>> 8
8
>>> print(type(8))
<class 'int'>
>>> 8.0
8.0
>>> print(type(8.0))
<class 'float'>
```

#### 浮点型不是严格意义的浮点
稍微大一点点

```
>>> .1+.1+.1
0.30000000000000004
>>> print(.1+.1+.1==.3)
False
```

#### 除法不能除以零

```
>>> 5/0
Traceback (most recent call last):
  File "<pyshell#6>", line 1, in <module>
    5/0
ZeroDivisionError: division by zero
>>> 5/.0
Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    5/.0
ZeroDivisionError: float division by zero
>>> 
```

