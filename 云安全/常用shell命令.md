shell命令主要是在渗透过程中去进行容器/云主机的命令执行等操作和绕过部分waf。
## 常见命令

云环境多数以Linux发行版本为基础来制作容器。使用命令可以收集云环境的基本信息

```bash
#查看CPU
lscpu
cat /proc/cpuinfo

#查看内存
free -h
cat /proc/meminfo


#查看磁盘分区情况，最近经常用来配合挂载硬盘到Linux服务器上
lsblk
cat /proc/partitions


#查看系统架构
arch


#查看内核版本
uname -r
uname -ar


#查看发行版本，不同版本可以用来搜索历史上的rce,提权等可以利用的漏洞
cat /etc/redhat-release
cat /etc/os-release
cat /etc/issue
```

## 命令行历史

在终端操作后，Linux系统会默认在内存中记录执行的命令，用户退出时保存到～/.bash_history。在信息收集时会有用，可能记录一些敏感信息，例如数据库连接账号密码等。

- -c 清空历史命令
- -d 指定删除命令
- n 显示最近n条命令
- -w 保存历史列表到指定文件

`HISTSIZE` 变量用于控制 **Bash 历史记录文件**（通常是 `~/.bash_history`）中**保存的命令的最大数量**。

