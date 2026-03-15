# Weblogic

## 未授权文件上传/弱口令

利用默认后台账号密码进入后台上传后门

## 未授权远程命令执行

CVE-2020-14882：允许未授权的用户绕过管理控制台的权限验证访问后台。

CVE-2020-14883：允许后台任意用户通过HTTP协议执行任意命令。

两个漏洞配合，目录穿越进入后台，http的get请求执行命令

## ssrf

CVE-2014-4210
Weblogic 的 uddi 组件实现包中有个 ddiexplorer.war文件，其下里 SearchPublicReqistries.jsp 接口存在 SSRF 漏洞，可以利用该漏洞可以发送任意 HTTP 请求，实现攻击内网中 Redis 等脆弱组件。

## 反序列化漏洞

Weblogic的事务管理组件WLS-WSAT，将xml数据转化为Java对象
WLS-WSAT 是 WebLogic Server 事务管理（WebLogic Server Transaction）的一个组件，它使用Java的反序列化机制来处理数据，调用 XMLDecoder类 将用户传入的XML数据转换为Java对象。\n\n在某些情况下攻击者可以构造恶意的序列化数据作为HTTP POST请求的一部分发送到WebLogic Server的T3协议端口（默认为7001），并且在请求头中设置一个特殊的“Content-Type”值来触发漏洞。当WebLogic Server处理该请求时，XMLDecoder将恶意的序列化数据反序列化为Java对象，并执行其中包含的恶意代码。
