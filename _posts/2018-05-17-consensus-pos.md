---
layout: post
title: 共识算法-pos原理
categories: [共识]
tags: [Pos]
date: 2018-05-17 15:26:18
comments: true
---



#### Pos

pos(proof-of-state)权益证明。区块的选择是根据不同节点的股份和时间进行随机选择。

假如发行一种币总量100个(有点少，哈哈...)。你买了一个币，下个区块的记账机会你就有百分之一。如果另一个人有10个币，他就有十分之一的记账机会，比你多十倍的概率来进行下一个区块的记账。

#### 完整代码


```
package main

import (
	"strconv"
	"crypto/sha256"
	"encoding/hex"
	"time"
	"math/rand"
	"fmt"
)

//创建区块
type Block struct {
	BMP       int
	PreHash   string
	HashCode  string
	TimeStamp string
	Index     int
	//区块验证者
	validator string
}

//创建区块链
var Blockchain []Block

//计算区块Hash
func GenerateHashValue(block Block) string {
	hashcode := block.PreHash + block.HashCode + block.TimeStamp + block.validator +
		strconv.Itoa(block.BMP) + strconv.Itoa(block.Index)

	sha := sha256.New()
	sha.Write([]byte(hashcode))
	hashed := sha.Sum(nil)
	return hex.EncodeToString(hashed)
}

//生成新区块
func GenerateNextBlock(oldBlock Block, BMP int, adds string) Block {
	var newBlock Block
	newBlock.Index = oldBlock.Index + 1
	newBlock.TimeStamp = time.Now().String()
	newBlock.PreHash = oldBlock.HashCode
	newBlock.validator = adds
	newBlock.BMP = BMP
	newBlock.HashCode = GenerateHashValue(newBlock)
	return newBlock
}

//当作网络系统的全节点
type Node struct {
	tokens int
	adds   string
}

//存放全节点
var n [2]Node

//全节点中的地址总数
var addr [3000] string

func main() {
	//全节点存放两个币的拥有者
	n[0] = Node{1000, "abc123"}
	n[1] = Node{2000, "def456"}

	//以下为Pos算法,形成拥有3000个币的地址
	cnt := 0
	for i := 0; i < 2; i++ {
		for j := 0; j < n[i].tokens; j++ {
			addr[cnt] = n[i].adds
			cnt++
		}
	}

	//随机数
	rand.Seed(time.Now().Unix())
	//通过随机值获取【0，3000）随机数
	var rd = rand.Intn(3000)

	var adds = addr[rd]

	var firstBlock Block
	firstBlock.BMP = 100
	firstBlock.PreHash = "0"
	firstBlock.TimeStamp = time.Now().String()
	firstBlock.validator = "abc123"
	firstBlock.Index = 1
	firstBlock.HashCode = GenerateHashValue(firstBlock)

	//将创世区块添加到区块链
	Blockchain = append(Blockchain,firstBlock)

	//挖矿成功
	var secondBlock =GenerateNextBlock(firstBlock,200,adds)
	Blockchain =append(Blockchain,secondBlock)

	fmt.Println(Blockchain)
}

```

运行结果：


```
[{100 0 35f6cced16f671a98597c7c5f31c161f855f91b74ae916a88a88aff927e4ef27 2018-05-22 10:49:18.67397031 +0800 CST m=+0.000583771 1 abc123} {200 35f6cced16f671a98597c7c5f31c161f855f91b74ae916a88a88aff927e4ef27 2b3442d8c1c79ae67dafc8c9a111c72788342a31bcb182525beae37d12838c53 2018-05-22 10:49:18.67409463 +0800 CST m=+0.000708082 2 def456}]

Process finished with exit code 0
```








