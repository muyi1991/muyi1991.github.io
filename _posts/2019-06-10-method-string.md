---
layout: post
title: 字符串方法
categories: [Python]
tags: [字符串]
date: 2019-06-10 14:00:08
comments: true
---


#### 常见方法
计算字符串长度、数据类型、打印

```
>>> len('Hello world!')
12
>>> type(8)
<class 'int'>
>>> print('Hello world!')
Hello world!
```

#### 检测字符串大小写方法
检测是全部大写或全部小写

```
>>> Mom_said='Tom is a good boy.'
>>> Mom_said.islower()
False
```

#### 统计字符串中某字母出现次数

```
>>> Mom_said.count('o')
4
```

#### 统计某个字母出现位置
只会统计该字母第一次出现的位置

```
>>> Mom_said.find('a')
7
```


