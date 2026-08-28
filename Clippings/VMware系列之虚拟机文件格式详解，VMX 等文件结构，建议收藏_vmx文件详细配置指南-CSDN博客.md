---
title: "VMware系列之虚拟机文件格式详解，VMX 等文件结构，建议收藏_vmx文件详细配置指南-CSDN博客"
source: "https://blog.csdn.net/youmangu/article/details/134416573"
author:
  - "[[成就一亿技术人!]]"
  - "[[hope_wisdom 发出的红包]]"
published:
created: 2025-08-27
description: "文章浏览阅读8.7k次，点赞4次，收藏26次。本文详细介绍了VMwareWorkstation中虚拟机的9种主要文件类型，包括.vmx配置文件、vmdk磁盘文件、.lck锁定文件、.log日志文件、.vmxf辅助配置等，帮助用户理解虚拟机内部机制并处理常见问题。"
tags:
  - "clippings"
---
## VMware系列之虚拟机文件格式详解，VMX 等文件结构，建议收藏

最新推荐文章于 2025-07-07 09:03:41 发布

转载 于 2023-11-15 11:40:56 发布 · 8.7k 阅读 · 26 ·

CC 4.0 BY-SA版权

原文链接： [https://baijiahao.baidu.com/s?id=1710247089629246016&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1710247089629246016&wfr=spider&for=pc)

本文详细介绍了VMwareWorkstation中虚拟机的9种主要文件类型，包括.vmx配置文件、vmdk磁盘文件、.lck锁定文件、.log日志文件、.vmxf辅助配置等，帮助用户理解虚拟机内部机制并处理常见问题。

摘要生成于 [C知道](https://ai.csdn.net/?utm_source=cknow_pc_ai_abstract) ，由 DeepSeek-R1 满血版支持， [前往体验 >](https://ai.csdn.net/?utm_source=cknow_pc_ai_abstract)

来自： [https://baijiahao.baidu.com/s?id=1710247089629246016&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1710247089629246016&wfr=spider&for=pc "https://baijiahao.baidu.com/s?id=1710247089629246016&wfr=spider&for=pc")

这篇文章和大家一起研究一下VMware中的这个“一”。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/99ec817b6196cf726e0aa6254a3a0bfe.jpeg)

在VMware Workstation 中创建虚拟机后，会生成一系列文件。这些文件的 [文件格式](https://so.csdn.net/so/search?q=%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F&spm=1001.2101.3001.7020) 是怎样的？都有何作用？

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/53bb9496a23d248dd0cc7528a26fb0bd.jpeg)

从最核心、最重要的几个文件说起。如果您正在使用VMware，那么耐心看完对您一定大有裨益。

[VMware虚拟机](https://so.csdn.net/so/search?q=VMware%E8%99%9A%E6%8B%9F%E6%9C%BA&spm=1001.2101.3001.7020) 有9种类型的文件，对于刚安装系统的虚拟机，默认只存在6种。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f4859bfe2ba7bf2e4c405f4bede10cfe.jpeg)

熟悉这9种文件，才算真正了解VMware，当出现问题时，手中有术，心中不慌。

如果嫌多，那就学习前三种。还嫌多？则非vmdk莫属。还嫌多？？？好吧，可以考虑改行了。

#### 一、.vmx

VMware虚拟机的配置文件。通常打开虚拟机时，打开的就是这个文件以。反之，可以通过编辑它以实现修改某种配置，当需要手动更改配置文件以达到对虚拟机硬件方面的更改时，可使用文本编辑器进行编辑。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a14a47e3b6df3e8390ae66f6f6b12f46.jpeg)

如果在 Linux 环境下使用VM虚拟机，这个配置文件的扩展名则是.cfg。

#### 二、.vmdk or -s###.vmdk

VMware虚拟机的磁盘文件。虚拟机的操作系统和所有文件都在这个文件中，它就相当于我们电脑主机中的硬盘。

一台虚拟机可以由一个或多个虚拟磁盘文件组成。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9141dcb2247a5b74da49d9a3674d12b2.jpeg)

如果在新建虚拟机时指定虚拟机磁盘文件为单独一个文件时，系统将只创建一个.vmdk文件，该文件包括了虚拟机磁盘分区信息，以及虚拟机磁盘的所有数据。 随着数据写入虚拟磁盘，虚拟磁盘文件将变大，但始终只有这一个磁盘文件。

**以单文件方式存储的vmdk是二进制文件。**

如果在新建虚拟机时指定创建多个磁盘文件的话，系统将创建一个<vmname>.vmdk文件和多个<vmname>-s###.vmdk文件（s###为磁盘文件编号）， 其中<vmname>.vmdk文件只包括磁盘分区信息，而<vmname>-s###.vmdk文件则存储磁盘数据信息。 随着数据写入某个虚拟磁盘文件，该虚拟磁盘文件将变大，直到文件大小为2GB， 然后新的数据将写入到 其他 s###编号的磁盘文件中。

**以多文件方式存储的vmdk是ASCII码文件，可以用记事本打开。**

如果虚拟机是直接使用物理硬盘而不是虚拟磁盘的话，虚拟磁盘文件则保存着虚拟机能够访问的分区信息。

当虚拟机出问题了，我们又想利用里面的数据，怎么打开它呢？

- **方法1：虚拟机映射**

利用vmware虚拟机软件的映射打开.vmdk虚拟磁盘文件。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d388ae2551396ec9f498876da0dcbdc3.jpeg)

- **方法2：借助第三方软件**

借助diskgenius等第三方磁盘管理软件，可以打开vmware的虚拟磁盘，进行虚拟机文件文件交互。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/43c24bf83db9f3ac87cde9980b6116a4.jpeg)

#### 三、.lck（动态文件，打开时有，关闭时无）

这个是目录，其作用是用于锁定vmx的文件夹，在虚拟机开机的时候，就会自动创建以.lck结尾的目录，虚拟机关机后会自动删除。而当虚拟机异常退出时，则不会删除.lck结尾的文件，用于保护虚拟磁盘文件数据。

对于出现虚拟机正在被使用，获取所有权的报错，就是这个\*.lck结尾目录搞的鬼，将其删除后就可正常开启虚拟机了。大多数情况下都会解决。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/940b3b9b02572034baf210b3ff1f0296.jpeg)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/643ce3ce9b8c491139d7440dc1d05216.jpeg)

#### 四、.log

这种log文件会有很多，vmware-0.log、vmware-1.log等等。用来记录vmware工作日志。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/122d681391dae73e259841d8a74eb0be.jpeg)

#### 五、.vmxf

该文件为虚拟机组team中的虚拟机的辅助配置文件。

#### 六、.vmsd

该文件储存了虚拟机快照的相关信息和元数据，并将vmsn和vmdk绑定在一起，也就是说记录里vmsn信息和vmdk信息。

以文本文件的方式记录，可以用记事本打开。

#### 七、.nvram

虚拟出来的BIOS，一般不能修改。

八、其他动态存在的文件

- **vmem**

表示虚拟内存文件，与pagefile.sys（亦称分页文件）同。当虚拟系统执行关机操作后，vmem文件消失，但挂起关闭时，该文件依然操作。

- **vmsn**

虚拟机快照文件，不创建默认不存在。

当虚拟机建立快照时，就会自动创建该文件。有几个快照就会有几个此类文件。这是虚拟机快照的状态信息文件，它记录了在建立快照时虚拟机的状态信息。##为数字编号，根据快照数量自动增加。

**Snapshotxxx.vmsn文件和Snapshotxxx.vmem文件是成对出现的。两者之间有依赖关系。**

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/37ca20226927555f0cc30ebf5f843f95.jpeg)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/55d2ae451ce5941b53745b235b330fe1.jpeg)

以上就是IT悟道总结的虚拟机文件格式，您的 **转发、分享** 可以帮助到更多有需求的人。如果您还有其他补充，欢迎在下方留言，我们一起交流学习~

  

显示推荐内容

实付 元

[使用余额支付](https://blog.csdn.net/youmangu/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4b5d638b57b796bf4228e36a12fd3244.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) ![程序员都在用的中文IT技术交流社区](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6ddf638effe61d927f03a0789e73f41e.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4832e9a5e9fe3a702f3d81b1ce7c9ca3.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e311c5ce3170b9f4471a19010a5c3bd8.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

返回顶部

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/99ec817b6196cf726e0aa6254a3a0bfe.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/53bb9496a23d248dd0cc7528a26fb0bd.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f4859bfe2ba7bf2e4c405f4bede10cfe.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a14a47e3b6df3e8390ae66f6f6b12f46.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9141dcb2247a5b74da49d9a3674d12b2.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d388ae2551396ec9f498876da0dcbdc3.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/43c24bf83db9f3ac87cde9980b6116a4.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/940b3b9b02572034baf210b3ff1f0296.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/643ce3ce9b8c491139d7440dc1d05216.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/122d681391dae73e259841d8a74eb0be.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/37ca20226927555f0cc30ebf5f843f95.jpeg) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/55d2ae451ce5941b53745b235b330fe1.jpeg)