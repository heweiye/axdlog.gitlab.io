---
title: Using SSH Port Forwarding To Build SSH Tunnel On GNU/Linux
slug:  Using SSH Port Forwarding To Build SSH Tunnel On GNU Linux
date: 2016-08-26T00:55:18+08:00
lastmod: 2019-04-26T16:50:16-04:00
draft: false
keywords: ["ssh", "port forward"]
description: "Using SSH Port Forwarding To Build SSH Tunnel On GNU/Linux"
categories:
- SSH
- Security
tags:
- SSH
- iptables
comment: true
toc: true

---

出於安全考慮，公司的服務器需通過跳板機(front)主機才能登陸。但在實際操作過程中遇到一些不便之處，如無法直接從本地上傳文件到目標主機。故而想實現在本機通過跳板機直接連接目標主機，實現文件的上傳、下載和命令操作。本文嘗試使用SSH通過`跳板主機`在`本機`與`目標主機`之間建立Tunnel(隧道)，實現遠程連接。相關實現方式可參考[What's ssh port forwarding and what's the difference between ssh local and remote port forwarding](http://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot#answer-115906)。

<!--more-->

## Simple Operation
`April 26,2019`更新，通過如下方式更爲簡單、快捷

在ssh配置文件`~/.ssh/config`進行如下配置

```
# https://infosec.mozilla.org/guidelines/openssh.html
# Directives Explanation Start
# - port forward, execute 'ssh -fNg'
# LocalForward [127.0.0.1:]8080 127.0.0.1:80  --  forward remote host tcp port '80' to local host tcp port '8080' over the secure channel
# RemoteForward [127.0.0.1:]80 127.0.0.1:8080  --  forward local host tcp port '80' to remote host tcp port '8080' over the secure channel
# DynamicForward 127.0.0.1:8888  --  create a socks5 secure channel at local host which socks5 port is '8888', the application protocol is then used to determine where to connect to from the remote machine

# - login internal host via jump host (Host remote_jump_host)
# https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
# ProxyJump remote_jump_host1[,remote_jump_host2,...]  -- ProxyJump add in OpenSSH 7.3
# ProxyCommand ssh remote_jump_host -W %h:%p %r  -- recommended way if has no ProxyJump
# ProxyCommand ssh remote_jump_host nc %h %p %r  -- need install pack 'nc' in internal host
# ProxyCommand nc -X 5 -x localhost:9050 %h %p    -- SSH over Tor, need install netcat, netcat-openbsd
# Directives Explanation End


# Sample Start
# execute 'ssh sample_internal_host1' to directly login internal host 'sample_internal_host1' via jump host 'sample_jump'
# Host sample_jump
# 	HostName 122.144.200.62
# 	Port 22000
# 	IdentityFile /PATH/jump_key
#
# Host sample_internal_*
# 	#ProxyJump sample_jump
# 	#ProxyCommand ssh sample_jump nc %h %p %r
# 	ProxyCommand ssh sample_jump -W %h:%p %r
#   # /PATH/internal_key is ssh keygen own by host sample_jump
# 	IdentityFile /PATH/internal_key
#
# Host sample_internal_host1
#     HostName 192.168.6.8
#     User user_1
#     Port 22222
# Sample End
```


例如

```
Host digital_ocean
    User root
    HostName 104.131.148.149
    DynamicForward 127.0.0.1:9999
    #LocalForward 127.0.0.1:9050 127.0.0.1:9050
    #LocalForward 127.0.0.1:8080 127.0.0.1:8080
```

配置完成後，只需執行`ssh -fNg digital_ocean`即可成功創建端口轉發。

以下內容爲原始內容

## Preparation
本文中的所有實驗都在 [Digital Ocean](https://www.digitalocean.com/) 的VPS中進行，因本機使用的是`CentOS 7.2`，為保證環境一致，故而VPS的操作系統也選擇`CentOS 7.2`，登陸VPS的SSH Key保存在本地主機用戶家目錄的`~/.ssh`中。

**注意**：所有操作都是在`root`用戶下進行。

<!-- SSH的配置和使用可參考本人之前Blog [SSH Configurations And Usages](https://lempstacker.com/tw/SSH-Configurations-And-Usages/)，如何生成SSH-Keygen Key，可參考文中的[Generate Authentication SSH-Keygen Keys](https://lempstacker.com/tw/SSH-Configurations-And-Usages/#Generate-Authentication-SSH-Keygen-Keys)部分。 -->

SSH的配置和使用可參考本人之前Blog

* [OpenSSH Config File Usage Introduction]({{< relref "2017-03-09-OpenSSH-Config-File-Usage-Introduction.md" >}})
* [OpenSSH Configurations Optimization & Simple Usages]({{< relref "2016-02-02-OpenSSH-Configurations-Optimization-Simple-Usages.md" >}})

如何生成SSH-Keygen Key，可參考文中的[Generating SSH-Keygen Keys]({{< relref "2016-02-02-OpenSSH-Configurations-Optimization-Simple-Usages.md#generating-ssh-keygen-keys" >}})部分。

### Environmental Design
本地主機可通過SSH連接遠程主機`Front`的外網IP，而遠程主機`Target`則只能與遠程主機`Front`進行內網通信，即防火牆規則中只允許遠程主機`Target`的內網IP通過。

先爲遠程主機`Target`設置root用戶密碼可通過密碼登陸，之後在安裝SSH-Keygen Key，使之可以通過`ssh -i`免密碼登陸。

### Server Info

以下是本地主機、遠程主機的相關信息

**本地主機**

item|detail
---|---
Operating System|`CentOS Linux release 7.2.1511 (Core)`
Kernel Version|`3.10.0-327.28.3.el7.x86_64`
Time Zone|`Asia/Shanghai (CST, +0800)`
OpenSSH Version|`OpenSSH_6.6.1p1`

**遠程主機**

Machine|Host Name|Outer IP|Inner IP|SSH Port
---|---|---|---|---
跳板機|`Front`|`104.131.148.149`|`10.134.5.221`|`22`
目標主機|`Target`|`104.131.150.251`|`10.134.12.148`|`22`


## Configuration
遠程主機配置主要包括以下方面

1. 更改遠程主機`Front`、`Target`的時區為`Asia/Shanghai`，安裝`chrony`服務自動同步時間；
2. 為遠程主機`Target`設置root密碼，默認是通過SSH Key連接；
2. 在遠程主機`Target`上安裝[iptables](https://en.wikipedia.org/wiki/Iptables)防火牆，並設置只允許遠程主機`Front`訪問主機`Target`的`22`號端口；

[Digital Ocean](https://www.digitalocean.com/)的VPS默認`SELinux`服務被禁用，`firewalld`服務未啟用，`iptables`服務未安裝。

### Front & Target
分別在遠程主機`Front`、`Target`中執行如下命令

```sh
# 禁用SELinux
sed -i -r 's@SELINUX=enforcing@SELINUX=disabled@g;s@(SELINUXTYPE=targeted)@#\1@g' /etc/selinux/config
setenforce 0

# 關閉firewalld
systemctl stop firewalld
systemctl disable firewalld

#設置時區
timedatectl set-timezone Asia/Shanghai

#安裝chrony服務並啟用
yum install -y chrony
systemctl start chronyd
systemctl enable chronyd
```

操作完成後可通過`timedatectl`命令查看時區信息，信息顯示如下
```sh
# Front
[root@Front ~]# timedatectl
      Local time: Fri 2016-08-26 09:08:20 CST
  Universal time: Fri 2016-08-26 01:08:20 UTC
        RTC time: Fri 2016-08-26 09:08:20
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: yes
      DST active: n/a

Warning: The system is configured to read the RTC time in the local time zone.
         This mode can not be fully supported. It will create various problems
         with time zone changes and daylight saving time adjustments. The RTC
         time is never updated, it relies on external facilities to maintain it.
         If at all possible, use RTC in UTC by calling
         'timedatectl set-local-rtc 0'.
[root@Front ~]#


# Target
[root@Target ~]# timedatectl
      Local time: Fri 2016-08-26 09:08:26 CST
  Universal time: Fri 2016-08-26 01:08:26 UTC
        RTC time: Fri 2016-08-26 09:08:25
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: yes
      DST active: n/a

Warning: The system is configured to read the RTC time in the local time zone.
         This mode can not be fully supported. It will create various problems
         with time zone changes and daylight saving time adjustments. The RTC
         time is never updated, it relies on external facilities to maintain it.
         If at all possible, use RTC in UTC by calling
         'timedatectl set-local-rtc 0'.
[root@Target ~]#
```

### Target Only

在遠程主機`Target`中執行如下命令

```sh
# 設置root密碼為 axdlog.com
echo "axdlog.com" | passwd --stdin root

#安裝、啓用iptables服務，並添加規則
yum install -y iptables iptables-services
systemctl start iptables

#查看現有的規則
iptables -L -n

# 關閉iptables服務
systemctl stop iptables

#清除已有規則
iptables -F
iptables -X
iptables -Z

# 保存規則
service iptables save

#只允許遠程主機Front的內網IP通信
iptables -A INPUT -s 10.134.5.221 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 10.134.5.221 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

#保證yum服務能正常使用
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp -m udp --sport 53 -j ACCEPT
iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p udp -m udp --dport 53 -j ACCEPT

#關閉所有端口(因是通過SSH連接修改iptable規則，如果執行會導致SSH連接斷開)
# iptables -P INPUT DROP
# iptables -P OUTPUT DROP
# iptables -P FORWARD DROP

#保存規則
service iptables save

#重啟iptables服務
systemctl restart iptables
```


再次嘗試從本地主機連接遠程主機`Target`已經無法連接

```sh
[flying@maxdsre ~]$ ssh root@104.131.150.251
ssh: connect to host 104.131.150.251 port 22: Connection timed out
[flying@maxdsre ~]$
```

從遠程主機`Front`可通過遠程主機`Target`的內網IP進行連接

```sh
[root@Front ~]# ssh root@10.134.12.148
The authenticity of host '10.134.12.148 (10.134.12.148)' can't be established.
ECDSA key fingerprint is 7b:39:56:00:5b:ff:f6:b2:b6:53:7f:3c:c7:7b:6b:f0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.134.12.148' (ECDSA) to the list of known hosts.
root@10.134.12.148's password:
Last failed login: Fri Aug 26 09:23:42 CST 2016 from 220.181.167.182 on ssh:notty
There were 9 failed login attempts since the last successful login.
Last login: Fri Aug 26 09:12:19 2016 from 10.134.5.221
[root@Target ~]# hostname
Target
[root@Target ~]# exit
logout
Connection to 10.134.12.148 closed.
[root@Front ~]# exit
logout
Connection to 104.131.148.149 closed.
[flying@maxdsre ~]$
```

## Port Forwarding
SSH端口轉發主要分為三種
1. Local Port Forwarding
2. Remote Port Forwarding
3. Dynamic Port Forwarding

具體的命令形式
```sh
# Local Port Forwarding
ssh -C -f -N -g -L 127.0.0.1:6666:10.134.12.148:22 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149

# Remote Port Forwarding
ssh -C -f -N -g -R 127.0.0.1:7777:10.134.12.148:22 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149

# Dynamic Port Forwarding
ssh -C -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149  2>/dev/null
```

相關參數解釋

* `-C`: Requests compression of all data (including stdin, stdout, stderr, and data for forwarded X11 and TCP connections).
* `-f`: Requests ssh to go to background just before command execution.
* `-N`: Do not execute a remote command (protocol version 2 only).
* `-g`: Allows remote hosts to connect to local forwarded ports.
* `-L [bind_address:]port:host:hostport`: Specifies that the given port on the **local** (client) host is to be forwarded to the given host and port on the **remote** side.
* `-R [bind_address:]port:host:hostport`: Specifies that the given port on the **remote** (server) host is to be forwarded to the given host and port on the **local** side.
* `-D [bind_address:]port`: Specifies a local “dynamic” application-level port forwarding.
* `-i identity_file`: Selects a file from which the identity (private key) for public key authentication is read.
* `-p port`: Port to connect to on the remote host.


**注意**：tunnel(隧道)的建立需要一定時間，請耐心等待。

## Local Port Forwarding

### Using Password
執行如下命令，會在本地主機上創建端口號為`6666`的端口，將該端口號的數據通過`Front`(104.131.148.149)轉發到`Target`內網IP(10.134.12.148)的`22`號端口。

執行如下命令建立SSH Tunnel實現本地端口轉發

```sh
# Local Port Forwarding
ssh -C -f -N -g -L 127.0.0.1:6666:10.134.12.148:22 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149
```

在本地主機執行如下命令
```sh
ssh -p 6666 root@127.0.0.1
```
即可正常登陸遠程主機`Target`

以下是操作過程
```
[flying@maxdsre ~]$ ssh -p 6666 root@127.0.0.1
The authenticity of host '[127.0.0.1]:6666 ([127.0.0.1]:6666)' can't be established.
ECDSA key fingerprint is 7b:39:56:00:5b:ff:f6:b2:b6:53:7f:3c:c7:7b:6b:f0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:6666' (ECDSA) to the list of known hosts.
Last login: Fri Aug 26 09:28:12 2016 from 10.134.5.221
[root@Target ~]# hostnamectl
   Static hostname: target
   Pretty hostname: Target
Transient hostname: Target
         Icon name: computer-vm
           Chassis: vm
        Machine ID: c3f4abf4b4024198ade2d41e6f26652d
           Boot ID: a890b11a7cc84ec9b6a63359ac21a014
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.28.2.el7.x86_64
      Architecture: x86-64
[root@Target ~]# echo 'This is Target Host' > /tmp/hostInfo.txt
[root@Target ~]# cat /tmp/hostInfo.txt
This is Target Host
[root@Target ~]# exit
logout
Connection to 127.0.0.1 closed.
[flying@maxdsre ~]$
```

在遠程主機`Target`的`/tmp`下創建文件`hostInfo.txt`，內容為`This is Target Host`。

**注** ：可使用如下命令在本地主機查看端口`6666`相關信息
```sh
ss -tnl | grep 6666

ps -ef | awk '$0~/6666/&&$0!~/grep/'

sudo netstat -apn | awk '$0~/6666/&&$0~/LISTEN/'

# 獲取父進程信息
ps -ef | grep `sudo netstat -apn | awk '$0~/6666/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'`

# 獲取父進程ID
ps -ef | grep `sudo netstat -apn | awk '$0~/6666/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'` | awk '{print $2}'

# 通過Kill命令終止父進程實現該端口的關閉
ps -ef | grep `sudo netstat -apn | awk '$0~/6666/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'` | awk '{print $2}' | xargs kill -s 9

```

#### SFTP Connection
通過`sftp`連接遠程主機`Target`

`sftp`命令格式如下
```
sftp [-1246aCfpqrv] [-B buffer_size] [-b batchfile] [-c cipher] [-D sftp_server_path] [-F ssh_config]
          [-i identity_file] [-l limit] [-o ssh_option] [-P port] [-R num_requests] [-S program]
          [-s subsystem | sftp_server] host
sftp [user@]host[:file ...]
sftp [user@]host[:dir[/]]
sftp -b batchfile [user@]host
```

**注意**： 指定端口在`ssh`命令中是小寫字母`-p`，而在`sftp`中是大寫字母`-P`；

執行如下命令即可以`ftp`形式進入遠程主機`Target`
```sh
sftp -P 6666 root@127.0.0.1
```

操作過程
```sh
[flying@maxdsre ~]$ sftp -P 6666 root@127.0.0.1
Connected to 127.0.0.1.
sftp> cd /tmp/
sftp> ls
hostInfo.txt    
sftp> exit
[flying@maxdsre ~]$
```

#### get data
在`stfp`中使用`get`命令可將遠程主機中的文件下載到本地主機

命令格式
```
get [-afPpr] remote-path [local-path]
```

>Retrieve the remote-path and store it on the local machine. If the local path name is not specified, it is given the same name it has on the remote machine. remote-path may contain glob(3) characters and may match multiple files. If it does and local-path is specified, then local-path must specify a directory.

此處將文件`hostInfo.txt`下載到本地主機的`/tmp`目錄下，並更名爲`SaveToLocalHost.txt`，操作過程

```sh
[flying@maxdsre ~]$ ls -lh /tmp/SaveToLocalHost.txt
ls: cannot access /tmp/SaveToLocalHost.txt: No such file or directory
[flying@maxdsre ~]$ sftp -P 6666 root@127.0.0.1
Connected to 127.0.0.1.
sftp> cd /tmp/
sftp> ls
hostInfo.txt    
sftp> get hostInfo.txt /tmp/SaveToLocalHost.txt
Fetching /tmp/hostInfo.txt to /tmp/SaveToLocalHost.txt
/tmp/hostInfo.txt                             100%   20     0.0KB/s   00:00    
sftp> exit
[flying@maxdsre ~]$ cat /tmp/SaveToLocalHost.txt
This is Target Host
[flying@maxdsre ~]$
```

#### put data
在`stfp`中使用`put`命令可將本地主機中的文件上傳到遠程主機中

命令格式
```
put [-fPpr] local-path [remote-path]
```

>Upload local-path and store it on the remote machine. If the remote path name is not specified, it is given the same name it has on the local machine. local-path may contain glob(3) characters and may match multiple files. If it does and remote-path is specified, then remote-path must specify a directory.

此處將本地主機目錄`/home/flying/Downloads`中的文件`Linux From Scratch-Version7.9.pdf`上傳到遠程主機的`/tmp`目錄下，操作過程

```sh
[flying@maxdsre ~]$ namei -l ~/Downloads/Linux\ From\ Scratch-Version7.9.pdf
f: /home/flying/Downloads/Linux From Scratch-Version7.9.pdf
dr-xr-xr-x root   root   /
drwxr-xr-x root   root   home
drwx------ flying flying flying
drwxr-xr-x flying flying Downloads
-rw-rw-r-- flying flying Linux From Scratch-Version7.9.pdf
[flying@maxdsre ~]$ sftp -P 6666 root@127.0.0.1Connected to 127.0.0.1.
sftp> cd /tmp/
sftp> ls
hostInfo.txt    
sftp> put /home/flying/Downloads/Linux\ From\ Scratch-Version7.9.pdf /tmp
Uploading /home/flying/Downloads/Linux From Scratch-Version7.9.pdf to /tmp/Linux From Scratch-Version7.9.pdf
/home/flying/Downloads/Linux From Scratch-Ver 100% 1696KB 106.0KB/s   00:16    
sftp> ls
Linux From Scratch-Version7.9.pdf       hostInfo.txt                            
sftp> exit
[flying@maxdsre ~]$
```

### Using SSH-Keygen Key

**重要**： 如果遠程主機`Front`是通過SSH Key形式連接遠程主機`Target`，則在連接時，命令`ssh -p 6666 root@127.0.0.1`是無法連接成功的，可添加參數`-vvv`查看具體信息。它需要的是跳板機`Front`的私鑰。故而需將遠程主機`Front`目錄`/root/.ssh/`下的`id_rsa`(私鑰)、`id_rsa.pub`(公鑰)保存到本地主機中，且文件讀寫權限設置為`600`，然後通過參數`-i`指定，形式如下
```
ssh -p 6666 -i /LOCALHOST/PATH/Front/id_rsa root@127.0.0.1
```
這樣才能正常登入遠程主機`Target`

在遠程主機`Front`中執行如下命令，創建SSH-Keygen並安裝到遠程主機`Target`中
```sh
# 生成密鑰，設置passphrase爲 grepsedawk
ssh-keygen -t rsa -b 4096 -C 'SSH Tunnel Testing'

# 安裝ssh-keygen key
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 root@10.134.12.148
#或執行該命令
cat ~/.ssh/id_rsa.pub | ssh -p 22 root@10.134.12.148 'umask 077; test -d .ssh|| mkdir .ssh; cat >> .ssh/authorized_keys'

```

操作過程
```sh
[flying@maxdsre ~]$ ssh root@104.131.148.149
Last failed login: Fri Aug 26 09:55:00 CST 2016 from 116.31.116.8 on ssh:notty
There were 12 failed login attempts since the last successful login.
Last login: Fri Aug 26 09:52:32 2016 from 116.228.89.242
[root@Front ~]# ssh-keygen -t rsa -b 4096 -C 'SSH Tunnel Testing'
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):

#指定passphrase，相當於爲ssh-keygen key再加一層安全保護
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
ca:1c:f6:bb:2a:34:e9:09:03:83:ca:e4:4c:3f:95:4a SSH Tunnel Testing
The key's randomart image is:
+--[ RSA 4096]----+
|                 |
|                 |
|.     .          |
|+o E o           |
|Boo o.o S        |
|.+o+++ +         |
|   =.o+ .        |
|    +    .       |
|     ...o.       |
+-----------------+
[root@Front ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 root@10.134.12.148
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.134.12.148's password:
# 此處輸入的是遠程主機Target的root用戶的登陸密碼

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh -p '22' 'root@10.134.12.148'"
and check to make sure that only the key(s) you wanted were added.

#此處輸入的是創建ssh-keygen時設置的passphrase
[root@Front ~]# ssh -p 22 root@10.134.12.148
Enter passphrase for key '/root/.ssh/id_rsa':
Last login: Fri Aug 26 09:35:57 2016 from 10.134.5.221
[root@Target ~]# tail -1 ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3DDMrKZmux2vAZqH5gg3W9Im6HaFAJh1te2fHQmOYOvNgQcTCNQ8Bw9jaIrUqqilL6AiHgrpyjvixW7roOsE7DCoNOVAHTFH9XiovsybLSqbmT8lq11mZvMLQUvZj0O34GMIVoDFYg1FrL5vH5Lxs5yGsd14a5rY6Fpchuf0k+zqaCUvjR5ragU+kdLA7Oe2YqljMZXTDy9qv30HIbc/qb7nyj0PCaLit48qhILgMEuDw627iifooaYkKgbk35nb75JKGVKVaDGVcheDVyPyThuQ2sNTZXbpB+FDb+Gmx0OpUtYxVW970/tSH6eMUudnE64KmWl61hYUrcxvczq1ybhBvAbBm8FY7gGvvs3+2HtC5/hRUmM3defc3+IkAiJoDgSIaLjWSovwQrJFN7HNvL3qiIX23BjqtYcQpJAXkIrSxa4fi+mRI66mWgVK5KrluOAp921VZ+Gcf/6x11JVdU3TAS+zAAQTtwaDBMvYyGa9Wpjoj8yAtc139HKNQo1gh4lFghv/jvb+aAtOazvH16gUKaLdAaYTUfRRCgEwf0cM8vbThAeVFTFSIVM1rY04NzpUeCGVlsSALDQR1Xh1DA6p4hdwW0KJld5GldvqN6iO+hlhBSslF3Zhfs8LvMxCoPdzLslLC0zAbYdaWssW86R6SEePFqvY//QAU4o3Csw== LempStacker.com SSH Tunnel Testing
[root@Target ~]# exit
logout
Connection to 10.134.12.148 closed.
[root@Front ~]# exit
logout
Connection to 104.131.148.149 closed.
[flying@maxdsre ~]$
```

將遠程主機`Front`目錄`~/.ssh`中的密鑰文件下載到本地主機中，以保存到目錄`/tmp/front`中爲例
```sh
[flying@maxdsre ~]$ mkdir /tmp/front
[flying@maxdsre ~]$ sftp -P 22 root@104.131.148.149
Connected to 104.131.148.149.
sftp> cd .ssh/
sftp> ls
authorized_keys     id_rsa              id_rsa.pub          known_hosts         
sftp> get id
id_rsa       id_rsa.pub
sftp> get id_rsa /tmp/front/
Fetching /root/.ssh/id_rsa to /tmp/front/id_rsa
/root/.ssh/id_rsa                             100% 3326     3.3KB/s   00:01    
sftp> get id_rsa.pub /tmp/front/
Fetching /root/.ssh/id_rsa.pub to /tmp/front/id_rsa.pub
/root/.ssh/id_rsa.pub                         100%  760     0.7KB/s   00:00    
sftp> exit
[flying@maxdsre ~]$ ls -lh /tmp/front/
total 8.0K
-rw------- 1 flying flying 3.3K Aug 26 10:16 id_rsa
-rw-r--r-- 1 flying flying  760 Aug 26 10:16 id_rsa.pub

#修改文件讀寫權限爲600
[flying@maxdsre ~]$ chmod 600 /tmp/front/id_rsa*
[flying@maxdsre ~]$ ls -lh /tmp/front/
total 8.0K
-rw------- 1 flying flying 3.3K Aug 26 10:16 id_rsa
-rw------- 1 flying flying  760 Aug 26 10:16 id_rsa.pub
[flying@maxdsre ~]$
```

執行如下命令kill使用端口`6666`的進程

```sh
ps -ef | grep `sudo netstat -apn | awk '$0~/6666/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'` | awk '{print $2}' | xargs kill -s 9
```

再次執行如下命令創建SSH Tunnel

```sh
# Local Port Forwarding
ssh -C -f -N -g -L 127.0.0.1:6666:10.134.12.148:22 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149
```

#### root password
在本地主機執行如下命令,會要求輸入root用戶密碼，才能正常登陸遠程主機`Target`

```sh
ssh -p 6666 root@127.0.0.1

sftp -P 6666 root@127.0.0.1
```

```sh
[flying@maxdsre ~]$ ssh -p 6666 root@127.0.0.1
root@127.0.0.1's password:
Last failed login: Fri Aug 26 10:33:47 CST 2016 from 10.134.5.221 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Fri Aug 26 10:29:20 2016 from 10.134.5.221
[root@Target ~]# hostname
Target
[root@Target ~]# exit
logout
Connection to 127.0.0.1 closed.
[flying@maxdsre ~]$
```

```sh
[flying@maxdsre ~]$ sftp -P 6666 root@127.0.0.1
root@127.0.0.1's password:
Connected to 127.0.0.1.
sftp> exit
[flying@maxdsre ~]$
```

#### ssh-keygen key
在本地主機執行如下命令,指定從遠程主機`Front`下載下來的ssh-keygen key，會要求輸入passphrase密碼，才能正常登陸遠程主機`Target`

```sh
ssh -p 6666 -i /tmp/front/id_rsa root@127.0.0.1

sftp -P 6666 -i /tmp/front/id_rsa root@127.0.0.1
```


操作過程
```
[flying@maxdsre ~]$ ssh -p 6666 -i /tmp/front/id_rsa root@127.0.0.1
Enter passphrase for key '/tmp/front/id_rsa':
Last login: Fri Aug 26 10:34:54 2016 from 10.134.5.221
[root@Target ~]# exit
logout
Connection to 127.0.0.1 closed.
[flying@maxdsre ~]$ sftp -P 6666 -i /tmp/front/id_rsa root@127.0.0.1
Enter passphrase for key '/tmp/front/id_rsa':
Connected to 127.0.0.1.
sftp> exit
[flying@maxdsre ~]$
```

### Actual Case
遠程主機`Target`是內網主機，只有遠程主機`Front`能與之連接通信，本地主機可以連接遠程主機`Front`。在遠程主機`Target`中安裝Nginx服務，並設置只允許遠程主機`Front`能訪問`80`端口。現通過遠程主機`Front`建立SSH Tunnel，將遠程主機`Target`的`80`端口映射到本地主機的`8080`端口，實現在本地瀏覽器URL中輸入`http://localhost:8080/`即可訪問遠程主機`Target`的Web服務。

#### Preparation
在遠程主機`Front`中執行如下操作

```sh
# 安裝epel源
yum install -y epel-release

# 安裝epel源後安裝Nginx，在epel源中
yum install -y nginx

# 啓動Nginx服務
systemctl start nginx

# 查看80端口是否啓用
ss -tnl | grep -w 80

# 添加iptables規則
iptables -A INPUT -s 10.134.5.221/32 -p tcp -m tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 10.134.5.221/32 -p tcp -m tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT

# 保存iptables規則
service iptables save

# 創建Web頁面
tee /usr/share/nginx/html/index.html <<-'EOF'
<h1>This is Target Host <strong>axdlog.com</strong> on CentOS 7!</h1>
EOF

```

在本地主機中執行如下命令創建SSH Tunnel

```sh
ssh -C -c aes256-ctr -o StrictHostKeyChecking=no -f -N -g -L 127.0.0.1:7777:10.134.12.148:80 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149
```

#### Operation Process
在遠程主機`Front`中可以正常訪問該服務
```sh
[root@front ~]# curl http://10.134.12.148:80
<h1>This is Target Host <strong>axdlog.com</strong> on CentOS 7!</h1>
[root@front ~]# curl -I http://10.134.12.148:80
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 26 Aug 2016 07:31:21 GMT
Content-Type: text/html
Content-Length: 75
Last-Modified: Fri, 26 Aug 2016 07:29:13 GMT
Connection: keep-alive
ETag: "57bfefc9-4b"
Accept-Ranges: bytes

[root@front ~]#
```

在本地主機使用`127.0.0.1:7777`
```sh
[flying@maxdsre ~]$ curl http://127.0.0.1:7777
<h1>This is Target Host <strong>axdlog.com</strong> on CentOS 7!</h1>
[flying@maxdsre ~]$ curl -I http://127.0.0.1:7777
HTTP/1.1 200 OK
Server: nginx/1.6.3
Date: Fri, 26 Aug 2016 07:37:00 GMT
Content-Type: text/html
Content-Length: 75
Last-Modified: Fri, 26 Aug 2016 07:29:13 GMT
Connection: keep-alive
ETag: "57bfefc9-4b"
Accept-Ranges: bytes

[flying@maxdsre ~]$
```

可以看到在本機能正常訪問遠程主機`Front`Web服務中的內容。


## Remote Port Forwarding

~~遠程端口轉發尚未完全理解，暫時擱置~~

本地主機可通過外網IP連接遠程主機`Front`，遠程主機`Target`可連接遠程主機`Front`，但遠程主機`Front`無法直接連接本地主機。即本地主機和遠程主機`Target`都沒有公網IP，但都能連接遠程主機`Front`。

### Scenario 1
情境設置：我在自己電腦的本地開發環境中開發了一個項目，想給外地的客戶或朋友看，他們無法直接在我電腦上看。但是我有一臺有獨立外網IP的VPS，我可以在自己的電腦上通過SSH創建遠程端口轉發，將本地的`80`端口的數據轉發到VPS(104.131.148.149)上的指定端口`7777`。身在外地的客戶或朋友直接在瀏覽器中輸入`http://104.131.148.149:7777`即可看到我電腦上的Web項目。


**重要**： 使用遠程主機進行端口轉發，必須在VPS的`/etc/ssh/sshd_config`文件中設置
```
GatewayPorts yes
```

然後執行如下命令重啓sshd服務

```bash
systemctl restart ssh
# systemctl restart sshd
```

在本地主機執行如下命令
```sh
ssh -C  -c aes256-ctr -o StrictHostKeyChecking=no -f -N -g -R 7777:127.0.0.1:80 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149
```

在遠程主機`Target`中即可查到已經監聽了`7777`端口。

測試過程
```
[flying@maxdsre ~]$ cat /usr/share/nginx/html/index.html
This is Local Host. axdlog.com
[flying@maxdsre ~]$ curl http://104.131.148.149:7777/
This is Local Host. axdlog.com
[flying@maxdsre ~]$ curl -I http://104.131.148.149:7777/
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 26 Aug 2016 11:20:36 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 36
Last-Modified: Fri, 26 Aug 2016 11:20:07 GMT
Connection: keep-alive
ETag: "57c025e7-24"
Accept-Ranges: bytes

[flying@maxdsre ~]$
```

### Scenario 2
情景設置：兩臺分處於不同地域的主機，主機無獨立IP，通過路由器分配內網地址聯網。如果一臺主機`A`想通過SSH連接到另一臺主機`B`，因彼此無獨立IP，無法直接連接，可通過VPS實現。

將VPS中`/etc/ssh/sshd_config`的`GatewayPorts`指令設置爲`yes`，若文件內無該指令，在末尾添加即可，重啓sshd服務。

VPS主機的IP爲`104.131.148.149`，登錄用戶名爲`user_vps`，SSH爲的`33333`號端口。
主機`B`的SSH端口爲`22222`，登錄用戶名爲`user_b`。

在主機`B`中執行如下命令，在VPS中創建遠程端口，此處假設要創建的端口號爲`1080`。
```bash
ssh -f -N -g -R 1080:127.0.0.1:22222 -p 33333 user_vps@104.131.148.149
```

在VPS中通過`ss -tnl`查看端口`1080`是否已經監聽。如果已經監聽，在主機`A`中執行如下命令登錄主機`B`

```bash
ssh -p 1080 user_b@104.131.148.149
```

注意：執行該命令時，指定的端口號爲創建的遠程轉發端口`1080`，用戶名是主機`B`的用戶名`user_b`，而主機IP則是VPS的主機。

如果啓用了防火牆，須對該遠程轉發端口進行放行。


## Dynamic Port Forwarding

`-D [bind_address:]port`: Specifies a local “dynamic” application-level port forwarding.

在本地主機執行如下命令，為本地主機指定一個本地的`動態的`應用程序級別的端口轉發，端口號是`8888`，所有從此端口發出的數據都會轉發到遠程主機`Front`的`22`號端口中，可成功穿越firewall。

```sh
# Dynamic Port Forwarding
ssh -C -c aes256-ctr -o StrictHostKeyChecking=0 -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149  2>/dev/null
```

在系統或瀏覽器代理中設置`Socks Host`：主機`127.0.0.1`，端口號`8888`，即可實現全局代理。在瀏覽器中查看，此時本機的IP已經變成遠程主機`Front`的IP(104.131.148.149)。

**重要**：如果想在後臺執行該命令，可通過`setsid`或`nohup`命令實現，具體命令爲

```sh
#setsid
ssh -C -c aes256-ctr -o StrictHostKeyChecking=no -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149 | setsid 2>/dev/null

#nohup
nohup ssh -C -c aes256-ctr -o StrictHostKeyChecking=no -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149 2> /dev/null &
```

**注** ：可使用如下命令在本地主機查看端口`8888`相關信息
```sh
ss -tnlp | grep 8888

ps -ef | awk '$0~/8888/&&$0!~/grep/'

sudo netstat -apn | awk '$0~/8888/&&$0~/LISTEN/'

# 獲取父進程信息
ps -ef | grep `sudo netstat -apn | awk '$0~/8888/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'`

# 獲取父進程ID
ps -ef | grep `sudo netstat -apn | awk '$0~/8888/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'` | awk '{print $2}'

ss -tnlp | awk 'match($4,/8888/){print gensub(/.*,pid=(.*),.*/,"\\1"," ",$6)}'


# 終止父進程
ps -ef | grep `sudo netstat -apn | awk '$0~/8888/&&$0~/LISTEN/{print $NF}' | awk -v FS='/' '{print $1}'` | awk '{print $2}' | xargs kill -s 9

ss -tnlp | awk 'match($4,/8888/){print gensub(/.*,pid=(.*),.*/,"\\1"," ",$6)}' | xargs kill -s 9
```


## Error Occurring
### Error 1 (未解決)
執行
```sh
# Dynamic Port Forwarding
ssh -C -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149

```
建立`Dynamic Port Forwarding`後，使用本地IP和Port進行全局代理上網，使用瀏覽器時，CLI中會報如下錯誤信息
>open failed: administratively prohibited: open failed

參考

* [“channel 3: open failed: administratively prohibited: open failed” when creating a VNC session on a SSH tunnel](http://unix.stackexchange.com/questions/247788/channel-3-open-failed-administratively-prohibited-open-failed-when-creating#answer-247790)
* [how to solve the “open failed: administratively prohibited: open failed” when using a SSH tunnel proxy](http://serverfault.com/questions/535412/how-to-solve-the-open-failed-administratively-prohibited-open-failed-when-us#answer-535425)

修改遠程主機`/etc/ssh/sshd_config`中的參數`AllowTcpForwarding`、`GatewayPorts`爲`yes`，但無效，仍舊報錯。

最後只能參考

* [how to solve the “open failed: administratively prohibited: open failed” when using a SSH tunnel proxy](http://serverfault.com/questions/535412/how-to-solve-the-open-failed-administratively-prohibited-open-failed-when-us#answer-790831)

將報錯信息寫入`/dev/null`，即命令更改成

```sh
ssh -C -f -N -g -D 127.0.0.1:8888 -p 22 -i ~/.ssh/id_rsa root@104.131.148.149 2>/dev/null
```


## References
* [How to understand SSH perfectly?](http://infosecaddicts.com/understand-ssh-perfectly/)
* [Howto use SSH local and remote port forwarding](http://www.debianadmin.com/howto-use-ssh-local-and-remote-port-forwarding.html)
* [SSH原理與運用（二）：遠程操作與端口轉發](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)
* [建立SSH隧道（SSH端口轉發）](http://www.xushulong.com/ssh-tunnel/)
* [iptables只允許指定ip地址訪問指定端口](http://www.ccvita.com/529.html)
* [SSH Tunnel - Local and Remote Port Forwarding Explained With Examples](http://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html)
* [SSH/OpenSSH/PortForwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding)
* [DYNAMIC PORT FORWARDING WITH SSH AND SOCKS5](https://network23.org/inputisevil/2015/09/05/dynamic-port-forwarding-with-ssh-and-socks5/)


## Change Logs
* 2016.08.26 00:55 Fri Asia/Shanghai
    * 初稿完成
* 2016.08.26 19:35 Fri Asia/Shanghai
    * 完善實驗過程
* 2016.10.18 10:05 Tue Asia/Shanghai
    * 添加`setsid`實現後臺執行
* 2017.07.14 16:58 Fri Asia/Shanghai
    * 爲`Remote PortForwarding`添加`Scenario2`，實現SSH連接彼此無獨立IP，但能連接外網的主機
* 2017.09.09 19:01 Sat Asia/Shanghai
    * 添加`How to understand SSH perfectly?`
* 2019.04.26 16:50 Fri America/Boston
    * 勘誤，添加`Simple Operation`一節


<!-- End -->
