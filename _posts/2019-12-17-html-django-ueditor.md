---
layout: post
title: ueditor富文本的集成
categories: [后端]
tags: []
date: 2019-12-17 13:48:07
comments: true
---


#### 拷贝前端代码、后端代码

拷贝dist目录下面ueditor文件夹
拷贝ueditor这个app文件夹到django项目文件中

#### 后端配置ueditor

把ueditor这个app放到settings.py中。
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'apps.cmsauth',
    'apps.index',
    'apps.product',
    'utils',
    'apps.ueditor'
]
```

在主urls中加子url。


```
urlpatterns = [
    path('', include('apps.index.urls')),
    path('account/', include('apps.cmsauth.urls')),
    path('product/', include('apps.product.urls')),
    path('ueditor/', include('apps.ueditor.urls')),//在富文本编辑器的映射
] + static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT)

```


#### 配置前端代码

引用静态文件config.js和min.js
```
{% block head %}
    <script src=" 'ueditor/ueditor.config.js' " ></script>
    <script src=" 'ueditor/ueditor.all.min.js' " ></script>
{% endblock %}
```

在页面中定义script标签定义富文本编辑器给一个id，然后给id进行初始化就可以定义一个富文本编辑器了。

```
<script id="editor" type="text/plain" ></script>
```

在js中条用sdk方法，初始化编辑器。

```
Product.prototype.InitUEditorEvent = function(){
    window.ue = UE.getEditor('editor',{
        'initialFrameHeight':400,
        'serverUrl':'/ueditor/upload/'//配置url上传view对应的事件配置后可以上传图片
    })
};
```

在settings.py中配置图片上传的url
可以两种方式，上传到自己服务器或者上传到七牛云

```
# Qiniu配置
QINIU_ACCESS_KEY = 'WX2GvY4K3gpS_FrDEiZa9lq1aTqxeofaw5iM4ZMm'
QINIU_SECRET_KEY = 'POeH40wL2lTwlQIiBxgi_AerrzbODedvb3kRCbSj'
QINIU_BUCKET_NAME = 'xfzspace1'
QINIU_DOMAIN = 'http://q2fz60woz.bkt.clouddn.com'

# 七牛和自己的服务器，最少要配置一个
# UEditor配置
UEDITOR_UPLOAD_TO_QINIU = True
UEDITOR_QINIU_ACCESS_KEY = QINIU_ACCESS_KEY
UEDITOR_QINIU_SECRET_KEY = QINIU_SECRET_KEY
UEDITOR_QINIU_BUCKET_NAME = QINIU_BUCKET_NAME
UEDITOR_QINIU_DOMAIN = QINIU_DOMAIN

UEDITOR_UPLOAD_TO_SERVER = True
UEDITOR_UPLOAD_PATH = MEDIA_ROOT
UEDITOR_CONFIG_PATH = os.path.join(BASE_DIR,'cms','dist','ueditor','config.json')
```

#### 获取ueditor的内容

通过window.ue.getContent()拿到富文本编辑器里面的内容

```
Product.prototype.ListenPubEvent = function(){
    var pubBtn = $("#pub-btn");
    pubBtn.click(function (event) {
        event.preventDefault();
        var title = $("input[name='title']").val();
        var category = $("select[name='category']").val();
        var price = $("input[name='price']").val();
        var thumbnail = $(".custom-file-label").text();
        var detail = window.ue.getContent();
        // console.log(title,category,price,thumbnail,detail)
        xfzajax.post({
            'url':'/product/add_product/',
            'data':{
                'title':title,
                'price':price,
                'thumbnail':thumbnail,
                'detail':detail,
                'category':category
            },
            'success':function (result) {
                if (result['code'] === 200){
                    window.messageBox.showSuccess('发布成功')
                    setTimeout(function () {
                        window.location.reload()
                    },1500)
                }
            }
        })
    })
};
```


#### 自定义工具栏

参考链接：http://fex.baidu.com/ueditor/#start-toolbar






