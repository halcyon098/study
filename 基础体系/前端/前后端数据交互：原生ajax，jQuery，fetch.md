# 前后端数据交互：原生ajax，jQuery，fetch

总结这段时间学习工具开发所使用的前后端交互的方法。这三种方法都是基于js的。

## jQuery

jQuery 是一个高效、精简并且功能丰富的 JavaScript 工具库。它提供的 API 易于使用且兼容众多浏览器，这让诸如 HTML 文档遍历和操作、事件处理、动画和 Ajax 操作更加简单。目前超过90%的网站都使用了jQuery库，jQuery的宗旨：写的更少，做得更多！
官网：https://jquery.com/

jQuery使用需要使用使用标签引入，同引入js文件一样。

jQuery实现ajax语法为：$.ajax({})，其中{}为对象，对象里面的key键是固定的，比如:

- 1、type: 表示请求方式，一般为post或get
- 2、url表示请求的地址
- 3、contentType表示发送信息至服务器时内容编码类型
- 4、data表示发送到服务器的数据
- 5、dataType表示预期服务器返回数据的类型如：text，json，xml等等类型
- 6、success表示请求成功后的回调函数
- 7、error自动判断 (xml 或 html)。请求失败时调用此函数，等等

```js
				$.ajax({
					//请求方式
					type:"post",
					//请求地址
					url:"02ajax.php",
					//发送信息至服务器时内容编码类型
					contentType: 'application/x-www-form-urlencoded;charset=utf-8',  
					//传递的数据
					data:{username:username,password:password},
					//预期服务器返回数据的类型如：text，json，xml等等
					dataType:"json",
					//请求成功
					success:function(data){
					console.log(data);
					},
					//请求失败，打印错误信息。
					error : function(e){
						console.log(e.status);
						console.log(e.responseText);
					}
				})

```



jQuery在Ajax方面还封装了其他更简单的函数，如load(),load() 通常是从web服务器上获取静态的数据文件，如果需要传递一些参数给服务器中的页面，可以使用 .*g**e**t*()方法和.post() 方法（或 $.ajax() 方法）。注意：.*g**e**t*()方法和.post() 方法是 jQuery 中的全局函数。


## 原生Ajax

Ajax可以在网页不刷新的情况下可以请求数据然后实现网页局部刷新或者渲染。

```js
GET:
//第一步  先城建一个ajax的核心 XMLHttpRequest
let xhr = new XMLHttpRequerst();
//第二步  使用open 创建请求 第一个参数是请求方式 第二个是请求的地址  第三个是同步或者异步
xhr.open('GET',"txt.php?a=1&b=2",false)
//如果是post请求  必须要写请求头
xhr.setRequestHeader('') //设置请求头
//第三步  为xhr.onreadystatechange  设置监听事件
xhr.onreadystatechange = function(){
    if(xhr.readyState == 4) {
    if(xhr.status == 200){
    alert(xhr.responseTwxt)
//readyState  0 请求未初始化  刚刚实例化XMLHttpRequest
//readyState  1 客户端与服务器建立链接  调用open方法
//readyState  2 请求已经被接收
//readyState  3 请求正在处理中
//readyState  4 请求成功
            }
       }
}
// 第四步 发送请求数据  调用send 发送请求 如果不需要参数就写一个null
xhr.send();


POST:
//第一步  先城建一个ajax的核心 XMLHttpRequest
let xhr = new XMLHttpRequerst();
//需要传递的参数
let data = "a=1&b=2"
//第二步  使用open 创建请求 第一个参数是请求方式 第二个是请求的地址  第三个是同步或者异步
xhr.open('POST',"txt.php",false)
//如果是post请求  必须要写请求头
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded") //设置请求头
//第三步  为xhr.onreadystatechange  设置监听事件
xhr.onreadystatechange = function(){
    if(xhr.readyState == 4) {
    if(xhr.status == 200){
    alert(xhr.responseTwxt)
//readyState  0 请求未初始化  刚刚实例化XMLHttpRequest
//readyState  1 客户端与服务器建立链接  调用open方法
//readyState  2 请求已经被接收
//readyState  3 请求正在处理中
//readyState  4 请求成功
            }
       }
}
// 第四步 发送请求数据  调用send 发送请求
xhr.send(data);


//在网上收集的另外一个例子
var Ajax={
  get: function(url, fn) {
    // XMLHttpRequest对象用于在后台与服务器交换数据   
    var xhr = new XMLHttpRequest();            
    xhr.open('GET', url, true);
    xhr.onreadystatechange = function() {
      // readyState == 4说明请求已完成
      if (xhr.readyState == 4 && xhr.status == 200 || xhr.status == 304) { 
        // 从服务器获得数据 
        fn.call(this, xhr.responseText);  
      }
    };
    xhr.send();
  },
  // datat应为'a=a1&b=b1'这种字符串格式，在jq里如果data为对象会自动将对象转成这种字符串格式
  post: function (url, data, fn) {
    var xhr = new XMLHttpRequest();
    xhr.open("POST", url, true);
    // 添加http头，发送信息至服务器时内容编码类型
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");  
    xhr.onreadystatechange = function() {
      if (xhr.readyState == 4 && (xhr.status == 200 || xhr.status == 304)) {
        fn.call(this, xhr.responseText);
      }
    };
    xhr.send(data);
  }
}
```

1. open(method, url, async) 方法需要三个参数:

　 method：发送请求所使用的方法（GET或POST）；与POST相比，GET更简单也更快，并且在大部分情况下都能用；然而，在以下情况中，请使用POST请求：

- 无法使用缓存文件（更新服务器上的文件或数据库）
- 向服务器发送大量数据（POST 没有数据量限制）
- 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

　url：规定服务器端脚本的 URL(该文件可以是任何类型的文件，比如 .txt 和 .xml，或者服务器脚本文件，比如 .asp 和 .php （在传回响应之前，能够在服务器上执行任务）)；

　async：规定应当对请求进行异步（true）或同步（false）处理；true是在等待服务器响应时执行其他脚本，当响应就绪后对响应进行处理；false是等待服务器响应再执行。

2. send() 方法可将请求送往服务器。

3. onreadystatechange：存有处理服务器响应的函数，每当 readyState 改变时，onreadystatechange 函数就会被执行。

4. readyState：存有服务器响应的状态信息。

- 0: 请求未初始化（代理被创建，但尚未调用 open() 方法）
- 1: 服务器连接已建立（`open`方法已经被调用）
- 2: 请求已接收（`send`方法已经被调用，并且头部和状态已经可获得）
- 3: 请求处理中（下载中，`responseText` 属性已经包含部分数据）
- 4: 请求已完成，且响应已就绪（下载操作已完成）

5. responseText：获得字符串形式的响应数据。

6. setRequestHeader()：POST传数据时，用来添加 HTTP 头，然后send(data)，注意data格式；GET发送信息时直接加参数到url上就可以，比如url?a=a1&b=b1。

## fetch

偷懒直接使用别人的文章了。原文：http://t.csdn.cn/IxhCY

fetch 是 XMLHttpRequest 的升级版，使用[js脚本](https://so.csdn.net/so/search?q=js脚本&spm=1001.2101.3001.7020)发出网络请求，但是与 XMLHttpRequest 不同的是，fetch 方式使用 Promise，相比 XMLHttpRequest 更加简洁。所以我们告别XMLHttpRequest，引入 fetch 如何使用？

## 一、fetch介绍

fetch() 是一个全局方法，提供一种简单，合理的方式跨网络获取资源。它的请求是基于 Promise 的，需要详细学习 Promise ，请点击[《 Promise详解 》](https://www.toutiao.com/i6985411051487560205/?group_id=6985411051487560205)。它是专门为了取代传统的 [xhr](https://so.csdn.net/so/search?q=xhr&spm=1001.2101.3001.7020) 而生的。

**1.1、fetch使用语法**

```cobol
fetch(url,options).then((response)=>{



//处理http响应



},(error)=>{



//处理错误



})
```

url ：是发送网络请求的地址。

options：发送请求参数,

- body - http请求参数
- mode - 指定请求模式。默认值为cros:允许跨域;same-origin:只允许同源请求;no-cros:只限于get、post和head,并且只能使用有限的几个简单标头。
- cache - 用户指定缓存。
- method - 请求方法，默认GET
- signal - 用于取消 fetch
- headers - http请求头设置
- keepalive - 用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据。
- credentials - cookie设置，默认omit，忽略不带cookie，same-origin同源请求带cookie，inclue无论跨域还是同源都会带cookie。

**1.2、response 对象**

fetch 请求成功后，响应 response 对象如图：

![img](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/289f0101268a918580cc16513b279654_MD5.png)



- status - http状态码，范围在100-599之间
- statusText - 服务器返回状态文字描述
- ok - 返回布尔值，如果状态码2开头的，则true,反之false
- headers - 响应头
- body - 响应体。响应体内的数据，根据类型各自处理。
- type - 返回请求类型。
- redirected - 返回布尔值，表示是否发生过跳转。

**1.3、读取内容方法**

response 对象根据服务器返回的不同类型数据，提供了不同的读取方法。分别有：

1. response.text() -- 得到文本字符串
2. response.json() - 得到 json 对象
3. response.blob() - 得到二进制 blob 对象
4. response.formData() - 得到 fromData 表单对象
5. response.arrayBuffer() - 得到二进制 arrayBuffer 对象

上述 5 个方法，返回的都是 promise 对象，必须等到异步操作结束，才能得到服务器返回的完整数据。

**1.4、response.clone()**

stream 对象只能读取一次，读取完就没了，这意味着，上边的五种读取方法，只能使用一个，否则会报错。

因此 response 对象提供了 clone() 方法，创建 respons 对象副本，实现多次读取。如下：将一张图片，读取两次：

```
const response1 = await fetch('flowers.jpg');
const response2 = response1.clone();

const myBlob1 = await response1.blob();
const myBlob2 = await response2.blob();

image1.src = URL.createObjectURL(myBlob1);
image2.src = URL.createObjectURL(myBlob2);
```



**1.5、response.body()**

body 属性返回一个 ReadableStream 对象，供用户操作，可以用来分块读取内容，显示下载的进度就是其中一种应用。

```
const response = await fetch('flower.jpg');
const reader = response.body.getReader();
while(true) {
  const {done, value} = await reader.read();
  if (done) {
    break;
  }
  console.log(`Received ${value.length} bytes`)
}
```



response.body.getReader() 返回一个遍历器，这个遍历器 read() 方法每次都会返回一个对象，表示本次读取的内容块。

## 二、请求时 POST 和 GET 分别处理

请求方式不同，传值方式也不同。xhr 会分别处理 get 和 post 数据传输，还有请求头设置，同样 fetch 也需要分别处理。

**2.1、get 方式**

只需要在url中加入传输数据，options中加入请求方式。如下面代码所示：

```
<input type="text" id="user"><br>
<input type="password" id="pas"><br>
<button οnclick="login()">提交</button>
<script>
 function login(){
  fetch(`http://localhost:80/fetch.html?user=${user.value}&pas=${pas.value}`,{
   method:'GET'
  }).then(response=>{
   console.log('响应',response)
  })
 }
</script>
```



**2.2、post 方式**

使用 post 发送请求时，需要设置请求头、请求数据等。

将上个实例，改写成 post 方式提交数据，代码如下：

```
fetch(`http://localhost:80/ES6练习题/53fetch.html`,{
 method:'POST',
 headers:{
  'Content-Type':'application/x-www-form-urlencoded;charset=UTF-8'
 },
 body:`user=${user.value}&pas=${pas.value}`
 }).then(response=>{
  console.log('响应',response)
})
```



如果是提交json数据时，需要把json转换成字符串。如

```js 
body:JSON.stringify(json)js
```

如果提交的是表单数据，使用 formData转化下，如：

```js
body:new FormData(form)
```

上传文件，可以包含在整个表单里一起提交，如：

```js
const input = document.querySelector('input[type="file"]');

const data = new FormData();
data.append('file', input.files[0]);
data.append('user', 'foo');

fetch('/avatars', {
  method: 'POST',
  body: data
});
```



上传二进制数据，将 bolb 或 arrayBuffer 数据放到body属性里，如：

```js
let blob = await new Promise(resolve =>   
  canvasElem.toBlob(resolve,  'image/png')
);

let response = await fetch('/article/fetch/post/image', {
  method:  'POST',
  body: blob
});
```

## 三、fetch 常见坑

**3.1、fetch兼容性**

fetch原生支持率如图：

![img](https://cdn.jsdelivr.net/gh/Dmup227/Dimags@main/D:%5CDocument%5Cimages202308170953457.png)



fetch 是相对较新的技术，IE浏览器不支持，还有其他低版本浏览器也不支持，因此如果使用fetch时，需要考虑浏览器兼容问题。

解决办法：引入 polyfill 完美支持 IE8 以上。

- 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: es5-shim, es5-sham
- 引入 Promise 的 polyfill：es6-promise
- 引入 fetch 探测库：fetch-detector
- 引入 fetch 的 polyfill： fetch-ie8
- 可选：如果你还使用了 jsonp，引入 fetch-jsonp
- 可选：开启 Babel 的 runtime 模式，现在就使用 async/await

polyfill 的原理就是探测fetch是否支持，如果不支持则用 xhr 实现。支持 fetch 的浏览器，响应中文会乱码，所以使用 fetch-detector 和 fetch-ie8 解决乱码问题。

**3.2、fetch默认不带cookie**

传递cookie时，必须在header参数内加上 credentials:'include'，才会像 xhr 将当前cookie 带有请求中。

**3.3、异常处理**

fetch 不同于 xhr ，xhr 自带取消、错误等方法，所以服务器返回 4xx 或 5xx 时，是不会抛出错误的，需要手动处理，通过 response 中的 status 字段来判断。
