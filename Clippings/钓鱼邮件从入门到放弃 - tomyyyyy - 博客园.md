---
title: "钓鱼邮件从入门到放弃 - tomyyyyy - 博客园"
source: "https://www.cnblogs.com/tomyyyyy/p/15648358.html#tid-ehdDtQ"
author:
published:
created: 2025-03-18
description: "钓鱼邮件从入门到放弃 钓鱼邮件，即一种伪造邮件，是指利用伪装的电邮，一般目的是用来欺骗收件人将账号、口令或密码等信息回复给指定的接收者，或附有超链接引导收件人连接到特制的钓鱼网站或者带毒网页，这些网页通常会伪装成和真实网站一样，如银行或理财的网页，令登录者信以为真，输入收集号码、账户名称及密码等而可"
tags:
  - "clippings"
---
钓鱼邮件，即一种伪造邮件，是指利用伪装的电邮，一般目的是用来欺骗收件人将账号、口令或密码等信息回复给指定的接收者，或附有超链接引导收件人连接到特制的钓鱼网站或者带毒网页，这些网页通常会伪装成和真实网站一样，如银行或理财的网页，令登录者信以为真，输入收集号码、账户名称及密码等而可能被盗取。

在大型企业边界安全做的越来越好的情况下，不管是 APT 攻击还是红蓝对抗演练，钓鱼和水坑攻击被越来越多的应用。

## 一、钓鱼邮件的基本概念

## 1.1 钓鱼邮件的伪造方式

### 1.1.1 购买域名搭建邮箱服务器

在真实的钓鱼场景中，一般是用一些近似的邮箱或邮件服务商注册的邮箱，在给甲方做钓鱼邮件演练的时候，如果预算高，会注册一个和甲方域名相似的域名，可以使用[urlcrazy](https://github.com/urbanadventurer/urlcrazy)工具自动寻找：

```sh
#工具在kali里面是预安装的
urlcrazy -i  weibo.com
```

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a4329d930ce3dc2b5245cf97ac54aa59.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a4329d930ce3dc2b5245cf97ac54aa59.png)

> valid为false的就是没有人使用的，我们就可以挑一个便宜的注册一个。然后利用域名搭建一个邮箱服务器，这里推荐ewomail，下文有具体的搭建方式。

### 1.1.2 伪造发件人

有时候我们找不到比较合适的域名，或者就是单纯的没有钱购买域名，这个时候我们就需要去伪造发件人。

我们先快速介绍一下邮件传递的过程，我们使用的邮箱客户端程序，网页版的qq邮箱、Foxmail、outlook统称为MUA（Mail User Agent）,他们只跟自己代理邮箱的smtp服务器（MTA）交互，而你使用qq邮箱向sina邮箱发送一封邮件，实际上最少经过4台机器，即：

> 发件MUA -> QQ MTA-> SINA MTA ->收件MUA。

然后我们可以仿造swaks直接伪装成一台邮件服务器（MTA），自己去进行身份的认证和邮件的发送

```sh
#telnet到邮件服务器的25端口
telnet smtp.mysun.org 25

# 用ehlo申明，表示自己需要身份验证，MTA将返回一些元信息
EHLO + 域名

#进行用户身份认证
auth login 
#然后服务器会返回base64加密过的user和pass字段，你也需要输入base64加密过的账号和密码

#发到本系统中域名下的账户可跳过身份认证。
#接下来进入邮件发送部分，使用mail from命令指定邮件由谁发出：
mail from: <test1@domain.com>

#递送给地址 test2@domain.com
rcpt to: <test2@domain.com>

#使用DATA命令指定邮件正文，以[.]结束
data

#结束连接
quit
```

然后我们可以发现上面的发送邮件流程中，涉及到域名后缀的有三个地方：

- 与接收者MTA打招呼时的EHLO命令
- 指定发送人的MAIL FROM命令
- 正文里的From头部

所以原理上，我们可以按照上面讲述原理的过程，直接与对端MTA交互，自定义以上三个部分，就可以模拟任何地址发送邮件了，因为MTA之间没有任何认证的过程。当然这个过程我们可以利用工具去替代，swaks就是这样的一款工具（该工具下文会介绍）。

## 1.2 三个邮件安全协议

### 1.2.1 SPF

SPF 是 Sender Policy Framework 的缩写，中文译为发送方策略框架。

主要作用是防止伪造邮件地址。

由于发送电子邮件的传统规范 - 1982年制定的《简单邮件传输协议（SMTP）》对发件人的邮件地址根本不进行认证，导致垃圾邮件制造者可以随意编造寄件人地址来发送垃圾信，而接收者则毫无办法，因为你无法判断收到的邮件到底是谁寄来的。

在SPF体系中，每个需要发送电子邮件的企业在其对外发布的DNS域名记录中，列出自己域名下需要发送邮件的所有IP地址段；而接收到邮件的服务器则根据邮件中发件人所属的域名，查找该企业发布的合法IP地址段，再对照发送邮件的机器是否属于这些地址段，就可以判别邮件是否伪造的。

查询是否开启 SPF

```cmd
#windows:
nslookup　-type=txt　qq.com
#linux:
dig　-t　txt　qq.com
```

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2007fc3bc280fac89384156e997ab521.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2007fc3bc280fac89384156e997ab521.png)

> spf可以配置四种规则：
> 
> "+" Pass（通过）
> 
> "-" Fail（拒绝）
> 
> "~" Soft Fail（软拒绝）
> 
> "?" Neutral（中立）

记录中有spf1说明使用了 spf ,这个时候所谓伪造邮件的时候是发送不到qq邮箱的。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/421395abe7d06aa2e70f80951e5b1937.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/421395abe7d06aa2e70f80951e5b1937.png)

再看看`spf.mail.qq.com`,它也是引用了其它域名的SPF策略，继续查询，可以看到有很多的ip段，我们在使用qq邮箱发邮件时，收件方会查询到这些IP，如果发件人的源IP不在这些IP列表里，则说明是一封伪造的邮件。

但是一般甲方是没有设置此协议的。如果甲方没有开启spf的话，我们就可以进行伪造

一般情况下没有SPF可以直接用[swaks](http://www.jetmore.org/john/code/swaks/)伪造，通过swaks修改发件邮箱进行发送即可。

```sh
-t  –to 目标地址 -t   test@test.com
-f –from 来源地址 (发件人)  -f "text<text@text.com>"
–protocol 设定协议(未测试)
--body "http://www.baidu.com"    //引号中的内容即为邮件正文；
--header "Subject:hello"   //邮件头信息，subject为邮件标题
-ehlo 伪造邮件ehlo头
--data ./Desktop/email.txt    //将正常源邮件的内容保存成TXT文件，再作为正常邮件发送；
```

### 1.2.2 DKIM

DKIM是一种在邮件中嵌入数字签名的技术，DKIM签名会对邮件中的部分内容进行HASH计算，最后在邮件头中增加一个DKIM-Signature头用于记录签名后的HASH值，接收方接收到邮件后，通过DNS查询得到公开密钥后进行验证， 验证不通过，则认为是垃圾邮件。

DKIM 的基本工作原理同样是基于传统的密钥认证方式，他会产生两组钥匙，公钥(public key)和私钥(private key)，公钥将会存放在 DNS 中，而私钥会存放在寄信服务器中。私钥会自动产生，并依附在邮件头中，发送到寄信者的服务器里。公钥则放在DNS服务器上，供自动获得。收信的服务器，将会收到夹带在邮件头中的私钥和在DNS上自己获取公钥，然后进行比对，比较寄信者的域名是否合法，如果不合法，则判定为垃圾邮件。

那么我们如何获取到发件方的DKIM的密钥呢？在DKIM中有一个选择器（selector）的概念，通过此功能可以为不同的用户提供不同的签名，想要找到发件方的DKIM服务器，首先需要找到selector，在邮件的DKIM头中，s字段的值即为DKIM的selector，获取到selector后，我们就可以在如下域名中找到密钥。

```none
selector._domainkey.xxxxxxxx.com
```

以阿里云的邮件为例，查看邮件源文件，通过s字段可以得到selector的值为s1024,因此阿里云的DKIM服务器域名为s1024.\_domainkey.aliyun.com，同样通过dig或者nslookup即可获取解密密钥。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d7e0f363263039068c3e4a46fd71aece.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d7e0f363263039068c3e4a46fd71aece.png)

### 1.2.3 DMARC

DMARC是2012年1月30号由Paypal，Google，微软，雅虎等开发的。DMARC是基于SPF和DKIM协议的可扩展电子邮件认证协议，通常情况下，它与SPF或DKIM结合使用，并告知收件方服务器当未通过SPF或 DKIM检测时该如何处理。

DMARC由Mail Sender方（域名拥有者Domain Owner）在DNS里声明自己采用该协议。当Mail Receiver方（其MTA需支持DMARC协议）收到该域发送过来的邮件时，则进行DMARC校验，若校验失败还需发送一封report到指定\[URI\]（常是一个邮箱地址）。

以阿里云为例，获取\_dmarc.aliyun.com的txt记录即可获取DMARC策略。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/de3796110a168fba54f1c77f1469140b.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/de3796110a168fba54f1c77f1469140b.png)

v=DMARC1：版本

p=：接收者根据域名所有者的要求制定的策略，取值和含义如下：

- quarantine : 邮件接收者将DMARC验证失败的邮件标记为可疑的。
- reject : 域名所有者希望邮件接收者将DMARC验证失败的邮件拒绝

pct：域名所有者邮件流中应用DMARC策略的消息百分比。

rua：用于接收消息反馈的邮箱。

## 1.3 钓鱼邮件的利用方式

### 1.3.1 链接钓鱼邮件

你收到一封系统升级的通知，点开链接立刻升级；或者账号被冻结，点击链接解冻等等，这类钓鱼邮件其实也很常见，攻击者在邮件内直接嵌入了钓鱼链接，点开链接是攻击者做的以假乱真的钓鱼网站，这类网站通常会要求用户输入账户信息之类以获取用户敏感信息；另一种链接指向的网页暗藏木马程序，用户点开的同时就中招了。

### 1.3.2 附件钓鱼邮件

顾名思义，这类钓鱼邮件的风险点在附件，很多人看到邮件中有附件时，就习惯性的点开查看。附件钓鱼邮件以Exe/Scr后缀的附件的风险程度最高，一般是病毒执行程序。其他常见的还有Html网页附件、Doc附件、Excel附件、PDF附件等。

### 1.3.3 鱼叉式钓鱼邮件

鱼叉式钓鱼邮件是一种只针对特定目标进行攻击的网络钓鱼攻击，由于鱼叉式网络钓鱼锁定之对象并非一般个人，而是特定公司、组织之成员，故受窃之资讯已非一般网络钓鱼所窃取之个人资料，而是其他高度敏感的资料。

### 1.3.4 BEC钓鱼邮件

BEC诈骗（Business Email Compromise），又被叫商务邮件诈骗，攻击者通过将邮件发件人伪装成你的领导、同事、商业伙伴，以此骗取商业信息、钱财、或者获取其他重要资料。

### 1.3.5 二维码钓鱼邮件

当用户处于内网，无法向外网发送信息时，骗子就会引诱收件人扫描二维码钓鱼邮件进行攻击~

## 二、钓鱼环境的搭建

## 2.1 网站克隆工具setoolkit

既然要发送钓鱼邮件了，那么我们就会需要搭建一个钓鱼网站，而这个一般是并不需要我们自己去从头搭建一个网站的，可以利用相应的网站克隆工具，这种工具也有不少，我就用kali自带的这个setoolkit举一个例子吧

我们直接在kali上打开setoolkit

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5a8361459981905c4ff60bdf201b2f59.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5a8361459981905c4ff60bdf201b2f59.png)

然后我们选择**Social-Engineering Attacks**，

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/420b6d2c044ecc3e61170c8dfc8f3435.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/420b6d2c044ecc3e61170c8dfc8f3435.png)

然后我们再选择选择**Website Attack Vectors**

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3e17d5e95c8f828ad17c21de462477b4.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3e17d5e95c8f828ad17c21de462477b4.png)

再选择**Credential Harvester Attack Method**

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9b30dea711dd22d0c957b9138746b012.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9b30dea711dd22d0c957b9138746b012.png)

最后选择**Site Cloner**，输入要克隆的网址，然后它克隆好之后就会返回克隆后的地址。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4abf546f3347b7a40103b9c516e3b3d7.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4abf546f3347b7a40103b9c516e3b3d7.png)

> 填写需要监听的IP，默认直接回车  
> 填写需要克隆的网站地址

然后我们就可以在kali看到输入的账号和密码

## 2.2 钓鱼平台gophish

当然现在还有一些专门用来钓鱼的平台工具，集成了邮件发送、网站克隆、邮件追踪、数据统计等功能，其中gophish是比较成熟的一个。

- Gophish项目地址：[https://github.com/gophish/gophish](https://github.com/gophish/gophish)
- Gophish官网地址：[https://getgophish.com/](https://getgophish.com/)

Gophish：开源网络钓鱼工具包

Gophish是为企业和渗透测试人员设计的开源网络钓鱼工具包。 它提供了快速，轻松地设置和执行网络钓鱼攻击以及安全意识培训的能力。

gophish自带web面板，对于邮件编辑、网站克隆、数据可视化、批量发送等功能的使用带来的巨大的便捷，并且在功能上实现分块，令钓鱼初学者能够更好理解钓鱼工作各部分的原理及运用。

### 2.2.1 安装教程

#### 2.2.1.1 Linux安装

```shell
#使用wget下载安装包
wget https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip

#解压
mkdir gophish
unzip gophish-v0.10.1-linux-64bit.zip -d ./gophish

#修改配置文件
cd gophish
vim config.json
```

若需要远程访问后台管理界面，将listen\_url修改为0.0.0.0:3333，端口可自定义。（这项主要针对于部署在服务器上，因为一般的Linux服务器都不会安装可视化桌面，因此需要本地远程访问部署在服务器上的gophish后台）  
如果仅通过本地访问，保持127.0.0.1:3333即可

```json
{
		//后台管理配置
        "admin_server": {
                "listen_url": "0.0.0.0:3333",  // 远程访问后台管理
                "use_tls": true,
                "cert_path": "gophish_admin.crt",
                "key_path": "gophish_admin.key"
        },
        "phish_server": {
                "listen_url": "0.0.0.0:80",
                "use_tls": false,
                "cert_path": "example.crt",
                "key_path": "example.key"
        },
        "db_name": "sqlite3",
        "db_path": "gophish.db",
        "migrations_prefix": "db/db_",
        "contact_address": "",
        "logging": {
                "filename": "",
                "level": ""
        }
}
```

然后运行gophish

 ```shell
./gophish
```

**访问后台管理系统：**  
本地打开浏览器，访问`https://ip:3333/` （注意使用https协议）  
可能会提示证书不正确，依次点击 `高级` — `继续转到页面` ，输入默认账密进行登录：`admin/gophish`

> 最新版本的gophsih(v0.11.0)删除了默认密码“ gophish”。取而代之的是，在首次启动Gophish时会随机生成一个初始密码并将其打印在终端中

#### 2.2.1.2windows安装

关于gophish的Windows平台安装包仅有64位的，如果是32的win可能会不兼容（未测试），不过现在的PC大多数都是64位了，因此还是具有普遍性

下载安装包：  
[gophish-v0.11.0-windows-64](https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-windows-64bit.zip)  
下载完成后，使用压缩工具解压到文件夹中

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/577fb2bba0df37663c4327572a3f8c9c.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/577fb2bba0df37663c4327572a3f8c9c.png)

（可选）修改配置文件：  
如果有远程访问gophish的后台管理系统的需求，则修改配置文件。具体参考Linux下修改gophish配置文件  
使用编辑器打开config.json文件，修改字段listen\_url的值为0.0.0.0:3333 (默认为127.0.0.1:3333，仅本地访问)，端口可自定义

运行gophish：  
双击目录下的gophish.exe启动gophish，这里需要保持小黑框不被关闭，关闭则脚本终止（如果看着碍眼可以自行搜索后台运行的方法）

**访问后台管理系统：**  
本地浏览器访问：`https://127.0.0.1:3333` 或 `https://远程ip:3333` (注意使用https协议)  
输入默认账密进行登录：`admin/gophish`

> 最新版本的gophsih(v0.11.0)删除了默认密码“ gophish”。取而代之的是，在首次启动Gophish时会随机生成一个初始密码并将其打印在终端中

### 2.2.2功能介绍

进入后台后，左边的栏目即代表各个功能，分别是Dashboard仪表板 、Campaigns钓鱼事件 、Users & Groups用户和组 、Email Templates邮件模板 、Landing Pages钓鱼页面 、Sending Profiles发件策略六大功能  
由于实际使用中并不是按照该顺序来配置各个功能，因此下面通过实际使用顺序来详细介绍各功能的使用方法

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a342c2435106b7b027139d3b2da9e772.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a342c2435106b7b027139d3b2da9e772.png)

#### 2.2.2.1 Sending Profiles

作用：设置发件人的邮箱  
点击New Profile新建一个策略，依次来填写各个字段。（可选）Email Headers 是自定义邮件头字段，例如邮件头的X-Mailer字段，若不修改此字段的值，通过gophish发出的邮件，其邮件头的X-Mailer的值默认为gophish。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d804adfa535dfe9d54b587451e17ffd9.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d804adfa535dfe9d54b587451e17ffd9.png)

Send Test Email测试邮箱验证是否通过

password需要填写邮箱的授权码

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0e9778642d6a709362b19317f8ca750e.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0e9778642d6a709362b19317f8ca750e.png)

> 可以看到发送成功，成功送达目标邮箱  
> 还可以伪造任意x-mailer头 (如果不设置的话默认是gophish)  
> x-mailer头表示邮件从哪个客户端发出来的

```none
wlmhymexqpljchcb
```

#### 2.2.2.2 Users & Groups

作用：设置收件人的邮箱

这个是用来设置攻击目标，即你要发送邮件的收件人地址，可以导入csv文档

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4b512043fa22b4c7a7c161e82106d96c.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4b512043fa22b4c7a7c161e82106d96c.png)

#### 2.2.2.3 Email Templates

作用：创建钓鱼邮件模版

设置好钓鱼邮件的发送方和接收方，便可以设置钓鱼邮件的模板。gophish支持手动编辑生成钓鱼邮件，也支持导入现有邮件内容。现有邮件eml导入:(QQ邮件)下载eml文件，打开导入即可。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4825bc458cfe09a52124eaadc5c048ef.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4825bc458cfe09a52124eaadc5c048ef.png)

首先我先导出了qq邮件的一个eml文件保存到本地，然后用记事本打开

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ac339009042d0adfe1c92d5637b8577a.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ac339009042d0adfe1c92d5637b8577a.png)

然后点击import email导入相应的模板

#### 2.2.2.4 Landing Page

作用：伪造钓鱼页面  
gophish支持手动编辑生成钓鱼邮件，Import Site填写被伪造网站的URL即可通过互联网自动抓取被伪造网站的前端代码。也支持导入现有邮件内容，配置时勾选捕获提交的数据和捕获密码，服务器会把相关数据保存下来。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/29788baff31d5a394ba174c6d7ec508e.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/29788baff31d5a394ba174c6d7ec508e.png)

[https://personalbank.cib.com.cn/pers/main/login.do](https://personalbank.cib.com.cn/pers/main/login.do)

#### 2.2.2.5 Campaign

作用：发送钓鱼邮件  
填写好攻击的名称，选择钓鱼邮件模板，选择相应的钓鱼页面，目标邮箱，和发送邮箱。需要填上GoPhish服务端的地址，在钓鱼开始前，这个地址会将先前钓鱼邮件模板中的链接替换。

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5f72bb6b3e1c352d8bb25817fbadfac6.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5f72bb6b3e1c352d8bb25817fbadfac6.png)

## 2.3 利用EwoMail搭建邮箱服务器

- [官方网站](http://www.ewomail.com/list-9.html)
- [github项目地址](https://github.com/gyxuehu/EwoMail)
- [官方docker教程](https://hub.docker.com/r/bestwu/ewomail/)

### 2.3.1 利用docker安装ewomail

然后在vps中安装EwoMail，ewomail要求 centos ，但是此时并不需要重装系统，使用docker 进行搭建即可。

```sh
docker search ewomail
docker pull bestwu/ewomail

#把命令中mail.ewomail.com 替换成你自己的域名 mail.*.格式
docker run  -d -h mail.tomyyyyy.xyz --restart=always \
  -p 25:25 \
  -p 109:109 \
  -p 110:110 \
  -p 143:143 \
  -p 465:465 \
  -p 587:587 \
  -p 993:993 \
  -p 995:995  \
  -p 80:80 \
  -p 8080:8080 \
  -v \`pwd\`/mysql/:/ewomail/mysql/data/ \
  -v \`pwd\`/vmail/:/ewomail/mail/ \
  -v \`pwd\`/ssl/certs/:/etc/ssl/certs/ \
  -v \`pwd\`/ssl/private/:/etc/ssl/private/ \
  -v \`pwd\`/rainloop:/ewomail/www/rainloop/data \
  -v \`pwd\`/ssl/dkim/:/ewomail/dkim/ \
  --name ewomail bestwu/ewomailserver
```

### 2.3.2 centos直接搭建

如果你使用的是centos系统，可以直接按照下面的方式进行安装：

```sh
git clone https://github.com/gyxuehu/EwoMail.git
cd ./EwoMail/install
##需要输入一个邮箱域名，不需要前缀，列如下面的ewomail.cn
#国外网络 请在安装域名后面加空格加en，例如 sh ./start.sh ewomail.cn en
sh ./start.sh ewomail.cn
```

查看安装的域名和数据库密码

```sh
cat /ewomail/config.ini
```

### 2.3.3 安装后的常规配置

安装好之后，就会出现下面这些平台

```none
邮箱管理后台：[http://IP:8010](http://ip:8010/) （默认账号admin，密码ewomail123）
ssl端口 [https://IP:7010](https://ip:7010/)

web邮件系统：[http://IP:8000](http://ip:8000/)
ssl端口 [https://IP:7000](https://ip:7000/)

域名解析完成后，可以用子域名访问，例如下面
[http://mail.xxx.com:8000](http://mail.xxx.com:8000/) (http)
[https://mail.xxx.com:7000](https://mail.xxx.com:7000/) (ssl)
```

然后我们就可以通过`http://域名:8080`,访问管理平台，利用admin/ewomail123登录

然后配置邮箱系统设置

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f1afcc3ab80e94bb8d89013e4c18c3af.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f1afcc3ab80e94bb8d89013e4c18c3af.png)

## 2.4 利用腾讯域名邮箱

### 2.4.1 注册并配置腾讯企业邮箱

利用[https://mail.qq.com/cgi-bin/loginpage?t=dm\_loginpage&c=1](https://mail.qq.com/cgi-bin/loginpage?t=dm_loginpage&c=1),该链接直接创建你的域名邮箱

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c844affd769190a807e2a1a84b9925cd.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c844affd769190a807e2a1a84b9925cd.png)

如果你的域名是在腾讯注册的话，可以直接在域名管理这里进行开通即可，更加方便

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e70d7bfef7ecfea91e90e3736f68d344.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e70d7bfef7ecfea91e90e3736f68d344.png)

然后按照他的流程完成企业邮箱的开通即可，开通之后会进入到这样的页面

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dc677b093f71200d9bcc7861e8c2955f.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dc677b093f71200d9bcc7861e8c2955f.png)

然后点击右侧的链接进行激活，填写个人信息即可，激活成功之后会进入到企业微信的后台管理平台了

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e944a6be2af87012886885a6b0067551.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e944a6be2af87012886885a6b0067551.png)

然后我们就可以在通讯录这个地方，添加，修改，或者删除我们的邮箱，同时可以在手机或者电脑的企业邮箱中收发我们的邮件了。

### 2.4.2 gophish配置腾讯企业邮箱

首先根据腾讯企业邮箱的[官方文档](https://service.exmail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001254),发送邮件服务器我们可以配置为smtp协议

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3819822c590db7a7dc944997bba1ea6e.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3819822c590db7a7dc944997bba1ea6e.png)

## 三、各种方式绕过邮箱的安全机制

## 3.1 邮件伪造工具

在线伪造： [http://tool.chacuo.net/mailanonymous](http://tool.chacuo.net/mailanonymous)

在线匿名邮件： [http://tool.chacuo.net/mailsend](http://tool.chacuo.net/mailsend)

在线邮件伪造：[https://emkei.cz/](https://emkei.cz/)

github邮件伪造工具：[https://github.com/Macr0phag3/email\_hack](https://github.com/Macr0phag3/email_hack)

## 3.2 利用swaks发送邮件

swaks（SWiss Army Knife Smtp），SMTP瑞士军刀Swaks是由John Jetmore编写和维护的一种功能强大，灵活，可脚本化，面向事务的SMTP测试工具。可向任意目标发送任意内容的邮件  
kali自带  
官方网站：[http://www.jetmore.org/john/code/swaks/](http://www.jetmore.org/john/code/swaks/)

### 3.2.1 基本使用

```sh
#测试邮箱连通性可以成功发送。
swaks -to xxx@qq.com

#可以添加的参数
-t  –to 目标地址 -t   test@test.com
-f –from 来源地址 (发件人)  -f "text<text@text.com>"
–protocol 设定协议(未测试)
--body "http://www.baidu.com"    //引号中的内容即为邮件正文；
--header "Subject:hello"   //邮件头信息，subject为邮件标题
-ehlo 伪造邮件ehlo头
--data ./Desktop/email.txt    //将正常源邮件的内容保存成TXT文件，再作为正常邮件发送；
```

或者直接利用写好的邮件内容(记得删除邮件头部的from和to)

```sh
swaks --data readmail.txt --header "Subject:标题" --to xxxxxxxx@qq.com --from "qianxiao996@126.com" --attach qq.txt   #qq.txt 为附件。
```

## 3.3 利用IDN伪造域名(又称Punycode钓鱼攻击)

什么是IDN？是指在域名中包含至少一个特殊语言字母的域名，特殊语言包括中文、法文、拉丁文等等非英语的语言。在DNS系统工作中，这种域名会被编码成ASCII字符串，并通过Punycode进行翻译。Punycode是一个根据RFC 3492标准而制定的编码系统,主要用于把域名从地方语言所采用的Unicode编码转换成为可用于DNS系统的编码。

我们用`bücher.example`这个域名来说明它的编码过程。`bücher`部分通过Punycode编码转换为`bcher-kva`，然后以`xn--`作为前缀，因此DNS记录将会变成`xn--bcher-kva.example`。浏览器里面会自动对IDN域名进行Punycode转码，Chrome团队将修补程序包含在Chrome 58之后的版本中，但Firefox仍然未解决这个问题，因为这一问题的具体范围仍然未定。

之前有人使用西里尔语中的`'a'`字符构建一个`apple.com`的同态域名，最后浏览器只显示了编码后的域名。访问[https://www.apple.com](https://www.xn--80ak6aa92e.com/),实际上该链接地址是`https://www.xn--80ak6aa92e.com/`，但是浏览器显示的却是`https://www.apple.com`,

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e1916c56e9150ef9f9e0e817d878dd75.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e1916c56e9150ef9f9e0e817d878dd75.png)

而作为邮箱服务器，对IDN的检测就更加的少了，而且检测的话也没有什么比较有效的方式，因此如果我们利用IDN域名作为我们的邮箱服务器的话，简直可以以假乱真。

如果我们想要去去注册一个这样子的域名的话，首先要先找到对应字母的替换形式，可以找到西方其它国家的一些字母进行替换，例如上文所提到的西里尔语，我们可以直接复制相应的字母到我们的域名中，例如[西里尔语中的一些字母](https://zh.wikipedia-on-ipfs.org/wiki/%E6%B1%89%E8%AF%AD%E8%A5%BF%E9%87%8C%E5%B0%94%E5%AD%97%E6%AF%8D%E8%BD%AC%E5%86%99%E7%B3%BB%E7%BB%9F)，[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9e38699174402e71cdd4a66ea7b8f846.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9e38699174402e71cdd4a66ea7b8f846.png)

我们可以直接复制他们中和英文类似的字母，例如`a、e、o`,之类的，然后利用IDN转码工具查看一下转码之后的效果

- [https://idn.bmcx.com/](https://idn.bmcx.com/)
- [https://www.cha127.com/cndm/](https://www.cha127.com/cndm/)

像是之前的`www.apple.com`,如果我们只替换字母`a`的话，就是这样的

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4da8db00b87c54acebd2deeed4b5bef9.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4da8db00b87c54acebd2deeed4b5bef9.png)

然后我们就可以到可以购买idn域名的服务厂商去购买域名了，例如https://sg.godaddy.com/，这种全球性的域名厂商，我们可以直接购买带有西里尔字母的域名就可以了

## 3.4 绕过邮件网关域名检测

一些公司的邮件服务都设置的有邮件网关的检测，或者是其它的安全防护设备。而这些防护设备的原理其实大都是利用的黑白名单，既然是黑白名单，那么一定是人为设置的，而人为设置难免会有设置疏忽的地方，这时候需要我们对目标多做一些域名收集，寻找一些没有SPF记录的域名，比如xxxx.com、xxxx.cn或者一些三级域名，甚至可以尝试任意伪造一个不存在的三级域名，比如1111.xxxx.com。

同时需要多收集一些正常的发件邮箱，比如公司IT的邮箱`it@xxxx.com`，通常此类业务邮箱是在白名单中的，因此在伪造发件人时，可选择一些白名单关键字来尝试匹配防护设备的白名单正则策略，比如`itit@xxxx.com`、`aa@it.xxxx.com`等，当然这只是一个思路，具体能不能成功还需要在实战中多尝试。

## 四、钓鱼邮件主题参考

## 4.1 重置密码

攻击者伪装成管理员，让受害者点击钓鱼邮件中的链接来修改各种各样的密码！如下：

```none
xxx,您好
  我是xx部门的信息，我的oa系统账号密码忘记了，麻烦帮我重置一下我的oa帐号密码，然后将新的账号密码发到这个邮箱，十分感谢！
```
```none
大家好：
  近期由于我们公司的邮箱密码泄露，为防止不法分子利用，影响到我们的数据安全，各位员工的密码均需要及时更新修改，在收到邮件的第一时间，请登录如下平台，立即修改自己邮箱的账号密码。
```

## 4.2 账号解冻

攻击者伪装成系统管理员，让受害者点击钓鱼邮件中的链接解冻账号。

```none
您好！
  上网行为管理近几日发现您的账号存在异常行为，为了防止您的账号被不法分子盗取，我们暂时将您的账号进行了冻结，如果不是您本人的操作的话，可以点击下面的链接进行解冻。
```

## 4.3 升级补丁

攻击者伪装成系统管理员，让受害者点击钓鱼邮件中的附件 升级补丁。

```none
各位同事，大家好！
  近日微软发布了本月安全更新补丁，其中包含一个RDP（远程桌面服务）远程代码执行漏洞的补丁更新，对应CVE编号为CVE-2019-0708,现需要所有员工的电脑都打上补丁，漏洞补丁直接下载附件即可
```

## 4.4 信息收集

利用疫情，让受害者点击钓鱼邮件中的链接反馈相关信息。

```none
各位同事，大家好：
近日，在xx的来往人员中进行核算检查时，又发现了核酸检测呈阳性的人员。
    为了更好的配合政府的疫情防控工作的顺利展开，我们公司主动承担了内部人员的信息收集工作，需要员工填写一些个人基本信息、核酸检测信息、行程信息等信息。 
可以扫描二维码填写基本信息
```
```none
各位同事，大家好：
    为落实“xx市疫情防控策略”的工作要求，推进疫情防控相关的数据共享，提升排查效率。我司开发人员设计出一个“疫情防控信息公示系统”，将个人基本信息，行程信息，温度信息，是否打疫苗等相关信息整合到一个公示平台之上，员工可以在线查看有关信息，享受”一站式”服务，同时推进疫情防控工作的展开。
目前该系统处于推广阶段，开发人员将根据用户体验，不断优化业务流程，继续完善平台共功能，欢迎各位领导、同事提出宝贵意见。
平台网址:http:xx.xx.com
```

## 4.5 节假日礼包

```none
各位同事好：    
    春节临近，旧的一年即将过去，崭新的一年即将到来，我们将为各位员工准备春节大礼包，现在需要填写一下家庭住址等基本信息
```
  

\_\_EOF\_\_

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/deeb4e4f0259239fdbbe1b92ebfd1543.png)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/deeb4e4f0259239fdbbe1b92ebfd1543.png)

- **本文作者：** [tomyyyyy](https://www.cnblogs.com/tomyyyyy)
- **本文链接：** [https://www.cnblogs.com/tomyyyyy/p/15648358.html#tid-ehdDtQ](https://www.cnblogs.com/tomyyyyy/p/15648358.html#tid-ehdDtQ)
- **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/tomyyyyy)我。
- **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-nd/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
- **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](https://www.cnblogs.com/tomyyyyy/p/)】**一下。

posted @   [tomyyyyy](https://www.cnblogs.com/tomyyyyy)  阅读(5189)  评论(0)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=15648358)  [收藏](https://www.cnblogs.com/tomyyyyy/p/)  [举报](https://www.cnblogs.com/tomyyyyy/p/)

[升级成为园子VIP会员](https://cnblogs.vip/)

编辑 预览

56935e0b-d73d-42a8-63f6-08da5c229458

[不改了](https://www.cnblogs.com/tomyyyyy/p/) [退出](https://www.cnblogs.com/tomyyyyy/p/) [订阅评论](https://www.cnblogs.com/tomyyyyy/p/ "订阅后有新评论时会邮件通知您")

\[Ctrl+Enter快捷键提交\]

[【推荐】还在用 ECharts 开发大屏？试试这款永久免费的开源 BI 工具！](https://dataease.cn/?utm_source=cnblogs)  
[【推荐】国内首个AI IDE，深度理解中文开发场景，立即下载体验Trae](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)  
[【推荐】编程新体验，更懂你的AI，立即体验豆包MarsCode编程助手](https://www.marscode.cn/?utm_source=advertising&utm_medium=cnblogs.com_ug_cpa&utm_term=hw_marscode_cnblogs&utm_content=home)  
[【推荐】抖音旗下AI助手豆包，你的智能百科全书，全免费不限次数](https://www.doubao.com/?channel=cnblogs&source=hw_db_cnblogs)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  

[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e9fdd2be0defa817090d7885a26762a7.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e9fdd2be0defa817090d7885a26762a7.jpg)

**编辑推荐：**  
· [Linux系列：如何调试 malloc 的底层源码](https://www.cnblogs.com/huangxincheng/p/18750484)  
· [AI与.NET技术实操系列：基于图像分类模型对图像进行分类](https://www.cnblogs.com/code-daily/p/18764803)  
· [go语言实现终端里的倒计时](https://www.cnblogs.com/apocelipes/p/18753961)  
· [如何编写易于单元测试的代码](https://www.cnblogs.com/CareySon/p/18762884/how_to_write_testable_code)  
· [10年+ .NET Coder 心语，封装的思维：从隐藏、稳定开始理解其本质意义](https://www.cnblogs.com/code-daily/p/18769455)  

**阅读排行：**  
· [25岁的心里话](https://www.cnblogs.com/nanwanqiu/p/18775135)  
· [因为Apifox不支持离线，我果断选择了Apipost！](https://www.cnblogs.com/ifmeme/p/18776889)  
· [零经验选手，Compose 一天开发一款小游戏！](https://www.cnblogs.com/wavky/p/18775843)  
· [Trae 开发工具与使用技巧](https://www.cnblogs.com/wgjava/p/18776746)  
· [通过 API 将Deepseek响应流式内容输出到前端](https://www.cnblogs.com/zhouyunbaosujina/p/18776269)  

点击右上角即可分享

![微信分享提示](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/555e49d6a3ba2183a3b8a2b6083355df.gif)