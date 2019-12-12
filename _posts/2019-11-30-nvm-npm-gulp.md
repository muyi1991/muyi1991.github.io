---
layout: post
title: nvm和node.js环境配置，npm的使用和gulp的安装
categories: [前端]
tags: [前端]
date: 2019-11-30 13:48:07
comments: true
---


#### 实现步骤

步骤：
1. nvm的安装和环境变量设置
2. 安装node.js
3. 安装npm
4. 安装自动化流程工具gulp


#### nvm的安装和nvm的环境变量设置

启用终端Terminal，输入脚本，然后回车。

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```

安装完 nvm 后，输入nvm，当看到有输出时，则 nvm 安装成功。 
安装完成后配置环境变量。
在终端输入 "open -e .bash_profile"，在弹出的环境变量设置中输入一下代码。

```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" 

```
保存后关闭环境变量文件，然后 source 一下 .bash_profile，即可完成安装配置。

以下时nvm常用命令。


```

最后nvm的常用命令 ：

nvm install stable ## 安装最新稳定版 node
nvm install <version> ## 安装指定版本
nvm uninstall <version> ## 删除已安装的指定版本，语法与install类似
nvm use <version> ## 切换使用指定的版本node
nvm ls ## 列出所有安装的版本
nvm ls-remote ## 列出所有远程服务器的版本（官方node version list）
nvm current ## 显示当前的版本
nvm alias <name> <version> ## 给不同的版本号添加别名
nvm unalias <name> ## 删除已定义的别名
nvm reinstall-packages <version> ## 在当前版本 node 环境下，重新全局安装指定版本号的 npm 包
nvm  所有命令  自行logo
```

#### 安装node.js

在终端输入一下命令即可完成安装。

```
nvm install 6.4.0
```

#### 安装npm

npm在安装node的时候已经安装好了。
nvm 6.4.0 的版本对应的npm版本时 3.10.3
nvm use 6.4.0 切换版本后输入 npm -v 查看npm的版本。

使用npm工具安装node.js需要用到的包。

示例：
```
npm install express //本地安装
npm install express -g //全局安装，可以在命令行中使用
```

#### 安装自动化流程工具gulp

需要在本地和全局中都要安装，需要require使用和在终端中使用。

```
npm install gulp --save-dev //本地安装，安装完成后保存记录package.json中
npm install gulp -g //全局安装，可以在命令行中使用
```



 




    
    










