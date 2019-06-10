---
layout: post
title: 列表
categories: [Python]
tags: [list列表]
date: 2019-06-10 14:00:07
comments: true
---


#### 列表是什么
下标代表举例元素的距离，如：0代表‘Hello’到‘Hello’的距离，距离是0.
-1代表最后一个元素，-2代表倒数第二个元素。

```
>>> A_list_of_things=['Hello',5,8,8.0,.6,True,False]
>>> A_list_of_things[0]
'Hello'
>>> A_list_of_things[1]
5
>>> A_list_of_things[-1]
False
>>> A_list_of_things[-2]
True
```

#### 利用len()获取元素
先得到list的元素个数，然后-1就能够得到最后一个元素的坐标，通过坐标找到最后一个元素。

```
>>> len(A_list_of_things)
7
>>> A_list_of_things[len(A_list_of_things)-1]
False
```



