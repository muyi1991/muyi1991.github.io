---
layout: post
title: 哈希加密-算法原理
categories: [密码学]
tags: [Hash加密]
date: 2018-05-01 09:11:18
comments: true
---

#### 哈希算法

哈希散列算法主要思想：把存储内容通过一定的函数关系（称为散列函数），计算出对应的函数值，把值解析为存储内容的地址，将存储内容放到地址对应的单元中。检索时，用同样方法计算出地址，去相应的单元中取要找的存储内容。通过散列方法可以对存储内容进行快速检索。

哈希散列（通过散列函数）以较短的信息来保证文件唯一性，这个标志与文件的每一个字节信息都相关，且很难通过散列函数后的值推导出存储内容。

当源文件发生任何改变，值也会发生改变，从而告诉使用者当前文件已经不是你所需要的文件了。

取模算法作为一种不可逆的计算方法，就可以满足这些条件。

先来看看简单的哈希算法。

#### 简单哈希算法


```
package main

import "fmt"

func main() {
	//定义一个变量
	a:=10
	//做取模运算
	fmt.Println(a%3)
}
```

这就是一个简单的哈希算法啦，10%3，结果1。

其中，10是明文，%3是密钥（哈希算法），1是密文（哈希值）。给出明文和密钥，可以推理出密文。给出密文和密钥，无法准确推出铭文，因为模3余1的数无数个。所以只能进行单向推导。

这样过于简单，我们加异或，让结果不那么有明显的规律性。

```
a:=10
fmt.Println(a+(a<<2)%3^5)
```

这样之后结果规律性就不那么明显了，当然这样算法依旧不是很安全。那么什么样的哈希算法是安全优秀的算法呢？我们先大致了解下哈希函数的分类。

#### 哈希函数分类、特点
* 加法哈希：就是把输入内容一个个加起来组成最后结果。
* 位运算哈希：通过各种位运算（移位和异或），来充分混合输入元素。
* 乘法哈希：利用乘法没有那么强的相关性就行散列。
* 除法哈希：和乘法类似。
* 查表哈希：通过特定的算法，比较有名的有Universal Hashing和Zobrist Hashing ，他们表格是随机生成的，有兴趣可以了解一下。
* 混合哈希：混合哈希就是利用以上各种方式，常见算法如，Md5就是用的混合哈希。

无论采用什么算法，对于密码学来说，安全始终是最重要的，目前流行的Sha256、Md5都是在混合哈希算法上对哈希做了二次封装，是比较安全的算法，他们满足以下特点：

* 正向快速：给出明文和密钥，在有限时间和有限资源内计算密文。
* 逆向困难：给出密文，在有限时间内很难逆推明文。
* 输入敏感：原始明文信息发生任何改变，新的密文有很大不同。
* 冲突避免：很难找到两个不同的明文，使得他们的密文一致。

先来看看Sha256。

#### Sha256

在特比特中Pow共识挖矿就是使用了SHA256算法，通过不断调整nonce值，找到符合条件哈希就算挖矿成功。

在go语言中自带SHA256的库，我们可以导入包直接使用。在C++中需要调用OPenssl第三方库，这个库包含所有加密库。

我们通过两种方式，把“hello world”进行加密，看看效果。

```
package main

import (
	//导入包
	"crypto/sha256"
	"fmt"
)

func main() {
	//定义变量
	s := "hello world"
	//调用函数
	mysha256(s)
}

//定义函数
func mysha256(s string) {
	//第一种方法
	sum := sha256.Sum256([]byte(s))
	fmt.Printf("%x\n", sum)

	//第二种方法
	h:=sha256.New()
	h.Write([]byte(s))
	fmt.Printf("%x\n", h.Sum(nil))

}
```

运行结果：

```
b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9
b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9

Process finished with exit code 0
```
哈希值的输出一般是16进制的字符串，而16进制的字符串每两个字符占一个字节。

我们可以看到两种方式得到哈希值是一样的，相比之下第一种方式看着更简单。

#### 文件哈希

我们现在桌面新建一个名为test.txt的文件，里面输入“hello world”，然后读取里面内容，进行哈希。


```
package main

import (
	"fmt"
	"os"
	"crypto/sha256"
	"io"
)

func main() {
	//调用函数
	MyIOSha256()
}

//将文件中的内容做sha256加密
func MyIOSha256()  {
	f,_ := os.Open("/...文件路径.../test.txt")
	defer f.Close()
	
	h:=sha256.New()
	//复制文件内容
	io.Copy(h,f)
	fmt.Printf("%x\n", h.Sum(nil))

}
```

运行结果：

```
b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9

Process finished with exit code 0
```

可以看出对文件内容进行哈希和上面我们直接哈希字符串结果是一样的，明文一致，得到的密文一样。

#### Md5

Md5和Sha256有些类似，我们也通过两种方式，把“hello world”进行加密，看看效果。

```
package main

import (
	//导入包
	"fmt"
	"crypto/md5"
	"encoding/hex"
)

func main() {
	d := []byte("hello world")
	//调用函数
	MyMd5(d)
}

//定义Md5函数
func MyMd5(d []byte)  {
	//第一种方法
	s := fmt.Sprintf("%x", md5.Sum(d))
	fmt.Println(s)

	//第二种方法
	h := md5.New()
	h.Write(d)
	s = hex.EncodeToString(h.Sum(nil))
	fmt.Println(s)
}

```

运行结果：

```
5eb63bbbe01eeed093cb22bb8f5acdc3
5eb63bbbe01eeed093cb22bb8f5acdc3

Process finished with exit code 0
```

两种方式结果一样，相比第一种更简单。

#### Ripemd160

同样加密“hello world”看看效果。

```
package main

import (
	"golang.org/x/crypto/ripemd160"
	"fmt"
)

func main() {
	//调用函数
	MyRipemd160()
}

//ripemd160 实现加密
func MyRipemd160() {
	hasher := ripemd160.New()
	hasher.Write([]byte("hello world"))

	hashString := fmt.Sprintf("%x", hasher.Sum(nil))

	fmt.Println(hashString)
}
```

运行结果：

```
98c615784ccb5fe5936fbc0cbe9dfdb408d92f0f

Process finished with exit code 0
```

好啦，哈希算法就说到这里了。




