

之前写的笔记：
[sql注入--盲注&报错&二次&堆叠](St/xioadi2022/sql注入--盲注&报错&二次&堆叠.md)
网上参考的资料：
https://www.cnblogs.com/c1047509362/p/12806297.html
**在探讨SQL注入之报错注入之前，有一个前提就是页面能够响应详细的错误描述**，然而mysql数据库中显示错误描述是因为开发程序中采用了print_r  mysql_error()函数，将mysql错误信息输出。

还有就是一起默写一下SQL注入的核心语句吧，巩固记忆的同时，方便后续注入的使用~

information_schema

schemata(schema_name)

tables(table_schema,table_name)

columns(table_schema,table_name,column_name)

select schema_name from information_schema.schemata;

select table_name from information_schema.tables where table_schema='dvwa';

select column_name from information_schema.columns where table_name='users' and table_schema='dvwa';

select concat(username,password) from dvwa.users;

##  xpath报错注入（extractvalue和updatexml）

- 在mysql高版本（大于5.1版本）中添加了对XML文档进行查询和修改的函数：  
      
    updatexml（）  
    
    extractvalue（）  
    
    当这两个函数在执行时，如果出现xml文档路径错误就会产生报错  
    
- **updatexml（）函数**  
  
    - updatexml（）是一个使用不同的xml标记匹配和替换xml块的函数。
      
    - 作用：改变文档中符合条件的节点的值
      
    - 语法： updatexml（XML_document，XPath_string，new_value） 第一个参数：是string格式，为XML文档对象的名称，文中为Doc 第二个参数：代表路径，Xpath格式的字符串例如//title【@lang】 第三个参数：string格式，替换查找到的符合条件的数据
      
    - updatexml使用时，当xpath_string格式出现错误，mysql则会爆出xpath语法错误（xpath syntax）
      
    - 例如： select * from test where ide = 1 and (updatexml(1,0x7e,3)); 由于0x7e是~，不属于xpath语法格式，因此报出xpath语法错误。
      
      
      
      
    
- ****extractvalue（）函数  
    ****
    - 此函数从目标XML中返回包含所查询值的字符串 语法：extractvalue（XML_document，xpath_string） 第一个参数：string格式，为XML文档对象的名称 第二个参数：xpath_string（xpath格式的字符串） select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));
      
    - extractvalue使用时当xpath_string格式出现错误，mysql则会爆出xpath语法错误（xpath syntax）
      
    - select user,password from users where user_id=1 and (extractvalue(1,0x7e));
      
    - 由于0x7e就是~不属于xpath语法格式，因此报出xpath语法错误。

## floor（）函数报错注入

### 一、概述

原理：利用select count(*),floor(rand(0)*2)x from information_schema.character_sets group by x;导致数据库报错，通过concat函数连接注入语句与floor(rand(0)*2)函数，实现将注入结果与报错信息回显的注入方式。

### 二、函数理解

附带一下本次解释函数的表创建步骤（不再附图）以及数据的填充。

create database test1;  
use test1；  

create table czs(id int unsigned not null primary key auto_increment, name varchar(15) not null);

insert into czs(id,name) values(1,'chenzishuo');  
insert into czs(id,name) values(2,'zhangsan');  
insert into czs(id,name) values(3,'lisi');  
insert into czs(id,name) values(4,'wangwu');

- rand()函数  
    rand()可以产生一个在0和1之间的随机数  
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/e2293f03e238aa6c53a8d6f728632585_MD5.png)
    
     ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/bca6e35f55b411e7ad9d878980ab9739_MD5.png)
    
     可以看出，直接使用rand函数每次产生的数值不一样，但当我们提供了一个固定的随机数的种子0之后，每次产生的值都是相同的，这也可以称之为伪随机。  
    
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/37792f7db6ae2b0dca0d67c9bf7ba5d5_MD5.png)
    
      
    
- floor (rand(0)*2)函数  
    floor函数的作用就是返回小于等于括号内该值的最大整数。  
    rand()本身是返回0~1的随机数，但在后面*2就变成了返回0~2之间的随机数。  
    配合上floor函数就可以产生确定的两个数，即0和1。  
    并且结合固定的随机数种子0，它每次产生的随机数列都是相同的值。  
    此处的myclass 表为含有四行数据的表。  
    结合上述的函数，每次产生的随机数列都是 0 1 1 0  
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/076f9651e793f313c26bf784c5dce516_MD5.png)
    
- group by 函数  
    group by 函数，作用就是分类汇总。  
    等一下再说group by，我们首先看一下我的表。  
    
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/04d231dd8075963ed12fa34c34478574_MD5.png)
    
     再在id 和 name后分别放入a x，意思就是id显示为a name显示为x。  
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/fafa76ab352794a48271297ab1bbf452_MD5.png)
    
     然后使用group by 函数进行分组，并且按照x（name）进行排序。
    
    
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/7b527610de3f17e3885082b1131e08eb_MD5.png)
    
     友情提示：在使用group by 函数进行分类时，会因为mysql版本问题而产生问题，主要是启用了ONLY_FULL_GROUP_BY SQL模式（默认情况下），MySQL将拒绝选择列表，HAVING条件或ORDER BY列表的查询引用在GROUP BY子句中既未命名的非集合列，也不在功能上依赖于它们。（或者自行百度解决）  
    https://blog.csdn.net/weixin_41991232/article/details/82803170
    
- count（*）函数  
    count（*）函数作用为统计结果的记录数。  
    ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/9b54fc7415abefd2019a00dc8fa4fade_MD5.png)
    
     这就是对重复的数据进行整合计数，x就是每个name的数量，我这里每个只有一个当然count（*）都为1了。
    
- ******综合使用产生报错  
    select count(*),floor(rand(0)*2) x from czs group by x;  
    当count（*）和group by x同时执行时，就会爆出duplicate entry错误。  
    ******
    
     ![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/474c05c1ba4ff1eca51519dacd3f1b60_MD5.png)
    
     根据前面的函数，这句话是统计后面的floor（rand（0）*2）from czs产生的随机数种类并计算数量，0110，应该是两个两个，但是最后却报错了。  
    
    **报错原因解析  
    **
    
    通过 floor 报错的方法来爆数据的本质是 group by 语句的报错。group by 语句报错的原因
    
    是 floor(random(0)*2)的不确定性，即可能为 0 也可能为 1
    
    group by key 执行时循环读取数据的每一行，将结果保存于临时表中。读取每一行的 key 时，
    
    如果 key 存在于临时表中，则更新临时表中的数据（更新数据时，不再计算 rand 值）；如果
    
    该 key 不存在于临时表中，则在临时表中插入 key 所在行的数据。（插入数据时，会再计算
    
    rand 值）
    
    如果此时临时表只有 key 为 1 的行不存在 key 为 0 的行，那么数据库要将该条记录插入临
    
    时表，由于是随机数，插时又要计算一下随机值，此时 floor(random(0)*2)结果可能为 1，就
    
    会导致插入时冲突而报错。即检测时和插入时两次计算了随机数的值
    
    实际测试中发现，出现报错，至少要求数据记录为 3 行，记录数超过 3 行一定会报错，2 行
    
    时是不报错的。