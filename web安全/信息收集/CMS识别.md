# CMS识别

## cms简述

内容管理系统，也可以理解为模板。

cms管理系统识别：通过某项特征来识别，获得网站某个文件的MD5或者⽤正则表达式与字典里面的关键字进行匹配，如果匹配成功就说明这个站点是对于的CMS。

通俗来讲：CMS可以理解为CMS帮你把一个网站的程序部分的事全做完了；你要做的只是一个网站里面美工的部份。
可以用指纹查询来判断目标站点用的什么cms。

国内指纹识别：

云悉 http://www.yunsee.cn/

潮汐指纹 http://finger.tidesec.net/

who am i web指纹识别 http://whatweb.bugscaner.com/

也可以在github上下载使用cms识别工具。

## 如何识别CMS

- 有些网站会直接暴露cms,寻找关键词。

- robots协议
  robots协议也叫robots.txt（统一小写）是一种存放于网站根目录下的ASCII编码的文本文件，它通常告诉网络搜索引擎的漫游器（又称网络蜘蛛），此网站中的哪些内容是不应被搜索引擎的漫游器获取的，哪些是可以被漫游器获取的。
  使用方法：http://dedecms.com/robots.txt
  有时可以在其中了解到cms信息。

- cms指纹库：

  [cmsprint](https://github.com/Lucifer1993/cmsprint#cmsprint)：CMS和中间件指纹库

  查找到文件哈希值（MD5值）与网站对应文件对比，在文件路径下执行命令：
  
  ```shell
  certutil -hashfile 文件名
  ```
  
  

## 指纹识别的工具

1.kali中的whatweb

```linux 
whatweb -v www.baidu.com
```

2.浏览器插件

Wappalyzer：Find out what websites are built with - Wappalyzer 

whatruns:WhatRuns — Discover What Runs a Website

3.在线网站

TideFinger 潮汐指纹 TideFinger 潮汐指纹 (tidesec.com)

已支持识别的的cms列表,cms识别,源码识别,在线工具--BugScaner

云悉：https://www.yunsee.cn/

4.离线网站

御剑指纹扫描器（需要.NET Framework 3.5)

Test404轻量CMS指纹识别 v2.1

5.其他开源程序

Tuhinshubhra/CMSeeK: CMS Detection and Exploitation suite - Scan WordPress, Joomla, Drupal and over 180 other CMSs (github.com)

asp源码的cms网站源码中有.mdb文件存放网站与数据库交互的信息，包括数据库密码，信息，管理员账号，密码

大致方法
CMS网站渗透基本步骤
1、判断是什么类型的CMS
2、根据CMS类型查找已知的漏洞，进行敏感信息搜集（找到后台管理员入口，找到管理员账号、密码等）
3、提权（上传一句话木马，连接菜刀，得到数据库，远程控制）