---
layout: post
title: Django前端环境搭建和安装相应包
categories: [后端]
tags: [前端]
date: 2019-12-01 13:48:07
comments: true
---


#### 实现步骤

步骤：
1. 创建cms文件夹，在里面创建src,dist,template文件夹
2. 初始化cms，在终端输入"npm init"初始化成为gulp项目
3. 复制"package.json"中的包名称，安装相应的包
4. 在cms文件夹下创建“gulpfile.js”文件，编写gulp命令
5. 检查最终修改scss文件保存直接在浏览器看到效果，检查目录结构

#### 创建文件夹

在apps的同级目录上创建一个cms文件夹，存放前端的模版和静态文件。结构如下：

```Python
apps
cms
    dist
    src
        css
        images
        js
   templates
        index
        goods
        order
        ...
```


#### 初始化成为gulp项目

在终端中cd到cms项目文件夹下，运行"npm init"命令。
在终端弹出的提示框输入name,description,author等等内容...然后输入"yes"，看到cms文件夹下面生成了"package.json"文件，表示初始化成功。


#### 复制"package.json"中的包名称，安装相应的包

复制devDependencies中的内容，然后粘贴到package.json文件中。代码如下:


```Json
{
  "name": "xfz",
  "version": "1.0.0",
  "description": "xfz front code",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "muyi",
  "license": "ISC",
  "devDependencies": {
    "browser-sync": "^2.24.4",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^5.0.0",
    "gulp-cache": "^1.0.2",
    "gulp-concat": "^2.6.1",
    "gulp-concat-folders": "^1.3.1",
    "gulp-connect": "^5.5.0",
    "gulp-cssnano": "^2.1.3",
    "gulp-imagemin": "^4.1.0",
    "gulp-rename": "^1.2.3",
    "gulp-sass": "^4.0.1",
    "gulp-sourcemaps": "^2.6.4",
    "gulp-uglify": "^3.0.0",
    "gulp-util": "^3.0.8",
    "underscore": "^1.9.1"
  }
}

```

完成后cd到cms该文件夹下，运行“npm install”命令，安装所有"devDependencies"下的包。
安装完成后回到cms文件夹会看到多了一个"node_modules"文件，里面包含刚刚安装的所有包。


#### 在cms文件夹下创建“gulpfile.js”文件，编写gulp命令

代码如下：


```Js
// 导入需要用到的包
var gulp = require("gulp");
var cssnano = require("gulp-cssnano");
var rename = require("gulp-rename");
var uglify = require("gulp-uglify");
var cache = require('gulp-cache');
var imagemin = require('gulp-imagemin');
var bs = require('browser-sync').create();
var sass = require("gulp-sass");
// gulp-util：这个插件中有一个方法log，可以打印出当前js错误的信息
var util = require("gulp-util");
var sourcemaps = require("gulp-sourcemaps");


var path = {
    'html': './templates/**/',
    'css': './src/css/**/',
    'js': './src/js/',
    'images': './src/images/',
    'css_dist': './dist/css/',
    'js_dist': './dist/js/',
    'images_dist': './dist/images/'
};

// 处理html文件的任务
gulp.task("html",function () {
    gulp.src(path.html + '*.html')
        .pipe(bs.stream())
});

// 定义一个css的任务
gulp.task("css",function () {
    gulp.src(path.css + '*.scss')
        .pipe(sass().on("error",sass.logError))
        .pipe(cssnano())
        .pipe(rename({"suffix":".min"}))
        .pipe(gulp.dest(path.css_dist))
        .pipe(bs.stream())
});

// 定义处理js文件的任务
gulp.task("js",function () {
    gulp.src(path.js + '*.js')
        .pipe(sourcemaps.init())
        .pipe(uglify().on("error",util.log))
        .pipe(rename({"suffix":".min"}))
        .pipe(sourcemaps.write())
        .pipe(gulp.dest(path.js_dist))
        .pipe(bs.stream())
});

// 定义处理图片文件的任务
gulp.task('images',function () {
    gulp.src(path.images + '*.*')
        .pipe(cache(imagemin()))
        .pipe(gulp.dest(path.images_dist))
        .pipe(bs.stream())
});

// 定义监听文件修改的任务
gulp.task("watch",function () {
    gulp.watch(path.html + '*.html',['html']);
    gulp.watch(path.css + '*.scss',['css']);
    gulp.watch(path.js + '*.js',['js']);
    gulp.watch(path.images + '*.*',['images']);
});

// 初始化browser-sync的任务
gulp.task("bs",function () {
    bs.init({
        'server': {
            'baseDir': './'
        }
    });
});

// 创建一个默认的任务
// gulp.task("default",['bs','watch']);
gulp.task("default",['watch','bs']);

```

在cmd终端运行"gulp"命令，会自动打开默认浏览器，输入模版的html路径就可以直接访问到页面了。
如果遇到错误，在终端重新安装下imagemin和cache就行了，多数情况就是这两个包出问题。
因为版本的问题，cache要安装1.1.1版本，imagemin要安装低版本比如5.0.3，命令如下:

```cmd
npm install gulp-cache@1.1.1 --save-dev 安装到本地
npm install gulp-cache@1.1.1 -g 安装到全局
npm install gulp-imagemin@5.0.3  --save-dev 安装到本地
npm install gulp-imagemin@5.0.3  -g 安装到全局
```

#### 最终目录结构


```
cms
    dist //存放可以放到线上压缩过的静态文件css,js,images等
        css
        images
        js
        ...
    node_modules //存放已经安装的包
        ...
    src //存放开发环境下的静态文件css,js,images,font等
    templates //存放html代码，可以再创建目录，存放二级html代码
        index
        goods
        order
        ...
    gulpfile.js //存放需要执行的gulp命令，开发时在终端进入cms文件下执行“gulp”命令即可
    package.json //存放初始化信息，包含所有安装的包的信息。
```
    配置好了之后，以后开发时就可以在终端进入cms文件夹下执行gulp,gulp css,gulp js,gulp images等命令就可以实现保存样式，直接在浏览器看到效果了。
    
    










