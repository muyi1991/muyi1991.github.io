---
layout: post
title: 共识算法-pow原理
categories: [共识]
tags: [Pow]
date: 2018-05-16 15:26:18
comments: true
---



#### Pow

Pow(proof-of-work)工作量证明，利用哈希算法改变nonce值来碰撞符合条件的哈希。

在区块链结构，用链表的形式进行维护。

上个节点哈希值，prevhash。当前区块哈希，hash。区块高度index。时间戳：timeStamp。数据：data，data存储交易信息。随机数：nonce。难度系数：diff。

#### 完整代码


```
package main

import (
	"fmt"
	"strings"
	"time"
	"strconv"
	"crypto/sha256"
	"encoding/hex"
)


//创建区块
type Block struct {
	//区块高度
	Index int
	//时间戳
	Timestamp string
	//当前网络的难度系数
	Diff int
	//上一个区块的哈希值
	PrevHash string
	//当前区块的哈希
	Hash string
	//随机数
	Nonce int
	//交易信息
	Data string
}


//创建创世区块
func GenertateFirstBlock(data string) Block {
	var firstBlock Block
	firstBlock.Index = 1
	firstBlock.Timestamp = time.Now().String()
	firstBlock.Diff = 4
	firstBlock.Nonce = 0
	firstBlock.Data = data
	firstBlock.Hash = GenerataBlockHashValue(firstBlock)

	return firstBlock
}

//生产区块哈希
func GenerataBlockHashValue(block Block) string {
	//将区块属性拼接产生当前区块哈希
	var hashdata =strconv.Itoa(block.Index)+block.Timestamp+strconv.Itoa(block.Diff)+
		strconv.Itoa(block.Nonce)+block.Data

	//Hash算法
	sha := sha256.New()
	sha.Write([]byte(hashdata))
	hashed := sha.Sum(nil)

	//将字节转换为字符串
	return hex.EncodeToString(hashed)
}

//产生新区块
func GenerateNextBlock(data string, oldBlock Block) Block {
	newBlock := Block{}
	newBlock.Data =data
	newBlock.Diff = 4
	newBlock.Timestamp = time.Now().String()
	//暂设为0。由矿工调整
	newBlock.Nonce = 0
	newBlock.PrevHash = oldBlock.Hash
	newBlock.Index = 2
	newBlock.Hash = pow(newBlock.Diff,&newBlock)

	return newBlock
}

//挖矿算法 proof-of-work 工作量证明
func pow(diff int, block *Block) string {

	for{
		hash := GenerataBlockHashValue(*block)
		fmt.Println(hash)
		//判断前缀有没有符合条件,前缀有没有重复符合条件的次数
		if strings.HasPrefix(hash,strings.Repeat("0",diff)){
			//挖矿成功
			fmt.Println("挖矿成功")
			return hash
		} else {
			block.Nonce++
		}
	}

	return ""

}


//通过链表的形式，维护区块链中的业务
type Node struct {
	//指针域
	NextNode *Node
	//数据域
	Data *Block
}


//创建头节点，保存创世区块
func CreateHeaderNode(data *Block) *Node  {
	var headerNode *Node= new(Node)
	headerNode.NextNode = nil
	headerNode.Data = data
	return  headerNode
}

//添加节点，当挖到框，添加区块
func AddNode(data *Block,node *Node) *Node {
	var newNode *Node = new(Node)
	newNode.Data = data
	newNode.NextNode = nil
	node.NextNode = newNode
	return  newNode
}

func ShowNodes(node *Node)  {
	n:=node
	for {
		if n.NextNode == nil {
			fmt.Println(n.Data)
			break
		} else{
			fmt.Println(n.Data)
			n = n.NextNode
		}
	}
}

func main() {
	first :=GenertateFirstBlock("创始区块")
	//通过挖矿产生第二区块
	second := GenerateNextBlock("第二区块",first)

	//创建链表，将两个区块放入链表中
	header :=CreateHeaderNode(&first)
	AddNode(&second,header)

	ShowNodes(header)
}
```

运行结果：


```
2a2b0cba542095b026ddeb06844fc42fef42cc5a251fc0aa21772944b5e8b1a9
...
0000063762d5691b842b1f4baf1a04b8e2395a159b25e4cc13178a2eb7fbf78f
挖矿成功
&{1 2018-05-22 13:00:35.316831097 +0800 CST m=+0.000599583 4  6b0231ba12848e4689d5c944cecc061205e7792ec61cc9dfc6e49d4672ae8d0c 0 创始区块}
&{2 2018-05-22 13:00:35.316963036 +0800 CST m=+0.000731519 4 6b0231ba12848e4689d5c944cecc061205e7792ec61cc9dfc6e49d4672ae8d0c 0000063762d5691b842b1f4baf1a04b8e2395a159b25e4cc13178a2eb7fbf78f 42760 第二区块}

Process finished with exit code 0
```

