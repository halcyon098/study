# XSS（跨站脚本攻击）

XSS主要基于JavaScript语言完成恶意的攻击行为，因为JavaScript可以非常灵活的操作html、CSS和浏览器

常用标签：

https://www.freebuf.com/articles/web/340080.html

https://xz.aliyun.com/t/4067

## **原理**

XSS就是通过利用网页开发时留下的漏洞（由于Web应用程序对用户的输入过滤不足），巧妙的将恶意代码注入到网页中，使用户浏览器加载并执行攻击者制造的恶意代码，以达到攻击的效果。

会受到不同浏览器的安全策略从而影响JS代码的执行

这些恶意代码通常是JavaScript，但实际上也可以包括Java、VBScript、ActiveX、Flash或者普通的HTML

简单来说，XSS就是通过攻击者精心构造的JS代码注入到网页中，并由浏览器解释，运行这段JS代码，以达到恶意攻击浏览器的效果。XSS攻击的对象是用户浏览器，属于
被动攻击。因此XSS攻击涉及到三个角色：

- 攻击者
- 用户浏览器
- 服务器

可能受到XSS攻击的位置：只要对用户的输入没有进行严格的过滤，就有可能遭到XSS攻击

微博、留言板、聊天室等收集用户输入信息的地方，都有可能遭到XSS攻击

## XSS的危害

​		针对用户

- 窃取cookie劫持的会话

- 网络钓鱼

- 放马挖矿

- 广告刷流量

  

  针对web服务

- 劫持后台（常见）

- 篡改页面

- 传播蠕虫

- 内网扫描（常见）

## 攻击利用

(js代码能做什么，XSS就能做什么攻击)：

```bash
盲打(不知道哪些地方存在XSS，直接能插XSS的地方都插入试试)，COOKIE盗取，凭据窃取，页面劫持，网络钓鱼，权限维持等

```

网络钓鱼：

通过精心构造与目标网站一样的页面，极度相似的域名，诱骗用户输入账号密码后，获取凭证和信息。有些获取后会跳转到正常网站，使受害者不知道自己已经被钓鱼。

测试流程：看输出想输入在哪里，更改输入代码看执行（标签，过滤决定）

### XSS 可以插在哪里？

- 用户输入作为 script 标签内容
- 用户输入作为 HTML 注释内容
- 用户输入作为 HTML 标签的属性名
- 用户输入作为 HTML 标签的属性值
- 用户输入作为 HTML 标签的名字
- 直接插入到 CSS 里

### 数据交互的地方

```txt
get、post、headers
反馈与浏览
富文本编辑器
各类标签插入和自定义
```

### 数据输出的地方

```txt
用户资料
数据输出
评论，留言等
关键词、标签、说明
文件上传
```

## 反射型

非持久化(一次性)，需要欺骗用户自己去点击链接才能触发 XSS 代码（服务器中没有这样的页面和内容），一般容易出现在搜索页面。反射型 XSS 大多数是用来盗取用户的 Cookie 信息。

因为原理决定反射xss需要用户主动或被动的配合才能完成。一般极难单独造成较大危害，实战中大部分反射型xss厂商是不收的，收的也多数为低危。

常见情况是攻击者通过构造一个恶意链接的形式，诱导用户传播和打开,由于链接内所携带的参数会回显于页面中或作为页面的处理数据源，最终造成[XSS攻击](https://so.csdn.net/so/search?q=XSS攻击&spm=1001.2101.3001.7020)。

因此反射型一般会出现在url参数的位置。



![](https://cdn.jsdelivr.net/gh/Dmup227/Dimags@main/D:%5CDocument%5Cimages202403191410664.png)

## 存储型 XSS

存储型 XSS，持久化，代码是存储在服务器中的，如在个人信息或发表文章等地方，插入代码，如果没有过滤或过滤不严，那么这些代码将储存到服务器中，用户访问该页面的时候触发代码执行。这种 XSS 比较危险，容易造成蠕虫，盗窃 cookie 。

存储型XSS是持久化的XSS攻击方式，将恶意代码存储于服务器端，

当其他用户再次访问页面时触发，造成XSS攻击。

一般会出现在用户可以控制输入，且输入内容会存储进数据库被调用的情况里。

![](https://cdn.jsdelivr.net/gh/Dmup227/Dimags@main/D:%5CDocument%5Cimages202403191411037.png)

## DOM型XSS

 漏洞是基于文档对象模型(Document Objeet Model,DOM)的一种漏洞，DOM-XSS 是通过 url 传入参数去控制触发的，其实也属于反射型 XSS。 DOM 的详解：DOM 文档对象模型

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/f192096892e127f1c9edf7eee27e9b6d_MD5.png)

DOM漏洞存在JS代码（由**JS代码直接处理，不需要经过服务端处理**），其他类型漏洞存在脚本代码（**反射，存储，都需要经过服务端处理**）

例如：我们在 URL 中传入参数的值，然后客户端页面通过 js 脚本利用 DOM 的方法获得 URL 中参数的值，再通过 DOM 方法赋值给选择列表，该过程没有经过后端，完全是在前端完成的。所以，我们就可以在我们输入的参数上做手脚了。

如果要挖DOM型漏洞，一定要查看网页源代码并在HTML代码中找到相关js代码

可能触发 DOM 型 XSS 的属性

> document.referer
>
> window.name
>
> location
>
> innerHTML
>
> documen.write



## 其他类型的xss

mxxs-浏览器版本造成的漏洞，出现较少，几年可能会爆出来一个

xss flash视频文件swf，引用js文件造成（没啥用了）

pdfxss 在pdf中插入js代码，在浏览器中打开触发代码，一般配合钓鱼使用

别人总结好的博客：

https://www.fooying.com/the-art-of-xss-1-introduction/#mxss

## 安全防御

有翻译过OWASP Xenotix XSS 漏洞利用框架作者Ajin Abranham写的一个[《给开发者的终极XSS防御备忘录》](https://github.com/fooying/Papers/blob/master/给开发者的终极XSS防护备忘录.pdf)

### CSP

> CSP (Content Security Policy 内容安全策略)  各种语言都存在，只不过设置不同
>
> 内容安全策略是一种可信白名单机制，来限制网站中是否可以包含某来源内容。
>
> 该制度明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单(当外
>
> 部资源不在白名单内，
>
> 禁止网站访问外部资源)，它的实现和执行全部由浏览器完成，开发者只需提供配置。
>
> 禁止加载外域代码，防止复杂的攻击逻辑。
>
> 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
>
> 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
>
> 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
>
> 合理使用上报可以及时发现XSS，利于尽快修复问题。
> 

php开启语句:

> header("Content-Security-Policy:img-src 'self'");

**绕过：有但鸡肋**
https://xz.aliyun.com/t/12370
https://blog.csdn.net/a1766855068/article/details/89370320

### HttpOnly

> 禁止页面的JavaScript访问带有HttpOnly属性的Cookie。
> php和java都有该设置
>
> PHP.INI设置或代码引用，三种方式设置：
> -session.cookie_httponly =1
> -ini_set("session.cookie_httponly", 1);
> -setcookie('', '', time() + 3600, '/xss', '', false, true);

### XSSFilter(过滤器的意思)

检查用户输入的数据中是否包含特殊字符， 如<、>、’、”,进行实体化等。

#### 1、无任何过滤

```js
<script>alert()</script>
```

#### 2.实体化 输入框没有

```js
">  <script>alert()</script>  <" 
```

#### 3.全部实体化 利用标签事件 单引号闭合

```js
' οnfοcus=javascript:alert() '
```

#### 4.全部实体化 利用标签事件 双引号闭合

```bash
" οnfοcus=javascript:alert() "
```

#### 5.事件关键字过滤 利用其他标签调用 双引号闭合

```js
"> <a href=javascript:alert()>xxx</a> <"
```

#### 6、利用大小写未正则匹配

```bash
"> <sCript>alert()</sCript> <"
```

#### 7、利用双写绕过匹配

```bash
"> <a hrehreff=javasscriptcript:alert()>x</a> <"
```

#### 8、利用Unicode编码

```js
&#x006a&#x0061&#x0076&#x0061&#x0073&#x0063&#x0072&#x0069&#x0070&#x0074&#x003a&#x0061&#x006c&#x0065&#x0072&#x0074&#x0028&#x0029
```

#### 9、利用Unicode编码（内容检测） 

```js
&#x006a&#x0061&#x0076&#x0061&#x0073&#x0063&#x0072&#x0069&#x0070&#x0074&#x003a&#x0061&#x006c&#x0065&#x0072&#x0074&#x0028&#x0029;('http://')
```

