# 绕waf

## sqlmap流量绕waf

### SQLMAP伪造IP白名单

在网站架构中，会经常使用IP白名单技术，把一些可以信任的IP地址添加到IP白名单里面，这些IP地址通常是服务器自身的IP地址或者是管理员常用的IP地址，WAF对IP白名单里面的IP地址进行的访问，通常不进行处理，直接放行。基于此原理，我们在使用SQLMAP进行SQL注入的渗透测试的时候，可以将自己伪造成服务器自身的IP地址，以便绕过WAF的检测机制。当然，我们无法伪造网路层的IP地址，我们可以伪造的是HTTP头中的X-Forwarded-For、X-Remote-IP、X-Originating-IP、X-Remote-Addr、X-Real-IP等参数。

我们在使用SQLMAP的时候，可以使用–headers，指定这些HTTP头部，加入到SQLMAP发送的数据包中，如果要使用多个HTTP头，则可以在–headers参数里面使用\n（换行）来分隔。

### SQLMAP伪造静态资源

除了伪造IP白名单以外，我们还可以通过伪造静态资源的方式来进行绕过。一般而言，对网站静态资源的访问不会造成安全威胁，比如访问一张图片、JS脚本等等。基于此，很多WAF不会对静态资源的访问来进行检测。我们则可以利用这一点，来将我们的访问伪造成对静态资源的访问。
例如，目标站点URL为：

> http://x.x.x.x/sqli/Less-1/index.php?id=1

我们可以将其改造为：

> http://x.x.x.x/sqli/Less-1/index.php?id=1/123.js?id=1

当然，这种改造方式，需要站点服务器将上述改造后的URL当作index.php文件来进行解析，并且WAF错误的将上述URL当作静态资源。因此，这种方法在本质上就是利用了站点服务器和WAF对URL的解析差异来进行的绕过。
我们在使用SQLMAP来伪造静态资源时，需要手动的对URL进行改造，这样SQLMAP就可以使用我们改造后的payload了。

### SQLMAP伪造URL白名单

很多站点都具有管理后台，通过管理后台可以方便的管理整个站点的资源、模块、数据库等等。因为站点管理后台的很多操作都是对站点的操作，因此经常被WAF所误报拦截。为了解决这一问题，很多WAF都会设置一个URL的白名单，对该白名单的URL，不进行检测，直接放行。常见的站点管理后台URL包含有manage、system、admin等关键字，因此很多WAF对后台的判断也是基于URL中是否具有上述关键字。
因此，我们可以利用这一点来绕过SQLMAP的检测。例如，目标站点URL为：

> http://x.x.x.x/sqli/Less-1/index.php?id=1

为了使得我们的SQL注入payload绕过WAF的检测，我们可以构造如下payload：

> http://x.x.x.x/sqli/Less-1/index.php?a=manage&b=system&c=admin&id=1

在该payload中，我们使用了无关的参数构造了含有manage、system和admin的URL，这样则可以试图利用URL白名单的方式，绕过WAF的检测。
我们在使用SQLMAP进行SQL注入时，也可以认为构造相应的参数，然后进行检测。

### SQLMAP伪造User-Agent

最后，给大家介绍一下SQLMAP伪造User-Agent的原理。

如果说前面的三种小技巧都是锦上添花的手段，那么对于User-Agent的伪造则是使用SQLMAP进行SQL注入绕过WAF时的必须了。

这是因为在默认情况下，SQLMAP的数据包User-Agent会带有其独有的User-Agent，而很多WAF会检测这样的User-Agent，检测到之后会直接视作攻击处理。

SQLMAP的User-Agent携带有sqlmap/1.6.8.2#dev(http://sqlmap.org)的User-Agent
因此，我们需要修改SQLMAP自带的User-Agent，可以考虑将其修改为常用浏览器的User-Agent，常用浏览器的User-Agent有很多，例如：
火狐浏览器：

> Mozilla/5.0(WindowsNT6.1;rv:2.0.1)Gecko/20100101Firefox/4.0.1

360浏览器：

> Mozilla/4.0(compatible;MSIE7.0;WindowsNT5.1;360SE)

类似的User-Agent内容我们都可以在网上搜索到。

我们在使用SQLMAP进行SQL注入测试时，如果要修改SQLMAP数据包中的User-Agent内容，可以使用–user-agent参数。例如，构造如下SQLMAP命令：

```bash
python .\sqlmap.py -u http://127.0.0.1/sqli/Less-1/?id=1 --proxy "http://127.0.0.1:8080"  --user-agent="Mozilla/5.0(WindowsNT6.1;rv:2.0.1)Gecko/20100101Firefox/4.0.1"
```

## shrio反序列化流量绕waf

### **HTTP请求方法随机**

首先最被大家常用的绕waf方法就是HTTP请求头变为随机字符串，在本案例过程中，将“**GET**”请求方法变为“**xxxxT**”方法，发现是能正常执行成功的。这种方法与Web应用所处的中间件有关，在部分中间件下不适用。

### **HTTP请求方法置空**

一些朋友可能只关注了HTTP请求方法随机化的问题，但是对于tomcat，将HTTP请求方法置空也是可以正常发包并返回命令执行结果，这种畸形数据包在经过waf设备会被放行，因为waf设备解析不了。这种绕过方法与中间件有关，在Weblogic中间件下不适用。

### **Shiro数据包添加脏数据**

这种方法在网上很少被提起，“**rememberMe=**”后面的数据包添加一些特殊字符仍然是可以正常发包的，原因是shiro组件在处理点号、反引号等特殊字符，会替换为空。

### **Shiro字段添加空白字符**

前面我们提到了，“**rememberMe=**”后面可以掺杂特殊字符，那么“**rememberMe**”关键词附近可否动动手脚呢？经过fuzz测试，发现添加“**Tab**”等空白字符是可以正常执行命令的。

### **Host头域名变IP地址**

很多甲方公司购买了waf或者一些云waf，但是可能目标网站只对“***.xxx.com**”域名进行了waf防护，这时候将host头的域名替换为域名解析出来的ip，就可以绕过waf了。

### 文中提到的添加点号等特殊字符绕过waf的思路，对于Struts2框架同样适用