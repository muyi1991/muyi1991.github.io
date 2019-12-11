---
layout: post
title: django中的csrf的使用
categories: [前端]
tags: [django]
date: 2019-12-10 15:48:07
comments: true
---


#### 如何关闭csrf

关闭某个视图函数的csrf,可以给某个视图添加装饰器。

```
from django.views.decorators.csrf import csrf_exempt
from utils.decorators import xfz_login_required
from django.utils.decorators import method_decorator

@method_decorator([xfz_login_required,csrf_exempt], name='dispatch')
class AddOrder(View):
    def get(self,request,*args,**kwargs)
        return render(request,"index:index.html")
```

关闭所有的csrf

settings.py中关闭csrf的中间件关掉就可以了。
```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```



#### 使用csrf

普通的表单提交,在form第一行增加csrftoken就可以啦。

```
<form>
{% csrf_token %}
...
</form>
```

ajax提交：

使用封装好的就可以啦。


```
/**
 * Created by hynev on 2018/5/15.
 */

function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

var xfzajax = {
    'get': function (args) {
        args['method'] = 'get';
        this.ajax(args);
    },
    'post': function (args) {
        args['method'] = 'post';
        this._ajaxSetup();
        this.ajax(args);
    },
    'ajax': function (args) {
        var success = args['success'];
        args['success'] = function (result) {
            if(result['code'] === 200){
                if(success){
                    success(result);
                }
            }else{
                var messageObject = result['message'];
                if(typeof messageObject == 'string' || messageObject.constructor == String){
                    console.log(messageObject)
                    window.messageBox.showError(messageObject);
                }else{
                    // {"password":['密码最大长度不能超过20为！','xxx'],"telephone":['xx','x']}
                    for(var key in messageObject){
                        var messages = messageObject[key];
                        var message = messages[0];
                        console.log(message);
                        window.messageBox.showError(message);
                    }
                }
                if(success){
                    success(result);
                }
            }
        };
        args['fail'] = function (error) {
            console.log(error);
            window.messageBox.showError('服务器内部错误！');
        };
        $.ajax(args);
    },
    '_ajaxSetup': function () {
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (!/^(GET|HEAD|OPTIONS|TRACE)$/.test(settings.type) && !this.crossDomain) {
                    xhr.setRequestHeader("X-CSRFToken", getCookie('csrftoken'));
                }
            }
        });
    }
};

```

    
    

