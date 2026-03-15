## 环境

参考博客：https://www.oryoy.com/news/shi-yong-vscode-jin-xing-php-xiang-mu-de-yuan-cheng-bu-shu-yu-diao-shi-ji-qiao-xiang-jie.html

### vps

作为网站的运行环境，使用小皮面板

官网下载命令：https://www.xp.cn/download

```bash
if [ -f /usr/bin/curl ];then curl -O https://dl.xp.cn/dl/xp/install.sh;else wget -O install.sh https://dl.xp.cn/dl/xp/install.sh;fi;bash install.sh
```

![image-20251124194108002](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20251124194108002.png)



> ```
>  面板版本: v1.3.27                                                                             
>  外网面板地址: http://113.45.128.31:51910/8f1999                                               
>  ****** 检测到本机有3个内网IP ******                                                           
>  内网面板地址-1: http://192.168.14.128:51910/8f1999                                            
>  内网面板地址-2: http://172.17.0.1:51910/8f1999                                                
>  内网面板地址-3: http://172.24.0.1:51910/8f1999                                                
>  面板账号: 87309d07                                                                            
>  面板密码: 2b2a0787   
> ```



### VSCode

- 首先确保你已经安装了最新版本的 VSCode。
- 安装以下插件：
  - **PHP Debug**：用于 PHP 代码的调试。
  - **PHP IntelliSense**：提供 PHP 代码智能提示。
  - **Remote - SSH**：用于远程连接服务器。