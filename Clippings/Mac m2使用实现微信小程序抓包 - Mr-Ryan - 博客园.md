---
title: "Mac m2使用实现微信小程序抓包 - Mr-Ryan - 博客园"
source: "https://www.cnblogs.com/mr-ryan/p/17680899.html"
author:
published:
created: 2025-04-15
description: "# Mac m2使用实现微信小程序抓包 最近换了MacBook Pro，芯片是M2 Pro，很多东西跟windows是不一样的，所以重新配置相应环境，这里介绍一下微信小程序抓包的方法。 ## 使用burp+proxifier实现小程序抓包 burp的配置这里就不做介绍了。 Proxifier是一款功"
tags:
  - "clippings"
---
最近换了MacBook Pro，芯片是M2 Pro，很多东西跟windows是不一样的，所以重新配置相应环境，这里介绍一下微信小程序抓包的方法。

## 使用burp+proxifier实现小程序抓包

burp的配置这里就不做介绍了。

Proxifier是一款功能非常强大的socks5客户端。

Proxifier功能:

- 通过代理服务器运行任何网络应用程序。软件不需要特殊配置，整个过程十分透明。
- 通过代理服务器网关访问限制网络的互联网。
- 绕过防火墙限制。
- 通过代理服务器解析 DNS 名称。
- 使用带有主机名和应用程序名称通配符的灵活的代理规则。
- 通过隐藏 IP 地址确保隐私。
- 使用不同协议通过一组代理服务器链。
- 实时查看关于当前网络活动（连接，主机，时间，带宽使用等）的信息。  
	... 等等。

### 安装激活proxifier

1. 访问proxifier官网进行下载， [下载地址](https://proxifier.com/download/)
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213730808-2012868662.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213730808-2012868662.png)
	下载完成后自行安装即可。
2. proxifier软件需要收费，也可以免费使用31天，激活码可以自行在网上搜索。
	注意：windows和mac的激活码不通用
	这里也附上我找到的激活码
	highlighter-
	```
	用户名：V2
	激活码：P427L-9Y552-5433E-8DSR3-58Z68
	用户名：V3
	激活码：3CWNN-WYTP4-SD83W-ASDFR-84KEA
	```

### 配置proxifier代理

1. 点击 `Proxies`
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213750449-511656421.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213750449-511656421.png)
2. 配置代理服务器
	点击 `Add` —>地址默认为 `127.0.0.1` ，端口为 `8080`,与burp一致，进行保存
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213802929-2103352877.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213802929-2103352877.png)
3. 进行规则配置
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213820028-1332520490.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213820028-1332520490.png)
4. 点击 `Add` 进行添加规则， `name` 自己随意命名， `Action` 选择前面配置好的代理 `127.0.0.1:8080` ， `Applications` 中点击 `➕` 选择微信小程序。
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213836813-1837133991.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213836813-1837133991.png)
5. 点击➕后，在访达中使用快捷键 `command+shift+G` 搜索小程序的位置
	highlighter-
	```
	/Applications/WeChat.app/Contents/MacOS/WeChatAppEx.app/Contents/
	```
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213853391-897426413.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213853391-897426413.png)
6. 选择 ` WeChatAppEx Helper.app`
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213907036-954394802.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213907036-954394802.png)
7. 确认无误，点击 `Save` 进行保存
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213920454-1654665766.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213920454-1654665766.png)
8. 打开burp，打开小程序进行测试
	[![](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213935916-682528946.png)](https://img2023.cnblogs.com/blog/3164322/202309/3164322-20230905213935916-682528946.png)
	成功抓取
  

\_\_EOF\_\_

posted @ [Mr-Ryan](https://www.cnblogs.com/mr-ryan)   阅读(4719)  评论(2) [收藏](https://www.cnblogs.com/mr-ryan/p/) [举报](https://www.cnblogs.com/mr-ryan/p/)

[升级成为园子VIP会员](https://cnblogs.vip/)

\[Ctrl+Enter快捷键提交\]

**相关博文：**  

· [Github工具库](https://www.cnblogs.com/mr-ryan/p/17381525.html "Github工具库")

· [vulnhub 靶场DC-8实战指南](https://www.cnblogs.com/mr-ryan/p/18433615 "vulnhub 靶场DC-8实战指南")

· [微信PC端小程序抓包-Burp](https://www.cnblogs.com/NoCirc1e/p/17467478.html "微信PC端小程序抓包-Burp")

· [微信小程序如何抓包](https://www.cnblogs.com/maohai-kdg/p/17582905.html "微信小程序如何抓包")

· [MAC burpsuite抓取微信小程序数据包](https://www.cnblogs.com/thespace/p/16714616.html "MAC   burpsuite抓取微信小程序数据包")

**阅读排行：**  
· [7 个最近很火的开源项目「GitHub 热点速览」](https://www.cnblogs.com/xueweihan/p/18825970)  
· [DeepSeekV3：写代码很强了](https://www.cnblogs.com/cicada-smile/p/18826084)  
· [记一次.NET某固高运动卡测试 卡慢分析](https://www.cnblogs.com/huangxincheng/p/18824441)  
· [Visual Studio 2022 v17.13新版发布：强化稳定性和安全，助力.NET 开发提](https://www.cnblogs.com/Can-daydayup/p/18825782)  
· [C# LINQ 快速入门实战指南，建议收藏学习！](https://www.cnblogs.com/Can-daydayup/p/18824060)