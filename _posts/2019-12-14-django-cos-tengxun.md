---
layout: post
title: 上传文件到腾讯云的cos对象存储
categories: [前端]
tags: [html,小程序]
date: 2019-12-14 13:48:07
comments: true
---


#### 步骤

前端利用ajax向后端发送请求获取授权，然后再利用腾讯的sdk向指定的存储桶存放数据。

1. 后端：安装sdk，完成token的请求视图
2. 配置cors规则，一定要配置，不然请求失败
2. 前端：完成前端html的布局
3. js：安装sdk，向后端发送请求授权，授权后存储数据


#### 完成后端请求视图

安装sdk
参考链接：https://github.com/tencentyun/qcloud-cos-sts-sdk/tree/master/python

进入项目所在虚拟环境运行一下命令：
```
pip install -U qcloud-python-sts
```

完成获取token请求视图

```
from django.shortcuts import render
from sts.sts import Sts//导入sdk
import json
from utils import restful//这里使用到了restfulapi
from django.views.decorators.http import require_GET//获取授权走的是get请求


@require_GET
def get_token(request):
//request请求会自带bucket和region，这两个参数也可以由前端传过来
    config = {
        # 临时密钥有效时长，单位是秒
        'duration_seconds': 1800,
        # 固定密钥
        'secret_id': 'AKIDd362UmEoUxFTmdPdvvkMjspbL7mGalma',//自己账户的id
        # 固定密钥
        'secret_key': 'oNOHyRDrLdheprflV5Ut3XzMArgJiqoE',//自己账户的key
        # 是否需要设置代理
        'proxy': {
            # 'http': 'http://127.0.0.1:9000/'//好像可以不填
            # 'https': 'XXX'
        },
        # 换成你的 bucket
        'bucket': 'couuer-1258368129',//换成自己的存储桶名称，可以前端传过来
        # 换成 bucket 所在地区
        'region': 'ap-beijing',//存储桶所在区域，可以前端传过来
        # 这里改成允许的路径前缀，可以根据自己网站的用户登录态判断允许上传的具体路径
        # 例子： a.jpg 或者 a/* 或者 * (使用通配符*存在重大安全风险, 请谨慎评估使用)
        'allow_prefix': '*',//可以填*
        # 密钥的权限列表。简单上传和分片需要以下的权限，其他权限列表请看 https://cloud.tencent.com/document/product/436/31923
        'allow_actions': [//授权哪些权限给前端
            # 简单上传
            'name/cos:PutObject',
            # 表单上传
            'name/cos:PostObject',
            # 分片上传： 初始化分片
            'name/cos:InitiateMultipartUpload',
            # 分片上传： 查询 bucket 中未完成分片上传的UploadId
            "name/cos:ListMultipartUploads",
            # 分片上传： 查询已上传的分片
            "name/cos:ListParts",
            # 分片上传： 上传分片块
            "name/cos:UploadPart",
            # 分片上传： 完成分片上传
            "name/cos:CompleteMultipartUpload"
        ]

    }

    sts = Sts(config)
    response = sts.get_credential()
    print('get data : ' + json.dumps(dict(response), indent=4))
    data = (json.dumps(dict(response), indent=4))//拿到json对象
    dict11 = json.loads(s=data)//把json变成字典
    return restful.result(data=dict11)//利用restfulapi把字段传给前端，这一步必做，不然前端拿不到数据


def index(request):
    return render(request,'index.html')
```

完成路由映射

```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index,name='index'),
    path('get_token/', views.get_token,name='get_token'),
]
```

向/get_token/发送get请求就可以拿到授权


#### 完成html布局


```
<head>
    <link rel="stylesheet" href="{% 'plugins/fontawesome-free/css/all.min.css' %}">
    <link rel="stylesheet" href="{% 'dist/css/adminlte.min.css' %}">
    <script src="{% 'js/jquery-3.3.1.min.js' %}" ></script>//jquery必须导入
    <script src="{% 'js/xfzajax.min.js' %}" ></script>//ajax必须导入
    <script src="{% 'js/cos-js-sdk-v5.min.js' %}"></script>//sdk
    <script src="{% 'js/index_main.js' %}" ></script>//我们自己的js代码
</head>
<body>
<input type="file" id="file-selector" >
<label for="file-selector" class="btn btn-outline-primary" id="btn">上传图片</label>
</body>
```

##### 发送请求获取授权，上传文件

首现需要到存储桶去配置cors，必须配置，这个如果不配置是不能上传成功的。

参考链接：https://cloud.tencent.com/document/product/436/11459

在"来源Origin"里配置：

```
http://a.qcloud.com
https://b.qcloud.com
http://127.0.0.1:9000//自己的域名请求地址，域名后面不加“/”
```

下载sdk，在html文件中引入该文件。
下载地址：https://github.com/tencentyun/cos-js-sdk-v5
在该文件dist目录下面，下载cos-js-sdk-v5.min.js，该js文件，并在html中引入。


发送js请求。


```

var Bucket = 'couuer-1258368129';
var Region = 'ap-beijing';

// 初始化实例//这种方式不推荐，会直接暴露自己的私钥在html文件中，不安全
// var cos = new COS({
//     SecretId: 'AKIDd362UmEoUxFTmdPdvvkMjspbL7mGalma',
//     SecretKey: 'oNOHyRDrLdheprflV5Ut3XzMArgJiqoE',
// });

$(function () {

var cos = new COS({
    getAuthorization: function (options, callback) {
        // 异步获取临时密钥

        $.get('/get_token/', {//向get_token发送请求获取授权
            bucket: options.Bucket,//传递bucket后端可以用起来
            region: options.Region,//传递region后端可以用起来
        }, function (result) {
            var credentials = result.data.credentials;
            callback({
                 TmpSecretId: credentials.tmpSecretId,
                 TmpSecretKey: credentials.tmpSecretKey,
                 XCosSecurityToken: credentials.sessionToken,
                 ExpiredTime: result.data.expiredTime
            });
        });
    }
});

// 监听选文件
document.getElementById('file-selector').onchange = function () {
    var file = this.files[0];//拿到文件的第一个数据
    var key = (new Date()).getTime() + '.' + file.name.split('.')[1];//重命名时间戳
    if (!file) return;
    cos.putObject({//调用cos上传数据
        Bucket: Bucket,
        Region: Region,
        Key: key,
        Body: file,
    }, function (err, data) {
        console.log(err);//上传失败打印错误信息
        if (data.statusCode === 200){
            console.log(data.Location);
            var src = 'http://' + data.Location;
            var img = $('#img');
        }
    });
};
});

```


更详细教程参考官方文档。
ptython：https://github.com/tencentyun/qcloud-cos-sts-sdk/tree/master/python
js：https://github.com/tencentyun/cos-js-sdk-v5



