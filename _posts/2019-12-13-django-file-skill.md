---
layout: post
title: 上传文件到自己的服务器
categories: [后端]
tags: []
date: 2019-12-13 13:48:07
comments: true
---


#### 步骤

1. 完成视图函数
2. 完成配置
3. 完成html代码
4. 完成js请求


#### 视图函数


完成视图函数
```

import os
from django.conf import settings

@method_decorator(xfz_login_required,name='dispatch')
class UploadFile(View):
    def post(self,request,*args,**kwargs):
        file = request.FILES.get('file')
        name = file.name
        with open(os.path.join(settings.MEDIA_ROOT,name),'wb') as fp:
            for chunk in file.chunks():
                fp.write(chunk)
        url = request.build_absolute_uri(settings.MEDIA_URL+name)
        print(url)
        return restful.result(data={'url':url})
```

#### 完成配置

在settings.py中配置好media路径


```
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR,'media')

```

新建文件夹取名media

apps
cms
media //上传的文件存放在这里
utils
xfz
manage.py

在主url.py中完成完整路径的配置


```
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('', include('apps.index.urls')),
    path('account/', include('apps.cmsauth.urls')),
    path('product/', include('apps.product.urls')),
] + static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT) //配置路径

```

#### 完成前端html代码


```
<div class="form-group col-6">
{#                <img class="img-circle elevation-2" src="{{ url }}" id="img" alt="">#}
                <label for="thumbnail-form">缩略图</label>
                <div class="input-group">
                    <input type="file" class="custom-file-input" id="thumbnail-form" value="s">
                    <label for="thumbnail-form" class="custom-file-label"></label>
                </div>
            </div>
```

#### 完成js发送请求


```
function Product() {

};

Product.prototype.run=function () {
    var self = this;
    // this.listenSaveProductEvent();
    self.ListenUploadEvent();
};

Product.prototype.ListenUploadEvent = function(){
    var uploadBtn = $("#thumbnail-form");
    var img = $("#img");
    uploadBtn.change(function () {
        var file = uploadBtn[0].files[0]
        var formData = new FormData();
        formData.append('file',file);
        xfzajax.post({
            "url":'/product/upload_file/',
            'data':formData,
            'processData':false, 
            'contentType': false,
            'success':function (result) {
                if (result['code'] === 200){
                    console.log(result['data']);
                    var url = result['data'].url;
                    img.attr('src',url);
                    var label = $(".custom-file-label")
                    label.text(url)
                }
            }
        })
    })
};


// Product.prototype.listenSaveProductEvent = function(){
//     var saveBtn = $("#save-btn")
//     saveBtn.click(function () {
//         xfzalert.alertOneInput({
//             'title':'添加分类',
//             'placeholder':'输入新闻分类'
//         })
//     })
//
// };




$(function () {
    var product = new Product()
    product.run()
})


```



