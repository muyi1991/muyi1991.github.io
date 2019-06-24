---
layout: post
title: zip迭代器
categories: [Python]
tags: [迭代生成器]
date: 2019-06-23 11:48:07
comments: true
---


#### zip普通迭代器
类似于range()，只能迭代，还需要list才能成为一个列表

```
names=['jim','tim','gary','gili']
height=[180,174,190,160]
print(list(zip(names,height)))
```

#### 利用len打印出编号

```
names=['jim','tim','gary','gili']

for i,name in zip(range(len(names)),names):
    print(i+1,name)
```

#### 利用enumerate打印列表编号
效果事一样的

```
names=['jim','tim','gary','gili']

for i,name in enumerate(names):
    print(i+1,name)

```

#### zip反向拿出数据

```
names=['jim','tim','gary','gili']
height=[180,174,190,160]

a=list(zip(names,height))

names1,height=zip(*a)

print(names1)
```


