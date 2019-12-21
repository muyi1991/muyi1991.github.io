---
layout: post
title: arttemplate的使用
categories: [前端]
tags: [arttemplate]
date: 2019-12-22 13:48:07
comments: true
---


#### 步骤

下载地址：https://github.com/aui/art-template
下载后打开example，打开basic.html看到以下内容，就是基本思路，理解思路。

1. 理解思路
2. 引用arttemplate的sdk，也就是js文件
3. 在html定义一个script，存放需要渲染的内容。需要有id。
4. 在js文件中构建一个html，找到一定的id，然后把data放进去。
5. 把插入的内容放到指定位置。



```
<!DOCTYPE HTML>
<html>
<head>
<meta charset="UTF-8">
<title>basic-demo</title>
<script src="../../lib/template-web.js"></script>
</head>

<body>
<div id="content"></div>
<script id="test" type="text/html">
{{if isAdmin}}

<h1>{{title}}</h1>
<ul>
    {{each list value i}}
        <li>索引 {{i + 1}} ：{{value}}</li>
    {{/each}}
</ul>

{{/if}}
{{$data}}
</script>

<script>


var data = {
	title: '基本例子',
	isAdmin: true,
	list: ['文艺', '博客', '摄影', '电影', '民谣', '旅行', '吉他']
};
var html = template('test', data);
document.getElementById('content').innerHTML = html;
</script>
</body>
</html>
```

#### 加载更多新闻案例


html文件中的代码如下：

```



    <script src="'arttemplate/template-web.js' "></script>//导入sdk文件
    <script id="news-item" type="text/html">//定义一个js文件ID
        {% verbatim %}//在django中告诉Django模版这部分代码不是django的模版代码
            {{ each newses news index }}
            <li>
                <div class="thumbnail-group">
                    <a href="/news/{{ news.id }}/">
                        <img src="{{ news.thumbnail }}" alt="">
                    </a>
                </div>
                <div class="news-group">
                    <p class="title">
                        <a href="/news/{{ news.id }}/">{{ news.title }}</a>
                    </p>
                    <p class="desc">
                        {{ news.desc }}
                    </p>
                    <p class="more">
                       <span class="category">{{ news.category.name }}</span>
                        <span class="pub-time">{{ news.pub_time|timeSince }}</span>
                        <span class="author">{{ news.author.username }}</span>
                    </p>
                </div>
            </li>
            {{ /each }}
        {% endverbatim %}
    </script>

```

js文件中的代码如下：

```
function Index() {
    var self = this;
    self.page = 2;
    self.category_id = 0;
    self.loadBtn = $("#load-more-btn");
}

Index.prototype.listenLoadMoreEvent = function () {
    var self = this;
    var loadBtn = $("#load-more-btn");
    loadBtn.click(function () {
        xfzajax.get({
            'url': '/news/list/',
            'data':{
                'p': self.page,
                'category_id': self.category_id
            },
            'success': function (result) {
                if(result['code'] === 200){
                    var newses = result['data'];
                    if(newses.length > 0){
                        var tpl = template("news-item",{"newses":newses});
                        var ul = $(".list-inner-group");
                        ul.append(tpl);
                        self.page += 1;
                    }else{
                        loadBtn.hide();
                    }
                }
            }
        });
    });
};

Index.prototype.listenCategorySwitchEvent = function () {
    var self = this;
    var tabGroup = $(".list-tab");
    tabGroup.children().click(function () {
        // this：代表当前选中的这个li标签
        var li = $(this);
        var category_id = li.attr("data-category");
        var page = 1;
        xfzajax.get({
            'url': '/news/list/',
            'data': {
                'category_id': category_id,
                'p': page
            },
            'success': function (result) {
                if(result['code'] === 200){
                    var newses = result['data'];
                    var tpl = template("news-item",{"newses":newses});//把模版赋值给script，找到模版的id
                    // empty：可以将这个标签下的所有子元素都删掉
                    var newsListGroup = $(".list-inner-group");
                    newsListGroup.empty();
                    newsListGroup.append(tpl);//追加某个元素到后面
                    self.page = 2;
                    self.category_id = category_id;
                    li.addClass('active').siblings().removeClass('active');//移除其他兄弟元素的选中状态
                    self.loadBtn.show();
                }
            }
        });
    });
};

Index.prototype.run = function () {
    var self = this;
    self.listenLoadMoreEvent();
    self.listenCategorySwitchEvent();
};

$(function () {
    var banner = new Banner();
    banner.run();

    var index = new Index();
    index.run();
});
```


