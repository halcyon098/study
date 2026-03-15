**1、在 docker-compose.yml 配置映射端口让 jvm debug 端口能外部访问。**
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165506639.png)
**2、在 docker-compose.yml 中使用 command 字段添加自定义启动命令：`java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=xxxx -jar jar包名称.jar`。**
是不是突然发现自己不知道 jar 包名字叫啥？😆😆 咱们可以先启动容器，然后执行 **`docker ps --no-trunc`** 不截断输出完整的容器描述，就可以看到容器名称以及容器中的路径。
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165527428.png)
3、容器启动后，从容器中把运行的 jar 包复制出来，新建一个文件夹 (例：CVE-2017-18349)，IDEA 点击 Open 打开 CVE-2017-18349 文件夹， 复制粘贴 fastjsondemo.jar, 右键jar包，选择 Add as Lirary 添加到项目依赖库中，并在代码上打上几个断点。
启动容器后，执行命令：**`docker cp 容器id:jar包路径 目标路径`**, 将 jar 复制出来。  
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165553338.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165559137.png)
**4、 IDEA 配置 Remote Debug，点击 Debug 运行失败。**
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165615382.png)
**==Project Structure，Modules->Dependencies 添加要调试的class文件的目录 BOOT-INF==。**
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165644464.png)
**再次测试远程断点调试是否生效：**生效
**==但是还是有问题，断点调试无法进入第三方依赖包中，我们需要把lib文件夹复制到根目录，并右键 Add as Library 。==**
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165728379.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241217165734408.png)
**最终远程断点调试的前期准备全部完成，可以快快乐乐的打断点了~**
❗️❗️❗️**==如果你是一名Java开发人员，请注意如果生产环境下迫不得已需要远程调试，调试完一定要关闭JDWP服务，或者JDWP服务监听的端口不对公网开放。==**❗️❗️❗️