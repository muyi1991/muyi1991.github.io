---
layout: post
title: url详解和Django的安装
categories: [后端]
tags: []
date: 2020-1-3 13:48:07
comments: true
---


#### url详解


```
scheme://host:port/path/?query-string=xxx#anchor
```

* scheme:协议。http默认端口80.https默认端口443.
* host:主机名，域名。如；www.baidu.com
* port:端口号，访问一个网站，浏览器默认使用80端口号。
* path:查找路径。如：www.jianshu.com/newses_list/，后面的/newses_list/就是path
* query-string:查询字符串。如：www.baidu.com/?wd=python，后面的wd=python就是查询字符串。
* anchor:锚点。前端页面用来定位的。

#### 安装Django2.0

进入项目的虚拟环境，然后通过“pip install django==2.0”，我们使用到的是Django2.0


安装pycharm profession 2017或以上版本。

#### 安装数据库

安装最新版本的mysql，安装5.7版本的数据库。

安装数据库引擎，“pip install pymysql”，进行安装数据库引擎。python操作数据库的中间件。

