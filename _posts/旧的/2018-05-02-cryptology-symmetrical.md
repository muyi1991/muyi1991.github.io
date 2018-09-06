---
layout: post
title: 对称加密-算法原理
categories: [密码学]
tags: [对称加密]
date: 2018-05-02 15:12:18
comments: true
---

### 对称加密

对称加密就是用相同的密钥加密和解密。

![](http://p8r45ciyt.bkt.clouddn.com/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86.png)

对称加密是相对于非对称加密而言的，非对称加密在加密和解密时使用不同的密钥，公钥和私钥。

对称加密使用时，都需要使用其他人不知道的唯一秘钥进行加密和解密。如果秘钥被泄露，那么加密信息也就不安全了。所以密钥的安全性甚至比算法安全性更为重要。

常用算法：DES，3DES，AES等。

#### Des

给“hello world”加密，然后解密，看看效果。

```
package main


import (
	"bytes"
	"crypto/cipher" //cipher密码
	"crypto/des"
	"encoding/base64" //将对象转换成字符串
	"fmt"
)


//DES加密方法
func MyDesEncrypt(origData, key []byte) {

	block, _ := des.NewCipher(key)

	//将明文按秘钥的长度做补码运算
	origData = PKCS5Padding(origData, block.BlockSize())

	//设置加密方式
	blockMode := cipher.NewCBCDecrypter(block, key)

	//创建明文长度的字节数组
	crypted := make([]byte, len(origData))

	//加密明文
	blockMode.CryptBlocks(crypted, origData)

	//将字节数组转换成字符串
	fmt.Println(base64.StdEncoding.EncodeToString(crypted))

}

//DES解密方法
func MyDESDecrypt(data string, key []byte) {

	//将字符串转换成字节数组
	crypted, _ := base64.StdEncoding.DecodeString(data)

	//将字节秘钥转换成block快
	block, _ := des.NewCipher(key)

	//设置解密方式
	blockMode := cipher.NewCBCEncrypter(block, key)

	//创建密文大小的数组变量
	origData := make([]byte, len(crypted))

	//解密密文到数组origData中
	blockMode.CryptBlocks(origData, crypted)

	//去补码
	origData = PKCS5UnPadding(origData)

	//打印明文
	fmt.Println(string(origData))

}

//实现明文的补码
func PKCS5Padding(ciphertext []byte, blockSize int) []byte {

	padding := blockSize - len(ciphertext)%blockSize

	padtext := bytes.Repeat([]byte{byte(padding)}, padding)

	return append(ciphertext, padtext...)

}

//实现去补码
func PKCS5UnPadding(origData []byte) []byte {

	length := len(origData)

	// 去掉最后一个字节 unpadding 次

	unpadding := int(origData[length-1])

	return origData[:(length - unpadding)]

}

func main() {
	//声明秘钥,利用此秘钥实现明文的加密和密文的解密
	key := []byte("12345678")

	//加密
	MyDesEncrypt([]byte("hello world"), key)

	//解密
	MyDESDecrypt("4Ducrt8n/7EfRBOkEUjD5w==", key)

}

```

输出结果：

```
4Ducrt8n/7EfRBOkEUjD5w==
hello world

Process finished with exit code 0
```

#### 3Des

3Des是Des的升级版。


```
package main

import (
	"bytes"
	"crypto/des"
	"crypto/cipher"
	"fmt"
	"encoding/base64"
)

//补码
func PKCS5Padding(ciphertext []byte, blocksize int) []byte {
	padding := blocksize - len(ciphertext)%blocksize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

//去码
func PKCS5unPadding(origData []byte) []byte {
	length := len(origData)
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}

//3Des加密
func TripleDesEncrypt(origData []byte, key []byte) []byte {
	//3Des的密钥长度必须为24位
	block, _ :=des.NewTripleDESCipher(key)

	//补码
	origData = PKCS5Padding(origData,block.BlockSize())

	//设置加密模式
	blockMode := cipher.NewCBCEncrypter(block,key[:8])

	//创建密文数组，加密
	crypted := make([]byte,len(origData))

	blockMode.CryptBlocks(crypted,origData)

	//将字节数组转换成字符串
	fmt.Println(base64.StdEncoding.EncodeToString(crypted))

	return crypted
}

//解密
func TipleDesDecrypt(crypted ,key  []byte) []byte  {
	block,_:=des.NewTripleDESCipher(key)
	blockMode := cipher.NewCBCDecrypter(block,key[:8])
	origData := make([]byte,len(crypted))

	blockMode.CryptBlocks(origData,crypted)

	origData = PKCS5unPadding(origData)


	return origData
}

func main()  {

	key :=[]byte("123456789012345678901234")
	encryptcode :=TripleDesEncrypt([]byte("hello world"),key)

	decryptcode := TipleDesDecrypt(encryptcode,key)

	fmt.Println(string(decryptcode))
}

```

运行输出：

```
WJ+EfR2QSeRw87h8u1yGbw==
hello world

Process finished with exit code 0
```

#### Aes

前面的Des和3Des现在很少用到，Aes目前更常用，所以我们重点掌握它的基本原理和实现。


```
package main

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"fmt"
)

//补码
func PkCS7Padding(ciphertext []byte, blocksize int) []byte {
	padding := blocksize - len(ciphertext)%blocksize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

//去码
func PKCS7UnPadding(origData []byte) []byte {
	length := len(origData)
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}

//通过Aes做加密
func AesEncrypt(origData []byte, key []byte) []byte {
	//分组密钥
	block,_ := aes.NewCipher(key)
	//获得密钥块的长度
	blocksize := block.BlockSize()
	//补码
	origData = PkCS7Padding(origData, blocksize)
	//加密模式,不同模式其加密算法不一样，攻击方式也不一样
	blockMode := cipher.NewCBCEncrypter(block, key[:blocksize])
	//创建数组
	cryted := make([]byte, len(origData))
	//加密
	blockMode.CryptBlocks(cryted,origData)

	return cryted
}

//Aes解密
func AesDecrypt(cryted [] byte, key [] byte) []byte {
	block,_ :=aes.NewCipher(key)
	blocksize := block.BlockSize()
	//设置解密方式
	blockMode := cipher.NewCBCDecrypter(block,key[:blocksize])
	origData :=make([]byte,len(cryted))
	blockMode.CryptBlocks(origData,cryted)
	origData = PKCS7UnPadding(origData)

	return origData
}

func main()  {
	//16,24,32位的密钥进行加密
	key :=[]byte("1234567890abcdef")
	encryptcode :=AesEncrypt([]byte("hello world"),key)

	decryptcode :=AesDecrypt(encryptcode,key)

	fmt.Println(string(decryptcode))
}

```
运行输出：

```
hello world

Process finished with exit code 0
```

