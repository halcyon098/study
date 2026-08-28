---
title: "安服水洞系列|CORS跨域资源共享漏洞 - FreeBuf网络安全行业门户"
source: "https://www.freebuf.com/articles/web/362156.html"
author:
  - "[[进击的HACK]]"
  - "[[新华三攻防实验室]]"
  - "[[雷石安全实验室]]"
  - "[[助安社区]]"
published:
created: 2025-06-09
description: "安服水洞系列|CORS跨域资源共享漏洞"
tags:
  - "clippings"
---


## 前言

CORS跨域资源共享漏洞与JSONP劫持漏洞类似，都是程序员在解决跨域问题中进行了错误的配置。攻击者能够利用web应用未严格校验用户请求包的Origin头，诱骗受害者访问精心构造的恶意网站，窃取用户敏感信息，例如cookie等身份认证信息。

## CORS介绍

CORS（Cross-Origin Resource Sharing）是一种浏览器安全机制，它允许 Web 应用程序从不同源加载所需的 Web 资源。由于浏览器的同源策略限制，Web 应用程序只能访问与其自身源相同的资源，因此需要 CORS 来解决跨域资源访问的问题。

CORS 是通过 HTTP 头来进行控制的，主要涉及到以下几个头部：

- Access-Control-Allow-Origin：授权访问的源站
- Access-Control-Allow-Credentials：是否允许发送 Cookie 等凭据信息
- Access-Control-Allow-Methods：授权访问的 HTTP 方法
- Access-Control-Allow-Headers：授权访问的自定义头部

## CORS存在漏洞原因

因为同源策略的存在，不同源的客户端脚本不能访问目标站点的资源，如果目标站点配置不当，没有对请求源的域做严格限制，导致任意源都可以访问时，就存在cors跨域漏洞问题。

### 什么是同源策略？

同源策略用于限制应用程序之间的资源共享，确保一个应用里的资源只能被本应用的资源所访问。如果要跨域通信，就必须要引入跨域资源共享。如果没有正确配置，就会导致漏洞产生。

同源是指协议，域名，端口三个都相同，即使是同一个ip，不同域名也不是同源，同源策略里允许发出请求，但是不允许访问响应。

**同源示例**

设 http://test.com/a/123.html 源

| URL | 是否同源 | 不同处 |
| --- | --- | --- |
| http://test.com/a/asd.html | 同源 | 路径不同 |
| http://test.com/abcvb/123.html | 同源 | 路径不同 |
| http://test.com/abc/123458.html | 同源 | 路径不同 |
| https://test.com/a/1234.html | 不同源 | 协议不同 |
| http://test.com:88/a/1234.html | 不同源 | 端口不同 |
| http://test.com:82/a/1234.html | 不同源 | 端口不同 |
| http://bbb.test.com/a/1234.html | 不同源 | 域名不同，子域名不算同源 |

### CORS 存在漏洞主要有以下几种

1. 未正确设置 Access-Control-Allow-Origin

如果 Access-Control-Allow-Origin 头未正确设置，攻击者可以通过构造特定的请求，绕过同源策略，从而获取到需要访问的数据。攻击者可以将自己的网站伪装成合法的源站，然后在自己的网站中通过 XMLHttpRequest 发送跨域请求，获取到需要的数据。

1. Access-Control-Allow-Origin 设置为 \*

Access-Control-Allow-Origin 头部设置为 \* ，表示所有站点都可以访问该资源，这可能会导致安全问题。攻击者可以通过伪造 HTTP 请求头中 Origin 字段的值，绕过同源策略，获取到需要的数据。

1. 未正确设置 Access-Control-Allow-Credentials

Access-Control-Allow-Credentials 头部用于指示是否允许发送 Cookie 等凭据信息。如果未正确设置该头部，攻击者可以伪造 HTTP 请求头中的 Origin 和 Cookie 字段，从而绕过同源策略，获取到需要访问的数据。

1. 使用不安全的 HTTP 方法和头部

Access-Control-Allow-Methods 和 Access-Control-Allow-Headers 用于授权访问的 HTTP 方法和头部。如果未正确设置这两个头部，攻击者可以通过构造特定的请求，使用不安全的 HTTP 方法和头部，获取到需要访问的数据。

1. CSRF

由于 CORS 的存在，攻击者可以利用 CSRF 漏洞发起跨域请求，进而绕过同源策略，获取到需要的数据。在这种情况下，应该使用 CSRF Token 等措施来防止 CSRF 攻击。

## CORS跨域漏洞的危害

CORS跨域资源共享漏洞可以导致恶意网站或者应用程序通过跨域请求访问用户敏感信息、执行非授权操作等，对用户隐私和数据安全造成威胁。具体危害包括：

1. 窃取用户敏感信息：攻击者可以通过CORS跨域请求窃取用户敏感信息，例如cookie等身份认证信息，用于身份冒充或者进行其他攻击。
2. 执行未授权操作：攻击者可以利用CORS跨域请求执行未授权的操作，例如发起POST请求进行数据修改、发起DELETE请求进行数据删除等，从而对服务器和应用程序造成破坏。
3. 数据篡改：攻击者可以利用CORS跨域请求修改数据内容，例如通过跨域POST请求篡改数据库中的数据，从而对应用程序造成破坏。
4. 资源耗尽：攻击者可以通过CORS跨域请求耗尽服务器资源，例如发起大量跨域请求占用服务器带宽、内存等资源，导致服务不可用。

综上，CORS跨域资源共享漏洞的危害非常严重，需要及时修复和防范。

## CORS跨域漏洞检测

使用burpsuite抓包对http请求添加 **Origin: http://www.attacker.com** 进行测试：

- 如果返回头是以下情况，那么就是漏洞，这种情况下漏洞最好利用：

Access-Control-Allow-Origin: https://www.attacker.com

Access-Control-Allow-Credentials: true

- 如果返回头是以下情况，那么也可以认为是漏洞，只是利用起来麻烦一些：

Access-Control-Allow-Origin: null

Access-Control-Allow-Credentials: true

- 果返回以下，则不存在漏洞，因为Null必须是小写才存在漏洞：

Access-Control-Allow-Origin: Null

Access-Control-Allow-Credentials: true

- 如果返回以下，可认为不存在漏洞，因为CORS安全机制阻止了这种情况下的漏洞利用，也可以写上低危的CORS配置错误问题。

Access-Control-Allow-Origin: \*

Access-Control-Allow-Credentials: true

- 如果返回以下，可认为不存在漏洞，也可以写上低危的CORS配置错误问题。

Access-Control-Allow-Origin: \*

## CORS跨域漏洞复现

Java复现可以参考https://mp.weixin.qq.com/s/PSU8T-IO3mAz4MEVvAeUug中的part3

PHP的CORS跨域漏洞可以参考https://mp.weixin.qq.com/s/ViSR-l41Z9qsazxI2MAhTA的利用方式

- 其中用到dorabox靶场，搭建方式可以参考https://mp.weixin.qq.com/s/juseXiuKATSTRUwoMjLI-Q
- 文章还有CORS的进一步利用方式，用来扩大战果或者在挖SRC的时候用来提升漏洞等级。

## CORS跨域工具化检测

自动搜集CORS与JSONP请求的burp插件

https://github.com/p1g3/JSONP-Hunter

结果：  
![图片.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6addcbecd92702db37cdcfb0728fbaf0.jpg)  
扫描存在CORS跨域漏洞的网站 https://github.com/p1g3/CORS-SCAN

使用：python3 cors\_test.py url\_file.txt

xray也可以扫描这类漏洞，但是很少用xray去扫，因为很多cors请求读取的并不是敏感信息，这类漏洞交到SRC大概率是要被忽略的，我们需要人工判断响应数据中的数据是否足够敏感，xray默认配置即可扫描CORS  
![图片.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/437ee76593ef164066c7fa58843bdd36.jpg)  
发现存在错误配置的CORS请求后，还需考虑请求是否有refer校验，因为跨域漏洞的触发和csrf是类似的，需要通过交互让受害者访问我们的恶意脚本，如果因为refer校验不通过还需要进一步bypass。这也是大多数SRC让提供有效poc的原因。js代码编写经验较少的师傅可以用工具生成poc：

https://github.com/0verSp4ce/PoCBox

docker启动：

```
docker container run -d -p 8088:80 registry.cn-hangzhou.aliyuncs.com/pocbox/pocbox:1.0
```

本地搭建好后在图示处填入相关数据后即可生成poc code，可以选择online test，也可以将poc code保存为html后本地访问，验证是否可以读取响应数据：

![图片.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d1872336ea4ed03fdf9d4ee6dbed4ee3.jpg)

## 参考链接

https://mp.weixin.qq.com/s/PSU8T-IO3mAz4MEVvAeUug

https://mp.weixin.qq.com/s/ViSR-l41Z9qsazxI2MAhTA

https://mp.weixin.qq.com/s/21jwvrWhbkVdE5EUovvNZQ

https://mp.weixin.qq.com/s/u-Eysz8vn9UszdnJuCowPA

chatGPT

\# 漏洞 \# 渗透测试 \# web安全 \# 网络安全技术 \# SRC漏洞挖掘

本文为 进击的HACK 独立观点，未经授权禁止转载。  
如需授权、对文章有疑问或需删除稿件，请联系 FreeBuf 客服小蜜蜂（微信：freebee1024）

被以下专辑收录，发现更多精彩内容

相关推荐

[![安服水洞系列|Vue源码泄露](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/807b865c622afa68c1d3aa6cb24db65b.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/807b865c622afa68c1d3aa6cb24db65b.jpg)

[Web安全](https://www.freebuf.com/articles/web/362520.html)

[![攻防潜力股：ChatGPT的实习报告](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a8cd15be1d0b96d3f0079bea5245e4c7.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a8cd15be1d0b96d3f0079bea5245e4c7.png)

[资讯](https://www.freebuf.com/news/362480.html)

[![威胁情报-telegram消息监控](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1535ecf2577ad75af73faecfb4494a61.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1535ecf2577ad75af73faecfb4494a61.png)

[基础安全](https://www.freebuf.com/articles/network/362429.html)

[![记一次绕过安全设备Getshell的实战经历](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7b5606d178df0a2d32375a05f54c779c.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7b5606d178df0a2d32375a05f54c779c.png)

[Web安全](https://www.freebuf.com/articles/web/362390.html)

[![CouchDB 漏洞复现](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/904887680cc6153466701e4d842a1775.png)

Web安全

](https://www.freebuf.com/articles/web/362382.html)

[前言CouchDB 是一个开源的面向文档的数据库管理系统，可以通过 RESTful JavaScript Object Notation...](https://www.freebuf.com/articles/web/362382.html)

[142571 围观](https://www.freebuf.com/articles/web/362382.html) 2023-04-03

进击的HACK LV.6

欢迎关注公众号：进击的HACK 坚持每日更新，一起变得更强。

- 37 文章数
- 50 关注者

流程审批模块常见漏洞挖掘点

nuclei介绍与持续更新的poc集合

钓鱼前邮件收集

文章目录