# [ACTF2020 新生赛]Include 1（php伪协议）
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20230226135059613.png)
可发现是文件包含，已经明确给出，文件包含直接读取的是文件，而不是文件源码，所以要想办法读取源码方法。
那么就要涉及到 PHP 伪协议。

## php支持的伪协议

- 1 file:// — 访问本地文件系统

- 2 http:// — 访问 HTTP(s) 网址

- 3 ftp:// — 访问 FTP(s) URLs

- 4 php:// — 访问各个输入/输出流（I/O streams）

- 5 zlib:// — 压缩流

- 6 data:// — 数据（RFC 2397）

- 7 glob:// — 查找匹配的文件路径模式

- 8 phar:// — PHP 归档

- 9 ssh2:// — Secure Shell 2

- 10 rar:// — RAR

- 11 ogg:// — 音频流

- 12 expect:// — 处理交互式的流
## 1 php://filter

php://filter 是一种元封装器， 设计用于数据流打开时的筛选过滤应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() file_get_contents()， 在数据流内容读取之前没有机会应用其他过滤器。

简单通俗的说，这是一个中间件，在读入或写入数据的时候对数据进行处理后输出的一个过程。

php://filter可以获取指定文件源码。当它与包含函数结合时，php://filter流会被当作php文件执行。所以我们一般对其进行编码，让其不执行。从而导致 任意文件读取。

### 协议参数



| 名称                 | 描述                                          |
| :----------------- | ------------------------------------------- |
| resource=<要过滤的数据流> | 这个参数是必须的。它指定了你要筛选过滤的数据流。                    |
| read=<读链的筛选列表>     | 该参数可选。可以设定一个或多个过滤器名称，以管道符（                  |
| write=<写链的筛选列表>    | 该参数可选。可以设定一个或多个过滤器名称，以管道符（                  |
| <；两个链的筛选列表>        | 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。 |

**常用：**

```php
# php://filter 元封装器
#read参数 convert.base64-encode以base64加密数据流（目标文件）
#resource=index.php 要过滤的数据流
php://filter/read=convert.base64-encode/resource=index.php
php://filter/resource=index.php

```

利用filter协议读文件，将index.php通过base64编码后进行输出。这样做的好处就是如果不进行编码，文件包含后就不会有输出结果，而是当做php文件执行了，而通过编码后则可以读取文件源码。而使用的convert.base64-encode，就是一种过滤器。

## 过滤器

字符串过滤器，该类通常以string开头，对每个字符都进行同样方式的处理。

**string.rot13**

一种字符处理方式，字符右移十三位。

**string.toupper**

将所有字符转换为大写。

**string.tolower**

将所有字符转换为小写。

**string.strip_tags**
这个过滤器就比较有意思，用来处理掉读入的所有标签，例如XML的等等。在绕过死亡exit大有用处。

## 转换过滤器

对数据流进行编码，通常用来读取文件源码。

**convert.base64-encode & convert.base64-decode**

base64加密解密

**convert.quoted-printable-encode & convert.quoted-printable-decode**

可以翻译为可打印字符引用编码，使用可以打印的ASCII编码的字符表示各种编码形式下的字符。

## 压缩过滤器

注意，这里的压缩过滤器指的并不是在数据流传入的时候对整个数据进行写入文件后压缩文件，也不代表可以压缩或者解压数据流。压缩过滤器不产生命令行工具如 gzip的头和尾信息。只是压缩和解压数据流中的有效载荷部分。

用到的两个相关过滤器：**zlib.deflate（压缩）**和 **zlib.inflate（解压）**。zilb是比较主流的用法，至于bzip2.compress和 bzip2.decompress工作的方式与 zlib 过滤器大致相同。

## 加密过滤器

**mcrypt.**和 **mdecrypt.**使用 libmcrypt 提供了对称的加密和解密。

更多妙用：https://www.leavesongs.com/PENETRATION/php-filter-magic.html

## 利用filter伪协议绕过死亡exit

什么是死亡exit：
			死亡exit指的是在进行写入PHP文件操作时，执行了以下函数：

``` php
file_put_contents($content, '<?php exit();' . $content);
```

亦或者

```php
file_put_contents($content, '<?php exit();?>' . $content);
```

这样，当你插入一句话木马时，文件的内容是这样子的：

```php
<?php exit();?>
<?php @eval($_POST['snakin']);?>
```

这样即使插入了一句话木马，在被使用的时候也无法被执行。这样的死亡exit通常存在于缓存、配置文件等等不允许用户直接访问的文件当中。

## payload

由以上原理构造payload

```php
?file=php://filter/read=convert.base64encode/resource=flag.php
```


![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20230226142411790.png)
base64解码即可得到flag
