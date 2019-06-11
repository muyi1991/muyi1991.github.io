---
layout: post
title: 字典
categories: [Python]
tags: [字典]
date: 2019-06-11 22:00:07
comments: true
---


#### 什么是字典
字典是一个键值对，通过索引能够查找到对应的值

```
>>> names={'Tom':'Tom is very tall.','muyi':'muyi is very handsome.'}
>>> names
{'Tom': 'Tom is very tall.', 'muyi': 'muyi is very handsome.'}
>>> print(names['Tom'])
Tom is very tall.
```
#### 利用get()方法判断字典中的值程序不会奔溃

```
>>> print(names['Mary'])
Traceback (most recent call last):
  File "<pyshell#5>", line 1, in <module>
    print(names['Mary'])
KeyError: 'Mary'

>>> print(names['Tom'])
Tom is very tall.
>>> print(names.get('Mary'))
None
```

#### 判断某个元素是否在字典中

```
>>> x=names.get('Mary')
>>> print(x is None)
True
>>> print(x is not None)
False
```

##### get()自定义返回值
给get一个默认值，当get不到时返回该默认的提示,默认返回值是一个字符串，需要用引号

```
>>> print(names.get('Mary','There is no Mary here.'))
There is no Mary here.
```

#### =、==、is之间的区别
==指值相当，但是is的意思是这个东西就是那个东西，两者含义不一样

```
>>> a=[1,2,3,4,5]
>>> b=a
>>> c=[1,2,3,4,5]
>>> print(a==b)
True
>>> print(a is b)
True
>>> print(b is a)
True
>>> print(a == c)
True
>>> print(a is c)
False

```


