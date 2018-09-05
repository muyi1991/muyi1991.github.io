---
layout: post
title: 哈希散列-链表
categories: [密码学]
tags: [Hash加密]
date: 2018-04-15 15:12:11
comments: true
---

#### 哈希散列

如果给你一个数组里面有一百万个数，从中去找某一个数，你会怎么做？

首先我们可以遍历这个数组，然后找出这个数所在位置。但是会发现这样可能会遍历很多次，甚至近一百万次才能找到这个数。会花费很长的时间和资源，有没有更好的方法呢？

有！我们可以把一百万个数放在一个长度为16的数组中(不同数组有不同的算法，本文以16位数组为例)，但是很显然，16位的数组时放不下一百万个数的。解决办法是放到最后之后折回来继续放。

![](http://p8r45ciyt.bkt.clouddn.com/1.png)

折回来放了之后，同一个数组下标对应的值他们的内存地址不是连续的，他们就叫做链表。

每一个数有两个属性，一个是Key，一个是value。value就是这个数，key则是通过16位数组的算法算出，把一百万个数随机平均散列到16位数组中。

这样对一个数操作就只需要找到数组对应的下标，也就是这个数的key，然后遍历一列的数就可以了。

我们先来看看链表。

#### 链表

链表是一种动态数据结构，需要添加一个元素时，程序可以自动的申请内存空间添加元素，需要减少元素时，程序可以自动释放该元素占用内存空间。

主要是利用结构体并配合指针实现。

来看看完整代码。

#### 完整代码

```
package main

import (
	"fmt"
)

//声明全局变量保存头节点
var head *Node
var curr *Node

//声明节点类型
type Node struct {
	//数据域
	Data string
	//地址域
	NextNode *Node
}

func main() {
	CreateHeadNode("头节点")
	AddNode("第二个节点")
	AddNode("第三个节点")
	AddNode("第四个节点")

	InsertNodeByIndex(1,"新节点")
	DeleteNodeByIndex(1)
	UpdataNodeByIndex(1,"修改节点")

	ShowNodes()

	fmt.Println("节点数量:",NodeCount())

}



//创建头节点
func CreateHeadNode(data string) *Node {
	var node *Node = new(Node)
	node.Data = data
	node.NextNode = nil

	//保存头节点
	head = node
	curr = node

	return node
}

//添加新节点
func AddNode(data string) *Node {
	var newNode *Node = new(Node)
	newNode.Data = data
	newNode.NextNode = nil

	//挂接节点
	curr.NextNode = newNode
	curr = newNode

	//返回值
	return newNode
}

//遍历链表
func ShowNodes() {
	var node = head
	for {
		if node.NextNode == nil {
			fmt.Println(node.Data)
			break
		} else {
			fmt.Println(node.Data)
			node = node.NextNode
		}
	}
}

//计算节点个数
func NodeCount() int {
	var count int = 1
	var node = head

	for {
		if node.NextNode == nil {
			break
		} else {
			node = node.NextNode
			count = count + 1
		}
	}
	return count

}

//插入节点
func InsertNodeByIndex(index int, data string) *Node {
	if index == 0 {
		//添加新的头节点
		var node *Node = new(Node)
		node.Data = data
		node.NextNode = head
		head = node
	} else if index > NodeCount()-1 {
		//添加新节点
		AddNode(data)
	} else {
		//中间插入节点
		var n = head
		for i := 0; i < index-1; i++ {
			n = n.NextNode
		}

		var newNode *Node = new(Node)
		newNode.Data = data
		newNode.NextNode = n.NextNode
		n.NextNode = newNode
	}

	return nil
}

//删除节点
func DeleteNodeByIndex(index int) {
	var node = head

	if index == 0 {
		//删除头节点，就是第二个节点为头节点
		head = node.NextNode
	} else {
		//删除中间节点
		for i := 0; i < index-1; i++ {
			node = node.NextNode
		}
		node.NextNode = node.NextNode.NextNode
	}
}

//修改节点
func UpdataNodeByIndex(index int, data string) {
	var node = head

	if index==0{
		head.Data=data
	}else {

		for i := 0; i < index-1; i++ {
			node = node.NextNode
		}
		node.NextNode.Data=data
	}
}
```

运行结果：

```
头节点
修改节点
第三个节点
第四个节点
节点数量: 4

Process finished with exit code 0
```

接下来详解代码。

#### 定义头节点

链表的内存地址不是连续的，那么一个节点中不光要存储这个节点的值，还要存储下一个节点的内存地址，通过内存地址把这些节点连接起来。值叫数据域，内存地址叫地址域。

我们用结构体来定义一个节点。

```
//声明节点类型
type Node struct {
	//数据域
	Data string
	//地址域
	NextNode *Node
}
```


链表中至少有一个节点，我们先声明全局变量保存头节点。再写一个定义头节点的方法。


```
//声明全局变量保存头节点
var head *Node
var curr *Node

//创建头节点
func CreateHeadNode(data string) *Node {
	var node = new(Node)
	node.Data = data
	node.NextNode = nil

	//保存头节点
	head = node
	curr = node

	return node
}
```

接下来我们就可以添加新节点啦。

#### 添加新节点
```
//添加新节点
func AddNode(data string) *Node {
	var newNode = new(Node)
	newNode.Data = data
	newNode.NextNode = nil

	//挂接节点
	curr.NextNode = newNode
	curr = newNode

	//返回值
	return newNode
}
```


写个函数遍历链表，写个函数结算节点个数。

```
//遍历链表
func ShowNodes() {
	var node = head
	for {
		if node.NextNode == nil {
			fmt.Println(node.Data)
			break
		} else {
			fmt.Println(node.Data)
			node = node.NextNode
		}
	}
}

//计算节点个数
func NodeCount() int {
	var count int = 1
	var node = head

	for {
		if node.NextNode == nil {
			break
		} else {
			node = node.NextNode
			count = count + 1
		}
	}
	return count

}
```

我们在主函数中添加几个节点，先看看效果。


```
func main() {
	CreateHeadNode("头节点")
	AddNode("第二个节点")
	AddNode("第三个节点")
	AddNode("第四个节点")
	
	ShowNodes()
	fmt.Println("节点数量:", NodeCount())
}
```

#### 插入节点

插入节点有三种情况：

第一种，若果插入的头节点

补图

第二种，如果插入的是新节点，直接调用添加新节点函数

补图

第三种，如果中间插入节点

补图


```
//插入节点
func InsertNodeByIndex(index int, data string) *Node {
	if index == 0 {
		//添加新的头节点
		var node  = new(Node)
		node.Data = data
		node.NextNode = head.NextNode
		head = node
	} else if index > NodeCount()-1 {
		//添加新节点
		AddNode(data)
	} else {
		//中间插入节点
		var n = head
		for i := 0; i < index-1; i++ {
			n = n.NextNode
		}

		var newNode  = new(Node)
		newNode.Data = data
		newNode.NextNode = n.NextNode
		n.NextNode = newNode
	}

	return nil
}
```

#### 删除节点
删除节点有两种情况：

第一种，删除头节点

补图

第二种，删除中间节点

补图

```
//删除节点
func DeleteNodeByIndex(index int) {
	var node = head

	if index == 0 {
		//删除头节点，就是第二个节点为头节点
		head = node.NextNode
	} else {
		//删除中间节点
		for i := 0; i < index-1; i++ {
			node = node.NextNode
		}
		node.NextNode = node.NextNode.NextNode
	}
}
```

#### 修改节点

修改节点有两种情况：

第一种：修改头节点

补图

第二种：修改中间节点

补图


```
//修改节点
func UpdataNodeByIndex(index int, data string) {
	var node = head

	if index == 0 {
		head.Data = data
	} else {

		for i := 0; i < index-1; i++ {
			node = node.NextNode
		}
		node.NextNode.Data = data
	}
}
```





