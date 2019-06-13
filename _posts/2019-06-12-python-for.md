---
layout: post
title: for语句
categories: [Python]
tags: [条件语句]
date: 2019-06-12 22:00:08
comments: true
---


#### for语句读数据
title()函数可以把字符串进行首字母大写

```
names=['mary','jim','tom','james']
for name in names:
    print(name.title())
```
   
#### for语句写数据

```   
names=['mary','jim','tom','james']
capitalized_names=[]

for name in names:
    capitalized_names.append(name.title())

print(capitalized_names)
```







