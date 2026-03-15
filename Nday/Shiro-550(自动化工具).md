# Shiro

练习平台

太阿漏洞练习平台(https://taie.hantaosec.com/#/dashboard)

## shiro 反序列化 （CVE-2016-4437）

打开镜像

![image-20241121134442712](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20241121134442712.png)

打开访问地址，进入登录页面。

![image-20241121134601974](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20241121134601974.png)

打开shiro利用工具，输入登录网址，先爆破密钥，再爆破利用链

![image-20241121135142961](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20241121135142961.png)

提示爆破成功后，尝试利用

![image-20241121135258618](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20241121135258618.png)

使用whoami,ls等命令，返回回显，flag在目录下。

**原理：**shiro550，密钥硬编码在源码中，可以使用默认或者互联网收集的密钥进行爆破尝试，爆破成功后攻击者可以使用密钥伪造rememberMe字段，rememberMe在后端代码中会进行反序列化，触发readObject函数，可以使用cc链进行任意命令执行。