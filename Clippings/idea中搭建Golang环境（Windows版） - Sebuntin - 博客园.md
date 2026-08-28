---
title: "idea中搭建Golang环境（Windows版） - Sebuntin - 博客园"
source: "https://www.cnblogs.com/sebuntin2020/p/12434134.html"
author:
published:
created: 2025-03-24
description: "Golang学习笔记 1、环境搭建 IDEA+Go SDK 1.14 for Windows 下载 Go SDK [下载网址 继续点击next，等待安装成功 读完进度条之后，点击finish，到这一步我们的go1.14的SDK就已经按照成功了，接下来就是环境变量的设置。 设置环境变量 在1.11版之"
tags:
  - "clippings"
---
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)

## Golang学习笔记

## 1、环境搭建---------IDEA+Go SDK 1.14 for Windows

### 下载 Go SDK

[下载网址<<<猛戳这里](https://studygolang.com/dl)

进入下载页面后，根据操作系统选择下载的版本，我本次使用的是 Windows 10系统，所以直接选择 [go1.14.windows-amd64.msi](https://studygolang.com/dl/golang/go1.14.windows-amd64.msi) 下载。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/baf2f5824d850d80271feaaee2059d98.png)

下载完成之后，双击下好的安装包进行安装。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ef9b0e43e20940811b8b2caa752867b4.png)

双击之后，弹出安装界面，我们直接一路next，这里我选择的安装路径是 `D:\Program Files\go`

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5363af70c792e069c4beac051967b7b4.png)

继续点击next，等待安装成功

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3076dbe353db9e082b8c2295e083bd42.png)

读完进度条之后，点击finish，到这一步我们的go1.14的SDK就已经按照成功了，接下来就是环境变量的设置。

### 设置环境变量

在1.11版之后的Go，为了更好的管理我们的包，我们一共需要设置的环境变量有四个：

- GOPATH
- GOROOT
- GO111MODULE
- GOPROXY

go命令依赖一个重要的环境变量：$GOPATH，首先我们需要在系统变量中设置GOPATH，GOPATH可以设置多个在windows系统中用 `;` 来隔开，我这里设置了两个GOPATH：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/55d7e18d7c9fc7642d1451d27518495f.png)

为了在终端上运行 `go` 命令，我们还需要在 Path 环境变量中设置存放go命令的目录：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4930b123414e5c6fdc6b5c68770c50d8.png)

点击 `浏览` ，选择我们之前安装 go sdk 路径下的 `bin` 目录，然后一路确定下去。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0d1642617a8197e41506647a5a1c7867.png)

完成上述步骤之后，我们就可以在终端中输入 `go` 命令了（如果之前开启了终端，就需要重启），接下来我们来查看一下我们安装的 go 版本，在cmd终端中输入 `go version` 。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/22f9d4ec0a737f5272c35b2734be5e90.png)

在Go中没有项目的概念，一切皆为包，当我们在自己的包中导入了其他依赖，Go会先从 GOROOT 目录中去寻找是否有对应的包，所以我们还需要在环境变量中设置 GOROOT，GOROOT的值就是我们安装的go sdk的路径：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1697f15ce6fed522a728836bf0620ad7.png)

### IDEA中的配置

在这里就不介绍IDEA编辑器的下载与激活了，直接介绍如何通过IDEA来搭建Go环境，在golang 1.11版本之后，Go推出了 Go modules依赖库版本管理，从此之后我们就不需要把我们自己的包文件写在GOPATH下了。

这篇文章介绍了如何使用Go modules来管理我们的包和依赖 ： [新版本Golang的包管理入门教程](https://zhuanlan.zhihu.com/p/60703832)

我们直接看看如何在IDEA中进行设置：

**在系统中配置GO111MODULE和GOPROXY环境变量**

> 还是按照上述环境变量的配置方式进入系统变量中输入变量名和变量值，
> 
> 变量名：GO111MODULE 变量值： auto 或者 on 或者 off （推荐选择auto）
> 
> - `GO111MODULE=off` 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。
> - `GO111MODULE=on` 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 `go.mod` 下载依赖。
> - `GO111MODULE=auto` 在 `$GOPATH/src` 外面且根目录有 `go.mod` 文件时，开启模块支持。
> 
> 变量名：GOPROXY 变量值： [https://goproxy.cn](https://goproxy.cn/)

**首先创建Go project**

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b81b181a7d91fbdbea7cefee7dfa352d.png)

下载Go插件，安装完成后需要重启IDEA，点击 `Restart IDE` ：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/cbbef48a6a964ba751baf8b7faca6172.png)

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b021e4c64d711f09cfeab90c3bf2a884.png)

重启IDEA后，创建项目，选择SDK：  
选择SDK：

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a55a3e28d6ebc527fbbcc65c40d5e0e1.png)

创建项目名和路径：  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b9d1d4e8f38fc162e90ba1a3e0d7a7af.png)

**我们需要设置Go proxy以便更快速的下载外部依赖**

> File ---> Setting ---> Language & Framework ----> Go ----> Go Modules

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3024a85bab52d73835c128b2e1677405.png)

在选择好SDK目录后，我们在Proxy这里填上代理地址： `https://goproxy.cn`

### 创建一个 hello.go 文件

我们在 `learn_go` project项目目录下创建了一个 `basic/hello` 目录，在该目录下新建了一个 hello.go 文件

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5e5244145de8cff1dd6148bda309c02b.png)

文件内容如图所示，cd到hello目录中，我们在 terminal 中输入：

```bash
go mod init hello
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/444ffec945364f8541010ab65df37639.png)

该命令执行完之后会在我们的当前目录下生成一个 go.mod 文件，文件内容如下：

```go
module hello

go 1.14
```

我们再尝试运行 hello.go 文件：

```bash
go run hello.go
```

运行结果如下:

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/43fea949a566bae6dcc9259c18e07de9.png)

我们发现 go 会自动查找代码中的包，并下载依赖包，并且把具体的依赖关系和版本写入到go.mod和go.sum文件中。

到此为止，我们的 IDEA+Go环境就搭建好了。

posted @ [Sebuntin](https://www.cnblogs.com/sebuntin2020)   阅读( 12807 )  评论( 0 ) [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=12434134) [收藏](https://www.cnblogs.com/sebuntin2020/p/) [举报](https://www.cnblogs.com/sebuntin2020/p/)

[升级成为园子VIP会员](https://cnblogs.vip/)

\[Ctrl+Enter快捷键提交\]

[【推荐】100%开源！大型工业跨平台软件C++源码提供，建模，组态！](http://www.uccpsoft.com/index.htm)  
[【推荐】还在用 ECharts 开发大屏？试试这款永久免费的开源 BI 工具！](https://dataease.cn/?utm_source=cnblogs)  
[【推荐】国内首个AI IDE，深度理解中文开发场景，立即下载体验Trae](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)  
[【推荐】编程新体验，更懂你的AI，立即体验豆包MarsCode编程助手](https://www.marscode.cn/?utm_source=advertising&utm_medium=cnblogs.com_ug_cpa&utm_term=hw_marscode_cnblogs&utm_content=home)  
[【推荐】抖音旗下AI助手豆包，你的智能百科全书，全免费不限次数](https://www.doubao.com/?channel=cnblogs&source=hw_db_cnblogs)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/16e642d75389d801a3c8776d1a653507.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/16e642d75389d801a3c8776d1a653507.jpg)

**编辑推荐：**  
· [dotnet 源代码生成器分析器入门](https://www.cnblogs.com/lindexi/p/18786647)  
· [ASP.NET Core 模型验证消息的本地化新姿势](https://www.cnblogs.com/himax/p/18785387/how_to_localize_validation_attrbuite_message)  
· [对象命名为何需要避免'-er'和'-or'后缀](https://www.cnblogs.com/CareySon/p/18781531/why_avoid_er_or_suffixes_in_object_naming)  
· [SQL Server如何跟踪自动统计信息更新?](https://www.cnblogs.com/kerrycode/p/18782081)  
· [AI与.NET技术实操系列：使用Catalyst进行自然语言处理](https://www.cnblogs.com/code-daily/p/18776627)  

**阅读排行：**  
· [dotnet 源代码生成器分析器入门](https://www.cnblogs.com/lindexi/p/18786647)  
· [官方的 MCP C# SDK：csharp-sdk](https://www.cnblogs.com/shanyou/p/18787725)  
· [一款.NET 开源、功能强大的远程连接管理工具，支持 RDP、VNC、SSH 等多种主流协议！](https://www.cnblogs.com/Can-daydayup/p/18787819)  
· [提示词工程师自白：我如何用一个技巧解放自己的生产力](https://www.cnblogs.com/anai/p/18788036)  
· [一文搞懂MCP协议与Function Call的区别](https://www.cnblogs.com/longronglang/p/18787489)  

<table><tbody><tr><td colspan="7"><table><tbody><tr><td><a href="https://www.cnblogs.com/sebuntin2020/p/">&lt;</a></td><td align="center">2025年3月</td><td align="right"><a href="https://www.cnblogs.com/sebuntin2020/p/">&gt;</a></td></tr></tbody></table></td></tr><tr><th align="center">日</th><th align="center">一</th><th align="center">二</th><th align="center">三</th><th align="center">四</th><th align="center">五</th><th align="center">六</th></tr><tr><td align="center">23</td><td align="center">24</td><td align="center">25</td><td align="center">26</td><td align="center">27</td><td align="center">28</td><td align="center">1</td></tr><tr><td align="center">2</td><td align="center">3</td><td align="center">4</td><td align="center">5</td><td align="center">6</td><td align="center">7</td><td align="center">8</td></tr><tr><td align="center">9</td><td align="center">10</td><td align="center">11</td><td align="center">12</td><td align="center">13</td><td align="center">14</td><td align="center">15</td></tr><tr><td align="center">16</td><td align="center">17</td><td align="center">18</td><td align="center">19</td><td align="center">20</td><td align="center">21</td><td align="center">22</td></tr><tr><td align="center">23</td><td align="center">24</td><td align="center">25</td><td align="center">26</td><td align="center">27</td><td align="center">28</td><td align="center">29</td></tr><tr><td align="center">30</td><td align="center">31</td><td align="center">1</td><td align="center">2</td><td align="center">3</td><td align="center">4</td><td align="center">5</td></tr></tbody></table>

点击右上角即可分享 ![微信分享提示](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/555e49d6a3ba2183a3b8a2b6083355df.gif)