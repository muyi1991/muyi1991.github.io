---
layout: post
title: 元组和元组解包
categories: [Python]
tags: [tuple]
date: 2019-06-11 10:00:07
comments: true
---


#### 元组是一个list，但是不可变
元组内数据相关性很高，但是不可变，用元组，常用于经纬度，三维坐标
元组中元素可以用小括号括起来，也可以不用小括号

```
>>> place_1=(1,2)
>>> print('longitude',place_1[0])
longitude 1
>>> print('latitude:',place_1[1])
latitude: 2
```

#### 元组的解包
一个三维数据，把里面的值都拿出来
全部解包，然后再拿自己想用的值

```
>>> demensions=1,2,3
>>> length,width,height = demensions
>>> print(length)
1
>>> print(width)
2
>>> print(height)
3
```

#### 元组的format打印
格式：‘x {}{}{}’.format(a,b,c)

```
>>> print('The demensions are {}*{}*{}'.format(length,width,height))
The demensions are 1*2*3
```


