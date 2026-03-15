## 浏览器http请求方式
### post方式
请求数据包含在请求体中，更加安全，没有长度限制，支持多种编码方式，不限制接受的数据类型，浏览器回退时会再次提交请求
### get请求
请求数据包含在url中，不保证安全，有长度限制，仅支持url编码方式，仅接受ASCII字符类型的数据类型，浏览器回退时会再次提交请求
### $_REQUEST 
php中_REQUEST可以获取以POST方法和GET方法提交的数据，
缺点：速度比较慢 。 
### $_GET
用来获取由浏览器通过GET方法提交的数据(参数)。
语法：**变量名=GET["name"];** //name指表单元素name属性值 
GET方式会将表单中的数据以URL字符串的形式发送给服务器
用test.php以GET方式提交，浏览器地址栏会显示
[http://localhost/test.php?username=admin](http://localhost/test.php?username=admin)
即$_GET['username']=admin

**$_GET[]缺点:**

1.  安全性不好，在URL中可以看得到 
2.  传送数据量较小，不能大于2KB。 
3.  在PHP中，$_POST[]主要用来获取表单中填入的值 

可以理解为用来获取由浏览器通过POST方法提交的数据(参数)

用test.php以POST方式提交，浏览器地址栏会显示
[http://localhost/test.php](http://localhost/test.php)

带有 POST 方法的表单发送的信息，对任何人都是不可见的（不会显示在浏览器的地址栏），并且对发送信息的量也没有限制。
他提交的大小一般来说不受限制，然而，默认情况下，POST 方法的发送信息的量最大值为 8 MB（可通过设置 php.ini 文件中的 post_max_size 进行更改）。相对于_GET方式安全性略高

### 三种方式的区别和联系
_$_REQUEST["参数"]具用$__POST["参数"],$ _GET["参数"]的功能,但是$_REQUEST["参数"]比较慢。
通过post和get方法提交的所有数据都可以通过$_REQUEST数组["参数"]获得
