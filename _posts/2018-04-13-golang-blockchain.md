---
layout: post
title: golang实现区块链-最小原型
categories: [区块链]
tags: [Blockchain,Bitcoin]
date: 2018-04-13 15:12:18
comments: true
---

#### 介绍
本质上，区块链是一个公开的分布式的记录数据库，和私有数据库不同，每个使用他的人都有完整或者部分副本，新记录的添加需要其他管理员的同意。

这个公开的数据库信息被存放在若干区块中，区块和区块连接起来就形成了区块链。

![](http://p8r45ciyt.bkt.clouddn.com/go%E8%AF%AD%E8%A8%80%E5%AE%9E%E7%8E%B0%E5%8C%BA%E5%9D%97%E9%93%BE-%E5%8C%BA%E5%9D%97%E9%93%BE.png)


最小原型区块链的区块中包含时间戳，交易数据，上一个区块哈希，当前区块哈希。

当前区块哈希是由时间戳，交易数据，上一个区块哈希经过SHA-256算法得来。其中任意一个数据变化都会导致当前区块哈希改变。

而下一个区块的哈希同样是由当前区块时间戳，交易数据，和上一个区块哈希得来，这样区块之间就形成了特定的先后顺序，从而极难篡改。

先来看看最小原型的完整代码。

#### 完整代码

```
package main

import (
	"strconv"
	"bytes"
	"crypto/sha256"
	"time"
	"fmt"
)

//定义一个区块
type Block struct {
	TimeStamp     int64
	Data          []byte
	PrevBlockHash []byte
	Hash          []byte
}

//生成区块哈希
func (b *Block) SetHash() {
	timestamp := []byte(strconv.FormatInt(b.TimeStamp, 10))
	headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
	hash := sha256.Sum256(headers)

	b.Hash = hash[:]
}

//定义一个数组结构的区块链
type Blockchain struct {
	blocks []*Block
}

//在定义的区块链中添加区块
func (bc *Blockchain) AddBlock(data string) {
	//上一个区块
	prevBlock := bc.blocks[len(bc.blocks)-1]
	//调用NEewBlock函数创建新区块
	newBlock := NewBlock(data, prevBlock.Hash)
	//把新的区块追加到区块链中
	bc.blocks = append(bc.blocks, newBlock)
}

func main() {
	//定义一个创世区块
	bc := NewBlockchain()

	//添加两个新区块
	bc.AddBlock("Send 1 BTC to zhangsan")
	bc.AddBlock("Send 2 BTC to lisi")

	//遍历出三个区块的数据信息
	for _, block := range bc.blocks {
		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		fmt.Println()
	}
}

//创世区块的函数
func NewGenesisBlock() *Block {
	//调用NweBlock函数创建创世区块，创世区块中prevBlockHash为空
	return NewBlock("Genesis Block", []byte{})
}

//把创世区块添加到区块链中
func NewBlockchain() *Blockchain {
	//把创世区块添加到区块链中
	return &Blockchain{[]*Block{NewGenesisBlock()}}
}

//创建新的区块，返回一个新的区块包含时间戳，date，上个区块哈希，当前区块哈希完整四个数据
func NewBlock(data string, prevBlockHash []byte) *Block {
	//新的区块需要时间戳，date，上个区块哈希，基于三个数据生成当前区块哈希
	block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}}
	//调用区块哈希方法生成当前区块哈希
	block.SetHash()
	return block
}

```

运行结果：

```
Prev. hash: 
Data: Genesis Block
Hash: fccad5089e9d39aa0f2df96f6c655560f22660590d39780d6394bb06f42dfb7d

Prev. hash: fccad5089e9d39aa0f2df96f6c655560f22660590d39780d6394bb06f42dfb7d
Data: Send 1 BTC to zhangsan
Hash: 65269288f2a0d5cbd5be3f6e1acf2668f3ab37b3a5393ddae94a3b3ad323f4e3

Prev. hash: 65269288f2a0d5cbd5be3f6e1acf2668f3ab37b3a5393ddae94a3b3ad323f4e3
Data: Send 2 BTC to lisi
Hash: 040fa0abac7124324bb0c07fda95406278c3a9ecc8c4390f2b143cefbf6c1b82


Process finished with exit code 0

```
接下来从区块链的区块开始，详解每部分代码。

#### 区块
区块中存储有价值的信息，如：比特币区块中存储着交易信息，这也是加密货币的本质。另外，还包含一些其他信息：版本号，时间戳，上一个区块hash值。

本文中我们实现一个最小原型，不是比特币标准区块，只包含最重要的信息。

像这样：

```
//定义一个区块
type Block struct {
	TimeStamp     int64
	Data          []byte
	PrevBlockHash []byte
	Hash          []byte
}
```

Timestamp是当前区块创建时的时间戳，Date是区块中所包含有实际价值交易信息，PrevBlockHash是存储上一个区块的hash，Hash是当前区块的hash。

当前区块的hash是如何得来的呢？hash的计算在区块链中非常重要，计算hash值是一个很难的操作，挖矿便是计算符合条件的hash值。

现在，我们把几个区块字段放一起，用SHA-256算法计算当前区块hash。

```
//生成区块哈希
func (b *Block) SetHash() {
	timestamp := []byte(strconv.FormatInt(b.TimeStamp, 10))
	headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
	hash := sha256.Sum256(headers)

	b.Hash = hash[:]
}
```

添加新的区块是一个很有意思的设计，他阻止了添加之后的修改。

接下来写一个函数，添加新区块。

```
//创建新的区块，返回一个新的区块包含时间戳，date，上个区块哈希，当前区块哈希完整四个数据
func NewBlock(data string, prevBlockHash []byte) *Block {
	//新的区块需要时间戳，date，上个区块哈希，基于三个数据生成当前区块哈希
	block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}}
	//调用区块哈希方法生成当前区块哈希
	block.SetHash()
	return block
}
```

#### 区块链
区块链保存着特定的数据结构的数据库，他是一个有序的，尾部相连的。

这就意味着插入的区块的顺序是被存储的，后一个区块连接着前一个区块。

这种结构我们通过array来实现。


```
//定义一个数组结构的区块链
type Blockchain struct {
	blocks []*Block
}
```

有了一个区块链，然后定义一个函数来向区块链中添加区块。


```
//在定义的区块链中添加区块
func (bc *Blockchain) AddBlock(data string) {
	//上一个区块
	prevBlock := bc.blocks[len(bc.blocks)-1]
	//调用NEewBlock函数创建新区块
	newBlock := NewBlock(data, prevBlock.Hash)
	//把新的区块追加到区块链中
	bc.blocks = append(bc.blocks, newBlock)
}
```

我们之前说过，添加新区块，需要一个已存在的区块，现在我们的区块链中还没有一个区块。

那我们就在链上创建第一个创世区块(genesis block),然后把创世区块添加到区块链中。


```
//创世区块的函数
func NewGenesisBlock() *Block {
	//调用NweBlock函数创建创世区块，创世区块中prevBlockHash为空
	return NewBlock("Genesis Block", []byte{})
}
```


```
//把创世区块添加到区块链中
func NewBlockchain() *Blockchain {
	//把创世区块添加到区块链中
	return &Blockchain{[]*Block{NewGenesisBlock()}}
}
```
#### 运行区块链

接下来我们确认下区块链是否正常工作，在主函数中打印下区块链中区块的数据信息。

```
func main() {
	//定义一个创世区块
	bc := NewBlockchain()

	//添加两个新区块
	bc.AddBlock("Send 1 BTC to zhangsan")
	bc.AddBlock("Send 2 BTC to lisi")

	//遍历出三个区块的数据信息
	for _, block := range bc.blocks {
		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		fmt.Println()
	}
}
```

运行结果：

```
Prev. hash: 
Data: Genesis Block
Hash: fccad5089e9d39aa0f2df96f6c655560f22660590d39780d6394bb06f42dfb7d

Prev. hash: fccad5089e9d39aa0f2df96f6c655560f22660590d39780d6394bb06f42dfb7d
Data: Send 1 BTC to zhangsan
Hash: 65269288f2a0d5cbd5be3f6e1acf2668f3ab37b3a5393ddae94a3b3ad323f4e3

Prev. hash: 65269288f2a0d5cbd5be3f6e1acf2668f3ab37b3a5393ddae94a3b3ad323f4e3
Data: Send 2 BTC to lisi
Hash: 040fa0abac7124324bb0c07fda95406278c3a9ecc8c4390f2b143cefbf6c1b82


Process finished with exit code 0


```
#### 最后

我们创建了一个超级简单的区块链原型，他只是一个包含区块的数组，每个区块和之前的区块连接起来。

在我们的区块链中添加一个区块很简单，真实的区块链中远比这个复杂，需要进行一些复杂运算来获取添加权限（这个过程被称作 工作量证明）。

而且区块链并非一个单节点决策的，区块链是一个分布式的数据库。

一个区块必须被网络上其他的区块确认和接收（这个过程被称作 一致性）。

