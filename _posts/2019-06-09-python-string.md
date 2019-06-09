---
layout: post
title: Python字符串
categories: [Python]
tags: [运算符,字符串]
date: 2019-06-09 14:00:04
comments: true
---


#### 字符串及其类型

```
>>> 'I am learning Python'
'I am learning Python'
>>> x='I am learning Python'
>>> x
'I am learning Python'
>>> type(x)
<class 'str'>
```

#### 字符串的单引号和双引号
可以在字符串中用“\”让Python不要读下一个字符

```
>>> 'I'm learning Python '
SyntaxError: invalid syntax
>>> 'I\'m learning Python '
"I'm learning Python "
>>> 
```

#### 字符串相加
字符串相加中间不会有空格，需要手动加上

```
>>> First_world = 'Hello'
>>> secend_world = 'World'
>>> print(First_world+secend_world)
HelloWorld
>>> print(First_world+' '+secend_world)
Hello World
```

#### 字符串不能相减

```
>>> print(First_world-secend_world)
Traceback (most recent call last):
  File "<pyshell#12>", line 1, in <module>
    print(First_world-secend_world)
TypeError: unsupported operand type(s) for -: 'str' and 'str'
```

#### 字符串可以乘以整数
不可以乘以字符串或者浮点型
不可以做除法运算

```
>>> print(First_world*12)
HelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHello
```

#### 字符串的内置方法“len()”
常用来计算字符串长度

```
>>> x = 'HelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHelloHello'
>>> print(len(x))
60
```

#### 判断字符串是否满足条件

```
>>> given_name='Luffy'
>>> middle_name='D'
>>> family_name='Monkey'
>>> name_length=len(given_name+middle_name+family_name)
>>> name_character_limit=15
>>> print(name_length<=name_character_limit)
True
>>> middle_name='DDDDDDDDDDDDDDDDDDD'
>>> name_length=len(given_name+middle_name+family_name)
>>> print(name_length<=name_character_limit)
False
```


