# 在Linux系统上初试MySQL服务

## 检查

先要检查 Linux 系统中是否已经安装了 MySQL，输入命令尝试打开 MySQL 服务：

```Linux
sudo service mysql start
```

输入密码后，如果出现以下提示，则说明系统中已经安装有 MySQL：

![1-01](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/7c378f8d93d7a0d5f285a1e1f5c226e3_MD5.png)

如果提示是这样的，则说明系统中没有 MySQL，需要继续安装：

```linux
mysql: unrecognized service
```

## 安装

在 Ubuntu 上安装 MySQL，最简单的方式是在线安装。只需要几行简单的命令（ `#` 号后面是注释）：

```linux
#安装 MySQL 服务端、核心程序
sudo apt-get install mysql-server

#安装 MySQL 客户端
sudo apt-get install mysql-client
```

在安装过程中会提示确认输入 YES，设置 root 用户密码（之后也可以修改）等，稍等片刻便可安装成功。

安装结束后，用命令验证是否安装并启动成功：

```linux
sudo netstat -tap | grep mysql
```

如果出现如下提示，则安装成功：

[图片缺失：原路径 ../assets/在Linux系统上初试MySQL服务/sql-01-02.png，仓库内无原图，需从旧电脑找回]

此时，可以根据自己的需求，用 gedit 修改 MySQL 的配置文件（my.cnf）,使用以下命令:

```linux
sudo gedit /etc/mysql/my.cnf
```

至此，MySQL 已经安装、配置完成，可以正常使用了。

## 尝试MySQL

使用如下两条命令，打开 MySQL 服务并使用 root 用户登录：

```linux
# 启动 MySQL 服务
sudo service mysql start

# 使用 root 用户登录，当环境的密码为空，直接回车就可以登录
mysql -u root
```



```linux
#当已经安装MySQL服务后
#查看有哪些数据库，一定注意后面不能漏分号
show databases;
#连接数据库
user 数据库名
#查看数据库有哪些表，一定注意后面不能漏分号
show tables;
#退出
quit或者exit
```

