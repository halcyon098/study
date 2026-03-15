## stp
### ensp
stp mode stp/rstp/mstp  #配置生成树模式stp
#rstp 快速生成树协议 mstp 多生成树协议
#省缺模式为mstp
stp  enable    #开启stp功能
undo stp root #删除根桥设置
stp priority (priority)  
#设置设备优先级 priority的取值范围是0～61440
#步长为4096  如0 40960 8192 省缺值为32768
stp root primary  
#直接指定本交换机为根桥，优先级为0 且不能被stp priority命令改变
stp root secondary
#指定本机为备用根桥，优先级为0 且不能被stp priority命令改变
display stp brief #查看stp简要信息

