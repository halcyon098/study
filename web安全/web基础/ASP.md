ASP即Active Server Pages，是Microsoft公司开发的服务器端脚本环境，可用来创建动态交互式网页并建立强大的web应用程序。当服务器收到对ASP文件的请求时，它会处理包含在用于构建发送给浏览器的[HTML](https://baike.baidu.com/item/HTML/97049?fromModule=lemma_inlink)（_Hyper Text Markup Language，超文本标记语言_）网页文件中的服务器端脚本代码。除服务器端脚本代码外，ASP文件也可以包含文本、HTML（_包括相关的客户端脚本_）和com组件调用。
ASP简单、易于维护 ， 是小型页面应用程序的选择 ，在使用DCOM （_Distributed Component Object Model_）和 MTS（_Microsoft Transaction Server_）的情况下， ASP甚至可以实现中等规模的企业应用程序。

环境(固定)：windows+iis+asp+access(sqlsever)
asp不能在Linux环境下运行,且基本已经被淘汰。环境搭建需要较老版本的windows
## CMS
动易([http://www.powereasy.net/](http://www.powereasy.net/))
乔客([http://www.joekoe.com/](http://www.joekoe.com/))
风讯([http://www.foosun.cn/](http://www.foosun.cn/))
TSYS([http://www.tsyschina.com/](http://www.tsyschina.com/))
新云( [http://www.newasp.cn/](http://www.newasp.cn/))
科汛([http://www.kesion.com/](http://www.kesion.com/) )
创力([http://www.aspoo.com/](http://www.aspoo.com/))
JTBC([http://www.jetiben.com/](http://www.jetiben.com/))
KINGCMS([http://www.kingcms.net/](http://www.kingcms.net/))
Feitec([http://www.feitec.com/](http://www.feitec.com/))
## 漏洞
### 数据库

1. mdb下载

使用默认数据库路径（或者知道修改后的数据库路径）可以下载数据库文件获取管理员账号密码

2. ASP后门植入连接

流程：留言板写入一句话木马–菜刀连接数据库地址
原理：网站开启了asp解析，直接访问数据库文件会返回信息，写入一句话木马后进行连接
### 中间件

1. IIS短文件名探针

原理：Windows系统为了兼容16位MS-DOS程序，为文件名较长的文件和文件夹生成了对应的Windows 8.3短文件名。比如文件名direct~1.asp中间有一个波浪号，这种就是短文件名了。
使用脚本扫描网站目录造成敏感信息泄露（不同于使用字典扫描）
参考连接：[https://www.freebuf.com/articles/web/172561.html](https://www.freebuf.com/articles/web/172561.html)

2. IIS文件上传解析

流程：发现网站存在上传点–上传asp木马–若存在文件名监测通过修改文件类型绕过检测–shell工具连接
例：1.jpg文件无法解析，修改为1.asp;.jpg即可绕过。或者1.jpg文件放在a.asp文件下也可进行解析

3. IIS配置目录读写

IS put写入漏洞
当网站为IIS6.0且开启写入权限、开启web服务拓展WebDAV则存在此漏洞
可以使用工具利用iis写权限可以配合解析漏洞
