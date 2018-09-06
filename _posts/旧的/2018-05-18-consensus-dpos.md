---
layout: post
title: 共识算法-Dpos原理
categories: [共识]
tags: [Dpos]
date: 2018-05-18 15:26:18
comments: true
---



#### Dpos

pos(delegated proof of state)股份授权证明。由持币者投票选出若干节点(具体多少社区可以进行调整)，让选出的节点循环依次生产下一个区块。

bts(比特股)和eos就是使用股份授权。

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
	Index     int
	PrevHash  string
	HashCode  string
	BMP       int
	validator string
	TimeStamp string
}

//创建区块链
var Blockchain []Block

//生成block
func GenerateNextBlock(oldBlock Block, BMP int, adds string) Block {
	var newBlock Block
	newBlock.Index = oldBlock.Index + 1
	newBlock.PrevHash = oldBlock.HashCode
	newBlock.BMP = BMP
	newBlock.TimeStamp = time.Now().String()
	newBlock.validator = adds
	newBlock.HashCode = GenerateHahsValue(newBlock)
	return newBlock
}

//产生区块哈希
func GenerateHahsValue(block Block) string {
	hashCode := block.PrevHash + block.validator + block.TimeStamp +
		strconv.Itoa(block.Index) + strconv.Itoa(block.BMP)

	sha := sha256.New()
	sha.Write([]byte(hashCode))
	hashed := sha.Sum(nil)
	return hex.EncodeToString(hashed)

}

//存放委托人,存放委托人的地址信息
var delegate = []string{"aaa", "bbb", "ccc", "ddd"}

//随机调换委托人
func RandDelegate()  {
	rand.Seed(time.Now().Unix())
	var r = rand.Intn(3)
	t:=delegate[r]
	delegate[r]=delegate[3]
	delegate[3]=t
}

func main() {
	RandDelegate()

	//创世区块
	var firstBlock Block
	Blockchain =append(Blockchain,firstBlock)

	//通过n按顺序让delegate作为矿工
	var n = 0

	for {
		//每间隔3秒产生新的区块
		time.Sleep(time.Second*3)
		var nextBlock = GenerateNextBlock(firstBlock,1, delegate[n])
		n++
		n = n%len(delegate)
		firstBlock = nextBlock
		Blockchain = append(Blockchain,nextBlock)
		fmt.Println(Blockchain)
	}
}
```







