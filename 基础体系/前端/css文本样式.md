# css文本样式

## 1.字体样式属性

### font-size:字号大小

该属性用于设置字号，可使用相对长度单位，也可以使用绝对长度单位。

1. em :相对于当前对象内文本的字体尺寸
2. px :像素,最常用，**推荐使用**
3. in :英寸
4. cm :厘米
5. mm :毫米

``` html
p{font-size:12px;}
```

### font-family:字体

font-fanily 属性用于设置字体。网页中常用的字体有宋体，微软雅黑，黑体等。

` p{font-size:12px;}`

可以同时指定多个字体，中间以逗号隔开，表示如果浏览器不支持第一种字体则会尝试下一个直到找到合适的字体

` body{font-family:"华文彩云","宋体","黑体";}`

使用font-family设置字体时，需注意以下几点：

+ 各种字体之间必须使用英语状态下的逗号隔开。
+ 中文字体需要加英语状态下的引号，英语字体一般不需要加引号。当需要设置英语字体，**英语字体名必须位于中文字体名之前**。
+ 如果字体名中包含空格，#，$等符号，字体名加英文状态下的单引号或双引号
+ 尽量使用系统默认字体。

### font-weight:字体粗细

属性值：

+ Normal:默认值，定义标准的字符
+ bold：粗体
+ bolder：更粗的字体
+ lighter：更细的字体
+ 100~900：定义由粗到细，400=normal，700=bold

### font-style:字体风格

属性值：

+ normal:默认
+ italic：斜体
+ oblique：斜体

italic和oblique都用于定义斜体，显示上无区别，但常用italic

### font:综合设置字体样式

语法格式：

`选择器{font:font-style font-weight font-size/line-height font-family;}`

使用时必须按上面语法格式中的顺序书写，以空格隔开。line-height是指行高。

```html
p{
	font-family:Arial,"宋体";
	font-size:30px;
	font-style:italic;
	font-weight:bold;
	line-height:40px;
}
```

等价于：

```html
P{font:italic bold 30px/40px Arial,"宋体";}
```

### @font-face规则

用于定义服务器字体

```html
@font-face{
	font-family:字体名称;
	src:字体路径;
}
```

### world-wrap:属性

用于实现长单词和URL地址的自动换行。

` 选择器{world-wrap:属性值;}`

属性值：

+ normal:只在允许的断字点换行（浏览器保持默认处理）
+ break-world:在长单词或URL地址内部进行换行

## 2.文本外观属性

### color：文本颜色

取值方式：

+ 预定义的颜色名，如 red,green,blue等
+ 十六进制，如#ff0000,#ff6600,#66cc00等。**实际中最常用**。十六进制的 颜色值是以#开头的6位十六进制数值组成，每两位为一个颜色分量，表示 红，黄，蓝 3个分量，当3个分量的2位十六进制相同时可使用缩写，如：%FF6600  缩写成#F60, #FF0000 —— #F00,#FFFFFF —— #FFF.
+ RGB代码。如红色可以表示为rgb(255,0,0)或rgb(100%,0%,0%)。

### letter-spacing:字间距

属性值为不同单位的值，允许使用负值，默认为normal。一般用于汉语，可用于英语，但设置的是字母间距。

### world-spacing

定义英语单词之间的间距，对中文无效，属性值同letter-spacing相同

### line-hight:行间距

常用的属性值单位有：px(像素) ,em(相对值), %(百分比)，常用像素（px）。

### text-transform:文本转换

用于控制英文字符大小写，可用属性值：

+ none:不转换（默认值）
+ capitalize:首字母大写
+ uppercase:全部转换成大写。
+ lowercase:全部转换成小写。

### text-decoration:文本装饰

+ none：没有装饰
+ underline：下划线
+ overline：上划线
+ line-through:删除线

### text-align:水平对齐方式

- left:左对齐（默认值）
- right：右对齐
- center：居中对齐

**该属性仅适用于块级元素，对行内元素无效**

### text-indent:文本缩进

建议使用em为设置单位，**仅适用于块级元素，对行内元素无效**

### white-space:空白符处理

- normal：常规（默认），文本中空格，空行无效，满行自动换行。
- pre:预格式化，原样保留。
- nowrap:空行空格无效，强制文本不能换行，除非遇到标签`<br/>`。内容超出边界也不换行，浏览器自动增加滚动条。

### text-shadow:阴影效果

`选择器{text-shadow:h-shadow v-shadow blur color;}`

- h-shadow:水平阴影距离
-  v-shadow：垂直阴影距离
-  blur：模糊半径
-  color：阴影颜色

### text-overflow:标示对象内文本的溢出

`选择器{text-overflow:属性值;}`

- elipsis:用省略号标签“···”标示被修剪文本，插入位置为最后一个字符
- clip:修剪溢出文本，不显示省略标签

设置具体步骤：

1. 为包含文本的对象定义宽度
2. 应用"white-space:nowrap;"样式强制文本不能换行
3. 应用"overflow-hidden;"样式隐藏溢出文本
4. 应用"text-overflow:elipsis;"样式显示省略标签