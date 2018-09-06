---
layout: post
title: 非对称加密-签名和验签
categories: [密码学]
tags: [非对称加密]
date: 2018-05-13 15:26:18
comments: true
---



#### 签名和验签

数字签名是非对称加密的一种应用。在非对称加密传递信息过程中可能被攻击者调包攻击，因此非对称加密也用来识别唯一的发送者。

只有发送者用私钥才能产生，别人无法伪造，然后接受者用公钥去验签，从而核实信息信息的真实性。

关于非对称加密两个应用方面：用来信息加密和解密，用来签名和验签。你可以这样理解：

```
既然是加密，那肯定是不希望别人知道我的消息，所以只有我才能解密，所以可得出公钥负责加密，私钥负责解密；
同理，既然是签名，那肯定是不希望有人冒充我发消息，只有我才能发布这个签名，所以可得出私钥负责签名，公钥负责验证。
```

go语言中签名和验签可以通过Rsa和Dsa（专业做验签的算法）完成。
Rsa作用加密，解密。签名，验签（需要密文中散列）。
Dsa专门做签名，验签。


#### Dsa

Dsa是专门做签名和验签的。


```
package main

import (
	"crypto/dsa"
	"crypto/rand"
	"fmt"
)
//验签作用，确保数据的完整性，还可以确保数据的来源。
func main() {
	//Dsa 专业做签名和验签
	var param dsa.Parameters
	//配置
	dsa.GenerateParameters(&param,rand.Reader,dsa.L1024N160)

	//生成私钥
	var priv dsa.PrivateKey
	priv.Parameters = param
	dsa.GenerateKey(&priv,rand.Reader)

	//通过私钥生成公钥
	pub := priv.PublicKey

	//利用私钥签名数据
	message := []byte("hello world")
	r,s,_ := dsa.Sign(rand.Reader,&priv,message)

	//利用公钥验签,返回布尔类型
	b := dsa.Verify(&pub,message,r,s)
	if b {
		fmt.Println("验签成功")
	}
}


```

运行结果：


```
验签成功

Process finished with exit code 0
```



#### Rsa

Rsa实现签名和验签。

```
package main

import (
	"crypto/rsa"
	"crypto/rand"
	"crypto/md5"
	"crypto"
	"fmt"
)

//Rsa签名和验签

func main() {
	//生成私钥
	priv , _:=rsa.GenerateKey(rand.Reader,1024)
	//产生公钥
	pub := &priv.PublicKey

	//设置明文
	plaintext := [] byte("hello world")

	//给明文做哈希散列
	h := md5.New()
	h.Write(plaintext)
	hashed := h.Sum(nil)

	//签名
	opts :=&rsa.PSSOptions{SaltLength:rsa.PSSSaltLengthAuto,Hash:crypto.MD5}

	sig ,_ :=rsa.SignPSS(rand.Reader,priv,crypto.MD5,hashed,opts)

	//认证
	e := rsa.VerifyPSS(pub,crypto.MD5,hashed,sig,opts)

	if e==nil {
		fmt.Println("验签成功")
	}


}

```

运行结果：

```
验签成功

Process finished with exit code 0
```

#### 椭圆曲线

用椭圆曲线实现签名和验签。

```
package main

import (
	"crypto/ecdsa"
	"crypto/elliptic"
	"crypto/rand"
	"crypto/sha256"
	"math/big"
	"fmt"
)

//通过椭圆曲线完成签名和验证
func main() {
	//声明明文
	message := []byte("hello world")

	//生成私钥
	privateKey, _ := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)

	//生成公钥
	pub := privateKey.PublicKey

	//将明文哈希散列
	digest := sha256.Sum256(message)

	//签名
	r, s, _ := ecdsa.Sign(rand.Reader, privateKey, digest[:])

	//设置私钥参数类型为曲线类型
	param := privateKey.Curve.Params()

	//获得私钥byte长度
	curveOrderByteSize := param.P.BitLen() / 8

	//获取签名返回值的字节
	rByte,sByte :=r.Bytes(),s.Bytes()

	//创建数组
	signature := make([]byte, curveOrderByteSize*2)

	//通过数组保存了签名结果的返回值
	copy(signature[curveOrderByteSize-len(rByte):], rByte)
	copy(signature[curveOrderByteSize*2-len(sByte):], sByte)


	//认证
	//将明文做哈希散列，为了做内容验证对比
	digest = sha256.Sum256(message)
	curveOrderByteSize = pub.Curve.Params().P.BitLen()/8

	//创建两个整形对象
	r,s =new(big.Int),new(big.Int)

	//设置整数值
	r.SetBytes(signature[:curveOrderByteSize])
	s.SetBytes(signature[curveOrderByteSize:])

	//认证
	e :=ecdsa.Verify(&pub,digest[:],r,s)

	if e {
		fmt.Println("验签成功")
	}else {
		fmt.Println("验签失败")
	}
}
```

运行结果：

```
验签成功

Process finished with exit code 0
```
















