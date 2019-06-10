---
layout: post
title: 列表的方法
categories: [Python]
tags: [list列表]
date: 2019-06-10 22:23:07
comments: true
---


#### 列表的长度，最大值，最小值，冒泡排序
len(),max(),min(),sorted(),sorted(x,reverse=True)
sorted()是冒大泡排序，sorted(x,reverse=True)是冒小泡排序

```
>>> stock_price=[3,1,5,8,4,2,9]
>>> len(stock_price)
7
>>> max(stock_price)
9
>>> min(stock_price)
1
>>> sorted(stock_price)
[1, 2, 3, 4, 5, 8, 9]
>>> sorted(stock_price,reverse=True)
[9, 8, 5, 4, 3, 2, 1]
>>> scores_card=['B','D','A','C']
>>> sorted(scores_card)
['A', 'B', 'C', 'D']
>>> sorted(scores_card,reverse=True)
['D', 'C', 'B', 'A']
```

#### join()方法
在列表中追加元素，可以用join()方法，给每一个元素都追加

```
>>> a=['A','B','C','D']
>>> b='\n'.join(a)
>>> print(b)
A
B
C
D
>>> '\n'.join(a)
'A\nB\nC\nD'
>>> 'I\' am a student.'
```

#### append()方法
每次追加一个元素到列表最后一个位置，一次只能追加一个

```
>>> name=['gary','jim','tom','sam']
>>> name.append('james')
>>> print(name)
['gary', 'jim', 'tom', 'sam', 'james']
>>> name.append('james','lucy')
Traceback (most recent call last):
  File "<pyshell#23>", line 1, in <module>
    name.append('james','lucy')
TypeError: append() takes exactly one argument (2 given)
```


