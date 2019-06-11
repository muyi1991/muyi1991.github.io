---
layout: post
title: 集合的去重
categories: [Python]
tags: [集合]
date: 2019-06-11 21:00:07
comments: true
---


#### 用set()方法给列表去重
set方法里面放的是赋值列表

```
>>> place=['hangzhou','beijing','shanghai','beijing','hangzhou']
>>> len(place)
5
>>> unique_place=set(place)
>>> print(unique_place)
{'shanghai', 'beijing', 'hangzhou'}
```

#### 判断某个元素是否在列表或者集合

```
>>> 'beijing' in place
True
>>> 'london' in place
False
>>> 'beijing' in unique_place
True
>>> 'london' in unique_place
False
>>> 
```

#### 集合增加、减少元素
集合中增加元素用add()，类似于列表中的append()，减少元素用pop()，需要注意的是减少方法里面不能写值，减少是从第1个开始减少的

```
>>> numbers={1,2,3,4,56,9}
>>> numbers.add(88)
>>> print(numbers)
{1, 2, 3, 4, 9, 56, 88}
>>> numbers.pop()
1
>>> print(numbers)
{2, 3, 4, 9, 56, 88}
```




