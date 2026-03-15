**插件说明： 一款适用于Burp的验证码识别插件**

项目地址：[https://github.com/f0ng/captcha-killer-modified](https://github.com/f0ng/captcha-killer-modified)
## 步骤概述

1. _下载captcha-killer-modified.jar包_
2. _burp安装captcha-killer-modified.jar包_
3. _搭建验证码识别服务（下载安装ddddocr）_
4. _启动验证码识别接口（codereg.py）_
5. _配置参数进行使用_
## 一、下载captcha-killer-modified.jar包
打开项目地址，点击右侧releases进入下载最新版本的jar包，我选择的是0.21-jdk11版，也可以直接点击下边地址下载（使用的burp的jdk版本是jdk17，win11操作系统）。

https://github.com/f0ng/captcha-killer-modified/releases/download/0.21-beta/captcha-killer-modified-0.21-beta-jdk11.jar

## 二、下载项目源码
主要是用codereg.py（需要提前准备python3的环境），且该项目源码包含用法和常见问题文档。

也可以直接点击下方地址下载。

https://github.com/f0ng/captcha-killer-modified/blob/main/codereg.py
## 三、安装插件

下载插件文件后，启动Burpsuite，点击Extensions模块，在Burp Extensions一栏点击Add，选择刚下好的jar包。并点击下一步完成安装。
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143619783.png)
## 四、安装ddddocr服务，并配置接口。
验证码识别接口用到的是Python库中的ddddocr，Web服务用 到的是aiohttp。 因此我们需要使用到Python环境，本次教程中使用的版本为 Python3.10.4（较新的python3都可以）。

执行命令：
```cmd
pip install -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com ddddocr aiohttp
```
也可以单独安装ddddocr，命令：
```cmd
pip install ddddocr
```
注：网络环境不同，可能第一种命令安装较快，第二种速度很慢，建议一种网速不好就换另一种。
## 五、安装完成后执行codereg.py

打开之前下载解压的项目源码的路径，在该路径下打开dos窗口（shift+鼠标右键，选择在此处打开powershell；），在dos界面输入“**python coderg.py**”，出现以下内容即代表验证码识别服务已搭建完成。
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143805226.png)

## 六、实战演示
1.获取目标验证码接口：
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143932707.png)

2.burp拦截验证码接口的请求包：
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143943378.png)


3.鼠标右键发送至captcha-killer-modified插件：
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143950436.png)

4.点击 “获取” 按钮，获取验证码

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209143958288.png)
从上图中可以看到，已经获取到验证码的返回包。
5.接下来就是设置 关键字 和 响应提取
注：关键字 是给插件判断从哪开始是验证码图形的位置，响应提取 是用来提取 “判断验证码是否失效的关键字”。
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144017955.png)

爆破时候网站后台会根据这个关键字判断验证码是否失效，所以需要每次进行提取，然后在intruder爆破页面进行调用，后边会有演示如何调用。
**关键字提取使用正则表达式**。

6.配置验证码识别
将下面的url粘贴进接口url输入框：
```url
http://127.0.0.1:8888
```

在 Requst template 下方空白处鼠标右键选择“ddddocr”，配置好请求模板。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144116282.png)

点击 “识别”，右侧Response raw栏会出现请求的结果，顺利的话，验证码就会出现在返回内容下方。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144125330.png)

7.爆破登录页面验证码识别相关配置
将Intruder 模块的 Attack type 设置为 “Pitch fork”。

此处我进行爆破账号和验证码的演示，共标记了两处爆破点，如图：
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144134468.png)


在Payloads一栏进行payload设置，payload set 选择 “2”，payload type 选择 “Extension-generated“（payload 1的设置不再演示，不懂的话自行百度）。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144143857.png)

在Payload Options栏点击Select generate ，选择 captcha-killer-modified，点击OK。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144150823.png)

在Positions一栏，验证码id处替换对应关键字，用@captcha-killer-modified@进行调用先前提取的关键字。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144157667.png)
点击Start attack，看效果。

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/file-20241209144209017.png)

如在使用中有别的问题，请查看该项目的 README.md 和 FAQ.md 文档。

教程中用到项目的官方链接

captcha-killer-modified：https://github.com/f0ng/captcha-killer-modified

ddddocr：https://github.com/sml2h3/ddddocr