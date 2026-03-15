# xss-labs

xss:跨站脚本攻击

参考博客：https://blog.csdn.net/l2872253606/article/details/125638898

## 基本绕过

```js
<script>alert('xss')</script>
```

当网页不进行任何过滤时使用,js弹窗函数

### **闭合绕过**

```js
">  <script>alert()</script>  <"
```

当回显中将<>转义时，观察到value属性仍能正常显示时，闭合双引号绕过

```php
htmlspecialchars($str) #php的html实体转化函数
```

### **onfocus事件绕过**

htmlspecialchars函数只针对<>大于小于号进行html实体化，因此当闭合也不能使用时可以考虑使用onfocus事件

onfocus事件在元素获得焦点时触发，最常与 <input>、<select> 和 <a> 标签一起使用，以上面图片的html标签<input>为例，<input>标签是有输入框的，简单来说，onfocus事件就是当输入框被点击的时候，就会触发myFunction()函数，然后我们再配合javascript伪协议来执行javascript代码
```js
' onfocus=javascript:alert() '
```

