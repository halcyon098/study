作用：域名解析系统，将服务器的域名转换成IP地址。例如：www.baidu.com -> **104.193.88.77**
## eNsp
该实验设置在DHCP实验设置之上添加
![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/89b560c0443b10099ecceb7ea8707dbb_MD5.png)
在DHCP实验的基础上添加一台服务器
![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/538a1834af509d558ee6d127f938a9cd_MD5.png)
![image.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/8f8ca437a0683f262e3a3ff5561eb4a1_MD5.png)
手动添加域名和相应的IP地址
在路由器上添加域名服务器
> ｓｙ
> ｉｎｔ　ｇ０／０／０
> ＃将添加ｄｎｓ服务器IP地址
> dhcp server dns-list 192.168.1.100

启动服务器，保存服务器IP地址，将ｐｃ设备调成静态模式后再改为DHCP模式重新分配地址
