---
layout: post
title: 非对称加密-算法原理
categories: [密码学]
tags: [Pow]
date: 2018-05-10 15:26:18
comments: true
---




#### 非对称加密

由于对称加密需要发送方加密后不光要把密文给接收方，还要把密钥给接收放，密钥泄露存在很大安全风险。

人们尝试过很多办法，比如事先共享密钥、通过密钥分配中心把密钥放到中央服务器等方法，但始终没有解决，直到非对称加密的出现。

非对称加密一次生成一个密钥对，公钥和私钥。公钥用来加密，私钥用来解密。

![](http://p8r45ciyt.bkt.clouddn.com/%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86.png)

在使用时接受者把公钥给发送者，发送者用公钥加密后的密文给接收者，接受者用自己的私钥解密。即使密文泄露，没有私钥也无法破解密文。

为什么公钥加私钥就无法破解密文呢？我们举个例子了解下原理：3乘以5等于15。

在这里“15”就是公钥（加密密钥），“3乘以5”是私钥（解密密钥）。可以看到1乘以15，15乘以1，5乘以3都可以，实际应用中可能性非常多，所以很难破解私钥。

我们实际使用中的密钥对的生成规则比这个复杂的多（涉及到小费马定理和欧拉函数，有兴趣可以去了解下），一般调用第三方库来生成密钥对。当然不会向我们的例子那么简单。

Rsa、椭圆加密就是非对称加密的典型应用，其中Rsa加密应用非常广泛，mac电脑下自带Rsa库。我们先来通过mac自带工具看看Rsa的公钥和私钥。

#### 查看公钥和私钥

通过自带工具看Rsa的公钥和私钥。

```
//mac打开终端
输入“openssl”

//制作私钥
输入“genrsa
-out rsa_private_key.pem 2048”

//设置私钥为PKCS8格式
输入“pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM”
//接下来会要求输入计算机密码，确认密码

//通过私钥产生公钥
输入“rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem”

//最后出现了“writing RSA key”，说明密钥对已经生成好了。
```

然后在当前用户目录下就可以看到公钥和私钥文件，用文本打开就可以看到了。

![](http://p8r45ciyt.bkt.clouddn.com/%E5%85%AC%E9%92%A5%E5%92%8C%E7%A7%81%E9%92%A5.png)

#### 完整代码

我们之前在mac系统下产生了一个密钥对，我们用文本把文件打开，用刚刚产生的密钥对来实现Rsa的加密和解密。

```
package main

import (
	"encoding/pem"
	"crypto/x509"
	"crypto/rsa"
	"crypto/rand"
	"fmt"
	"encoding/base64"
)

//通过Rsa的公钥加密，私钥解密

//私钥 `` 证明没有换行符，把生成的私钥用文本打开完整复制
var privateKey = []byte(`-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxd2ZR9vir5njuP2PH5whtYVPiLdpb+cK7zdFv3i0eX8kDWpv
5IOLz72n9Ck/Ybl424E6wHtZCddZgVo85yLi+K2AqCvZBnG8FcPD+olDFUG1txW6
8a82eA4AN+pBQxXRth0OwYS/74XrI8u+ZkGaAdeuGpbbHkl5AVFuhFLsGeG45T+e
Keg1wV1zuOjAzUfMK2Ls5SnxQMx6zYIVkOjvKj4GAw4QZakXkjlw/ngI+WH0XBme
Wj803jfnWoKBUDDOAFEOuasiR09DS3oiSPrgE9W/eGXkdW400Z3LGmehT+7Mb+R+
LeT4UgkkwCJ9fiSZTv9XbT5ZVh015kdLsFII1QIDAQABAoIBABYQW+NTdbe3JVmf
jLItquSe9Pt92FgOH34FX9W2FAnoT5DfaZLFyHVl5LCpWNQA5qUzh+Wm24rpZKWz
9k6f6UdpsYsPOKgrTpnuto/ddomMRkMgPTRuIVjJ1uRlTtm4OSZhnx+dOfnvqQTY
12Z3skC6WEPGxrBd6QxolDZVONa5zpOSXVowYYAid8cdsBOjlEdzC7+ExQWL+bng
RLxXT2oRK8P7g6vY+XrVh0R1neR6A67jhNXppZDYvWt/pIhLefcbPIqMMRbfBguP
gV1AZDP2cVef5f49wZarcWDEmHilXXz4B10ByxjZgG4H34GUM2OIiAxnJSaBJDhh
vECYVIECgYEA9EPPF9NwYhILpW13u3HMDTxkmuNCvv9B7qXwEAFNz1X22YZtOCBK
uYsiEBy1a9Tjm/93H10QHrkL8IIk5aUi72IVXhrHYAiKZjenHlzpOSD6cEyM6hDH
vsaQlJXJFMOI2qcKUVc1JkPEAOkcIrUzMdvjJehAaWTTS0KBAmiTDS0CgYEAz18h
LynZXJB+gd+jNlpf3V+shKQk+HH86UUQ4Ru5if4DbnFxouhOWNmWQQFBnuQKmaBX
QtPC7os9HqGKlezpFDGZBcBaUDhRyulQQCW8sC8n6M+a2RwTnvGDeRFfB/N2Z2dq
WPC1a/SbaMuqGf+Zp6JVYJE17lfqreoL52UCw0kCgYEAiWK+TztYgYB+1mvMpTwr
NeKa/1cFiqHNdqoUbRwepJhIQC7QrXnULano3cEX9W+HGY3FdXmFgJI5+etpT1Tj
Ylr7g7NyIjyLg1SYBYbikoRO9+zGcTxA7LeISFo7ABe+mKTNM9TmCwCgdJaogYkD
I272wrJv0BeqlDDymOUymH0CgYAcUcuAW2C5yWndZqMtaw4od0ZiHuCFpVt6p9n0
RAsEk1H4pTl1m/AHJj/kxL0na9Eexczk7XJzjURdiIYaj24NOfDB3lD3H5nb8hzp
hb3M+cOjgaaN+82aKTVhNUQbG96RpIfbeZPtGEyY9SdXwZZEVGEAfRQ2Zn9AHPRf
N2Y3MQKBgQCM346Or7fDH9dbH7GxXIYx+JIP5Kna5gS84vWEOIKlA2CJhgIgmms/
2h0Qh+f3KPgDJPshtkfRXwB0suPOQ5Juz/+wRuBzG4pPvbtZJWgTRrhSrx39sWah
pUJtmATuY6JF20UlTvLzt/LDANI+XWXg+TNPn3grhZ4/VDDX8OaTzA==
-----END RSA PRIVATE KEY-----
`)

//生成的私钥
var publicKey = []byte(`-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxd2ZR9vir5njuP2PH5wh
tYVPiLdpb+cK7zdFv3i0eX8kDWpv5IOLz72n9Ck/Ybl424E6wHtZCddZgVo85yLi
+K2AqCvZBnG8FcPD+olDFUG1txW68a82eA4AN+pBQxXRth0OwYS/74XrI8u+ZkGa
AdeuGpbbHkl5AVFuhFLsGeG45T+eKeg1wV1zuOjAzUfMK2Ls5SnxQMx6zYIVkOjv
Kj4GAw4QZakXkjlw/ngI+WH0XBmeWj803jfnWoKBUDDOAFEOuasiR09DS3oiSPrg
E9W/eGXkdW400Z3LGmehT+7Mb+R+LeT4UgkkwCJ9fiSZTv9XbT5ZVh015kdLsFII
1QIDAQAB
-----END PUBLIC KEY-----`)

//加密函数
func RsaEncrypt(origData []byte) [] byte {
	//公钥加密
	//把公钥整成block块
	block, _ := pem.Decode(publicKey)

	//解析公钥
	pubInterface, _ := x509.ParsePKIXPublicKey(block.Bytes)

	//设置刚才的公钥为publickey。确定类型就是公钥
	pub := pubInterface.(*rsa.PublicKey)

	//加密[]byte("hello world")明文
	bts, _ := rsa.EncryptPKCS1v15(rand.Reader, pub, []byte("hello world"))
	return bts

}

//解密函数
func RsaDecrypt(origData []byte) []byte {
	//通过私钥解密
	block, _ := pem.Decode(privateKey)

	//解析私钥
	priv, _ := x509.ParsePKCS1PrivateKey(block.Bytes)
	//解码
	bts, _ := rsa.DecryptPKCS1v15(rand.Reader, priv, origData)

	return bts
}

func main() {
	encryptecode := RsaEncrypt([]byte("hello world"))
	//base64转码，字节码编程string字符串类型
	fmt.Println(base64.StdEncoding.EncodeToString(encryptecode))

	decryptcode := RsaDecrypt(encryptecode)
	fmt.Println(string(decryptcode))
}


```

运行结果：


```
rJenDJbfxUFF4yZjK6XcYHgBUjunVsz+qvXR2tpWMTebJzsKU8oeNthBMIcyy3/00XF4fotrRsvA8+5lmBbgQVLTxSiYDpI0gXsQpi95IjjsQKNnxzunKI73otxJesstEin46YFJzgghmqA42wwFLlOr4HFjZedCkf8R5jjTeTTG80ELYYNQipQz+Vlj3l220vog2lmTGBHKyctyZ6vlyKpdcRsjz4RNBtbK+YrW6ca+BlqosKdY07/tRbT/WgTwJgaoSonOXHyAYQ3xWGXLBwn5YhSUdjCID+JZfyJIrsX6VAsrMgEvTmaB1NUHvFiZ45Q7AuhOAlKeimS0HDUT8w==
hello world

Process finished with exit code 0
```

其实Rsa是自带产生公钥和私钥的功能的，我们用自带的功能产品对钥对来试一下。

#### Rsa自带生成密钥对实现加密解密


```
package main

import (
	"crypto/rsa"
	"crypto/rand"
	"fmt"
	"crypto/md5"
	"encoding/base64"
)

//通过Rsa实现加密和解密
//利用Rsa的方法生成密钥对

func main() {
	//Rsa首先生成的私钥，然后根据私钥生成公钥
	//生成1024位私钥
	priv, _ := rsa.GenerateKey(rand.Reader, 1024)

	//根据私钥产生公钥。
	pub := &priv.PublicKey

	fmt.Println("私钥:", priv)
	fmt.Println("公钥:", pub)

	//定义明文
	plaintext := []byte("hello world")
	//加密成密文
	ciphertext, _ := rsa.EncryptOAEP(md5.New(), rand.Reader, pub, plaintext, nil)
	//打印密文
	fmt.Println(base64.StdEncoding.EncodeToString(ciphertext))

	//解密
	plaintext,_ =rsa.DecryptOAEP(md5.New(),rand.Reader,priv,ciphertext,nil)
	fmt.Println(string(plaintext))

}

```

运行结果：

```
kBIUNoYElVZEajD51p2UyZ4Dku9EAEgZe7L+bl3k9p4iM0iApKtfRRJPrj16ZyTJWw7vSDfOm8uZ42doGPFZ6pWGkRDmWJz5Wfn+QZzR/SCTdZ0xaVK1FbxYyZJnEXM3+jk6p4=
hello world
```

