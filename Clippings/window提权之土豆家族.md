---
title: "window提权之土豆家族"
source: "https://blog.csdn.net/qq_42699326/article/details/144334216"
author:
  - "[[qq_42699326]]"
published: 2024-12-08
created: 2025-06-09
description: "文章浏览阅读865次，点赞11次，收藏4次。适用系统: Windows Server 2008/2012/2016，Windows 7/10。适用系统: Windows Server 2008/2012，Windows 7/10。适用系统: Windows 7/10，Windows Server 2016/2019。适用系统: Windows Server 2016/2019，Windows 10。适用系统: Windows Server 2019，Windows 10（新版）。机制: 通过混合利用 RPC 和 Named Pipe，达到提权目的。_sweetpotato提权"
tags:
  - "clippings"
---
“土豆家族”是指一系列利用 Windows 系统漏洞实现 [提权](https://so.csdn.net/so/search?q=%E6%8F%90%E6%9D%83&spm=1001.2101.3001.7020) 的工具或方法，起源于 JuicyPotato。这些工具大多利用 COM 对象和服务中的权限提升漏洞，主要用于在 Windows 环境中从中低权限（如普通用户）提权到 SYSTEM 权限。以下是土豆家族中的主要工具和其提权类型：

  
\---

1\. JuicyPotato

功能: 基于 [COM 对象](https://so.csdn.net/so/search?q=COM%20%E5%AF%B9%E8%B1%A1&spm=1001.2101.3001.7020) 的 DCOM 提权。

适用系统: Windows Server 2008/2012/2016，Windows 7/10。

机制: 利用 NT AUTHORITY\\LOCAL SERVICE 或 NETWORK SERVICE 的特权，通过指定的 CLSID 和 COM 服务劫持 SYSTEM 权限。

\---

2\. RottenPotato

功能: 利用 NTLM 反射实现提权。

适用系统: Windows 7/10。

机制: 使用 Token Impersonation 技术，通过恶意 COM 对象劫持 SYSTEM Token。

\---

3\. SweetPotato

功能: 针对 JuicyPotato 不适用的现代系统进行提权。

适用系统: Windows Server 2019，Windows 10（新版）。

机制: 同样基于 COM 对象的漏洞，但兼容性更强，适配新系统。

\---

4\. PrintSpoofer

功能: 针对 Windows 打印服务的漏洞进行提权。

适用系统: Windows Server 2016/2019，Windows 10。

机制: 滥用打印假脱机程序（Print Spooler）的权限进行 Token Impersonation。

\---

5\. RoguePotato

功能: 替代 JuicyPotato 的一种新方法。

适用系统: Windows Server 2019，Windows 10（新版）。

机制: 通过伪造的远程服务与 NTLM 身份验证劫持 SYSTEM Token。

\---

6\. HotPotato

功能: 利用 WPAD 代理机制漏洞进行提权。

适用系统: Windows 7/10。

机制: 滥用 WPAD 自动代理设置，通过中间人攻击实现 NTLM 身份验证劫持。

\---

7\. PotatoX

功能: JuicyPotato 的改良版本。

适用系统: Windows 10，Windows Server 2019。

机制: 针对 JuicyPotato 的问题进行了修复，提高兼容性。

\---

8\. BloodyPotato

功能: 使用 Named Pipe 的一种提权方法。

适用系统: Windows Server 2008/2012，Windows 7/10。

机制: 通过混合利用 RPC 和 Named Pipe，达到提权目的。

\---

9\. GhostPotato

功能: 一种基于 NTLM Relay 的提权方法。

适用系统: Windows 10，Windows Server 2019。

机制: 通过绕过 NTLM 身份验证实现 SYSTEM 权限提升。

\---

10\. EvilPotato

功能: 专门针对 Terminal Services 提权。

适用系统: Windows 7/10，Windows Server 2016/2019。

机制: 滥用 RDP 相关服务权限，提升至 SYSTEM。

\---

总结

土豆家族工具以 JuicyPotato 为核心衍生出多种版本，根据 Windows 系统的不同版本和安全更新，开发了适用于新环境的工具。例如：

1\. 经典版本: JuicyPotato、RottenPotato、HotPotato。

  
2\. 现代兼容: SweetPotato、RoguePotato、PrintSpoofer。

  
3\. 特定场景: EvilPotato、GhostPotato。

每个工具的具体选择取决于目标系统的版本和环境配置。

打赏作者

¥1 ¥2 ¥4 ¥6 ¥10 ¥20

扫码支付： ¥1

获取中

扫码支付

您的余额不足，请更换扫码支付或 [充值](https://i.csdn.net/#/wallet/balance/recharge?utm_source=RewardVip)

打赏作者

实付 元

[使用余额支付](https://blog.csdn.net/qq_42699326/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4b5d638b57b796bf4228e36a12fd3244.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6ddf638effe61d927f03a0789e73f41e.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4832e9a5e9fe3a702f3d81b1ce7c9ca3.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e311c5ce3170b9f4471a19010a5c3bd8.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部