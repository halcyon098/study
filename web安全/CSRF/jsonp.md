# jsonp攻击

https://xz.aliyun.com/t/10051?u_atoken=b0ac9df4bd46da02383f24b399a17189&u_asig=0a472f9217364044980427338e003f

https://blog.csdn.net/sunshine19871211/article/details/105507219

https://www.freebuf.com/vuls/403125.html

https://cn-sec.com/archives/2242977.html

## jsonp介绍

JSONP 是 JSON with padding（填充式 JSON 或参数式 JSON）的简写。
 JSONP实现跨域请求的原理简单的说，就是动态创建`<script>`标签，然后利用`<script>`的src 不受同源策略约束来跨域获取数据。

JSONP 由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数。回调函数的名字一般是在请求中指定的。而数据就是传入回调函数中的 JSON 数据。

动态创建`<script>`标签，设置其src，回调函数在src中设置：

```js
var script = document.createElement("script");
script.src = "https://api.douban.com/v2/book/search?q=javascript&count=1&callback=handleResponse";
document.body.insertBefore(script, document.body.firstChild);
```

在页面中，返回的JSON作为response参数传入回调函数中，我们通过回调函数来来操作数据。

```js
function handleResponse(response){
    // 对response数据进行操作代码
}
```



Jsonp(JSON with Padding) 是 json 的一种"使用模式"，可以让网页从别的域名（网站）那获取资料，即跨域读取数据。

JSONP的语法和JSON很像，简单来说就是在JSON外部用一个函数包裹着。JSONP基本语法如下：

```
callback({ "name": "kwan" , "msg": "获取成功" });
```

JSONP原理就是动态插入带有跨域url的`<script>`标签，然后调用回调函数，把我们需要的json数据作为参数传入，通过一些逻辑把数据显示在页面上。

常见的jsonp形式类似：

```
http://www.test.com/index.html?jsonpcallback=hehe
```

传过去的hehe就是函数名，服务端返回的是一个函数调用，可以理解为：evil就是一个函数，(["customername1","customername2"])就是函数参数，网站前端只需要再编写代码处理函数返回的值即可。

## 原理

浏览器的同源策略

SOP，全称为同源策略 (Same Origin  Policy)，该策略是浏览器的一个安全基石，如果没有同源策略，那么，你打开了一个合法网站，又打开了一个恶意网站。恶意网站的脚本能够随意的操作合法网站的任何可操作资源，没有任何限制。浏览器要严格隔离两个不同源的网站，目的是保证数据的完整性和机密性。

浏览器的同源策略规定：不同域的客户端脚本在没有明确授权的情况下，不能读写对方的资源。那么何为同源呢，即两个站点需要满足同协议，同域名，同端口这三个条件。
 “同源”的定义：
    域名
    协议
    tcp端口号
 只要以上三个值是相同的，我们就认为这两个资源是同源的。

首先来看看同源策略到底有什么作用：当浏览器发现有一个跨域的请求，但是它在服务器的返回头中如果没有发现 `Access-Control-Allow-Origin` 值允许 http://x.x.x.x 的访问，那么便会将其给拦截。
 那么虽然浏览器受到了同源策略的限制，不允许实现跨域访问，但是由于在开发过程中，其中的前后端的交互过程中不可避免会涉及到跨域的请求（设计同源策略的人想必也发现了这个问题），于是设计者给我们留了一个后门，就是只要服务器响应头中返回允许这个源的选项，那么跨域请求就会成功。（这里纠正一个误区，不要认为浏览器默认支持同源策略就意味着不同源的请求就不能发出去，其实还是能发出去的，只是要看响应头，我们都知道在页面中有几个东西是对同源策略免疫的，有 `<img>` 的src 、`<link>` 的 href 还有就是`<script>`的 src , 那么JSONP 就是利用其中的 `<script>` 标签的sec 属性实现跨区域请求的。
 `<script>`标签的请求不论是不是同源一律不受同源策略的限制，那我们就找到了解决跨域访问的方法。



所以整个的具体过程其实就是：

我们把回调函数给了服务器，服务器把json参数给了回来。

`jsonp劫持`就是攻击者获取了本应该传给网站其他接口的数据。通过JSONP技术可以实现数据的跨域访问，必然会产生安全问题，如果网站B对网站A的JSONP请求没有进行安全检查直接返回数据，则网站B 便存在JSONP 漏洞，网站A 利用JSONP漏洞能够获取用户在网站B上的数据。



## 漏洞利用过程

1）用户在网站B 注册并登录，网站B 包含了用户的id，name，email等信息；

 2）用户通过浏览器向网站A发出URL请求；

 3）网站A向用户返回响应页面，响应页面中注册了JavaScript的回调函数和向网站B请求的`<script>`标签，示例代码如下：>标签，示例代码如下：

```js
<script type="text/javascript">
function Callback(result)
{
    alert(result.name);
}
</script>
<script type="text/javascript" src="http://B.com/user?jsonp=Callback"></script>
```

4）用户收到响应，解析JS代码，将回调函数作为参数向网站B发出请求；

 5）网站B接收到请求后，解析请求的URL，以JSON 格式生成请求需要的数据，将封装的包含用户信息的JSON数据作为回调函数的参数返回给浏览器，网站B返回的数据实例如下：

```json
Callback({"id":1,"name":"test","email":"test@test.com"})
```

6）网站B数据返回后，浏览器则自动执行Callback函数对步骤4返回的JSON格式数据进行处理，通过alert弹窗展示了用户在网站B的注册信息。另外也可将JSON数据回传到网站A的服务器，这样网站A利用网站B的JSONP漏洞便获取到了用户在网站B注册的信息。

![20210813165430-12bfea54-fc14-1](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/20210813165430-12bfea54-fc14-1.png)

### 危害

```
1.攻击者利用存在漏洞的网站，将链接通过邮件等形式推送给受害者，如果受害者点击了链接，则攻击者便可以获取受害者的个人敏感的信息。所以JSONP劫持漏洞会泄露信息。

2.可能导致用户权限被盗用;
攻击者通过JSON劫持构造盗取管理员或高权限用户的脚本，一旦被访问，权限立即被盗用。

3. 可以通过劫持对网页进行挂马;
在JSON劫持点构造引向漏洞后门木马，但访问直接利用漏洞批量挂马。

4. 可对劫持页进行网站钓鱼;
利用JSON劫持直接导向伪装网站地址。

5. 可做提权攻击;

6. 变种拒绝服务攻击;
劫持后将流量导向受害网站，直接发动DDOS攻击。
```

### 修复

##### 限制referer：

```
if ($_SERVER['HTTP_REFERER']!=='http://www.xxx.com/1.html') {
    exit("非法访问");
}
```

##### 使用token

随机的生成一段token值，每次提交表单都要检查，攻击者没有token就不能访问。

### 绕过

针对上面两种修复方式 也都有对应的绕过方式

##### data URI 绕过 referer

`data URI`不会发送referer头，data还可以使用base64编码

##### https转到http referer

https转到http会返回一个空的referer (为了防止数据泄露)

##### 绕过token

这里有一个比较好的例子：`http://www.91ri.org/13407.html`

### JSON劫持可能存在的点

1.Referer过滤不严谨；

 2.空Referer(在通过跨协议调用JS时，发送的http请求里的Referer为空)；

 3.CSRF调用json文件方式不安全，token可重复利用；

 4.JSON输出的Content-Type及编码不符合标准(gb2312可能存在宽字节注入)；

 5.未严格过滤callback函数名及JSON里数据的输出；

 6.未严格限制JSONP输出callback函数名的长度。

### 需要满足的条件

1.使用JSONP获取数据；

2.未检测referer字段或者验证了 referer字段，但是验证方式不严谨，如需要验证的referer字段为 www.xxx.com 域，但是 www.xxx.com.mydomain.com 同样能够绕过；

3.GET请求中不包含token相关的参数

## 其他

这个师傅的复现过程比较好理解：https://cn-sec.com/archives/2242977.html

总体理解就是b网站的策略没有做好，导致其他网站例如a网站可以访问本应该是b网站中只允许在该域中传递json数据。我们利用js代码，完成让用户发送自己在b网站上的敏感信息传递给a网站。

1. 最新版的谷歌浏览器Chrome对于JONSP劫持攻击做了防范，这也是为啥很多JSONP劫持漏洞别人能复现成功，而有的人却始终复现不成功的原因。这标志着JSONP劫持和CORS跨域资源共享漏洞危害性会逐步降低。

2. 想要理解一些web漏洞原理，还是得自己搭建环境，自己写代码从头到尾梳理一遍，从根源上理解这个漏洞，踩过坑后才发现原来是这么回事。

