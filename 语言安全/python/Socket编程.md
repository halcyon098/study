# Socket编程

socket，套接字，计算机直接进行网络连接的一套程序接口，相当于在发送端和接受端之间建立了一套程序接口

## 创建对象（实例化）

scoket(family,type[,protocol])

第一个参数 family 是指定应用程序使用的通信协议的协议族，有：

| Family参数      | 描述                               |
| --------------- | ---------------------------------- |
| socket.AF_UNIX  | 只能够用于单一的Unix系统进程间通信 |
| socket.AF_INET  | 服务器之间网络通信                 |
| socket.AF_INET6 | IPv6                               |

默认值为 AF_INET

第二个参数 type 为要创建套接字的类型

| Type参数           | 描述                                             |
| ------------------ | ------------------------------------------------ |
| socket.SOCK_STREAM | 流式socket , 当使用TCP时选择此参数               |
| socket.SOCK_DGRAM  | 数据报式socket ,当使用UDP时选择此参数            |
| socket.SOCK_RAW    | 原始套接字，允许对底层协议如IP、ICMP进行直接访问 |

第三个参数 protocol 是可选项，指明所要接收的协议类型，通常为 0 或者不填。

| Type参数           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| socket.IPPROTO_RAW | 相当于protocol=255，此时socket只能用来发送IP包，而不能接收任何的数据。发送的数据需要自己填充IP包头，并且自己计算校验和。 |
| socket.IPPROTO_IP  | 相当于protocol=0，此时用于接收任何的IP数据包。其中的校验和和协议分析由程序自己完成。 |

## 套接字对象方法（Socket 常用函数）

### 服务端函数

#### 1、bind 函数

该函数是服务端函数，会将之前创建的套接字与指定的IP地址和端口进行绑定，使用点直接调用该方法，**以元组（host, port）的形式表示地址**。

比如我们绑定本地的 12345 端口：

```python
s.bind(('127.0.0.1', 12345))
```

注意这里用到了两次括号，因为 bind 方法的参数需要是一个包含主机地址和端口号的元组。

#### 2、listen 函数

该函数也是服务端函数，用于在使用TCP的服务端开启监听模式，只有一个参数，指定在拒绝连接之前，操作系统可以挂起的最大连接数量，该值至少为1，大部分应用程序设为 5 即可。

在服务端开启监听，设置操作系统可以挂起的最大连接数量为 5 ：

```python
s.listen(5)
```

listen() 只是让套接字处于监听状态，并没有接收请求，接收请求需要使用 accept() 函数。 

#### 3、accept 函数

当套接字处于监听状态时，可以通过 accept() 函数来接收客户端请求，该函数也是服务端函数，接受 TCP 连接并返回（conn,address），返回一个新的套接字来和客户端通信，其中 conn 是新的套接字对象，可以用来接收和发送数据，address 是连接客户端的地址。

比如我们就使用 conn 和 address 这两个参数（可自定义）来接收 accept 函数返回的值

```python
conn, address = s.accept()
```

### 客户端函数

#### 4、connect 与 connect_ex 函数

connect() 是客户端程序用来连接服务端的方法，客户端连接到 address 处的套接字，一般 address 的格式为元组（host, port），如果连接出错，会返回 socket.error 错误。

比如我们连接到刚才创建的套接字，即本地的 12345 端口：

```python
import socket

c = socket.socket()
c.connect(('127.0.0.1', 12345))
```

只有经过 connect 连接成功后的套接字对象才能调用发送和接受方法（send/recv），所以服务端的 s 对象不能 send 或者 recv 。

还有一个 connect_ex 函数，功能与 connect 相同，只是成功返回 0，失败返回 error 的值。

### 常见的公共函数

#### 5、send 、 sendall 、sendto 函数

send() ：用于发送 TCP 数据

> 用法：s.send(string[,flag])

将 string 中的数据发送到连接的套接字，返回值是要发送的字节数量，该数量可能小于 string 的字节大小。

sendall() ：完整发送TCP数据

> 用法：s.sendall(string[,flag])

与 send() 类似，将 string 中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据，成功返回None，失败则抛出异常。

> 关于参数的说明：
>
> string: 这是要发送的数据，通常是一个字符串。这是必需的参数。
>
> flag : 这是一个可选的参数，用于指定发送数据的附加选项，在大多数情况下可以省略。

比如我们将 "Hello, server!" 字符串通过UTF-8编码转换为字节数据，然后使用套接字的 send 方法发送这些字节数据到连接的服务端：

```python
c.send("Hello, server!".encode('utf-8'))
```

sendto ：用于发送 UDP 数据

> 用法：s.sendto(string[,flag],address) 

将数据发送到套接字，address是形式为（ipaddr，port）的元组，指定远程地址，返回值是发送的字节数。

#### 6、recv 与 recvfrom 函数

recv ：接受TCP套接字的数据，数据以字符串形式返回。

> 用法：s.recv(bufsize[,flag])

参数说明：

> bufsize: 这是一个整数，表示要接收的最大字节数。接收的实际数据可能少于或等于这个值，取决于发送端发送的数据量。
>
> flag（可选）: 这是一个可选的参数，用于指定接收数据的附加选项。在大多数情况下，可以省略这个参数。

通过 recv 方法接收最多 1024 字节的数据，我们将接收到的字节数据解码为 UTF-8 编码的字符串，最后打印出来：

```python
re = c.recv(1024)
print(re.decode('utf-8'))
```

recvfrom：接受 UDP 套接字的数据，与 recv() 类似，但返回值是（data, address）。其中 data 包含接收数据的字符串，address 是发送数据的[套接字地址](https://so.csdn.net/so/search?q=套接字地址&spm=1001.2101.3001.7020)。

> 特别说明：recvfrom 函数是可以在没有进行 connect 的情况下使用的，对于 UDP 套接字，可以在未连接的状态下使用 recvfrom 来接收数据。recvfrom 通常与 UDP 一起使用，因为 UDP 是一个无连接协议，而 recvfrom 提供了从发送端获取地址信息的能力。在 TCP 中，由于是面向连接的，地址信息通常在连接建立时就已经确定了，所以 recv 通常就足够了。

示例：

```python
import socket

# 创建一个UDP套接字
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 绑定本地地址和端口
s.bind(('127.0.0.1', 12345))

# 接收数据，设置bufsize为最大接收字节数
data, address = s.recvfrom(1024)

# 打印接收到的数据和发送端地址信息
print(f"Received data: {data.decode('utf-8')}")
print(f"Received from: {address}")
```

#### 7、close 函数

该函数用于关闭套接字

直接使用点调用即可

```python
s.close()
```

## Socket 创建和配置：

socket(family, type, proto=0, fileno=None)：创建一个套接字对象。

gethostname()：获取主机名。 



### 通用套接字方法：

bind(address)：将套接字绑定到指定的地址。

listen(backlog)：开始监听传入连接请求。

accept()：接受连接，返回新的套接字和对端地址。

connect(address)：连接到远程套接字。

close()：关闭套接字。



### 数据收发：

send(bytes)：发送数据。

recv(bufsize)：接收数据。



### UDP 相关方法：

sendto(data, address)：发送UDP数据到指定地址。

recvfrom(bufsize)：接收UDP数据和对端地址。



### 设置和获取套接字选项：

setsockopt(level, optname, value)：设置套接字选项。

getsockopt(level, optname, buflen=None)：获取套接字选项。



### 获取主机信息：

gethostbyname(hostname)：通过主机名获取IP地址。

gethostbyaddr(ip_address)：通过IP地址获取主机名。

## 演示

### 服务端

```python
# socket编程初识
import socket

# 服务端 TCP
# 创建socket对象
s = socket.socket()

# 绑定端口,元组的形式
s.bind(('localhost', 8899))

# 等待客户端链接，5指定在服务端关闭前最大可以链接的数量
s.listen(1)

# 连接后返回一个新的套接字对象
c, addr = s.accept()
print(f'接受到客户端信息:{addr}')
# 发送数据
while True:
    # 接受客户端发送的数据
    data = c.recv(1024).decode("utf-8")
    print(f'客户端发来的消息{data}')

    # 服务端发送数据
    msg = input('输入服务端发送的消息')
    if 'exit' ==msg:
        break
    c.send(msg.encode('utf-8'))
# 关闭连接
c.close()
s.close()
```

### 客户端

```python
import socket
# 客户端 TCP
# 创建对象
s = socket.socket()

# 绑定IP（域名）和端口
s.connect(('localhost',8899))
# 发送信息
while True:
    # 发送信息到服务端
    msg = input('请输入发送服务端的信息：')
    if 'exit'==msg:
        break
    s.send(msg.encode('utf-8'))
    # 接收服务端的信息
    data = s.recv(1024).decode('utf-8')
    print(f'服务端发来的信息：{data}')
# 关闭连接
s.close()
```



