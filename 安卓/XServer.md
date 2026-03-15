https://www.freebuf.com/sectool/238841.html

下载安装https://github.com/monkeylord/XServer

框架中要启动这个插件。

使用usb连接电脑和手机

命令：

```cmd
# 列出所有连接的设备（包括 USB 连接的设备和通过网络连接的设备）
adb devices

# 查看指定设备的 Wi-Fi 接口（wlan0）的网络配置信息，包括 IP 地址
adb shell ifconfig wlan0

# 将设备切换到网络调试模式，并监听端口 5555
# 这一步通常用于将设备从 USB 调试切换到网络调试
adb tcpip 5555

# 通过网络连接到指定的设备
# 这里假设设备的 IP 地址是 10.92.100.87，端口是 5555
adb connect 10.92.100.87:5555

# 在指定设备上设置端口转发规则
# 将本地端口 8000 转发到设备上的远程端口 8000
adb -s 10.92.100.87:5555 forward tcp:8000 tcp:8000

# 查看指定设备上所有已设置的端口转发规则
# 这将列出设备 10.92.100.87:5555 上的所有转发规则
adb -s 10.92.100.87:5555 forward --list
```

本地访问127.0.0.1：8000，即可成功hook的web界面了，我没成功。