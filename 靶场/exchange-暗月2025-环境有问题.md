
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/492e22e4c4fd573240539e052b3d68ce_MD5.png)
nmap扫描：
```bash
╰─$ sudo nmap -sV 192.168.111.20
Starting Nmap 7.98 ( https://nmap.org ) at 2025-10-05 13:04 +0800
Nmap scan report for 192.168.111.20
Host is up (0.10s latency).
Not shown: 972 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
25/tcp   open  smtp          Microsoft Exchange smtpd
80/tcp   open  http          Microsoft IIS httpd 8.5
81/tcp   open  http          Microsoft IIS httpd 8.5
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/https?
444/tcp  open  ssl/snpp?
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
465/tcp  open  smtp          Microsoft Exchange smtpd
587/tcp  open  smtp          Microsoft Exchange smtpd
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
808/tcp  open  ccproxy-http?
1801/tcp open  msmq?
2103/tcp open  msrpc         Microsoft Windows RPC
2105/tcp open  msrpc         Microsoft Windows RPC
2107/tcp open  msrpc         Microsoft Windows RPC
2525/tcp open  smtp          Microsoft Exchange smtpd
3800/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3801/tcp open  mc-nmf        .NET Message Framing
3828/tcp open  mc-nmf        .NET Message Framing
5060/tcp open  sip?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
6001/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
6005/tcp open  msrpc         Microsoft Windows RPC
6006/tcp open  msrpc         Microsoft Windows RPC
6007/tcp open  msrpc         Microsoft Windows RPC
6009/tcp open  msrpc         Microsoft Windows RPC
9944/tcp open  unknown
Service Info: Host: exchange.redteam.lab; OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 154.09 seconds
```
信息：
>
exchange
microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Host: exchange.redteam.lab


无凭据
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/21e0fe11dd54de63f50ce73dc2d249d4_MD5.png)
web可以访问
版本判断：
https://rivers.chaitin.cn/blog/cq94vbh0lnechd244d3g
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/c0e97e95f48fc317fda7c0ced8281ca7_MD5.png)
http直接访问接口都失败了。猜测是否为http ntml认证。

```
# 指定要访问的接口，解析返回的 ntlmssp 包``nmap --script http-ntlm-info --script-args http-ntlm-info.root=/ews -p 443 192.168.60.116``nmap --script http-ntlm-info --script-args http-ntlm-info.root=/Autodiscover -p 443 192.168.60.116``   ``# MailSniper.ps1，仅支持 /Autodiscover /ews 两个接口``Invoke-DomainHarvestOWA -ExchHostname 192.168.60.116`
```


https://cloud.tencent.com/developer/article/1937716
![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/af85a312da5b10f466c69bbe81e46259_MD5.png)