因为感觉这个师傅写的很好没忍住摘抄了。
### 0x01 前言

本文记录一次攻防中比较苛刻场景下的 shiro 550 漏洞的不出网利用。

### 0x02 简介

主要内容：

- 绕过 WAF  对 rememberMe 长度的限制
    
- 加载本地字节码 defineClass 注入内存马
    

### 0x03 漏洞验证

1、验证后端反序列化功能是否正常

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/601152aaca3217fb9d65551c17559d61_MD5.webp)

  

（被拦截）

根据经验猜测大概率是限制了 rememberMe 的长度

2、删减到 300 左右，正常放行

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/8ed399ab9d4e2e61c3ec5ab6318b6a44_MD5.webp)

  

3、绕过 waf  对 rememberMe 长度的限制

OPTIONS 请求方式 + 静态资源 uri 路径

```
OPTIONS /app/login;index.css
```

成功将 rememberMe 的长度提升到 7000 左右

但超过 7000 依然会被拦截，没法再绕

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/dabf9231a68c5fae70a8687fc63bd08f_MD5.webp)

  

4、验证后端反序列化功能是否正常

反序列化炸弹

```
java -jar ysoserial-for-woodpecker-<version>.jar -g FindClassByBomb -a "java.lang.String|25"
```

延时成功，说明后端功能正常

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/dd47f1c9c01162a4a335662515341567_MD5.webp)

  

5、探测目标是否出网

通过 urldns/httplog/jrmp 的方式探测，发现目标不出网

6、探测反序列化链

通过延时探测反序列化链

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/212ed9ee6dea59a241ea2660546092ae_MD5.webp)

  

说明目标存在 gadget: CommonsCollections10

### 0x04 漏洞利用

7、梳理信息与思路

请求方式：OPTIONS

- 分离payload+动态类加载的姿势失效（OPTIONS 无法传递参数）
    
- 修改 maxHeaderSize 没用（nignx 反代）
    

目标不出网

- 远程加载字节码的姿势失效
    

利用思路

- 命令执行/代码执行 写文件马
    
- 代码执行 注内存马
    

8、判断目标操作系统

```
public class Payload extends AbstractTranslet {
```

延时6s，说明目标操作系统极大概率为 *nix

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/b0db6ccee72f84703a3d1f7edab2ee93_MD5.webp)

9、尝试写文件马

通过 find 静态资源文件获取web路径并写入文件马，失败

猜测原因

- 当前用户权限问题（不是root）
    
- 当前应用部署问题（以springboot fatjar的方式部署）
    

10、排查写入失败原因

在 home 目录写文件

```
echo 123321 > /home/tmp.txt
```

判断文件是否写入成功

```
File file = new File("/home/tmp.txt");
```

延时6s，说明当前权限足够，则很有可能是应用部署层面导致的，文件马的路到此为止，尝试内存马

11、尝试注入内存马

将字节码写到临时目录，然后从目标本地读取文件加载字节码

由于 paylaod 长度限制，内存马需要分散写入，经过测试发现每次最多能写长度为 1600 左右

1）使用 java-memshell-generator 辅助模块延时确认目标中间件为 tomcat

```
detect_way=Sleep
```

2）拆分内存马，限制每组长度为 1600

```
public static void main(String[] args) {
```

拆分成了 14 组

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/945559ed2defee37126d3f470a26c4cc_MD5.webp)

3）写入内存马的字节码

```
// 第1次
```

4）读取字节码进行 defineClass

- 注意换行符问题
    

```
ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
```

5）成功注入内存马

![图片](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a233a6ab8fae2dcdf252a82b35c8c1b2_MD5.webp)

### 0x05  总结

目标虽然成功拿下了，但这是一种不太优雅的利用方式，不管是落地文件还是追加文件内容的选择都有可能会遇到更苛刻的挑战，比如文件无法落地、有负载均衡等。