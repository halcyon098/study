---
title: "Nacos配置文件攻防思路总结-先知社区"
source: "https://xz.aliyun.com/news/15068"
author:
published:
created: 2025-05-23
description: "先知社区是一个安全技术社区，旨在为安全技术研究人员提供一个自由、开放、平等的交流平台。"
tags:
  - "clippings"
---
Nacos配置文件攻防思路总结

[渗透测试](https://xz.aliyun.com/news?cate_id=24) 1629浏览 · 2024-09-15 10:27

## 前言

笔者最近在进行内网渗透时，遇到了存在超多Nacos资产的情况。而笔者在搜索Nacos相关利用资料时，发现大家的大部分注意力，似乎都是聚焦在Nacos本身的问题上，比如是否存在某个漏洞，能否RCE。

但是对于Nacos配置文件的一些衍生利用思路，却很少提及。根据笔者遇到的情况来看，对Nacos配置文件的分析利用（尤其是超多Nacos资产情况下），重要性其实是不亚于甚至超过RCE的。

鉴于此，笔者临时写了一个利用脚本，可以利用nacos jwt弱密钥漏洞，导出nacos所有namespace的所有配置文件，然后通过正则匹配相关敏感信息，做初步的信息整理，方便后面的攻击思路的整理。（脚本获取方式在文末）

下面是笔者根据遇到的情况，总结的打法思路及拓展，欢迎师傅们补充、批评指正！

## 打法思路概述

根据笔者遇到的情况，通过nacos配置文件延伸出来的打法，主要有以下几条，供大家参考：

```
查找数据库账密，尤其是redis、postgresql这类能拿shell的-->横向/敏感数据
查找xxl-job账密-->拿业务系统服务器/横向
查找jwt key，伪造token登录-->减少爆破/进业务系统（但是个人感觉并不是很好伪造）
查找AK/SK-->存储桶接管/云主机接管
查找企微corpid和corpsecret-->获取员工信息/了解企业架构/接管企微应用/钓鱼
查找邮箱信息-->写到Nacos配置里的一般都是公共邮箱，可以查找邮件来往中的账密信息-->登录业务系统，还有就是利用公共邮箱来钓鱼
```
```
当然了站在甲方的立场，也可以是根据以上的攻击路径，来证明该资产的危害，加强领导的重视，促进业务方的整改（手动狗头）。
```

1、2两条没什么可说的，找到账密后都是比较常规的操作，就不演示了。  
下面说一下实际环境中遇到3/4/5/6这4条路径的一些操作，以供师傅们参考。若有不对的地方，该请见谅并指正！

## 企微corpid和corpsecret-人员信息&钓鱼

这块利用思路的话，主要有：

```
1、查找部门、人员信息；
2、创建企微成员，然后钓鱼（创建成员需要通讯录权限,笔者很少遇到）；
3、没有权限创建成员的话，利用企微应用发消息钓鱼；（划重点）
```

## 基础信息获取

### 获取企微access\_token

[https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=SafetyCockpit03&corpsecret=SafetyCockpit03](https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=SafetyCockpit03&corpsecret=SafetyCockpit03)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/348ee3fdbf65719ec9a48fe6f9bd4d22.png)

### 先查看access\_token权限

[https://open.work.weixin.qq.com/devtool/query](https://open.work.weixin.qq.com/devtool/query)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4d09ab4642278b2919f0ebc343358867.png)

### 获取企业微信API域名IP段

[https://qyapi.weixin.qq.com/cgi-bin/get\_api\_domain\_ip?access\_token=SafetyCockpit03](https://qyapi.weixin.qq.com/cgi-bin/get_api_domain_ip?access_token=SafetyCockpit03)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d03d9f10d6df7acfd0db652d89dbd8b6.png)

### 获取部门列表

[https://qyapi.weixin.qq.com/cgi-bin/department/list?access\_token=SafetyCockpit03](https://qyapi.weixin.qq.com/cgi-bin/department/list?access_token=SafetyCockpit03)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/13326308d4e219bd455e7cc7f13244d6.png)

### 获取部门成员/详情

[https://qyapi.weixin.qq.com/cgi-bin/user/simplelist?access\_token=SafetyCockpit03&department\_id=SafetyCockpit03&&fetch\_child=1](https://qyapi.weixin.qq.com/cgi-bin/user/simplelist?access_token=SafetyCockpit03&department_id=SafetyCockpit03&&fetch_child=1)  
我这里尝试，未带&&fetch\_child=1如果只出来的一条信息，那么一般就是部门老大的信息（划重点，大鱼，只要你敢钓（狗头保命））：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a62b898000afde19e42e0fe144b08ee5.png)  
而带&&fetch\_child=1遍历的话，就是全部门的人员信息  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e5332e6776a4efd09ac1a5d280570601.png)

## 获取加入企业二维码/创建成员

[https://qyapi.weixin.qq.com/cgi-bin/corp/get\_join\_qrcode?access\_token=SafetyCockpit03](https://qyapi.weixin.qq.com/cgi-bin/corp/get_join_qrcode?access_token=SafetyCockpit03)  
拿到secret当然是直接添加成员到企微，然后进行后续操作比如钓鱼的话方便一点。  
但有时候我们拿到的key可能没有那么大的权限，如图，在获取加入企业微信二维码和创建成员的时候，并没有权限：  
获取加入企业二维码：错误码48002  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3796c4a89de720c64debaa8346ff8a4c.png)  
创建成员：错误码48002  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5e618261e6d9848bd37445b86356177c.png)  
错误码48002解释：API接口无权限调用  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3ac1a8374d099a13205efb580e72e85a.png)  
这个时候没法加入企微，除了得到一些人员信息，还能怎么利用呢？  
答案是我们可以接管企微应用，利用企微应用钓鱼。

## 接管高权限应用

如图，查询access\_token的时候，可以查到应用的AgentId，这个发送消息会用到。  
[https://developer.work.weixin.qq.com/devtool/query](https://developer.work.weixin.qq.com/devtool/query)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cfaa984cc96b04952567e4390b91186a.png)  
然后在 [https://developer.work.weixin.qq.com/resource/devtool](https://developer.work.weixin.qq.com/resource/devtool) 工具里，发送应用消息  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b83ae3b7e0c09a10fd2536877a968d27.png)  
或者直接掏出数据包发送：

```
POST /cgi-bin/message/send?access_token=SafetyCockpit03 HTTP/1.1
Host: qyapi.weixin.qq.com
Content-Type: application/json

{
   "touser": "SafetyCockpit03", #一般是工号，结合上面获得的基础信息
   "toparty": "@all",
   "totag": "@all",
   "msgtype" : "news",
   "agentid" : 1000xxx, #上面查出来的应用的AgentId
   "news" : {
       "articles" : [
           {
               "title" : "中秋节礼品领取",
               "description" : "今年中秋节公司有豪礼相送",
               "url" : "钓鱼链接",
               "picurl" : "展示的图片链接",
         "appid": "",
             "pagepath": ""
           }
        ]
   },
   "enable_id_trans": 0,
   "enable_duplicate_check": 0,
   "duplicate_check_interval": 1800
}
```

测试效果图：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/32eace5187e1d4e4ba98d4d154f6aacd.png)  
点进去后：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/60e656203823f12d1b9e3349440d286d.png)

更多应用消息发送方式见官方文档，但是一般来讲上面提供的图文消息就够用了。

## 邮箱-敏感信息&JWT伪造&钓鱼

这块利用思路的话：主要有：

```
1、钓鱼；因为nacos配置文件中的邮箱，一般都是一些公共邮箱，非常适合拿来钓鱼；
2、查找账密信息；理由如上；
3、结合第二条，进行JWT伪造，接管高权限账号；
```

具体思路操作如下：  
比如打进去的公司邮箱域名为@xxx.com，那么就可以先搜索mail、mail.xxx.com，看有无网页登录邮箱的口子，要不然拿到账密后续也难利用。  
如图，这里通过搜索mail.xxx.com，找到两个域名：

```
email.xxx.com
szmail.xxx.com
```

经确认第一个outlook的登录页面，第二个是coremail的登录页面  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cff7ab8467b373f54395893d9b85172a.png)  
然后就是搜索关键字，比如：邮箱，@xxx.com  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/62a21051ef0355976cb22b07e70450da.png)  
最终找到N个公共邮箱系统，登录其中一个，是个核心公共邮箱，好几万封邮件：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0cb1e020d9db36cfc793cebbc2fed511.png)  
还可以搜索关键字，比如账号、开通、激活、经理……  
这里搜索出来一些最近的账号开通信息：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/03bfb1617e3d5a7ae9889e3840c08870.png)  
很贴心，账密系统URL都在里面：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/aa91654e6fbd280bbb82863273d22c01.png)  
成功拿邮件里的账密成功登录到某系统，但就是个普通用户：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/add0fa411e0c4e81f62a35dbe5fd6e45.png)  
F12查看该系统的验证方式，发现是JWT  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cce0a3efb56df59e9199da959da14dbb.png)  
复制到jwt.io看下信息：发现算法是HS256对称加密算法，可以在配置文件找找秘钥，看能否伪造管理员的token。  
不过看着data部分的rnStr字段，有种不妙的感觉。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f8559355c1f10424c9610bb16e462fe1.png)  
这里运气挺好，根据关键字找到了jwt key:  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b666b8bd6de78e7fbd9161dc0adf3302.png)  
拿爆破脚本验证下该key是不是正确的：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/716ad1b0b0d4ef986a98fbddd5a6dfe1.png)  
也可以直接在jwt.io验证，具体方式看图，稍微有点绕：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/77dbdf6fa083175769a09cd299ccad21.png)  
先改变一下userid，从原来的4087变成4086，测试下是否可以伪造。  
改变后发现直接提示401，感觉和上面提到的rnStr字段有关，试了删除了也不行，最终尝试无果，只能从其他地方找突破了。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2bcf841af8e160f5b8abafdc929ad43f.png)  
其他思路再有就是钓鱼了，因为本身就是公共账号，所以具有公信力，直接根据来往邮件信息，准备好话术开钓即可。

## AK/SK-敏感信息&云主机接管

这块不太熟，就不妄论了，可以网上找找更加详细的利用方式。  
但是需要注意的是，前期最好不要使用开源的云环境利用框架工具，因为大部分厂商都是有安全监测的，这里以阿里云为例：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/efca2407360f764cbcb8a80b97e8043d.png)  
所以建议先使用官方提供的工具看看存储桶有无敏感信息，没有敏感信息的话再用开源工具进行利用也不迟，以免浪费辛辛苦苦搜集到的key。  
阿里云官方工具下载链接：  
[https://gosspublic.alicdn.com/ossbrowser/1.18.0/oss-browser-win32-x64.zip](https://gosspublic.alicdn.com/ossbrowser/1.18.0/oss-browser-win32-x64.zip)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2a4f03d67a6d3502c7cc8dfb31835ed1.png)  
腾讯云的话，可以直接使用在线环境：  
[https://cosbrowser.cloud.tencent.com/login](https://cosbrowser.cloud.tencent.com/login)  
[https://cos.cloud.tencent.com/tools/cosbrowser](https://cos.cloud.tencent.com/tools/cosbrowser)  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3540b5266d334cc50da5a9426f8578fb.png)  
AWS官方工具下载链接：  
[https://s3browser.com/download/s3browser-11-7-5.exe](https://s3browser.com/download/s3browser-11-7-5.exe)  
巨多资料：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/61ff6bf08df291ee9011649152e95f95.png)

## 如何防护

写了这么多利用方式，站在甲方角度的话，该如何防护呢？

```
1、常规漏洞的话，根据官方修复建议修复即可；
2、配置文件的话，建议开发在写的时候，能加密的尽量加密，可以参考官方文档：
https://nacos.io/docs/v2/plugin/config-encryption-plugin/
```

效果图：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b0dab5c8e5579e1bb7883d2451bad30c.png)

## 参考

[https://xz.aliyun.com/t/11092](https://xz.aliyun.com/t/11092)  
[https://mp.weixin.qq.com/s?\_\_biz=MzkzMzYzNzIzNQ==&mid=2247483738&idx=1&sn=b1a7e73e4228d2008322d1ccd7671d89](https://mp.weixin.qq.com/s?__biz=MzkzMzYzNzIzNQ==&mid=2247483738&idx=1&sn=b1a7e73e4228d2008322d1ccd7671d89)

## 脚本

[https://github.com/3gpna/getNacosConfig](https://github.com/3gpna/getNacosConfig)

0 条评论

[发布投稿](https://xz.aliyun.com/news/release)

目录

- **
	前言
- **
	打法思路概述
- **
	企微corpid和corpsecret-人员信息&amp;钓鱼
	- [基础信息获取](https://xz.aliyun.com/news/)
	- [获取企微access\_token](https://xz.aliyun.com/news/)
- **
	先查看access\_token权限
- **
	获取企业微信API域名IP段
- **
	获取部门列表
- **
	获取部门成员/详情
- **
	获取加入企业二维码/创建成员
- **
	接管高权限应用
- **
	邮箱-敏感信息&amp;JWT伪造&amp;钓鱼
- **
	AK/SK-敏感信息&amp;云主机接管
- **
	如何防护
- **
	参考
- **
	脚本