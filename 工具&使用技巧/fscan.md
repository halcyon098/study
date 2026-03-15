下载链接：

[Releases · shadow1ng/fscan · GitHub](https://github.com/shadow1ng/fscan/releases "Releases · shadow1ng/fscan · GitHub")
1.简介
fscan是一款用go语言编写的开源工具，一款内网综合扫描工具，方便一键自动化、全方位漏扫扫描。
按照软件官方的说法，该工具支持主机存活探测、端口扫描、常见服务的爆破、ms17010、redis 批量写公钥、计划任务反弹 shell、读取 win 网卡信息、web 指纹识别、web 漏洞扫描、netbios 探测、域控识别等功能
**2.命令介绍**
```bash
cd fscan//进入fscan文件夹

//执行在windows下为fscan.exe,linux下为./fscan

//例如
./fscan -h 192.168.101.1/24//启动fscan并扫描网段

fscan.exe -h 192.168.x.x //默认使用全部模块
fscan.exe -h 192.168.x.x -rf id_rsa.pub //redis 写私钥
fscan.exe -h 192.168.x.x -c whoami //ssh爆破成功后，命令执行
fscan.exe -h 192.168.x.x -m ms17010 //指定模块
fscan.exe -h 192.168.x.x -m ssh -p 2222 //指定模块ssh和端口
fscan.exe -h 192.168.x.x -h 192.168.1.1/24 //C段
fscan.exe -h 192.168.x.x -h 192.168.1.1/16 //B段
fscan.exe -h 192.168.x.x -h 192.168.1.1/8  //A段的192.x.x.1和192.x.x.254,方便快速查看网段信息
fscan.exe -h 192.168.x.x -hf ip.txt //以文件导入

```
**完整命令**
```bash
  -c string
        ssh命令执行
  -cookie string
        设置cookie
  -debug int
        多久没响应,就打印当前进度(default 60)
  -domain string
        smb爆破模块时,设置域名
  -h string
        目标ip: 192.168.11.11 | 192.168.11.11-255 | 192.168.11.11,192.168.11.12
  -hf string
        读取文件中的目标
  -hn string
        扫描时,要跳过的ip: -hn 192.168.1.1/24
  -m string
        设置扫描模式: -m ssh (default "all")
  -no
        扫描结果不保存到文件中
  -nobr
        跳过sql、ftp、ssh等的密码爆破
  -nopoc
        跳过web poc扫描
  -np
        跳过存活探测
  -num int
        web poc 发包速率  (default 20)
  -o string
        扫描结果保存到哪 (default "result.txt")
  -p string
        设置扫描的端口: 22 | 1-65535 | 22,80,3306 (default "21,22,80,81,135,139,443,445,1433,3306,5432,6379,7001,8000,8080,8089,9000,9200,11211,27017")
  -pa string
        新增需要扫描的端口,-pa 3389 (会在原有端口列表基础上,新增该端口)
  -path string
        fcgi、smb romote file path
  -ping
        使用ping代替icmp进行存活探测
  -pn string
        扫描时要跳过的端口,as: -pn 445
  -pocname string
        指定web poc的模糊名字, -pocname weblogic
  -proxy string
        设置代理, -proxy http://127.0.0.1:8080
  -user string
        指定爆破时的用户名
  -userf string
        指定爆破时的用户名文件
  -pwd string
        指定爆破时的密码
  -pwdf string
        指定爆破时的密码文件
  -rf string
        指定redis写公钥用模块的文件 (as: -rf id_rsa.pub)
  -rs string
        redis计划任务反弹shell的ip端口 (as: -rs 192.168.1.1:6666)
  -silent
        静默扫描,适合cs扫描时不回显
  -sshkey string
        ssh连接时,指定ssh私钥
  -t int
        扫描线程 (default 600)
  -time int
        端口扫描超时时间 (default 3)
  -u string
        指定Url扫描
  -uf string
        指定Url文件扫描
  -wt int
        web访问超时时间 (default 5)


```
