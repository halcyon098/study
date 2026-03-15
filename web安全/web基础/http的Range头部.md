# http的Range头部

## **为什么？**

http协议中可能会遇到：请求取消或数据传输中断，这时客户端已经收到了部分数据，后面再请求时最好能请求剩余部分（断点续传）；或者，对于某个较大的文件，能够支持客户端多线程分片下载...，因此HTTP1.1 协议（RFC2616）中新增的header字段 Range 和 Content-Range 可以支持获取文件的部分内容，这为断点续传和并行下载提供了技术支持。

## Range 和 Content-Range

HTTP1.1 协议（RFC2616）中新增的header字段 Range 和 Content-Range 可以支持获取文件的部分内容，这为断点续传和并行下载提供了技术支持。

### Range

客户端进行请求时，通过header中配置 Range参数 可以 指定请求数据的 第一个字节的位置和最后一个字节的位置，格式为：

```
Range:(unit=first byte pos)-[last byte pos]

```

如：

```
Range: bytes=0-499  	// 第 0-499 字节的内容 
Range: bytes=500-999 	// 第 500-999 字节的内容 
Range: bytes=-500 		// 最后 500 字节的内容 
Range: bytes=500- 		// 第 500 字节开始到文件结束的内容 
Range: bytes=0-0,-1 	//第一个 和 最后一个字节 的内容 
Range: bytes=500-600,601-999 // 同时指定多个范围
```

### Content-Range

服务端响应请求时，通过header中返回 Content-Range参数表示当前发送数据的范围和文件总大小，格式为：

```
Content-Range: bytes (unit first byte pos) - [last byte pos]/[entity legth]
```

如：

```
Content-Range: bytes 0-499/22400 // 0－499 为当前发送数据的范围， 22400 为文件总大小
```

### 状态码

HTTP/1.1 200 Ok（不使用断点续传方式）
HTTP/1.1 206 Partial Content（使用断点续传方式）

### If-Range

服务端响应请求时，通过header中返回If-Range判断实体是否发生改变。如果实体未改变，则服务器继续发送客户端丢失部分，否则发送整个实体，格式为：

```
If-Range: Etag | HTTP-Date
```

If-Range 的值为 Etag 或者 Last-Modified 且优先使用ETag，当没有 ETage 却有 Last-modified 时，则使用 Last-modified 作为 If-Range 字段的值。

如：

```
If-Range: “627-4d648041f6b80” 
If-Range: Fri, 22 Feb 2013 03:45:02 GMT
```

If-Range 必须与 Range 配套使用。如果请求header中没有 Range，那么 If-Range 就会被忽略。

如果服务器不支持 If-Range，那么 Range 就会被忽略。

如果请求header中的 Etag 与服务器目标内容的 Etag 相同，则响应状态码为 206。

如果服务器目标内容发生了变化，那么应答报文的状态码为 200。

### 应用

网站视频播放可以通过拉动进度条播放视频，因为视频文件传输时进行了切片，通过拉动进度条可以发送请求对应范围的数据包。

众多号称多线程下载工具（如 FlashGet、迅雷等）实现多线程下载的核心所在。

### 补充：
tcp/ip协议进行通信时，链路层有最大的承载数据量限制，当两台主机之间的通信要通过多个具有不同MTU值的网络时，MTU的瓶颈是通信路径上最小的MTU值，它被称为路径MTU。

	在TCP/IP分层中，数据链路层用MTU（Maximum Transmission Unit，最大传输单元）来限制所能传输的数据包大小，MTU是指一次传送的数据最大长度，不包括数据链路层数据帧的帧头，如以太网的MTU为1500字节，实际上数据帧的最大长度为1512字节，其中以太网数据帧的帧头为12字节。

当发送的IP数据报的大小超过了MTU时，IP层就需要对数据进行分片，否则数据将无法发送成功。

MTU （最大传输单元）决定 IP报文是否分片。
