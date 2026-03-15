使用eNsp

新建两个路由设备

![image-20221230145452524](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20221230145452524.png)

启动设备，双击打开命令行

### AR1(命令均为缩写)

sy(进入系统视图)

int g0/0/0(进入AR1的G0/0/0端口)

ip add 192.168.1.1 255.255.255.0（为该端口配置IP地址为192.168.1.1 子网掩码为255.255.255.0）

q(退出当前视图命令缩写)

q

save(保存设备配置)

### AR2

sy(进入系统视图)

int g0/0/0(进入AR2的G0/0/0端口)

ip add 192.168.1.2 255.255.255.0（为该端口配置IP地址为192.168.2.1 子网掩码为255.255.255.0）

q(退出当前视图命令缩写)

q

save(保存设备配置)

### 测试设配是否能连通

例如：进入AR2设备命令行

ping  192.168.1.1（AR1的IP地址）

![image-20221230150423110](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/image-20221230150423110.png)

出现该图则设备可以通信