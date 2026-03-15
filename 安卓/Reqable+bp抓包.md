https://xz.aliyun.com/t/14651?time__1311=GqAhYKBIqIxGx0HW2D9DRxYTPWTokD1YD
reqable官网:https://reqable.com/zh-CN/

## 0x01)reqable

很多师傅应该使用过小黄鸟抓包工具，在手机上是比较好用的，然后这个开发者后续开发了reqable这款工具，它的功能非常的强大。这里可以在ios上安装reqable和电脑上的burp进行联动。（也是需要电脑和手机连接同一个WIFI，电脑最好把防火墙关了）  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153725-cbba6080-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153725-cbba6080-1b32-1.png)  
[https://reqable.com/zh-CN/download](https://reqable.com/zh-CN/download) 这里可以先下载Windows安装，然后安卓apk可以z在软件里直接下载了。  
下载后就开始配置信息

## 0x02)配置

首先就是安装好我们的一个证书

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153756-de920654-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153756-de920654-1b32-1.png)  
我这里已经安装了，如果没有安装就是直接点击自动安装（非常的人性化）

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153810-e693674e-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153810-e693674e-1b32-1.png)  
然后呢，我们点击Android安装证书，这里也是非常的贴心。注意这个ip

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153818-eb84c234-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153818-eb84c234-1b32-1.png)  
这里ip就是连接wifi的地址
这里可以选择ip  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153834-f513e870-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153834-f513e870-1b32-1.png)  
然后再回到上面的教程，直接下载安装加信任一条路结束。结束之后可以把wifi设置的代理给关了。（好像还有个同步证书的操作）

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153852-ffba6876-1b32-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153852-ffba6876-1b32-1.png)  
点击这里进行扫码和电脑联动

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153917-0ebe0af8-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153917-0ebe0af8-1b33-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153921-1120ba0c-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153921-1120ba0c-1b33-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153932-1794a95c-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153932-1794a95c-1b33-1.png)

电脑上就可以看到手机上的数据包了

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526153957-263ea26e-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526153957-263ea26e-1b33-1.png)

## 0x03)联动burp

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154012-2f26fe6c-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154012-2f26fe6c-1b33-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154015-316b461a-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154015-316b461a-1b33-1.png)  
burp这里选择和reqable一样的host和端口（这里因为截图ip变了所以不一样）  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154020-3407dde8-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154020-3407dde8-1b33-1.png)  
这里把全局关一下

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154044-42433808-1b33-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154044-42433808-1b33-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154730-346e1076-1b34-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154730-346e1076-1b34-1.png)

可以看到burp里已经出现了流量信息。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20240526154737-388cca30-1b34-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240526154737-388cca30-1b34-1.png)
## 0x04)踩坑
面具有自动安装reqable证书到模块。
使用面具MagiskHide功能时一定要将应用加入sulist(白名单)，抓取有ssl pinning的应用时在**LSPosed中的JustTrustMe,和TrustMeAlready要将抓流量的app加进去。**