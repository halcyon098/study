# 初识js

+ js的文件后缀为 “**.js**”
+ js代码的运行必须依靠html代码，使用<script></script>

## js的引用方式

回忆css样式如何在html里应用，有三种方式：外链样式，内部样式，行内样式。**同css一样**，js在html内有同样三种应用样式。**但与css不同的是**，引用css样式的<style>标签只能在头部标签<head>标签内，引用js的<script>标签的引入位置不受限制，因此一般习惯写在<body>标签下面。

1. 外链样式。使用<script></script>标签内src属性

```  <!--html文

   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title>初识js</title>
   		<!--外链式-->
   		<script src="jsdemo.js"></script>
   	</head>
   	<body>
   	</body>
   </html> ```

   
js文件：jsdemo.js
    alert('祖国万岁');



```

2. 内部样式

``` <!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>初识js</title>
		<!--外链式-->
		<!--<script src="jsdemo.js"></script>-->
		
		<!--内部样式-->
		<script>
			alert('祖国万岁');
		</script>
	</head>
	<body>
	</body>
</html>
```

3. 行内样式

``` <!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>初识js</title>
		<!--外链式-->
		<!--<script src="jsdemo.js"></script>-->
		
		<!--内部样式-->
		<!--<script>
			alert('祖国万岁');
		</script>-->
	</head>
	<body>
		<button onclick="alert('祖国万岁')">威武</button>
	</body>
</html>
```

## js的输出

JavaScript是没有任何输出或打印函数，但可以通过不同的方式来输出数据。

1. 使用**window.alert()**弹出警告框。
```html
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title>js的输出方式</title>
   	</head>
   	<body>
   		<h2>我的第一个页面</h2>
   		<p>我的第一个段落</p>

   		<!--先弹出js代码输出内容，再显示html文本-->
   		<script>
   			window.alert(5+6);
   		</script>
   	</body>
   </html>
```

2. 使用**document.write()**方法将内容写到HTML文档中。

```  html
<html>
	<head>
		<meta charset="utf-8">
		<title>js的输出方式</title>
	</head>
	<body>
		<h2>我的第一个页面</h2>
		<p>我的第一个段落</p>
		
		
		<script>
			document.write(Date());
            //请使用 document.write() 仅仅向文档输出写内容。
			//如果在文档已完成加载后执行 document.write，整个 HTML 页面将被覆盖。
		</script>
	</body>
</html>
```

3. **innerHTML**写入到HTML元素中
``` html
<html>
	<head>
		<meta charset="utf-8">
		<title>js的输出方式</title>
	</head>
	<body>
		<h2>我的第一个页面</h2>
		<p id="demo">我的第一个段落</p>
		
		
		<script>

			//getElementById(),里面的id名需要加 “” 。innerHTML后面是 = ，不是（）。
			document.getElementById("demo").innerHTML="段落已修改";
            //document.getElementById("demo") 是使用 id 属性来查找 HTML 元素的 JavaScript 代码 
			//innerHTML = "段落已修改。" 是用于修改元素的 HTML 内容(innerHTML)的 JavaScript 代码。
			
		</script>
	</body>
</html>

```

4. 使用 **console.log()** 写入到浏览器的控制台。

``` html
<html>
	<head>
		<meta charset="utf-8">
		<title>js的输出方式</title>
	</head>
	<body>
		<h2>我的第一个页面</h2>
		<p>我的第一个段落</p>

	<script>
		
		a = 2;
		b = 3;
		c = a+b;
		console.log(c);
		
	</script>
</body>
```

## js数组循环

