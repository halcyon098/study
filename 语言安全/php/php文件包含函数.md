empty()函数，判断变量内容是否为空。

文件包含函数：

require()

require_once()

include()

include_once()

区别：

- include和require的区别在于，如果包含的文件不存在的时候，include只是报警告错误，而不影响自身代码执行；而require会报致命错误，而且中断代码执行
- include和include_once区别：include不论如何都会执行包含操作，而include_once会记录是否已经包含过对应文件，对同一文件多次包含只操作一次（对于函数/类这种结构不允许重复的，是个好方法）。
  

