---
layout: post
title: 虚拟环境和virtualenvwrapper的使用
categories: [后端]
tags: [虚拟环境]
date: 2020-1-2 13:48:07
comments: true
---


#### 什么是虚拟环境

为了开发不同的项目时候用不同的包方便管理，我们使用不同的虚拟环境去开发项目，这样就不至于在一个系统级别的目录中去安装不同的包，导致项目管理的混乱。

virtualenv 是virtual(虚拟)和environment(环境)的缩写。

pip对应python2，pip3对应python3，如果你的系统中只有一个python环境，那么pip就直接对应的是python3。结论：只使用一个python3环境，不安装python2，如果安装了不用就卸载掉。

#### virtualenvwrapper

为了方便统一的管理的虚拟环境，方便使用，我们需要用到virtualenvwrapper来管理我们的虚拟环境。
运行一下命令安装virtualenvwrapper。如果安装的时候没有安装virtualenv，就帮我们顺带安装virtualenv。

```
pip install virtualenvwrapper
```
安装完成后就是如何去创建，查看，更换，退出，删除虚拟环境了。

#### 创建虚拟环境

运行以下命令创建虚拟环境.mkvirtualenv+虚拟环境名字

```
mkvirtualenv my_env
```

#### 配置环境变量

打开终端进入根目录“cd ~”打开虚拟环境配置
```
open -e  .bash_profile //修改环境变量命令
```

把以下命令放入.bash_profile里面

```
export WORKON_HOME=~/workspaces
source /usr/local/bin/virtualenvwrapper.sh
```

更新刚配置的环境变量

```
source .bash_profile
```

#### 查看虚拟环境

查看系统安装的所有虚拟环境列表

```
workon
```

#### 切换到某个虚拟环境

workon+虚拟环境名字

```
workon my_env
```

#### 退出虚拟环境

deactivate命令退出虚拟环境

```
deactivate
```

#### 删除某个虚拟环境

rmvirtualenv+虚拟环境名字

```
rmvirtualenv my_env
```

#### 进入到虚拟环境目录

```
cdvirtualenv
```




