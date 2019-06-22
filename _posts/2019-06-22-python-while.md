---
layout: post
title: while语句
categories: [Python]
tags: [条件语句]
date: 2019-06-22 21:00:07
comments: true
---


#### 普通while语句

```
deck=[1,2,3,4,5,6,7,8,9]
equiped=[]
while sum(equiped)<=20:
    equiped.append(deck.pop())

print(equiped)
```

#### break终止循环
input()可以让在控制台输入内容
capitalize()和title()都是可以把字符串首字母大写

```
while True:
    word=input('Enter string to capitalize, q to quit:')
    if word=='q':
        break
    print(word.capitalize())
```

#### continue跳过本次循环
continue()是跳过这一次循环，继续进行其他循环

```
while True:
    number=input('Even numble please~[enter q to quit]:')
    if number=='q':
        break
    number=int(number)
    if number %2 == 0:
        continue
    print('I said even numbel, are you blind??')
```


