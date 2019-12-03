---
layout: post
title: 如何发送ajax请求及toast弱提示
categories: [前端]
tags: [html]
date: 2019-12-03 15:48:07
comments: true
---


#### 步骤

```
ajax请求不需要写表单提交，也就是说input不用写在form表单里面，不需要被form包裹，但是input必须要有一个name名称，这样在js中才能找到该元素的值。

具体原理是利用jquery在网页加载完成后，利用$找到input的值，然后利用ajax提交数据。以登录功能为例，以下是具体步骤：


1. 在网页中导入文件
2. 在django视图函数中完成登录的视图函数
3. 利用ajax发送post请求
4. 导入message.js文件，在js文件中引用
```

#### 导入文件


```
<body class="hold-transition login-page">
<div class="login-box">
  <div class="login-logo">
    <a href="#"></a>
      酷客管理系统
  </div>
  <!-- /.login-logo -->
  <div class="card">
    <div class="card-body login-card-body">
      <p class="login-box-msg">登录</p>

{#      <form action="" class="signin-group"  >#}
          {% csrf_token %}
        <div class="input-group mb-3">
          <input type="text" class="form-control" name="telephone" placeholder="手机号">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-mobile"></span>
            </div>
          </div>
        </div>
        <div class="input-group mb-3">
          <input type="password" class="form-control" name="password" placeholder="密码">
          <div class="input-group-append">
            <div class="input-group-text">
              <span class="fas fa-lock"></span>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-8">
            <div class="icheck-primary">
              <input type="checkbox" id="remember" name="remember" value="1">
              <label for="remember">
                记住我
              </label>
            </div>
          </div>
          <!-- /.col -->

          <div class="col-4">
            <button  id="submit-btn" class="btn btn-primary btn-block"> 登录 </button>
          </div>
          <!-- /.col -->
        </div>
{#      </form>#}
{#      <p class="mb-1">#}
{#        <a href="forgot-password.html">忘记密码</a>#}
{#      </p>#}
    </div>
    <!-- /.login-card-body -->
  </div>
</div>
<!-- /.login-box -->

<!-- jQuery -->
<script src="{% static 'adminlte/plugins/jquery/jquery.min.js' %}"></script>
<!-- Bootstrap 4 -->
<script src="{% static 'adminlte/plugins/bootstrap/js/bootstrap.bundle.min.js' %}"></script>
<!-- AdminLTE App -->
<script src="{% static 'adminlte/dist/js/adminlte.min.js' %}"></script>
<script src="{% static 'js/login.min.js' %}"></script> //js文件
<script src="{% static 'js/jquery-3.3.1.min.js' %}"></script> //jquery文件
<script src="{% static 'js/xfzajax.min.js' %}"></script> //ajax文件
<script src="{% static 'js/message.min.js' %}"></script> // toast弱提示文件

</body>

```

注意：jqury文件和ajax需要在message文件前加载，所以放到前面。


#### Django完成post视图函数


注意：ajax发送的是post请求，所以在视图函数中用post视图函数去判断。


```python
from django.shortcuts import render, redirect, reverse
from django.contrib.auth import logout, login, authenticate
from .forms import LoginForm
from utils import restful


def index(request):
    return render(request, 'index/index.html')


def login_view(request):
    """GET请求返回登录模版"""
    if request.method == 'GET':
        return render(request, 'index/login.html')
    if request.method == 'POST': //这里定义的接受post请求的视图函数
        form = LoginForm(request.POST)
        if form.is_valid():
            telephone = form.cleaned_data.get('telephone')
            password = form.cleaned_data.get('password')
            remember = form.cleaned_data.get('remember')
            user = authenticate(request, username=telephone, password=password)
            if user:
                if user.is_active:
                    login(request, user)
                    if remember:
                        request.session.set_expiry(None)
                        # return redirect(reverse('index:index'))
                        return restful.ok()
                    else:
                        request.session.set_expiry(0)
                        # next_url = request.GET.get('next')
                        # if next_url:
                        #     return redirect(next_url)
                        # else:
                        #     return redirect(reverse('index:index'))
                        return restful.ok()
                else:
                    return restful.unauth(message='账号已经被冻结')

            else:
                return restful.params_error(message='手机号或密码错误')
        else:
            errors=form.get_errors()
            return restful.params_error(message=errors)


def logout_view(request):
    logout(request)
    return redirect(reverse('index:login'))

```


#### 利用ajax发送post请求给视图函数



```js
function Banner() {

};

Banner.prototype.run = function(){
    //id选择器在jquery中用#号来选择 $("#submit-btn")
    //calss选择器用  $("input[name='telephone']")
    var self = this;
    var telephoneInput = $("input[name='telephone']");
    var passwordInput = $("input[name='password']");
    var rememberInput = $("input[name='remember']");

    var submitBtn = $("#submit-btn");
    submitBtn.click(function () { //点击按钮式触发事件
        var telephone = telephoneInput.val(); //获取input中的值
        var password = passwordInput.val();
        var remember = rememberInput.prop("checked");
        console.log(telephone,password,remember)
        xfzajax.post({ //利用ajax发送请求post请求
            'url': '/login/', //请求的视图函数，在url.py里面定义的视图函数。要写在/../之间
            'data': {
                'telephone': telephone,
                'password': password,
                'remember': remember?1:0
            },
            'success': function (result) {
                if(result['code'] == 200){
                    // window.location.reload();
                    window.location.replace('/') //登录成功后跳转主页
                }else{
                    var messageObject = result['message'];
                    if(typeof messageObject == 'string' || messageObject.constructor == String){
                        window.messageBox.showError(messageObject);//表单验证错误toast弱提示
                    }else{
                        // {"password":['密码最大长度不能超过20为！','xxx'],"telephone":['xx','x']}
                        for(var key in messageObject){
                            var messages = messageObject[key];
                            var message = messages[0];
                            window.messageBox.showError(message);//表单验证错误toast弱提示
                        }
                    }
                }
                console.log(result)
            },
            'fail': function (error) {
                console.log(error);
            }
        });
    });
};


//$符号里面所有内容会在页面的所有元素都加载完成后执行
$(function () {
    var banner = new Banner();
    banner.run()
});
```


#### message.js文件及分析


以下message.js文件的toast弱提示分三种,success,error,info。我们可以利用css样式对其进行修改成我们想要的样式更符合自己的网页ui。

修改完成后，导入到html即可使用。


```js
// 错误消息提示框

function Message() {
    var self = this;
    self.isAppended = false;
    self.wrapperHeight = 48;
    self.wrapperWidth = 300;
    self.initStyle();
    self.initElement();
    self.listenCloseEvent();
}

Message.prototype.initStyle = function () {
    var self = this;
    self.errorStyle = {
        'wrapper':{
            'background': '#f2dede', //可以对error的背景颜色进行修改如：#ff0000
            'color': '#993d3d' //可以对toast文字颜色进行修改如:#fff
        },
        'close':{
            'color': '#993d3d' //可以对“x”关闭按钮颜色进行修改，改成和背景颜色一样则会隐藏如:#ff0000
        }
    };
    self.successStyle = {
        'wrapper':{
            'background': '#dff0d8',//可以对success的背景颜色进行修改如：#036bff
            'color': '#468847'//可以对toast文字颜色进行修改如:#fff
        },
        'close': {
            'color': "#468847"//可以对“x”关闭按钮颜色进行修改，改成和背景颜色一样则会隐藏如:#036bff
        }
    };
    self.infoStyle = {
        'wrapper': {
            'background': '#d9edf7',//同理
            'color': '#5bc0de'//同理
        },
        'close': {
            'color': '#5bc0de'//同理
        }
    }
};

Message.prototype.initElement = function () {
    var self = this;
    self.wrapper = $("<div></div>");
    self.wrapper.css({
        'padding': '10px',
        'font-size': '14px',
        'width': '300px',//可以增加toast的宽度如：'width': '500px',增加宽度后需修改margin-left的值，不然不会显示在顶部正中间位置
        'position': 'fixed',
        'z-index': '10000',
        'left': '50%',
        'top': '-48px',
        'margin-left':'-150px',//宽若为500，则margin-left为其一半-250px
        'height': '48px',
        'box-sizing': 'border-box',
        'border': '1px solid #ddd',//边框可以改为0
        'border-radius': '4px',//圆角可以改为0
        //'text-align':'center', //可以让提示的文字居中显示
        'line-height': '24px',
        'font-weight': 700
    });
    self.closeBtn = $("<span>×</span>");
    self.closeBtn.css({
        'font-family': 'core_sans_n45_regular,"Avenir Next","Helvetica Neue",Helvetica,Arial,"PingFang SC","Source Han Sans SC","Hiragino Sans GB","Microsoft YaHei","WenQuanYi MicroHei",sans-serif;',
        'font-weight': '700',
        'float': 'right',
        'cursor': 'pointer',
        'font-size': '22px'
    });

    self.messageSpan = $("<span class='xfz-message-group'></span>");

    self.wrapper.append(self.messageSpan);
    self.wrapper.append(self.closeBtn);
};

Message.prototype.listenCloseEvent = function () {
    var self = this;
    self.closeBtn.click(function () {
        self.wrapper.animate({"top":-self.wrapperHeight});
    });
};

Message.prototype.showError = function (message) {
    this.show(message,'error');
};

Message.prototype.showSuccess = function (message) {
    this.show(message,'success');
};

Message.prototype.showInfo = function (message) {
    this.show(message,'info');
};

Message.prototype.show = function (message,type) {
    var self = this;
    if(!self.isAppended){
        $(document.body).append(self.wrapper);
        self.isAppended = true;
    }
    self.messageSpan.text(message);
    if(type === 'error') {
        self.wrapper.css(self.errorStyle['wrapper']);
        self.closeBtn.css(self.errorStyle['close']);
    }else if(type === 'info'){
        self.wrapper.css(self.infoStyle['wrapper']);
        self.closeBtn.css(self.infoStyle['close']);
    }else{
        self.wrapper.css(self.successStyle['wrapper']);
        self.closeBtn.css(self.successStyle['close']);
    }
    self.wrapper.animate({"top":0},function () {
        setTimeout(function () {
            self.wrapper.animate({"top":-self.wrapperHeight});
        },3000);
    });
};

window.messageBox = new Message();
```



