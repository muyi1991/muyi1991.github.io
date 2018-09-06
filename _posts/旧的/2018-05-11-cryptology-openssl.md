---
layout: post
title: 非对称加密-编译Openssl库
categories: [密码学]
tags: [非对称加密]
date: 2018-05-11 17:26:18
comments: true
---

#### 下载文件

Openssl包含Md5，Sha256。Des，Aes。Rsa。
Openssl是一个生成非对称加密的公钥和私钥的第三方库，编译好这个文件后我们就可以在程序中调用了。

先去[https://www.openssl.org](https://www.openssl.org)网站下载源码。

解压下载的文件，得到一个文件夹。

#### 命令行

在终端打开文件夹。

![](http://p8r45ciyt.bkt.clouddn.com/Openssl%E6%96%87%E4%BB%B6.png)

然后按顺序输入以下命令。

* 输入命令“./config --prefix=/usr/local/openssl”。设定Openssl安装路径。
* 输入命令“./config -t”。
* 输入命令“make”。编译Openssl，编译需要等待一定时间。
* 输入命令“make install”。安装Openssl，安装也需要一定时间。


安装完成后，在文件夹中就可以找到libssl.a和libcrypto.a编译后的库文件。

![](http://p8r45ciyt.bkt.clouddn.com/Openssl%E7%BC%96%E8%AF%91%E5%90%8E.png)

然后就可以使用了。




