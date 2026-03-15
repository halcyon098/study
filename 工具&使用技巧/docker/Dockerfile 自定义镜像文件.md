# Dockerfile 自定义镜像文件

Dockerfile就是一个**文本文件**，其中**包含一个个的指令**(Instruction)，用指令来**说明要执行什么操作来构建镜像**。

理解起来类似于shell脚本，提前写好命令，进行批量执行。

## 规则

1. 格式 ：#是注释，指令建议要大写，内容小写。这样更能区分。比如：FROM centos

2. 执行顺序：docker 是按照 Dockerfile 指令顺序依次执行的，也就是说从上到下。

3. 其他：每一个 Dockerfile 的第一行都是非注释性的，也就是说第一行不能是注释，必须是 FROM 指令，来指定基础镜像，后面的指令都以基础镜像为运行环境。如果构建过程中本地没有指定镜像文件，就会去远端仓库拉。

## 常用指令

### **FROM**

 - 指定基础镜像，必须在文本的第一行。后面的一切操作都要基于此镜像
	- FROM centos:7 指定基础镜像为CentOS的7版本

### **ENV**

- 设置环境变量，在后面使用
- ENV <key>=<value>
  ENV <key> <value>
  - ENV MYPATH /usr/locad 设置变量MYPATH为 /usr/locad，调用MYPATH即为调用该路径



### **MAINTAINER**

-  该指令是描述的维护者信息。
- MAINTAINER <name>

### **WORKDIR**

- 该指令是切换到 WORKDIR 目录下工作，这个与 linux 里面的 cd 差不多。如果 WORKDIR 不存在，它将被创建。

- 需要注意的是通过 WORKDIR 设置工作目录后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT、ADD、 COPY 等命令都会在该目录下执行。在使用 docker run 运行容器时，可以通过 -w 参数覆盖构建时所设置的工作目录。

- 这条命令会等我们进入容器的时候生效

### RUN 标签

- 该指令用于在容器中执行命令。我们常用来安装基础软件，或者执行一些命令。
- 需要注意的是 RUN 指令创建的中间镜像会被缓存，并会在下次构建中使用。**如果不想使用这些缓存镜像，可以在构建时指定 --no-cache 参数**，如：docker build --no-cache

```dockerfile
# 在容器的指定目录下创建一个 ROOT 文件夹
RUN mkdir -p /usr/local/tomcat/webapps/ROOT/

# 安装 yum 工具
RUN yum -y install vim
 
# 安装网络监测工具
RUN yum -y install net-tools
```



### ADD **标签**

- 该指令是用来将宿主机某个文件或目录放到（复制）容器某个目录下面。
- 如果复制的文件是 **tar** 类型的，那么该文件会自动解压 ( 网络压缩资源不会被解压 ) ，可以访问网络资源，类似 **wget**。

```dockerfile

ADD <src>... <dest>
 
ADD ["<src>",... "<dest>"]

# 将 index.html 放到指定的目录下
ADD index.html /usr/local/tomcat/webapps/ROOT/index.html
```



### **COPY** 标签

​    该标签功能类似于 **ADD** 标签，但是是不会自动解压文件，也不能访问网络资源。

```dockerfile
COPY <源路径> <目标路径>
```



### EXPOSE **标签**

​    该指令用于暴露容器里的端口，**没有什么用**，加了和不加没有什么区别，仅仅是为了告诉使用这个镜像的人，我的这个容器用到了 xxxx 端口号。

```dockerfile
EXPOSE 端口
```

### **VOLUME** 标签

​    该标签用于在 **image** 中创建一个挂载目录，以挂载宿主机上的目录。

```dockerfile
# path 代表容器中的目录，与 docker run 不同，Dockerfile 中不能指定宿主机目录，默认使用 docker 管理的挂载点
VOLUME <path>
VOLUME ["path"]
```

### CMD 标签

-  该标签是构建容器后调用的，也就是在容器启动时才进行调用。
- 一个 Dockerfile 只有一个 CMD 指令，若有多个，只有最后一个 CMD 指令生效。
- CMD 主要目的：为容器提供默认执行的命令，这个默认值可以包含可执行文件，也可以不包含可执行文件，意味着必须指定 ENTRYPOINT 指令（第二种写法）。
- 注意和 RUN 指令的区别，**RUN 是构建镜像时执行的命令，执行的时期不同**。

```dockerfile
# shell格式
CMD <命令>
 
# exec格式 
CMD ["可执行文件", "参数1", "参数2", …]
```

### ENTRYPOINT 

- 该标签是指定容器启动的要运行的命令，可以追加命令。
- ENTRYPOINT 与 CMD 非常类似，不同的是通过 docker run 执行的命令不会覆盖ENTRYPOINT，而 docker run 命令中指定的任何参数，都会被当做参数再次传递给 ENTRYPOINT。
- Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的 ENTRYPOINT 指令。

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
 
ENTRYPOINT command param1 param2 (shell内部命令)
```

### CMD 和 ENTRYPOINT 配合使用

​        一般情况下，ENTRYPOINT 和 CMD 标签都是互相配合使用的，即：ENTRYPOINT 填写固定的命令，CMD 填写该固定命令对应的参数，CMD 将这个参数传递给 ENTRYPOINT命令。

可以理解为 CMD 参数为 ENTRYPOINT 的默认值，如果项目中使用的不是 CMD 的默认值，就可以在启动 docker 容器时添加上真实的参数值，用来覆盖 CMD 的默认值。

举个例子，比如要在镜像中通过 java -jar 的方式启动一个 java 工程，就可以采用下面的方式，默认启动的时候 commcon.jar 这个工程。

```dockerfile
ENTRYPOINT ["java", "-jar"]

CMD ["common.jar"
```

如果我们不想启动这个 common.jar 的工程了，我们在启动容器的时候更换下命令就可以了，如下所示：

```bash
docker run 容器名称 xxxx.jar
```

## 例子

```dockerfile

# 指定基础镜像为 tomcat:8 
FROM tomcat:8
 
# 作者的名字为 xhf ，邮箱是1982392926@qq.com
MAINTAINER xhf<1982392926@qq.com>
 
# 定义一个变量 MYPATH，路径为 /usr/local
ENV MYPATH /usr/local
 
# 切换到 MYPATH 的路径下
WORKDIR $MYPATH
 
# 在容器的指定目录下创建一个 ROOT 文件夹
RUN mkdir -p /usr/local/tomcat/webapps/ROOT/
 
# 将 index.html 放到指定的目录下
ADD index.html /usr/local/tomcat/webapps/ROOT/index.html
 
# 对外暴露 8080 端口
EXPOSE 8080
```



