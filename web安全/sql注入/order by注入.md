# order by注入

## SQL语法基础知识

### 复习order by

ORDER BY 语句一般是对指定的列进行排序，默认按照升序对记录进行排序。

如果想要降序利用DESC关键字对记录进行降序。

```sql
SELECT * FROM `admin` ORDER BY username DESC;
```

查询语句 order by 排序字段 降序（desc）/升序（默认）
### SQL语法顺序
平常书写SQL语句就是按照下面的语法顺序来书写的。
```sql
select[distinct]
from
join(如 left join)
on
where
group by
having
union
order by
limit
```
### SQL的执行顺序
但是语法顺序并不是SQL的执行顺序，SQL在执行的过程中是按照下面的顺序来执行的。
```sql
(1)from
(2)on
(3)join
(4)where
(5)group by：group by 子句将数据划分为多个分组；
(6)sum,count,max,min,avg：聚合函数
(7)having：使用 having 子句筛选分组
(8)select：选择需要的列
(9)distinct（去重）：对结果进行去重操作
(9)union:将多个查询结合联合，会重复上面的步骤
(10)order by：对结果进行排序
(11)limit：返回的条数
```
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/Pasted%20image%2020241205091502.png)
以上每个步骤都会产生一个虚拟表，该虚拟表被用作下一个步骤的输入。这些虚拟表对调用者(客户端应 用程序或者外部查询)不可用。只有最后一步生成的表才会会给调用者。如果没有在查询中指定某一个子句， 将跳过相应的步骤。

### 检测sql注入的常用方式
>1）布尔注入
>
>2）报错注入
>
>3）延时注入
>
>4）多语句注入（堆叠注入,就是添加分号，在写一个语句）就是可以执行多个语句，利用分号进行隔开
>
>5）联合注入（union)
>
>6）内联注入

## order by注入
### 判断注入类型

数字型order by注入时,语句`order by=2 and 1=2`,和`order by=2 and 1=1` 显示的结果一样,所以无法用来判断注入点类型

而rand()在数字型中，每次刷新会显示不同的排序结果

当在字符型中用`?sort=rand()`,则不会有效果,排序不会改变

因此用rand()可判断注入点类型

### 注入方式

#### 1.和union查询一块使用来判断字段有几个

（不是本主题讨论的重点）

在sql注入时经常利用`order by`子句进行快速猜解表中的列数

通过修改`order by`参数值，比如调整为较大的整型数如`order by 5`，再依据回显情况来判断具体表中包含的列数。

判断出列数后，接着使用`union select`语句进行回显。

#### 2.基于if语句盲注(数字型)

下面的语句只有`order=$id`,数字型注入时才能生效,

`order ='$id'`导致if语句变成字符串,功能失效

如下图为演示

- 字符串型时if()失效,排列顺序不改变
  
    ![image-20210806162140036](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/4630a49cf766ae5ec898e38c495e1ca1_MD5.png)
    
- 数字型时排列顺序改变
  
    ![image-20210806162608698](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/b6492298c7ecf50cf69fdbbff73024b1_MD5.png)
    

知道列名情况下

if语句返回的是字符类型，不是整型, 因此如果使用数字代替列名是不行的,如下图

![image-20210806162742207](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/7fefd0d1af8d3f136f21d6095670bf4a_MD5.png)

这是在知道列名的前提下使用

```sql
?order=if(表达式,id,username)
```

- 表达式为true时,根据id排序
- 表达式为false时,根据username排序

不知道列名

id总知道吧

```sql
?order=if(表达式,1,(select id from information_schema.tables))
```

- 如果表达式为true时，则会返回正常的页面。
  
    ![image-20210806162946764](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/34b2edf2c15190c4eb29e7392754b582_MD5.png)
    
- 如果表达式为false时，sql语句会报ERROR 1242 (21000): Subquery returns more than 1 row的错误，导致查询内容为空
  
    ![image-20210806162923360](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/95df35974e6fef5d5471940e2e7c4a6e_MD5.png)
    

#### 3.基于时间的盲注

```sql
order by if(表达式,1,sleep(1))
```

- 表达式为true时,正常时间显示
  
- 表达式false时,会延迟一段时间显示
  

延迟的时间并不是sleep(1)中的1秒，而是大于1秒。 它与所查询的数据的条数是成倍数关系的。

计算公式：**延迟时间=sleep(1)的秒数*所查询数据条数**

如果查询的数据很多时，延迟的时间就会特别长

在写脚本时，可以添加timeout这一参数来避免延迟时间过长这一情况。

#### 4.基于rand()的盲注(数字型)

rand() 函数可以产生随机数介于0和1之间的一个数

当给rand() 一个参数的时候，会将该参数作为一个随机种子，生成一个介于0-1之间的一个数,

种子固定,则生成的数固定

`order by rand`:这个不是分组，只是排序，rand()只是生成一个随机数,每次检索的结果排序会不同

```sql
order by rand(表达式)
```

当表达式为true和false时，排序结果是不同的，所以就可以使用rand()函数进行盲注了。

#### 5.报错注入

```sql
order by updatexml(1,if(1=2,1,(表达式)),1)
```

```sql
order by extractvalue(1,if(1=2,1,(表达式)));
```

因为`1=2`,所以执行表达式内容

#### 6用列名构造基于排序差异的布尔盲注

```sql
select * from user  order by if(1=1,id,email);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/2f751aa6281416708e5d97a078e9a057_MD5.png)

但前提是知道列名，以及列名确实能够产生排序不同。  
如果不知道列名，用数字代替是不行的。  

#### 7. 用rand构造基于排序差异布尔的盲注
rand是根据种子返回一个0-1的随机小数，rand(1=1)和rand(1=2)在大量数据时大概率会导致排序不同 ，因此可以构造基于排序不同的布尔条件。  

```sql
select * from user  order by rand(1=1);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/b67622a8583dbf66d14675b741bbc453_MD5.png)

显然基于排序差异的布尔盲注，如果碰到没有排序差异的查询结果，比如就一条数据，就没办法了。  



#### 8 基于报错的布尔盲注

在有些注入中，表里没有数据，或者查询出来的数据不展示在web，又或者是增删改的SQL注入。这个时候可能只能时间盲注，但通常可以将这个时间盲注升级成基于报错的布尔盲注。  
就是构造出一个条件，为true时执行错误的语句导致服务端抛错，为false时执行正确的语句，即使服务端不返回任何东西或者返回随机的东西，只要不抛错，即可构造布尔条件。  
直观点就是这样的SQL语句。  

```sql
select * from user where id =1 and if(1=1,exp(9999999),1);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/ac8c57aafbaf78f58568c862c30d5bb1_MD5.png)

用case when代替if  

```sql
select * from user where id =1 and (case when 1=1 then exp(9999999) else 1 end);
```



exp(9999999)这个报错行为，代替如下。  
cot(1-(1=1))  
pow(1+(1=1),99999)  
还有另外一种在其他数据库也能构造出来的报错行为，那就是联合查询使子查询有两条数据。  

```sql
select * from user where id =1 and (case when 1=1 then (select 1 union select 2) else 1 end);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/2b8de104a23e15c73ef84069bb9d8f2b_MD5.png)

基于报错的布尔盲注弥补了基于排序差异的布尔盲注的不足，那么order by能用它吗？  
答案是能用一部分，也就是联合查询那部分。  

```sql
select * from user where 1=1  order by (case when 1=1 then (select 1 union select 2) else 1 end);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/c0b6c8e468b8150594c33d50b69bfd2d_MD5.png)

为什么if+exp不能用呢？

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/26d9fa7f6dfd92a34c3f723b0a6b624e_MD5.png)

答案是order by正常情况下根本不会去执行方法，甚至连基础的运算都不支持。  
如下图，order by (2-1)的排序结果是跟没有order by或者order by null的结果是一样的。  

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/53e5aceaf0e7b845626d945f84155055_MD5.png)

什么情况下会执行运算呢？前面已经说了很多种情况了，比如sleep，比如联合查询。  
但这里面还要分情况，那就是运算的先后顺序，比如常用的substr()，它结合exp(9999999)也会导致报错，但却是恒定报错。  

```sql
select * from user where 1=1  order by if(1=1,substr(1,exp(9999999)),1);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/85be50de65a3cb6edfa7cac0f427a0d7_MD5.png)

也就是说，先运算exp()，再运算substr()，最后运算if(1=1)。  
而如果是sleep(exp(9999999))，运算顺序就变化了。  

```sql
select * from user where 1=1  order by if(1=1, sleep(exp(9999999)),1);
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/c96e83732bacf95584080268dff1b2a6_MD5.png)

先运算if(1=1)，再运算exp()，最后运算sleep()，这也是为什么sleep能够进行时间盲注而benchmark不行的的真正原因。  
而且即使不执行这个sleep()，仅仅只需要将其放在一个不会触发的地方，也能够改变运算顺序，从而构造出基于报错的布尔盲注。  

```sql
select * from user where 1=1  order by if(1=1, exp(9999999),if(1=1,1,sleep(0)));
```

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/28e3245377f4c848b2e8fa396f30c4a0_MD5.png)

甚至在if外面也可以

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/6ad477c25b7a2cf2f65794cd0077e786_MD5.png)



### fuzz

但可惜的是，目标环境禁用了sleep()和select，由于测试了很多冷门函数都不能用，反而经常被禁的substr都可以用，所以合理怀疑它是个框架内的白名单。于是我从mysql官网中将所有的非8.0函数几乎都搬了过来，构造出一个插入order by中不会报错的字典。  
[https://dev.mysql.com/doc/refman/8.0/en/built-in-function-reference.html](https://dev.mysql.com/doc/refman/8.0/en/built-in-function-reference.html)  

然后开始fuzz，最终只发现三个函数可以改变order by的运算顺序，使基于报错的布尔盲注在order by语句中实现。  
sleep(exp(99999999))  
get_lock('test',exp(99999999))  
rand(exp(99999999))  



![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/82110dde0f23d25d47827017611ead72_MD5.png)

也就是order by一共存在三种情况。  
1，if/length/hex/char/exp等单独使用完全不会执行的函数，结果恒定为null  
2，substr/updatexml/min/left等会优先执行的函数，配合exp(99999999)，恒定报错，此时可以使用报错注入。  
3，sleep/rand/get_lock/联合查询会改变执行顺序的函数，先执行if(1=1)，再根据结果是否执行exp(99999999)  
只有第三种情况才能做到order by基于报错的布尔盲注。  

最终结合目标的实际情况，使用rand(exp(99999999))盲注出CURRENT_USER交差。  
具体payload如下  

```perl
(case when substr(CURRENT_USER,1,1)='r' then rand(exp(99999999)) else 1 end)