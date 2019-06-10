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

#### 提取list数值
取list数值左包右不包

```
>>> A_list_of_things=[1,2,.3,5,6,7,8]
>>> A_list_of_things[1:3]
[2, 0.3]
```

从头开始或者到最后可以不用填

```
>>> A_list_of_things[0:5]
[1, 2, 0.3, 5, 6]
>>> A_list_of_things[5:]
[7, 8]
```

#### 判断某个元素是否在列表中
利用in、not in 判断某个元素是否在列表中

```
>>> 'am' in 'I am learning Python'
True
>>> 'are' in 'I am learning Python'
False
>>> 5 in [1,2,3,4,5]
True
>>> 5 not in [1,2,3,4,5]
False
```

#### 列表和字符串的可变性
列表是可以变换的，字符串不能变换赋值

```
>>> A_list_of_things
[1, 2, 0.3, 5, 6, 7, 8]
>>> A_list_of_things[0]
1
>>> A_list_of_things[0]='one'
>>> print(A_list_of_things)
['one', 2, 0.3, 5, 6, 7, 8]
>>> Hello='Hello hi~'
>>> Hello
'Hello hi~'
>>> Hello[0]
'H'
>>> Hello[0]='M'
Traceback (most recent call last):
  File "<pyshell#19>", line 1, in <module>
    Hello[0]='M'
TypeError: 'str' object does not support item assignment
```

#### 字符串的不可变性
I_said赋值给了You_said
小球在那一刻把值赋给了另一个小球

```
>>> I_said='I am muyi'
>>> You_said=I_said
>>> You_said
'I am muyi'
>>> I_said='I am not muyi'
>>> I_said
'I am not muyi'
>>> You_said
'I am muyi'
```

#### 列表的可变性
小球贴上了标签scores_card，同时小球又贴上了标签scores_card1

```
>>> scores_card=['B','D','C','A']
>>> scores_card1=scores_card
>>> scores_card[0]='C'
>>> scores_card
['C', 'D', 'C', 'A']
>>> scores_card1
['C', 'D', 'C', 'A']
```







