# log4j

Log4j漏洞

 本身是Apache日志功能，他有个日志遍历的功能，当碰到${jndi:// } ，会遍历执行，JNDI功能又可以使用ldap或者rmi来引入class文件，我们只需要在class文件中加入需要执行的恶意代码，就可以造成代码注入。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/073846a4866a27b3c7385ec0e8992712_MD5.jpg)