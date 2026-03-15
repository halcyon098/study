# Maven踩坑

## Maven安装和配置

Maven官网：[Maven – Download Apache Maven](https://maven.apache.org/download.cgi)

### 下载

在官网中下载需要版本的Maven

![image-20240807140028738](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/ad0b2753b1868937190d944808883e31_MD5.png)

Binary是可执行版本，已经编译好可以直接使用。

Source是源代码版本，需要自己编译成可执行软件才可使用。

tar.gz和zip两种压缩格式,其实这两个压缩文件里面包含的内容是同样的,只是压缩格式不同

tar.gz格式的文件比zip文件小很多,用于unix操作系统。

下载，解压到本地

### 配置环境变量

1.右键此电脑–>属性–>高级系统设置–>环境变量

2.新建变量MAVEN_HOME = D:\xxx\Maven\apache-maven-3.8.1（以自己的安装路径为准）

3.编辑变量Path，添加变量值%MAVEN_HOME%\bin

4.然后win+R运行cmd，输入mvn -version，出现Maven版本，电脑版本，jdk版本，成功

### 配置本地仓库

在D:\xxx\Maven\apache-maven-3.8.1路径下新建maven-repository文件夹，用作maven的本地库（maven-repository不一定非要是这个，只是方便）

在路径E:\Tools\Maven\apache-maven-3.8.1\conf下找到settings.xml文件

找到节点localRepository，在注释外添加

```xml
<localRepository>E:\Tools\Maven\maven-repository</localRepository>
```

> localRepository节点用于配置本地仓库，本地仓库其实起到了一个缓存的作用，它的默认地址是 C:\Users\用户名.m2。
> 当我们从maven中获取jar包的时候，maven首先会在本地仓库中查找，如果本地仓库有则返回；如果没有则从远程仓库中获取包，并在本地库中保存。
> 此外，我们在maven项目中运行mvn install，项目将会自动打包并安装到本地仓库中。
>
> 

### 配置镜像

1.在settings.xml配置文件中找到mirrors节点

2.添加如下配置（注意要添加在<mirrors>和</mirrors>两个标签之间，其它配置同理）

```xml
<!-- 阿里云仓库 -->
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

> 因为国外的服务器下载jar包很慢所以我们改为阿里云服务器
>
> 虽然mirrors可以配置多个子节点，但是它只会使用其中的一个节点，即默认情况下配置多个mirror的情况下，只有第一个生效，只有当前一个mirror无法连接的时候，才会去找后一个；而我们想要的效果是：当a.jar在第一个mirror中不存在的时候，maven会去第二个mirror中查询下载，但是maven不会这样做！

### 配置JDK

1. 在settings.xml配置文件中找到profiles节点
2. 添加如下配置

```xml
<!-- java版本 --> 
<profile>
      <id>jdk-1.8</id>
      <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
      </activation>
 
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
      </properties>
</profile>
```

win+R运行cmd，输入mvn help:system测试

>  首次执行 mvn help:system 命令，Maven相关工具自动帮我们到Maven中央仓库下载缺省的或者Maven中央仓库更新的各种配置文件和类库（jar包)到Maven本地仓库中。
> 下载完各种文件后， mvn help:system 命令会打印出所有的Java系统属性和环境变量，这些信息对我们日常的编程工作很有帮助。



## idea中 maven 本地仓库有jar包，但还是找不到，解决打包失败和无法引用的问题

通过阿里云公共仓库下载的jar，然后放入本地仓库后，还是无法引用，打包报错，百度了一下问题的解决办法，因为下载资源后，会生成对应的_remote.repositories文件，标示该资源，所以我们根据打包时候控制台输出的 jar 包所在的本地仓库，删除_remote.repositories 文件

![image-20240807135029874](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a9a24b5df5df1ba7505a253e29df0848_MD5.png)

如果还是不能引用 这些 jar 的话 ， 删除项目下的 .[iml](https://so.csdn.net/so/search?q=iml&spm=1001.2101.3001.7020) 文件，然后重启idea， 点击 maven 上面的刷新

## Maven中某个包下载失败

Maven 在本地仓库中缓存了某个依赖项的失败状态，并且在一段时间内不会再次尝试解析该依赖项，除非更新间隔已过或者[强制更新](https://so.csdn.net/so/search?q=强制更新&spm=1001.2101.3001.7020)。

有几种方法可以解决这个问题：

清理本地仓库：尝试清除 Maven 的本地仓库缓存，这将强制 Maven 重新下载所有依赖项。可以手动删除 Maven 本地仓库目录下的所有内容（默认情况下位于用户目录下的.m2/repository[文件夹](https://so.csdn.net/so/search?q=文件夹&spm=1001.2101.3001.7020)），然后重新构建项目。

强制更新依赖项：可以在 Maven 命令中使用 -U 或 --update-snapshots 参数，强制更新所有依赖项，而不管缓存状态。例如，运行 mvn clean install -U。

检查网络连接：确保你的网络连接正常，没有被防火墙或代理服务器阻止。有时候，网络问题可能导致无法正确下载依赖项。

检查远程仓库：如果你使用的是远程仓库，可以检查该仓库是否可用，并且其设置是否正确。可能需要更新远程仓库的 URL 或验证凭据等。

**mvn clean install -U -f pom.xml文件绝对路径，可以不依赖环境变量，在bin目录下使用**

## 手动加载依赖包到本地

依赖包官网：[Maven Repository: Search/Browse/Explore](https://mvnrepository.com/)

官网中依赖包的镜像：[org/apache/maven/plugins](https://repo.maven.apache.org/maven2/org/apache/maven/plugins/)

进入官网依赖包，搜索想要的包，进入

![image-20240807141206833](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/b67d24290ca576442d6721bc00b459e2_MD5.png)

进入需要的版本

![image-20240807141304349](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/416ad08c7dffb2398b37c049b8b36a02_MD5.png)

![image-20240807141350312](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a2d9e7b81db75b58df885d8c85c01efc_MD5.png)



![image-20240807141423603](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/3511b496d63dfa81540a724b8189fff3_MD5.png)

第二个网址可以直接进入最后一个jar包的目录，但是需要自己翻目录了。

**settings.xml**

只保留一个本地仓库的路径

**删除仓库里访问远程仓库的文件**

删除原来仓库里所有的 _remote. 和 lastUpate 文件

**使用命令手动安装jar 包**

```
mvn install:install-file -Dfile=sdk-1.0.jar -DgroupId=com.im -DartifactId=sdk -Dversion=1.0 -Dpackaging=jar
```

> mvn install:install-file -Dfile=your-artifact-1.0.jar
>
> [-DpomFile=your-pom.xml]
>
> [-Dsources=src.jar]
>
> [-Djavadoc=apidocs.jar]
>
> [-DgroupId=org.some.group]
>
> [-DartifactId=your-artifact]
>
> [-Dversion=1.0]
>
> [-Dpackaging=jar]
>
> [-Dclassifier=sources]
>
> [-DgeneratePom=true]
>
> [-DcreateChecksum=true]

安装完jar包后，把目录下生成的 _remote 删除掉

**在IDEA中Update 索引**

```
File => Settings => Build, Execution, Deployment => Build Tools => Maven => Repositories => Update
```

**reimport 包**

如果不行，pom.xml中多修改几次jar版本号，或者注释掉再删掉注释，让pom.xml多加载几次，多reimport几次。或者重新手动安装该jar包、删除_remote文件、更新索引、修改jar包版本号、reimport



## 国内依赖镜像源

其次在换国内镜像源，这里给大家整理了最新可用的镜像源

1. 阿里云：[**http://maven.aliyun.com/**](http://maven.aliyun.com/)

2. 中央仓库：[**https://repo1.maven.org/maven2/**](https://repo1.maven.org/maven2/)

3. 网易：[**http://maven.netease.com/repository/public/**](http://maven.netease.com/repository/public/)

4. 华为云：[**https://repo.huaweicloud.com/repository/maven/**](https://repo.huaweicloud.com/repository/maven/)

5. tencent：[**https://mirrors.cloud.tencent.com/repository/maven/**](https://mirrors.cloud.tencent.com/repository/maven/)

6. 中国科技大学：[**http://mirrors.ustc.edu.cn/maven/maven2/**](http://mirrors.ustc.edu.cn/maven/maven2/)

7. 南京大学：[**http://maven.nju.edu.cn/repository/**](http://maven.nju.edu.cn/repository/)

8. 清华大学：[**https://repo.maven.apache.org/maven2/**](https://repo.maven.apache.org/maven2/)

9. 北京理工大学：[**http://mirror.bit.edu.cn/maven/**](http://mirror.bit.edu.cn/maven/)

10. 东软信息学院：[**https://mirrors.neusoft.edu.cn/maven2/**](https://mirrors.neusoft.edu.cn/maven2/)

11. 中国科学院开源协会：[**http://maven.opencas.cn/maven/**](http://maven.opencas.cn/maven/)

12. 北京交通大学：[**http://maven.bjtu.edu.cn/maven2/**](http://maven.bjtu.edu.cn/maven2/)

```xml
<mirrors>
  <mirror>
    <id>aliyun</id>
    <url>http://maven.aliyun.com/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>central</id>
    <url>https://repo1.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>netease</id>
    <url>http://maven.netease.com/repository/public/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>huaweicloud</id>
    <url>https://repo.huaweicloud.com/repository/maven/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>tencent</id>
    <url>https://mirrors.cloud.tencent.com/repository/maven/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>ustc</id>
    <url>http://mirrors.ustc.edu.cn/maven/maven2/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>nju</id>
    <url>http://maven.nju.edu.cn/repository/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>tsinghua</id>
    <url>https://repo.maven.apache.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>bit</id>
    <url>http://mirror.bit.edu.cn/maven/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>neusoft</id>
    <url>https://mirrors.neusoft.edu.cn/maven2/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>opencas</id>
    <url>http://maven.opencas.cn/maven/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
  <mirror>
    <id>bjtu</id>
    <url>http://maven.bjtu.edu.cn/maven2/</url>
    <mirrorOf>central</mirrorOf>
  </mirror>
</mirrors>
```

