---
layout: post
title: 对称加密-写一个自己的算法
categories: [密码学]
tags: [对称加密]
date: 2018-05-03 12:12:18
comments: true
---

### 介绍

我们用“123”作为密钥给“abc”明文进行加密，然后再解密。

通过自己写，来了解对称加密是怎样一步步实现的。

先看完整代码。

#### 完整代码


```
package main

import (
	"fmt"
	"bytes"
	"encoding/base64"
)

//加密
func EnCrypt(orig []byte, key []byte) []byte {
	//实现将密钥中每个字节累加,通过sum累积求和实现orig的加密工作
	sum := 0
	for i := 0; i < len(key); i++ {
		sum = sum + int(key[i])
	}

	//个明文补码
	var pk_code = PKCS5Padding(orig, 8)

	//通过密钥对补码后的明文进行加密[97 98 99 5 5 5 5 5],每一位和sum相加[150+97 150+98 ... 150+5]
	for k := 0; k < len(pk_code); k++ {
		pk_code[k] = pk_code[k] + byte(sum)
	}

	return pk_code
}

//解密
func DeCrypt(orig []byte, key []byte) []byte {
	sum := 0
	for i := 0; i < len(key); i++ {
		sum = int(key[i]) + int(sum)
	}

	//解密
	//加密过程要有反函数
	for k := 0; k < len(orig); k++ {
		orig[k] = orig[k] - byte(sum)
	}

	//去码，去码之后就变成了明文的数组[96 97 98 5 5 5 5 5]
	var pk_un_code = PKCS5Unpadding(orig)
	//fmt.Println(pk_un_code)
	return pk_un_code

}

//去码函数
func PKCS5Unpadding(orig []byte) []byte {
	//获取数组中最后一位，len-1，最大下标，就是最后一位
	var tail = int(orig[len(orig)-1])
	return orig[:(len(orig) - tail)] //8-5=3,取前三位
}

func PKCS5Padding(orig []byte, size int) []byte {
	//计算明文长度
	length := len(orig)
	//要补的数字
	padding := size - length%size
	//向byte类型数组中重复添加55555
	repeats := bytes.Repeat([]byte{byte(padding)}, padding)
	//字节拼接
	return append(orig, repeats...)
}

func main() {
	var EnCrypttCode = EnCrypt([]byte("abc"), []byte("123"))
	//打印密文,base64把他打印成串
	fmt.Println(base64.StdEncoding.EncodeToString(EnCrypttCode))

	var DeCryptCode = DeCrypt(EnCrypttCode, []byte("123"))
	//打印解密的明文
	fmt.Println(string(DeCryptCode))
}

```

运行结果：


```
9/j5m5ubm5s=
abc

Process finished with exit code 0
```

#### 代码详解


加密过程：
举例abc
原理：先通过给abc长度取模一个size，假设size是8，3模8得3，然后8减去取模数得5，重复补足8位数的补码。比如abc对应[97 98 99]，做完补码后应该是[97 98 99 5 5 5 5 5]

然后我们把key中的值做一个累加sum，把补码后的结果一一很sum做累加，得到密文。然后用特定特使输出密文。

解密过程：
相反。




通过一步步了解，感觉对称加密算法是不是也没那么复杂？

哈哈，其实算法的实现原理解起来比创建算法本身简单多了，算法本身才是核心。然而很多经典算法都是数学家好了很大心血才研究出来的，大多数情况下我们直接用就可以了。

有句话这样说的“站在巨人的肩膀上我们可以看的更远！”。


