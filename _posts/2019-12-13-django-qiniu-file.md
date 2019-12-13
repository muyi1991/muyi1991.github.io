---
layout: post
title: 上传文件到七牛云
categories: [前端]
tags: [html,小程序]
date: 2019-12-13 20:48:07
comments: true
---


#### 步骤

流程：后端给前端一个token，前端拿到token通过ajax向七牛云上传图片。

七牛创建空间，安装sdk
后端完成token视图
前端请求获取token并上传至七牛云
上传进度提示


#### 安装sdk

在华东区域创建存储空间xfzspace1,在个人中心拿到access_key和secret_key。

进入项目所在的虚拟环境，workon django-env
安装七牛的sdk pip install qiniu


#### 完成token请求视图


```
import qiniu

@method_decorator(xfz_login_required,name='dispatch')
class QnToken(View):
    def get(self,request,*args,**kwargs):
        access_key = 'WX2GvY4K3gpS_FrDEiZa9lq1aTqxeofaw5iM4ZMm'
        secret_key = 'POeH40wL2lTwlQIiBxgi_AerrzbODedvb3kRCbSj'
        bucket = 'xfzspace1'
        q = qiniu.Auth(access_key,secret_key)
        token = q.upload_token(bucket)
        # print(token)
        return restful.result(data={'token':token})
```

完成映射


```
from django.urls import path
from . import views

app_name = 'product'

urlpatterns = [
    path('product_list/', views.ProductList.as_view(), name='product_list'),
    path('add_product/', views.AddProduct.as_view(), name='add_product'),
    path('product_category/', views.ProductCategory.as_view(), name='product_category'),
    path('edit_category/', views.EditCategory.as_view(), name='edit_category'),
    path('delete_category/', views.DeleteCategory.as_view(), name='delete_category'),
    path('upload_file/', views.UploadFile.as_view(), name='upload_file'),
    path('qntoken/', views.QnToken.as_view(), name='qntoken'),
]
```


完成前端html，js部分代码
需要先导入七牛云的js代码


```

<script src="https://unpkg.com/qiniu-js@2.4.0/dist/qiniu.min.js
" ></script>

<div class="form-group">
{#                <img class="img-circle elevation-2" src="{{ url }}" id="img" alt="">#}
                <label for="thumbnail-form">缩略图</label>
                <div class="input-group">
                    <input type="file" class="custom-file-input" id="thumbnail-form" value="">
                    <label for="thumbnail-form" class="custom-file-label"></label>
                </div>
            </div>
            <div class="form-group progress-group" style="display: none" id="progress-group">
                  上传进度
                  <span class="float-right">0</span>
                  <div class="progress progress-sm">
                    <div class="progress-bar bg-primary" style="width: 0%"></div>
                  </div>
            </div>
```

在js中先请求后台获得token，然后利用获取的token向七牛云上传图片。


```
function Product() {

};

Product.prototype.run=function () {
    var self = this;
    // this.listenSaveProductEvent();
    // self.ListenUploadEvent();
    self.ListenUploadQiniuFileEvent();
};

Product.prototype.ListenUploadQiniuFileEvent = function(){
    var self = this;
    var uploadBtn = $("#thumbnail-form");
    uploadBtn.change(function () {
        var file = this.files[0];
        xfzajax.get({
            'url':'/product/qntoken/',
            'success':function (result) {
                if(result['code'] === 200){
                    var token = result['data']['token']//拿到token
                    var key = (new Date()).getTime() + '.' + file.name.split('.')[1];//给文件加上时间戳
                    var putExtra = {//额外参数
                        fname : key,
                        params :{},
                        mimeType:['image/png','image/jpeg','image/jpg']
                    };
                    var config = {//其他参数
                        userCdnDomain :true,
                        retryCount :6,
                        region:qiniu.region.z0//上传空间对应的参数z0代表华东区域
                    };
                    var observable = qiniu.upload(file,key,token,putExtra,config);
                    observable.subscribe({//获取返回对象
                        'next': self.handleFileUploadProgress,//上传进度
                        'error': self.handleFileUploadError,//上传错误提示
                        'complete': self.handleFileUploadComplete,//上传完成执行
                    })
                }
            }
        })
    })
};

Product.prototype.handleFileUploadProgress = function(response){
    var total = response.total;
    var percent = total.percent;
    var percentText = percent.toFixed(0) +'%';
    var progressGroup = Product.progressGroup;
    progressGroup.show();
    var progressBar = $(".progress-bar");
    progressBar.css({'width':percentText});
    var span = progressGroup.children('span')
    span.text(percentText)
};

Product.prototype.handleFileUploadError = function(error){
    window.messageBox.showError(error.messages)
    var progressGroup = Product.progressGroup;
    progressGroup.hide();
};

Product.prototype.handleFileUploadComplete = function(response){
    var progressGroup = Product.progressGroup;
    progressGroup.hide();
    // window.messageBox.showSuccess('上传完成');
    var domain = 'http://q2fz60woz.bkt.clouddn.com';
    var filename = response.key;
    var url = domain + filename;
    var label =$(".custom-file-label");
    label.text(url)
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
    var product = new Product();
    product.run();
    Product.progressGroup = $(".progress-group");
});


```







