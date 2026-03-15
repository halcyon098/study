IIS
解析漏洞
6.x
*.asp；.png解析为asp文件
将*.asp/路径下文件解析成asp
罕见文件名解析为asp，例如，asa，cer，cdx
7.x
*.asp/.php文件解析为php


http.sys漏洞，wondows系统漏洞，进行内存读取，dos，使用msf模块进行攻击，指纹为请求头添加range:bytes=0-xxxxx，返回响应码416，和Requested Range Not Satisfiable

IIS-PUT任意文件写入
条件，webdve扩展开启，目录有写入权限，bp修改请求方式，使用put请求创建文件写入文件内容

短文件名猜解，存在404，不存在400
