# sql注入--数据类型&提交方式

数据库类型不同决定攻击手法（暴力爆破，低权限注入，高权限注入），payload不同

数据库类型不同要考虑payload闭合方式不同，提交的数据格式不同（数字型，字符型，编码，加密，json）

数据提交方式不同，数据请求不同，注入需要按照指定的方式去测试。（url没有参数不一定没有注入，有些数据在数据包中，只要http数据包任何一个地方被接收，都有可能产生漏洞）

## 数据类型

### 数字型

不用考虑payload闭合

### 字符型

需要考虑闭合方式，**常见的闭合方式有 ’，"， ')，")** 

### 搜索型

往往有搜索框的地方也是可能存在注入的，因为很多内容都是存储在数据库中，想要完成搜索，就需要与数据库进行数据交互，如果没有做到安全的过滤或其他的防护手段往往也是会存在SQL注入的。常规的搜索处后台SQL语句是

select value from test where value like "%$_GET['value']%" ,这个时候我们想要对其进行注入，就需要考虑到闭合双引号及通配符的问题了。

> select * from inkbamboo where name like '%$s%'     假设这条语句为代码中查询的sql语句
> %' UNION+ALL+SELECT+1,database,2,3,4,5 and '%'='    该语句是要进行注入的语句
> select * from inkbamboo where name like '%%' UNION+ALL+SELECT+1,database,2,3,4,5 and '%'='%'     语句融合

### 编码|加密

常见MD5加密，也有其他加密

#### 宽字节注入

##### **涉及函数：**

> magic_quotes_gpc（魔术引号开关）这个在PHP的高版本(5.4.45及以上)里面取消掉了，转化为了一种函数（addslashes()函数）
>
> 作用：当PHP的传参中有特殊字符就会再前面加转义字符'\',来做一定的过滤 单引号和双引号内的一切都是字符串，那我们输入的东西如果不能闭合掉单引号和双引号，我们的输入就不会当作代码执行，就无法产生SQL注入
>
> addslashes() 函数返回在预定义字符之前添加反斜杠的字符串
>
> mysql_real_escape_string() 函数转义 SQL 语句中使用的字符串中的特殊字符
>
> mysql_escape_string() 转义一个字符串

##### **原理分析：**

​		先了解一下什么是窄、宽字节已经常见宽字节编码：

当某字符的大小为一个字节时，称其字符为窄字节.

当某字符的大小为两个字节时，称其字符为宽字节.

所有英文默认占一个字节，汉字占两个字节

常见的宽字节编码：GB2312,GBK,GB18030,BIG5,Shift_JIS等

**为什么会产生宽字节注入**，其中就涉及到编码格式的问题了，宽字节注入主要是源于程序员设置数据库编码与PHP编码设置为不同的两个编码格式从而导致产生宽字节注入

如果数据库使用的的是GBK编码而PHP编码为UTF8就可能出现注入问题，原因是程序员为了防止SQL注入，就会调用我们上面所介绍的几种函数，将单引号或双引号进行转义操作，转义无非便是在单或双引号前加上斜杠（\）进行转义 ，但这样并非安全，因为数据库使用的是宽字节编码，两个连在一起的字符会被当做是一个汉字，而在PHP使用的UTF8编码则认为是两个独立的字符，如果我们在单或双引号前添加一个字符，使其和斜杠（\）组合被当作一个汉字，从而保留单或双引号，使其发挥应用的作用。但添加的字符的Ascii要大于128，两个字符才能组合成汉字 ，因为前一个ascii码要大于128，才到汉字的范围 ，这一点需要注意。

也是因此，这个漏洞大部分在中国。

##### 绕过方法

​    1、寻找不需要闭合的地方

​    2、仔细查看作用域(影响范围)

​    3、宽字节注入

当尝试单引号（'）不报错后，尝试经典的%df后报错，说明可能存在宽字节注入，单引号被转义了。

原理：

![image-20240224173244414](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/aea870191056097e1b11cb956b2a7a51_MD5.png)

> python sqlmap.py -r 1.txt --tamper=unmagicquotes.py 启用这个脚本即可，就可以让sqlmap智能得在payload中加入宽字节等方法绕过转义函数。


#### base64编码

例如参数进行了base64编码，在后端解码后再正常拼接到sql中

eg:

www.xxx.xom/?id=1

www.xxx.xom/?aWQ9MQ==

使用sqlmap进行扫描时，工具会判断不出扫描点，需要使用Tamper脚本。

tamper 脚本可以用来对 sqlmap 的 sql 查询语句进行一次处理，让其绕过过滤，tamper 脚本在 sqlmap 安装目录的tamper 目录下

```bash
use age：sqlmap.py --tamper="模块名.py"

base64encode.py
功能：用 base64 格式进行编码
平台：All
举例：1’ AND SLEEP (5)# ==> MScgQU5EIFNMRUVQKDUpIw==
```



### json

有时候会有 {"id":1}这种格式的参数传递时，我们注入就需要考虑到json的特殊格式。在进行构造payload

原理：下面是一个简单的json格式数据，当把数据取出来时，并不会取出双引号，而且取出来的值默认是以单引号包含的。所以在进行注入时需要考虑单引号闭合。

> {
>
> "firstName":"Brett",
>
> "lastName":"McLaughlin",
>
> "email":"brett@newInstance.com"
>
> }

## 提交方式

```bash
#httpt头
python sqlmap.py -r 1.txt #1.txt是http请求包数据,抓包得到
python sqlmap.py -r 1.txt
#post
python sqlmap.py -r 1.txt #1.txt是http请求包数据,抓包得到
python sqlmap.py --data "n=1&p=1"
#cookie
python sqlmap.py -r 1.txt #1.txt是http请求包数据,抓包得到
python sqlmap.py --cookie "JSESSIONID=EC114ABC4B90D3CB1FC79874ABB5E9C6"
```



### GET

常见在url参数中

### POST

一般出现在表单，上传等数据量较大，不让用户直接看到数据

### Cookie

概念：HTTP协议本身是无状态的，什么是无状态呢，即服务器无法判断用户身份，cookie实际上是一小段的文本信息（key-value格式），用于记录用户状态。

前提是：有交互记录，有cookie记录

有报错信息可以利用报错注入

### HTTP头

#### User-Agent方式注入

UA的基本概念： User Agent中文名为用户代理，简称UA，它是一个特殊字符串头，使得服务器能够识别客户使用的操作系统及版本，CPU类型，浏览器及版本，浏览器渲染引擎，浏览器语言，浏览器插件等。

#### Referer方式注入

REFERER的基本概念： HTTP Referer是heade的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器该网页是从哪个页面连接发过来的，服务器因此可以获得一些信息用于处理。

#### XFF方式注入

X-Forwarded-For：

简称XFF头，代表了HTTP的请求端真实IP。 是客户端通过HTTP代理或者负载均衡器连接到web服务端获取源IP地址的一个标准（通常一些网站的防注入功能会记录请求端真实IP地址并写入数据库或某文件，通过修改XXF头可以实现伪造IP）

### Request方式注入

概念：超全局变量 PHP中的许多预定义变量都是“超全局的”，这意味着它们在一个脚本的全部作用域中都可以用

> 这些超全局变量是：
>
> $_REQUEST（获取GET/POST/COOKIE）COOKIE在新版本已经无法获取了_
>
> $_POST（获取POST传参）_
>
> $_GET（获取GET传参）_
>
> $_COOKIE（获取COOKIE传参）_
>
> $_SERVER（包含了诸如头部信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组）
