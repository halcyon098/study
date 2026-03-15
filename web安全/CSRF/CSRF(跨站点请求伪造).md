# CSRF(跨站点请求伪造)

## 1、简介

　　CSRF的全名为Cross-site request forgery，它的中文名为 跨站请求伪造

　　CSRF是一种夹持用户在已经登陆的web应用程序上执行非本意的操作的攻击方式。相比于XSS，CSRF是利用了系统对页面浏览器的信任，XSS则利用了系统对用户的信任。

## 2、CSRF攻击原理

下面为CSRF攻击原理图：

![](https://cdn.jsdelivr.net/gh/Dmup227/Dimags@main/D:%5CDocument%5Cimages202403191516555.jpg)

由上图分析我们可以知道构成CSRF攻击是有条件的：

　　1、客户端必须一个网站并生成cookie凭证存储在浏览器中

　　2、该cookie没有清除，客户端又tab一个页面进行访问别的网站

## 3、CSRF例子与分析

　　我们就以游戏虚拟币转账为例子进行分析

### 　　3.1、简单级别CSRF攻击

　　假设某游戏网站的虚拟币转账是采用GET方式进行操作的，样式如：

```
1 http://www.game.com/Transfer.php?toUserId=11&vMoney=1000
```

 

　　此时**恶意攻击者**的网站也构建一个相似的链接：

　　1、可以是采用图片隐藏，页面一打开就自动进行访问第三方文章：

```html
<img src='%E6%94%BB%E5%87%BB%E9%93%BE%E6%8E%A5'>
```



　　2、也可以采用js进行相应的操作

```url
http://www.game.com/Transfer.php?toUserId=20&vMoney=1000         
#toUserID为攻击的账号ID
```

　

　1、假若客户端已经验证并登陆www.game.com网站，此时客户端浏览器保存了游戏网站的验证cookie

　　2、客户端再tab另一个页面进行访问恶意攻击者的网站，并从恶意攻击者的网站构造的链接来访问游戏网站

　　3、浏览器将会携带该游戏网站的cookie进行访问，刷一下就没了1000游戏虚拟币

### 　　3.2、中级别CSRF攻击

　　游戏网站负责人认识到了有被攻击的漏洞，将进行升级改进。

　　将由链接GET提交数据改成了表单提交数据

```html
//提交数据表单
<form action="./Transfer.php" method="POST">
　　　　<p>toUserId: <input type="text" name="toUserId" /></p>
　　　　<p>vMoney: <input type="text" name="vMoney" /></p>
　　　　<p><input type="submit" value="Transfer" /></p>
</form>
```

　　Transfer.php

```php
1 <?php
2  　　　　session_start();
3  　　　　if (isset($_REQUEST['toUserId'] &&　isset($_REQUEST['vMoney']))  #验证
4  　　　　{
5  　　　　     //相应的转账操作
6  　　　　}
7  ?>
```

　　恶意攻击者将会观察网站的表单形式，并进行相应的测试。

　　首先恶意攻击者采用（http://www.game.com/Transfer.php?toUserId=20&vMoney=1000）进行测试，发现仍然可以转账。 

　　那么此时游戏网站所做的更改没起到任何的防范作用，恶意攻击者只需要像上面那样进行攻击即可达到目的。

　　总结：

　　1、网站开发者的错误点在于没有使用$_POST进行接收数据。当$_REQUEST可以接收POST和GET发来的数据，因此漏洞就产生了。

### 　　3.3、高级别CSRF攻击

　　这一次，游戏网站开发者又再一次认识到了错误，将进行下一步的改进与升级，将采用POST来接收数据

　　Transfer.php

```php
1 <?php
2  　　　　session_start();
3  　　　　if (isset($_POST['toUserId'] &&　isset($_POST['vMoney']))  #验证
4  　　　　{
5  　　　　     //相应的转账操作
6  　　　　}
7  ?>
```

 

　　此时恶意攻击者就没有办法进行攻击了么？那是不可能的。

　　恶意攻击者根据游戏虚拟币转账表单进行伪造了一份一模一样的转账表单，并且嵌入到iframe中

嵌套页面：(用户访问恶意攻击者主机的页面，即tab的新页面)

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>攻击者主机页面</title>
    <script type="text/javascript">
    function csrf()
    {
        window.frames['steal'].document.forms[0].submit();
    }
    </script>
</head>
<body onload="csrf()">
<iframe name="steal" display="none" src="./csrf.html">
</iframe>
</body>
</html>
```

 

表单页面：（csrf.html）

```html
<!DOCTYPE html>
<html>
<head>
    <title>csrf</title>
</head>
<body>
<form display="none" action="http://www.game.com/Transfer.php" method="post" >
    <input type="hidden" name="toUserID" value="20">
    <input type="hidden" name="vMoney" value="1000">
</form>
</body>
</html>
```

　　客户端访问恶意攻击者的页面一样会遭受攻击。

**总结：**

　　CSRF攻击是源于Web的隐式身份验证机制！Web的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的

 

## 4、CSRF防御方法

　　服务器端防御：

　　1、重要数据交互采用POST进行接收，当然是用POST也不是万能的，伪造一个form表单即可破解

　　2、使用验证码，只要是涉及到数据交互就先进行验证码验证，这个方法可以完全解决CSRF。但是出于用户体验考虑，网站不能给所有的操作都加上验证码。因此验证码只能作为一种辅助手段，不能作为主要解决方案。

　　3、验证HTTP Referer字段，该字段记录了此次HTTP请求的来源地址，最常见的应用是图片防盗链。PHP中可以采用APache URL重写规则进行防御

可参考：http://www.cnblogs.com/phpstudy2015-6/p/6715892.html

　　4、为每个表单添加令牌token并验证

（可以使用cookie或者session进行构造。当然这个token仅仅只是针对CSRF攻击，在这前提需要解决好XSS攻击，否则这里也将会是白忙一场【XSS可以偷取客户端的cookie】） 

　　CSRF攻击之所以能够成功，是因为攻击者可以伪造用户的请求，该请求中所有的用户验证信息都存在于Cookie中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的Cookie来通过安全验证。由此可知，抵御CSRF攻击的关键在于：在请求中放入**攻击者所不能伪造的信息**，并且该信息不存在于Cookie之中。

　　鉴于此，我们将为每一个表单生成一个随机数秘钥，并在服务器端建立一个拦截器来验证这个token，如果请求中没有token或者token内容不正确，则认为可能是CSRF攻击而拒绝该请求。

　　由于这个token是随机不可预测的并且是隐藏看不见的，因此恶意攻击者就不能够伪造这个表单进行CSRF攻击了。

　　要求：

　　1、要确保同一页面中每个表单都含有自己唯一的令牌

　　2、验证后需要删除相应的随机数

**构造令牌类Token.calss.php**

```php
 1 <?php
 2 class Token
 3 {
 4     /**
 5     * @desc 获取随机数
 6     *
 7     * @return string 返回随机数字符串
 8     */
 9     private function getTokenValue()
10     {
11         return md5(uniqid(rand(), true).time());
12     }
13     
14     /**
15     * @desc 获取秘钥
16     *
17     * @param $tokenName string | 与秘钥值配对成键值对存入session中（标识符，保证唯一性）
18     *
19     * @return array 返回存储在session中秘钥值
20     */
21     public function getToken($tokenName)
22     {
23         $token['name']=$tokenName;      #先将$tokenName放入数组中
24         session_start();
25         if(@$_SESSION[$tokenName])      #判断该用户是否存储了该session
26         {                               #是，则直接返回已经存储的秘钥
27             $token['value']=$_SESSION[$tokenName];
28             return $token;
29         }
30         else                            #否，则生成秘钥并保存
31         {
32             $token['value']=$this->getTokenValue();
33             $_SESSION[$tokenName]=$token['value'];
34             return $token;
35         }
36     }
37 
38 }
39 #测试
40 $csrf=new Token();
41 $name='form1';
42 $a=$csrf->getToken($name);
43 echo "<pre>";
44 print_r($a);
45 echo "</pre>";
46 echo "<pre>";
47 print_r($_SESSION);
48 echo "</pre>";die;
49 
50 ?> 
```

 表单中使用：

```php+HTML
<?php
session_start();
include(”Token.class.php”);
$token=new Token();
$arr=$token->getToken(‘transfer’);    #保证唯一性（标识符）
?>
<form method=”POST” action=”./transfer.php”>
	<input type=”text” name=”toUserId”>
    <input type=”text” name=”vMoney”>
    <input type="hidden" name="<?php echo $arr['name'] ?>"  value="<?php echo $arr['value']?>" >
    <input type=”submit” name=”submit” value=”Submit”>
 </form>
```

 验证：

```php
 1 <?php
 2 #转账表单验证
 3 session_start();
 4 if($_POST['transfer']==$_SESSION['transfer'])       #先检验秘钥
 5 {
 6     unset($_SESSION['transfer']);    #删除已经检验的存储秘钥
 7     if ( &&isset($_POST['toUserId'] &&　isset($_POST['vMoney']))  #验证
 8     {
 9          //相应的转账操作
10     }
11 }
12 else
13 {
14     return false;
15 }
16 ?>
```

```php
 1 <?php
 2 #转账表单验证
 3 session_start();
 4 if($_POST['transfer']==$_SESSION['transfer'])       #先检验秘钥
 5 {
 6     unset($_SESSION['transfer']);    #删除已经检验的存储秘钥
 7     if ( &&isset($_POST['toUserId'] &&　isset($_POST['vMoney']))  #验证
 8     {
 9          //相应的转账操作
10     }
11 }
12 else
13 {
14     return false;
15 }
16 ?>
```

该方法套路：

\1. 用户访问某个表单页面。

\2. 服务端生成一个Token，放在用户的Session中，或者浏览器的Cookie中。【这里已经不考虑XSS攻击】

\3. 在页面表单附带上Token参数。

\4. 用户提交请求后， 服务端验证表单中的Token是否与用户Session（或Cookies）中的Token一致，一致为合法请求，不是则非法请求。

## 5. 漏洞利用

 检测：黑盒手工利用测试，白盒看代码检验（有无token，来源检验等）

使用bp抓取实现功能的数据包（转账，用户添加等）

生成：BurpSuite->Engagement tools->Generate CSRF Poc

利用：将文件放到自己的站点下，诱使受害者访问（或配合XSS触发访问）

绕过：https://blog.csdn.net/weixin_50464560/article/details/120581841

https://threezh1.com/2020/02/25/CSRF%E7%BB%95%E8%BF%87%E6%95%B4%E7%90%86/

### Referer同源

就是判断Referer这个值是不是同一个域名或者同一个源下，绕过方法是把referer来源改下就行

但是这里又有个问题，就是受害者不可能自己去改referer这个值呀。
只能尝试一下下面三种方式绕过

绕过1：配合文件上传绕过
绕过2：配合存储XSS绕过
绕过3：去掉检测来源头(代码逻辑问题)

```html
<meta name="referrer" content="no-referrer">
```

### Token校验

token(令牌,也可以理解为暗号,在数据传输之前，要先进行暗号的核对，暗号不一致则拒绝数据传输)
CSRF_token 对关键操作增加Token参数，token必须随机，每次都不一样，存储在cookie中，与验证码一样

**绕过：**
绕过0：将Token参数值复用（代码逻辑不严谨）能够重复使用token
绕过1：将Token参数删除（代码逻辑不严谨）把token整个参数值删掉