# Tomcat

文件上传漏洞
 Tomcat配置文件/conf/web.xml 配置了可写（readonly=false），导致可以使用PUT方法上传任意文件，攻击者将精心构造的payload向服务器上传包含任意代码的 JSP 文件。之后，JSP 文件中的代码将能被服务器执行。
利用配置文件可写，使用put请求上传jsp文件，服务器执行jsp文件

文件包含漏洞
Tomcat 配置了两个Connecto，它们分别是 HTTP 和 AJP ：HTTP默认端口为8080，处理http请求，而AJP默认端口8009，用于处理 AJP 协议的请求，而AJP比http更加优化，多用于反向、集群等，漏洞由于Tomcat AJP协议存在缺陷而导致，攻击者利用该漏洞可通过构造特定参数，读取服务器webapp下的任意文件以及可以包含任意文件，如果有某上传点，上传图片马等等，即可以获取shell。

利用ajp协议缺陷，读取，包含任意文件，配合上传点getshell

弱口令
在tomcat8环境下默认进入后台的密码为tomcat/tomcat，未修改造成未授权即可进入后台。
