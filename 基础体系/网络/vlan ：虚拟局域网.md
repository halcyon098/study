# vlan ：虚拟局域网

![image-20230111103722168](C:\Users\丁明明\AppData\Roaming\Typora\typora-user-images\image-20230111103722168.png)

vlan技术用于交换机设备，用于将网段中的设备进行隔离，当某台设备被病毒攻击后不会传播到相连的其他设备中去。

 ## 交换机端口有三种工作模式

分别是Access，Hybrid，Trunk。
  Access类型的端口只能属于1个VLAN，一般用于连接计算机的端口；
  Trunk类型的端口可以允许多个VLAN通过，可以接收和发送多个VLAN的报文，一般用于交换机之间连接的端口；
  Hybrid类型的端口可以允许多个VLAN通过，可以接收和发送多个VLAN的报文，可以用于交换机之间连接，也可以用于连接用户的计算机。 
  Hybrid端口和Trunk端口在接收数据时，处理方法是一样的，唯一不同之处在于发送数据时：Hybrid端口可以允许多个VLAN的报文发送时不打标签，而Trunk端口只允许缺省VLAN的报文发送时不打标签。

## 命令

 ###  Access

在eNsp中配置四台pc设备的IP和子网掩码

```Huawei
<Huawei>sy
[Huawei]vlan 10
[Huawei-vlan10]q
[Huawei]vlan 20
[Huawei-vlan20]q
#显示交换机中已存vlan情况
[Huawei]display vlan
[Huawei]int g0/0/1	
#将交换机端口切换成Access模式
[Huawei-GigabitEthernet0/0/1]port link-type access
#将该端口（g/0/0/1）添加到vlan 10中
[Huawei-GigabitEthernet0/0/1]port default vlan 10

```

进入其他端口重复上面的命令即可。可用ping命令观察配置vlan前后不同vlan中pc设备的通信情况。  

### Trunk

![image-20230112125647732](vlan ：虚拟局域网.assets/image-20230112125647732.png)

```Huawei
<Huawei>sy
[Huawei]vlan 10
[Huawei-vlan10]
[Huawei-vlan10]vlan 20
[Huawei-vlan20]q
[Huawei] int g0/0/2
[Huawei-GigabitEthernet0/0/2]port link-type access 
[Huawei-GigabitEthernet0/0/2]port default vlan 10
[Huawei-GigabitEthernet0/0/2]int g0/0/3
[Huawei-GigabitEthernet0/0/3]port link-type access
[Huawei-GigabitEthernet0/0/3]port default vlan 20
[Huawei-GigabitEthernet0/0/3]int g0/0/1
#设置该端口为trunk模式 ；
#端口 工作模式 trunk
[Huawei-GigabitEthernet0/0/1]port link-type trunk
#该端口允许携带vlan 10包头的数据包通过
#port trunk allow-pass vlan all 允许所有vlan通过
[Huawei-GigabitEthernet0/0/1]port trunk allow-pass vlan 10
[Huawei-GigabitEthernet0/0/1]port trunk allow-pass vlan 20

```

   
