https://xz.aliyun.com/t/14651?time__1311=GqAhYKBIqIxGx0HW2D9DRxYTPWTokD1YD
reqable官网:https://reqable.com/zh-CN/

## 0x01)reqable

很多师傅应该使用过小黄鸟抓包工具，在手机上是比较好用的，然后这个开发者后续开发了reqable这款工具，它的功能非常的强大。这里可以在ios上安装reqable和电脑上的burp进行联动。（也是需要电脑和手机连接同一个WIFI，电脑最好把防火墙关了）  
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5f3926288a500dea155086a704cc9a97.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5f3926288a500dea155086a704cc9a97.png)  
[https://reqable.com/zh-CN/download](https://reqable.com/zh-CN/download) 这里可以先下载Windows安装，然后安卓apk可以z在软件里直接下载了。  
下载后就开始配置信息

## 0x02)配置

首先就是安装好我们的一个证书

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cc5c4ecc4081cb2d6480f609eb1953b0.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cc5c4ecc4081cb2d6480f609eb1953b0.png)  
我这里已经安装了，如果没有安装就是直接点击自动安装（非常的人性化）

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f4861d524fdbf83397a8e2f68f6731c8.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f4861d524fdbf83397a8e2f68f6731c8.png)  
然后呢，我们点击Android安装证书，这里也是非常的贴心。注意这个ip

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/acc2f741005469de4243588af6c2aea1.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/acc2f741005469de4243588af6c2aea1.png)  
这里ip就是连接wifi的地址
这里可以选择ip  
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/32d69d9d318b1206a52cb6212318b69b.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/32d69d9d318b1206a52cb6212318b69b.png)  
然后再回到上面的教程，直接下载安装加信任一条路结束。结束之后可以把wifi设置的代理给关了。（好像还有个同步证书的操作）

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1f23f71384ed70b43a986c62586e57a1.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1f23f71384ed70b43a986c62586e57a1.png)  
点击这里进行扫码和电脑联动

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8aea91a850213cf808f09b1f18c2034c.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8aea91a850213cf808f09b1f18c2034c.png)

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/17339e86ea90d5b691bfc8c9214e752f.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/17339e86ea90d5b691bfc8c9214e752f.png)

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0eb176b3b66fdea2d14415e065f90be4.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0eb176b3b66fdea2d14415e065f90be4.png)

电脑上就可以看到手机上的数据包了

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9cc0b62a006e19ea792663de5d4134e4.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9cc0b62a006e19ea792663de5d4134e4.png)

## 0x03)联动burp

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/37e26d4626c37fe82c884535059843d2.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/37e26d4626c37fe82c884535059843d2.png)

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ca4502fc7dde915d4133f8c1a960a6f0.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ca4502fc7dde915d4133f8c1a960a6f0.png)  
burp这里选择和reqable一样的host和端口（这里因为截图ip变了所以不一样）  
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/273ca887cc917addfead28f9d0509bd2.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/273ca887cc917addfead28f9d0509bd2.png)  
这里把全局关一下

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5cdc0112109dab0d2040e5bd5708bbd6.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5cdc0112109dab0d2040e5bd5708bbd6.png)

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4d1974f874393f70627e8d795d0a02aa.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4d1974f874393f70627e8d795d0a02aa.png)

可以看到burp里已经出现了流量信息。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/71d4f41dde408413e770c71c5f6d5849.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/71d4f41dde408413e770c71c5f6d5849.png)
## 0x04)踩坑
面具有自动安装reqable证书到模块。
使用面具MagiskHide功能时一定要将应用加入sulist(白名单)，抓取有ssl pinning的应用时在**LSPosed中的JustTrustMe,和TrustMeAlready要将抓流量的app加进去。**