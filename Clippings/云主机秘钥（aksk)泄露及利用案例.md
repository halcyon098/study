---
title: "云主机秘钥（ak/sk)泄露及利用案例"
source: "https://www.cnblogs.com/backlion/p/17901047.html"
author:
  - "[[渗透测试中心]]"
published: 2023-12-14
created: 2026-01-06
description: "前言云平台作为降低企业资源成本的工具，在当今各大公司系统部署场景内已经成为不可或缺的重要组成部分，并且由于各类应用程序需要与其他内外部服务或程序进行通讯而大量使用凭证或密钥，因此在漏洞挖掘过程中经常会遇到一类漏洞：云主机秘钥泄露。此漏洞使攻击者接管云服务器的权限，对内部敏感信息查看或者删除等操作。此"
tags:
  - "clippings"
---
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e985866d71b146418899d291d274785f.webp)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e985866d71b146418899d291d274785f.webp)

## 前言

云平台作为降低企业资源成本的工具，在当今各大公司系统部署场景内已经成为不可或缺的重要组成部分，并且由于各类应用程序需要与其他内外部服务或程序进行通讯而大量使用凭证或密钥，因此在漏洞挖掘过程中经常会遇到一类漏洞：云主机秘钥泄露。此漏洞使攻击者接管云服务器的权限，对内部敏感信息查看或者删除等操作。此篇文章围绕如何发现秘钥泄露、拿到秘钥后如何利用展开。

## 0X01漏洞概述

ak、sk拿到后的利用，阿里云、腾讯云

云主机通过使用Access  
Key Id / Secret Access Key加密的方法来验证某个请求的发送者身份。Access Key  
Id（AK）用于标示用户，Secret Access Key（SK）是用户用于加密认证字符串和云厂商用来验证认证字符串的密钥，其中SK必须保密。

云主机接收到用户的请求后，系统将使用AK对应的相同的SK和同样的认证机制生成认证字符串，并与用户请求中包含的认证字符串进行比对。如果认证字符串相同，系统认为用户拥有指定的操作权限，并执行相关操作；如果认证字符串不同，系统将忽略该操作并返回错误码。

AK/SK原理使用对称加解密。

## 0x02秘钥泄露常见场景

通过上面描述我们知道云主机密钥如果泄露就会导致云主机被控制，危害很大。

在漏洞挖掘过程中常见的泄露场景有以下几种：

1、报错页面或者debug信息调试。

2、GITHUB关键字、FOFA等。

3、网站的配置文件

4、js文件中泄露

5、源码泄露。APK、小程序反编译后全局搜索查询。

6、文件上传、下载的时候也有可能会有泄露，比如上传图片、上传文档等位置。

7、HeapDump文件。

## 0x03实战举例

### 案例一：HeapDump文件中的ak\\sk泄露

HeapDump [文件](https://www.eolink.com/news/tags-985.html) 是JVM虚拟机运行时内存的一个快照，通常用于性能分析等，但是因为其保存了对象、类等相关的信息，如果被泄露也会造成信息泄露。

1、Spring Actuator heapdump文件造成的秘钥泄露。

扫描工具： [https://github.com/F6JO/RouteVulScan](https://github.com/F6JO/RouteVulScan)

解压工具： [https://github.com/wyzxxz/heapdump\_tool](https://github.com/wyzxxz/heapdump_tool)

访问某一网站时进行测试发现存在spring未授权，此时查看是否有heapdump文件，下载解压，全局搜索可发现秘钥泄露。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f9ceaeb34070bac18783258285be6146.png)

2、通过暴破路径的方式获取。

在文件存储位置会有一些敏感文件泄露，比如请求下载云服务器上某文件时候抓包分析。可以在请求位置暴破文件名，云服务器会返回带有访问秘钥的敏感文件。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/593d42b57a93baca3efececccfccb83a.png)

得到文件地址后访问下载，下载后用工具爬取内容。发现泄露ak\\sk

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/42e72ab92576dc7bfdd23d135a274596.png)

工具链接： [https://github.com/whwlsfb/JDumpSpider](https://github.com/whwlsfb/JDumpSpider)

### 案例二：Js文件泄露秘钥

使用工具：hog

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/65f18c43d77ce68e5c5711898d68eaad.png)

访问某网站，使用插件hog探测，会在Findings位置显示是否有密钥泄露。（网站采用异步加载也适用）

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0c7ec8d9d23dca408c871767fd3e3f96.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/88442c37de8df5890851e7b602d07d30.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ab2c52edd575e4914a9710a1e519f918.png)

### 案例三：小程序上传等功能点泄露。

某小程序打开后在个人中心头像位置

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ad2b29f981b358a1bed4765dc83f4834.png)

点击头像抓包：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cb0652da18cdbea21f2d446424bf99b7.png)

可以看到accesskeyid\\acesskeysecret泄露。

渗透测试过程中可以多关注上传图片、下载文件、查看图片等等位置，说不定就有ak\\sk泄露。

### 案例四：配置信息中的ak\\sk泄露

常见的nacos后台配置列表，打开示例可以看到一些配置信息，可以看到有ak\\sk泄露。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ed72813d191e8921fab3087594e5f06c.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2654f59bc9b1ae38fdf8f936fb54d725.png)

## 0x04漏洞利用

### 1、ak\\sk接管存储桶。

使用工具或者云主机管理平台可以直接接管存储桶，接管桶后可以对桶内信息进行查看、上传、编辑、删除等操作。

OSS Browser--阿里云官方提供的OSS图形化管理工具

[https://github.com/aliyun/oss-browser](https://github.com/aliyun/oss-browser)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/af8b5faeae8a6f79bbe82bdb78f19b54.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8d5dd7d65869fb3fe1a70f98ebd25500.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/eb851e0fd7a9d1891c4b22d91eb20fc6.png)

可以看到登入存储桶后可以查看、上传、删除、下载桶内文件，造成存储桶接管的危害。

腾讯云云主机接管平台：

[https://cosbrowser.cloud.tencent.com/web/bucket](https://cosbrowser.cloud.tencent.com/web/bucket)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2b8543e5a6205020e82eaf235b297d83.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/50da6a2b15710e7d23fb3afceba58bfe.png)

行云管家（支持多家云主机厂商）：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7369ef11757bdfe2d9e994369bfb61b7.png)

可以选不同厂商的云主机导入。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/67e4b4711366a3b1ff183729e8c98d60.png)

选择主机导入：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d99b94bb5977c66d2c7daddc16bcdda5.png)

通过行云管家接管主机后，不仅可以访问OSS服务，还可以直接重置服务器密码，接管服务器。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/07fd27374e0c28898636882e5a68ef3f.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fc6e17d0a39018e79275cfe5af6f370b.png)

可以对主机进行重启、暂停、修改主机信息等操作。

### 2、拿到ak\\sk后可以尝试对主机进行命令执行。

CF 云环境利用框架

[https://github.com/teamssix/cf/releases](https://github.com/teamssix/cf/releases)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f74440f7f510c9f61ebd82b74946546f.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8a92875c906afeb92c9637fa337e696e.png)

使用cf查看该主机可做的操作权限，可以看到能执行命令。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/90e7a24b0154ff39389c1f920734a73c.png)

cf tencent cvm exec -c whoami等等。

详情参考： [https://wiki.teamssix.com/CF/ECS/exec.html](https://wiki.teamssix.com/CF/ECS/exec.html)

针对阿里云主机rce

工具链接： [https://github.com/mrknow001/aliyun-accesskey-Tools](https://github.com/mrknow001/aliyun-accesskey-Tools)

输入ak\\sk查询主机，选择主机名填入，查看云助手列表是true或者false，为true可执行命令。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9d820a73a92d1d69f05d510b7652c79f.png)  

  

转自原文链接： [https://forum.butian.net/share/2376](https://forum.butian.net/share/2376)

分类:

免责声明：本内容来自平台创作者，博客园系信息发布平台，仅提供信息存储空间服务。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fc6adea6ccc5179d59eb91560feaddb8.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fc6adea6ccc5179d59eb91560feaddb8.png)

[渗透测试中心](https://home.cnblogs.com/u/backlion/)  
[粉丝 - 925](https://home.cnblogs.com/u/backlion/followers/) [关注 - 0](https://home.cnblogs.com/u/backlion/followees/)  

[+加关注](https://www.cnblogs.com/backlion/p/)

0

0

[升级成为会员](https://cnblogs.vip/)

[«](https://www.cnblogs.com/backlion/p/17900881.html) 上一篇： [2023年最新微信小程序抓包及测试案例](https://www.cnblogs.com/backlion/p/17900881.html "发布于 2023-12-14 11:41")  
[»](https://www.cnblogs.com/backlion/p/17901091.html) 下一篇： [记一次省护网红队案例](https://www.cnblogs.com/backlion/p/17901091.html "发布于 2023-12-14 14:25")

posted @ 2023-12-14 14:03 [渗透测试中心](https://www.cnblogs.com/backlion) 阅读(8497)  评论(0) [收藏](https://www.cnblogs.com/backlion/p/) [举报](https://www.cnblogs.com/backlion/p/)

<table><tbody><tr><td colspan="7"><table><tbody><tr><td><a href="https://www.cnblogs.com/backlion/p/">&lt;</a></td><td align="center">2026年1月</td><td align="right"><a href="https://www.cnblogs.com/backlion/p/">&gt;</a></td></tr></tbody></table></td></tr><tr><th align="center">日</th><th align="center">一</th><th align="center">二</th><th align="center">三</th><th align="center">四</th><th align="center">五</th><th align="center">六</th></tr><tr><td align="center">28</td><td align="center">29</td><td align="center">30</td><td align="center">31</td><td align="center">1</td><td align="center">2</td><td align="center">3</td></tr><tr><td align="center">4</td><td align="center">5</td><td align="center">6</td><td align="center">7</td><td align="center">8</td><td align="center">9</td><td align="center">10</td></tr><tr><td align="center">11</td><td align="center">12</td><td align="center">13</td><td align="center">14</td><td align="center">15</td><td align="center">16</td><td align="center">17</td></tr><tr><td align="center">18</td><td align="center">19</td><td align="center">20</td><td align="center">21</td><td align="center">22</td><td align="center">23</td><td align="center">24</td></tr><tr><td align="center">25</td><td align="center">26</td><td align="center">27</td><td align="center">28</td><td align="center">29</td><td align="center">30</td><td align="center">31</td></tr><tr><td align="center">1</td><td align="center">2</td><td align="center">3</td><td align="center">4</td><td align="center">5</td><td align="center">6</td><td align="center">7</td></tr></tbody></table>

### 随笔分类

- [更多](https://www.cnblogs.com/backlion/p/)

### 随笔档案

- [2025年11月(2)](https://www.cnblogs.com/backlion/p/archive/2025/11)
- [2025年10月(1)](https://www.cnblogs.com/backlion/p/archive/2025/10)
- [2025年9月(6)](https://www.cnblogs.com/backlion/p/archive/2025/09)
- [2025年5月(8)](https://www.cnblogs.com/backlion/p/archive/2025/05)
- [2025年4月(5)](https://www.cnblogs.com/backlion/p/archive/2025/04)
- [2024年12月(5)](https://www.cnblogs.com/backlion/p/archive/2024/12)
- [2024年11月(2)](https://www.cnblogs.com/backlion/p/archive/2024/11)
- [2024年10月(16)](https://www.cnblogs.com/backlion/p/archive/2024/10)
- [2024年9月(3)](https://www.cnblogs.com/backlion/p/archive/2024/09)
- [2024年8月(2)](https://www.cnblogs.com/backlion/p/archive/2024/08)
- [2024年7月(3)](https://www.cnblogs.com/backlion/p/archive/2024/07)
- [2024年5月(9)](https://www.cnblogs.com/backlion/p/archive/2024/05)
- [2024年3月(1)](https://www.cnblogs.com/backlion/p/archive/2024/03)
- [2024年1月(45)](https://www.cnblogs.com/backlion/p/archive/2024/01)
- [2023年12月(14)](https://www.cnblogs.com/backlion/p/archive/2023/12)
- [更多](https://www.cnblogs.com/backlion/p/)

### 相册

- [image(1)](https://www.cnblogs.com/backlion/gallery/1300217.html)

[字节旗下的 TRAE](https://www.trae.com.cn/?utm_source=advertising&utm_medium=cnblogs_ug_cpa&utm_term=hw_trae_cnblogs)