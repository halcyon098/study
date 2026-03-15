### 什么是bytes？
byte，也称为字节，是计算机唯一可以存储的东西。也就是说，你想要在计算机中存储任何东西，都需要对其进行编码，将其转换为字节。例如：

- 存储音乐，必须先使用MP3、WAV等方式对其编码。
- 存储图片，必须先使JPG、JPEG等方式对其编码。
- 存储文本，必须使用ASCII、UTF-8等方式对其编码。

这里面，MP3、WAV、JPG、JPEG、ASCII、UTF-8等都是编码的类型，每种类型的编码方式不同，所以在解码时也要通过编码时的方式解码。
### bytes与string的区别
同理，在Python中，bytes就是：字节序列。它只存储二进制的0和1，人类是无法理解的。
而 string 字符串是字符序列，人类可以理解的，但它无法直接存储在计算机中，必须要对其编码（转换为bytes）。有多种编码方式可以将string转换为bytes，如ASCII、UTF-8、GBK等。举例如下：
```python
i_string='I am a string'.encode('UTF-8')
print(i_string)
```
输出：b'I am a string'。
在这个例子中，Python使用UTF-8的编码方式对变量i_string的值“I am a string”进行了编码，然后存入到了计算机中。
如果我们用**print**函数打印出**i_string**的值，Python会将其表示为`b'I am a string'。这并不代表实际存储在计算机中的就是这个值，而是在打印时，Python使用UTF-8解码了它们，所以才能以字符的形式展现出来。但它在字符前添加了**“b”**这个字符，以标识它是**bytes**类型。
当然，我们也可以将bytes解码回string，如下：
```python
i_bytes = b'I am a string'.decode('UTF-8')
print(i_bytes)
```
上面变量i_bytes输出的结果为：I am a string。
编码解码是逆运算，计算机在将字符写入到磁盘前进行**编码**，从磁盘中读取时进行**解码**。
### bytes的作用和使用方式
**bytes类型**是Python3.0以上版本新增的类型。如上文所知，bytes只负责以二进制形式存储数据，至于这些二进制数据代表了什么内容，完全由程序的编码方式决定。
除了存储图片、视频、音乐等文件外，bytes类型的数据也非常适合在互联网上传输，所以一般用于网络通信。
在Python中，bytes和string类型关系最为紧密，你可以将字符串转换为bytes对象，你可以使用以下两种方式：
```python
#通过构造函数方式创建 bytes 变量
b1 = bytes()#无参数创建空的bytes
b2 = bytes('Python技术站',encoding='UTF-8')#指定UTF-8字符编码方式，转换为bytes
print(b1)
print(b2)

#通过字符形式创建 bytes 变量
bs1 = b''
bs2 = b'http://pythonjishu.com'
print(bs1)
print(bs2)
```
输出：
b''
b'Python\xe6\x8a\x80\xe6\x9c\xaf\xe7\xab\x99'
b''
b'[http://pythonjishu.com](http://pythonjishu.com/)'
从运行结果可以发现，变量b2的输出结果并没有原样输出，而是输出了十六进制形式的字符编码值，这是因为Python将bytes转换为字符时，是按照单个字节处理数据的，而非ASCII字符（例子中的中文）一般占用两个以上的字节，Python无法一次性处理，所以会以十六进制形式输出。
对于非ASCII字符，bytes有一个decode()方法，通过该方法可以将bytes对象重新转换成字符串。例如上面的b2变量：
b2.decode('UTF-8')
输出结果：
'Python技术站'
