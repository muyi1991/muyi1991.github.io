---
layout: post
title: for语句和range()
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

#### for和range()配合使用
利用range()给列表赋值

```
print(list(range(100)))

for a in range(100):
    print("My name is muyi.")
```

#### 利用range()改变列表的值

```
names=['tim','jim','mary','tom']
for index in range(len(names)):
    names[index]=names[index].title()
print(names)
```

#### 列表迭代
replace()改变字符串中某个字符

```
names=['Tim jie','Jim li','Mary wang','Tom zhang']
names_new=[]
for index in names:
    names_new.append(index.lower().replace(' ','_'))
print(names_new)
```

#### 字典迭代
通过items()对字典的所有元素进行迭代，通过format()方法，对字符串进行格式化输出

```
names={'Tim jie':'you are rich.','Jim li':'you are hot.','Mary wang':'you are handsome.',
       'Tom zhang':'you are cute.'}

for key,value in names.items():
    print('name:{},saying:{}'.format(key,value))
```










