# DHCP

作用：自动给电脑，手机配置IP地址

## eNsp

设备：AR2220路由器，一台交换机，三台PC

![image-20230101141154340.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/117a5cb54f3eeb8a63dc94c59d675710_MD5.png)

首先在未开启DHCP功能时查看PC1的IP地址

![image-20230101141341399.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/0be88f84b552db570ee71c9ecdfcdcfa_MD5.png)
可以看到PC设备未获取IP地址

### 在路由器开启DHCP功能
启动设备后打开AR1的命令行，先手动配置路由器的IP地址
> #在系统视图下
> #打开路由器DHCP功能
> dhcp enable  
> #进入路由器与交换机链接的端口g0/0/0
> int g0/0/0  
> #选择端口
> dhcp select interface
> 

将三台设备的IP配置调成DHCP模式，再次查看PC设备的IP，即可看到已经自动配置IP地址
