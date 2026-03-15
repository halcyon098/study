### Python中的字符串类型
字符串（String）指的是一个或多个字符的组合。在Python这门语言中，字符串必须由双引号""或者单引号''包围，格式如下：
"字符串内容"
'字符串内容'
字符串中的内容可以随便书写，包含数字、字母、标点、特殊符号、中文、英文等等。
下面都是合法的字符串：

- “12342”
- '12345abcd'
- "[http://pythonjishu.com/](http://pythonjishu.com/)"
- "Python技术站"

字符串用双引号、单引号包裹都可以，没有任何区别。
### 转义字符
如果字符串内容中出现引号时，会怎么样？
你可以直接输出这句话：
'I'm Metahuber!'
我们会发现解释器会报错：
SyntaxError: invalid syntax. Perhaps you forgot a comma?
原因是解释器将'I'当成了一个字符串，所以之后的内容被认定为是异常字符，导致语法错误而报错。
所以当字符串中出现引号时，就需要我们做特殊处理。
这时候我们就需要使用转义符反斜杠'\'对引号进行转义。如下：
'I\'m Metahuber!'
除了对引号进行转义的反斜杠‘\’外，还有给字符添加换行、回车等作用的转义符。关于转义字符的更多内容，可点击《[Python转义字符及用法](http://pythonjishu.com/python-escape-character/)》一文了解。
### Python长字符串
我们已知，**转义字符**可以在字符串中增加换行、缩进等内容，而Python中的长字符串，指的是**不需要转义符**，所有字符内容都直接原文本输出的写法，包括单双引号、空格、回车、缩进、空白符等等，你输入什么，就显示什么。
Python中的长字符串使用三个双引号"""或三个单引号'''包围，格式如下：
"""长字符串内容"""
'''长字符串内容'''
在长字符串中写入单引号或双引号不会导致解释器异常，而会原样输出里面的内容。
所以，当我们的程序中需要定义有大段的文本内容的字符串变量时，优先推荐使用长字符串形式，这样可以让我们从转义符的使用中解放。
实例如下：
```python
long_str='''Hello！My name is “Metahuber”。
This is My Website “http://pythonjishu.com/”。'''
print(long_str)
```
输出如下：
Hello！My name is "Metahuber"。
This is My Website "[http://pythonjishu.com/](http://pythonjishu.com/)"。
### Python原始字符串
当我们的变量是一个Windows路径‘C:\Program Files\Python 3.11\[python](https://pythonjishu.com/tag/python/).exe’这样的字符串时，如果使用普通的定义字符串的方式，直接这么写解释器肯定会报错，我们需要在每一个反斜杠前再加一个反斜杠，这样才不会出现解析错误。如下：
```python
print('C:\\Program Files\\Python 3.11\\python.exe')
```
这种写法的坏处是在编程中很容易出错，需要写得特别谨慎。
为了解决这个问题，Python支持原始字符串的写法。顾名思义，原始字符串的作用与长字符串相同，是将字符串中的内容保持原始的样子输出。
具体的写法是在字符串开头加上r前缀，如：
r_str=r'C:\Program Files\Python 3.11\python.exe'
print(r_str)
输出结果：
C:\Program Files\Python 3.11\python.exe
可以看到，这种写法是正确的，解释器并没有出现异常。
需要注意的是，原始字符串中的“\”仍然会对引号进行转义。所以原始字符串最后一个字符不能是反斜杠，因为它会对结尾处的引号进行转义，导致报错。
比如：
r_str2=r'C:\Program Files\Python 3.11\python.exe\'
print(r_str2)
此时会出现异常：
SyntaxError: unterminated string literal (detected at line 1)
解决这个问题也很简单，可以由两种方式。
一种是不要使用原始字符串，养成使用长字符串的方式书写。
另外一种方法比较麻烦，就是在字符后面追加反斜杠。写法如下：
r_str3=r'C:\Program Files\Python 3.11\python.exe' '\'
print(r_str3)
我们定义变量时，先写了最后字符不带反斜杠的原始字符串，然后添加一个空格，又单独写了个包含转义字符的反斜杠字符串。Pyhton会将这两个字符串自动拼接在一起，所以以上代码的输出结果是正常的：
C:\Program Files\Python 3.11\python.exe\
