---
title: OpenSSH Configurations Optimization & Simple Usages
slug: OpenSSH Configurations Optimization & Simple Usages
date: 2016-02-02T10:28:26+08:00
lastmod: 2018-07-31T21:38:18+08:00
draft: false
keywords: ["AxdLog", "ssh"]
description: "OpenSSH Configurations Optimization & Simple Usages"
categories:
- Secure Shell
tags:
- ssh
comment: true
toc: true

---


[OpenSSH][openssh]是基於[Secure Shell](https://en.wikipedia.org/wiki/Secure_Shell 'WikiPedia')協議的安全套件，通過加密網絡流量實現網絡通信安全，主要於遠程連接。同時提供創建安全隧道(tunnel)、多重認證機制等功能。

<!--more-->

OpenSSH由以下套件組成:

1. 遠程操作工具: [ssh](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh&sektion=1), [scp](http://www.openbsd.org/cgi-bin/man.cgi?query=scp&sektion=1) 和 [sftp](http://www.openbsd.org/cgi-bin/man.cgi?query=sftp&sektion=1)
2. 密鑰管理工具: [ssh-add](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-add&sektion=1), [ssh-keysign](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-keysign&sektion=8), [ssh-keyscan](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-keyscan&sektion=1) 和 [ssh-keygen](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-keygen&sektion=1)
3. 服務器端工具: [sshd](http://www.openbsd.org/cgi-bin/man.cgi?query=sshd&sektion=8), [sftp-server](http://www.openbsd.org/cgi-bin/man.cgi?query=sftp-server&sektion=8) 和 [ssh-agent](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1)


## OpenSSH Installation
OpenSSH的Client和Server在CentOS、Debian等發行版中是2個獨立的安裝包，而在OpenSUSE中，包`openssh`同時包含二者，只有一個安裝包。

### Installing

```bash
# CentOS/RedHat
sudo yum install openssh-clients -y
sudo yum install openssh-server -y

# Debian/Ubunut
sudo apt-get install openssh-clients -y
sudo apt-get install openssh-server -y

# OpenSUSE contain server & client side
sudo zpyyer install openssh -y
```

### Uninstalling

```bash
sudo systemctl stop sshd.service
sudo systemctl disable sshd.service

# CentOS/RedHat
sudo yum erase openssh-server -y

# Debian/Ubunut
sudo apt-get purge openssh-server -y
sudo apt-get autoremove -y

# OpenSUSE
sudo zypper remove -y openssh
```

如果配置了防火牆(firewalld/iptables/ufw/SuSEfirewall2)，需修改規則後重啓相關服務。


## Configuration Files
SSH相關配置文件，可通過如下命令查看

```bash
man ssh | sed -n '/^FILES/,/^EXIT STATUS/p' | sed '$d'
```


file|
---|---
~/.rhosts|
~/.shosts|
~/.ssh/|
~/.ssh/authorized_keys|
~/.ssh/config|
~/.ssh/environment|
~/.ssh/identity|
~/.ssh/id_dsa|
~/.ssh/id_ecdsa|
~/.ssh/id_ed25519|
~/.ssh/id_rsa|
~/.ssh/identity.pub|
~/.ssh/id_dsa.pub|
~/.ssh/id_ecdsa.pub|
~/.ssh/id_ed25519.pub|
~/.ssh/id_rsa.pub|
~/.ssh/known_hosts|
~/.ssh/rc|
/etc/hosts.equiv|
/etc/ssh/shosts.equiv|
/etc/ssh/ssh_config|
/etc/ssh/ssh_host_key|
/etc/ssh/ssh_host_dsa_key|
/etc/ssh/ssh_host_ecdsa_key|
/etc/ssh/ssh_host_ed25519_key|
/etc/ssh/ssh_host_rsa_key|
/etc/ssh/ssh_known_hosts|
/etc/ssh/sshrc|


常用配置文件

file|explanation
---|---
~/.ssh/|登錄用戶SSH配置文件目錄
~/.ssh/id_rsa|用於遠程登錄認證的私鑰(私有)
~/.ssh/id_rsa.pub|用於遠程登錄認證的公鑰(公開)
~/.ssh/authorized_keys|遠程用戶的用於遠程登錄該主機的公鑰信息
~/.ssh/known_hosts|遠程用戶用於遠程登錄該主機的主機信息
~/.ssh/config|登錄用戶配置文件(rwx 600)
/etc/ssh/ssh_config|OpenSSH客戶端配置文件
/etc/ssh/sshd_config|OpenSSH服務器端配置文件

文件的讀寫權限

file|attribute
---|---
`~/.ssh/`|700
`~/.ssh/id_ed25519`|600
`~/.ssh/id_ed25519.pub`|644
`~/.ssh/known_hosts`|644
`~/.ssh/authorized_keys`|600

**注**

1. SSH默認端口號是`TCP 22`
2. 目錄`~/`表示登錄用戶家目錄`$HOME`
3. 如果文件`/etc/nologin`存在，則sshd拒絕任何用戶登錄(root用戶除外)；
4. 如果遠程主機安裝了`tcp-wrappers`，則會生成文件`/etc/hosts.allow`、`/etc/hosts.deny`，用於訪問控制

可通過如下命令查看SSH支持的密碼算法

```bash
ssh -Q {cipher | cipher-auth | mac | kex | key}
```

## Configurations Optimization
參數修改的出發點是提升OpenSSH安全性，主要參考

* [Security/Guidelines/OpenSSH -- Mozilla](https://wiki.mozilla.org/Security/Guidelines/OpenSSH 'Mozilla')
* [Top 20 OpenSSH Server Best Security Practices](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html 'nixCraft') (含有防火牆iptables配置)


可使用如下命令生成強密碼

```bash
len=20

# method 1
tr -dc A-Za-z0-9_ < /dev/urandom | head -c $len | xargs

# method 2
openssl rand -base64 $len
```

### sshd_config Optimization
文件`/etc/ssh/sshd_config`中各指令的說明可通過如下命令查看

```bash
man 5 sshd_config
```

參數針對最新OpenSSH版本設置，不特意兼容舊版本。執行如下命令進行參數修改(不更改默認端口號`22`)

```bash
# Only Use SSH Protocol 2
sudo sed -i -r 's@^#?(Protocol 2)@\1@' /etc/ssh/sshd_config

# Limit Users' SSH Access, separated by spaces
# DenyUsers > AllowUsers > DenyGroups > AllowGroups

# Log Out Timeout Interval, just work for Protocol 2
[[ ! -z $(sed -n -r '/^#?ClientAliveCountMax/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(ClientAliveCountMax).*@\1 0@' /etc/ssh/sshd_config || sudo sed -i -r '$a ClientAliveCountMax 0' /etc/ssh/sshd_config

[[ ! -z $(sed -n -r '/^#?ClientAliveInterval/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(ClientAliveInterval).*@\1 180@' /etc/ssh/sshd_config || sudo sed -i -r '$a ClientAliveInterval 180' /etc/ssh/sshd_config

# Disallow the system send TCP keepalive messages to the other side
[[ ! -z $(sed -n -r '/^#?TCPKeepAlive/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(TCPKeepAlive).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a TCPKeepAlive no' /etc/ssh/sshd_config

# Don't read the user's ~/.rhosts and ~/.shosts files
[[ ! -z $(sed -n -r '/^#?IgnoreRhosts/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(IgnoreRhosts).*@\1 yes@' /etc/ssh/sshd_config || sudo sed -i -r '$a IgnoreRhosts yes' /etc/ssh/sshd_config

# Use Public Key Based Authentication
[[ ! -z $(sed -n -r '/^#?PubkeyAuthentication/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(PubkeyAuthentication).*@\1 yes@' /etc/ssh/sshd_config || sudo sed -i -r '$a PubkeyAuthentication yes' /etc/ssh/sshd_config

# Just Allow Public Key Authentication Login
[[ ! -z $(sed -n -r '/^#?AuthenticationMethods/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(AuthenticationMethods).*@\1 publickey@' /etc/ssh/sshd_config || sudo sed -i -r '$a AuthenticationMethods publickey' /etc/ssh/sshd_config

# Disable Host-Based Authentication
[[ ! -z $(sed -n -r '/^#?HostbasedAuthentication/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(HostbasedAuthentication).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a HostbasedAuthentication no' /etc/ssh/sshd_config

# Disable root Login via SSH {yes,without-password,forced-commands-only,no}
[[ ! -z $(sed -n -r '/^#?PermitRootLogin/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(PermitRootLogin).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a PermitRootLogin no' /etc/ssh/sshd_config

# Disable Password Authentication
[[ ! -z $(sed -n -r '/^#?PasswordAuthentication/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(PasswordAuthentication).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a PasswordAuthentication no' /etc/ssh/sshd_config

# Disallow Empty Password Login
[[ ! -z $(sed -n -r '/^#?PermitEmptyPasswords/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(PermitEmptyPasswords).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a PermitEmptyPasswords no' /etc/ssh/sshd_config

# Enable a Warning Banner After Login, just change /etc/motd
# Enable a Warning Banner Before Login, default none
[[ ! -z $(sed -n -r '/^#?Banner/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(Banner).*@\1 /etc/ssh/banner@' /etc/ssh/sshd_config || sudo sed -i -r '$a Banner /etc/ssh/banner' /etc/ssh/sshd_config

sed -r -n 's@PRETTY_NAME="(.*)"@\1@p' /etc/os-release | sudo tee /etc/ssh/banner

# Enable Logging Message {QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG, DEBUG1, DEBUG2, DEBUG3}
[[ ! -z $(sed -n -r '/^#?LogLevel/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(LogLevel).*@\1 VERBOSE@' /etc/ssh/sshd_config || sudo sed -i -r '$a LogLevel VERBOSE' /etc/ssh/sshd_config

# Turn on privilege separation
[[ ! -z $(sed -n -r '/^#?UsePrivilegeSeparation/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(UsePrivilegeSeparation).*@\1 sandbox@' /etc/ssh/sshd_config || sudo sed -i -r '$a UsePrivilegeSeparation sandbox' /etc/ssh/sshd_config

# Check file modes and ownership of the user's files and home directory before accepting login
[[ ! -z $(sed -n -r '/^#?StrictModes/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(StrictModes).*@\1 yes@' /etc/ssh/sshd_config || sudo sed -i -r '$a StrictModes yes' /etc/ssh/sshd_config

# Ciphers Setting
[[ ! -z $(sed -n -r '/^#?Ciphers.*aes.*/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(Ciphers).*@\1 chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr@' /etc/ssh/sshd_config || sudo sed -i -r '$a Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr' /etc/ssh/sshd_config

# Supported HostKey algorithms by order of preference
sudo sed -i -r 's@^#?(HostKey /etc/ssh/ssh_host_dsa_key)$@#\1@' /etc/ssh/sshd_config
sudo sed -i -r 's@^#?(HostKey /etc/ssh/ssh_host_rsa_key)$@\1@' /etc/ssh/sshd_config
sudo sed -i -r 's@^#?(HostKey /etc/ssh/ssh_host_ecdsa_key)$@\1@' /etc/ssh/sshd_config
sudo sed -i -r 's@^#?(HostKey /etc/ssh/ssh_host_ed25519_key)$@\1@' /etc/ssh/sshd_config

# Specifies the available KEX (Key Exchange) algorithms
sudo sed -i -r '/^#?KexAlgorithms/d;$a KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256' /etc/ssh/sshd_config

# Message authentication codes (MACs) Setting
sudo sed -i -r '/^#?MACs/d;$a MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-ripemd160-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,hmac-ripemd160,umac-128@openssh.com' /etc/ssh/sshd_config

#此處禁止TCP端口、X11轉發

# Disable TCP Port forwarding {yes,no,local,remote}
[[ ! -z $(sed -n -r '/^#?AllowTcpForwarding/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(AllowTcpForwarding).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a AllowTcpForwarding no' /etc/ssh/sshd_config

# Disable X11 forwarding
[[ ! -z $(sed -n -r '/^#?X11Forwarding/p' /etc/ssh/sshd_config) ]] && sudo sed -i -r 's@^#?(X11Forwarding).*@\1 no@' /etc/ssh/sshd_config || sudo sed -i -r '$a X11Forwarding no' /etc/ssh/sshd_config

```


### ssh_config Optimization
文件`~/.ssh/config`中各指令的說明可通過如下命令查看

```bash
man 5 ssh_config
```

執行如下命令進行參數修改

```bash
touch ~/.ssh/config
chmod 600 ~/.ssh/config

tee ~/.ssh/config <<-EOF
# Ensure KnownHosts are unreadable if leaked - it is otherwise easier to know which hosts your keys have access to.
HashKnownHosts yes

# Host keys the client accepts - order here is honored by OpenSSH
HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-ed25519,ssh-rsa,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256

KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256

MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
EOF

```


## SSH-Kegen Keys

### Generating SSH-Keygen Keys
創建SSH認證密鑰須使用`ssh-keygen`命令，密鑰類型建議使用`RSA`、`ED25519`，不建議使用`DSA`和`ECDSA`。需要向後兼容的(如CentOS6.8)使用`RSA`，不需要向後兼容的使用`ED25519`。

生成的密鑰默認保存`$HOME/.ssh/`中(`$HOME`是用戶家目錄)，如果該目錄不存在，則會自動創建，也可通過參數`-f`手動指定路徑。

執行如下命令創建認證密鑰

```bash
#　RSA
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_$(date +%Y-%m-%d) -C "Comments"

#Ｄ25519
# This is only compatible with OpenSSH 6.5+ and fixed-size (256 bytes).
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_$(date +%Y-%m-%d) -C "Comments"
```

**參數解釋**

option|explanation
---|---
`-t`|Specifies the type of key to create.
`-b bits`|Specifies the number of bits in the key to create.
`-f`|Specifies the filename of the key file.
`-C comment`|Provides a new comment.


### Installing Public Key To Remote Server
將公鑰安裝到目標主機中，之後再使用SSH遠程連接，無須輸入用戶密碼即可直接登錄，但可能會提示輸入私鑰的passphrase。

安裝方式主要2種，取決於本機是否安裝有`ssh-copy-id`

1. 直接使用`ssh-copy-id`命令安裝
2. 使用SSH連接，手動創建相關目錄文件

#### Via ssh-copy-id
語法格式

```
ssh-copy-id [-n] [-i [identity_file]] [-p port] [-o ssh_option] [user@]hostname
```

執行如下命令

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 22 flying@192.168.0.140
```

操作成功後，提示

>Now try logging into the machine, with:   "ssh 'flying@192.168.0.140'"
and check to make sure that only the key(s) you wanted were added.

在遠程主機`flying@192.168.0.140`中的`/home/flying`目錄下自動創建`.ssh/authorized_keys`文件，而其內容含有公鑰`~/.ssh/id_ed25519.pub`中內容。


#### Manual Create Related Files
執行如下命令進行操作

```bash
cat $HOME/.ssh/id_ed25519.pub | ssh -p 22 flying@192.168.0.140 '(umask 077; [[ -d ~/.ssh ]] || mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys)'

cat $HOME/.ssh/id_rsa.pub | ssh -p 22 flying@192.168.0.140 '(umask 077; [[ -d ~/.ssh ]] || mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys)'
```

命令解釋
* `umask 077`: 指定掩碼，爲創建目錄、文件指定默認權限，目錄是(777-077=700)，文件是(666-077=600)
* `[[ -d ~/.ssh ]]`: 測試命令，判斷目錄是否存在
* `mkdir`: 創建目錄
* `>>`: 追加輸出重定向，文件不存在則自動創建


直接使用

```bash
ssh -p 22 flying@192.168.0.140
```
登錄，提示輸入密鑰密碼後(假如創建密鑰時設置了密碼)，即可直接登錄，無需輸入用戶密碼。


## SSH Simage Usage
使用ssh操作實例

### Execute Remote Script

執行如下命令

```bash
ssh -p 22 flying@192.168.0.140 'bash /tmp/test.sh'
```

解釋： 通過ssh遠程連接主機192.168.0.140:22，用戶名爲flying，執行遠程主機中的腳本`/tmp/test.sh`


### Remote Backup and Restore Compressed Files
使用`tar`進行歸檔操作，默認使用相對路徑，如果直接使用絕對路徑，會報如下錯誤`tar: Removing leading '/' from member names`。使用絕對路徑須加參數`-p`，但不建議，原因是使用絕對路徑，解壓時會出現問題，可能會覆蓋已經存在的文件。

此處假定遠程服務器ip爲`192.168.0.140`

* 將本地目錄文件以壓縮包形式保存到遠程服務器

```bash
cd /home/$nowUser/Documents/

# method 1
tar Jcf - node_hexo/ | ssh -p 22 flying@192.168.0.140 "cat > /tmp/hexo.tar.xz"

# method 2
tar Jcf - node_hexo/ | ssh -p 22 flying@192.168.0.140 "dd of=/tmp/hexo.tar.xz"
```

* 將遠程服務器的壓縮包文件解壓到本地指定目錄(/tmp)

```bash
# method 1
ssh -p 22 flying@192.168.0.140 "cat /tmp/hexo.tar.xz" | tar Jxf - -C /tmp

# method 2
ssh -p 22 flying@192.168.0.140 "dd if=/tmp/hexo.tar.xz" | tar Jxf - -C /tmp
```

注意： tar後的`-`不可省去，會報錯；`-C`表示指定解壓縮目錄路徑；`dd`表示格式化和複製文件

### Remote Copy Database Data Cross Server
使用SSH和管道符`|`實現跨服務器數據複製，命令格式如下

```bash
mysqldump -uroot -pXXX database_name | ssh xxx@XXX -pXXX mysql -utianyun -pXXX database_name
```

具體實例參看本人之前寫的博文
* [MariaDB使用管道符和SSH進行跨服務器的表複製](https://github.com/LempStacker/Qingtianjiedu-Blog-Backup/blob/master/origin/MariaDB%E4%BD%BF%E7%94%A8%E7%AE%A1%E9%81%93%E7%AC%A6%E5%92%8CSSH%E9%80%B2%E8%A1%8C%E8%B7%A8%E6%9C%8D%E5%8B%99%E5%99%A8%E7%9A%84%E8%A1%A8%E8%A4%87%E8%A3%BD.md)
* [MariaDB使用管道符進行跨庫、跨服務器的表複製](https://github.com/LempStacker/Qingtianjiedu-Blog-Backup/blob/master/origin/MariaDB%E4%BD%BF%E7%94%A8%E7%AE%A1%E9%81%93%E7%AC%A6%E9%80%B2%E8%A1%8C%E8%B7%A8%E5%BA%AB%E3%80%81%E8%B7%A8%E6%9C%8D%E5%8B%99%E5%99%A8%E7%9A%84%E8%A1%A8%E8%A4%87%E8%A3%BD.md)

### Compare Remote and Locale File
使用SSH、管道符號`|`和`diff`命令對文件內容進行比較，此使用案例是`HA Cluster`中比較各節點主機間配置文件內容是否一致

```bash
ssh node2 'cat /etc/corosync/corosync.conf' | diff - /etc/corosync/corosync.conf
```

使用`diff`命，如果內容一致，不輸出任何內容

```bash
[maxdsreg@node1 ~]$ ssh node2 'cat /etc/corosync/corosync.conf' | diff - /etc/corosync/corosync.conf
Enter passphrase for key '/home/flying/.ssh/id_rsa':

#命令執行返回0，表示成功
[maxdsreg@node1 ~]$ echo $?
0
[maxdsreg@node1 ~]$
```

以下內容不一致的操作示例

```bash
[maxdsreg@node1 ~]$ ssh node2 'cat /etc/corosync/corosync.conf' | diff - /etc/corosync/corosync.conf
Enter passphrase for key '/home/flying/.ssh/id_rsa':
5c5
<     transport: udpu1
---
>     transport: udpu

#命令執行返回1，表示失敗
[maxdsreg@node1 ~]$ echo $?
1
[maxdsreg@node1 ~]$
```


## References
* [Security/Guidelines/OpenSSH -- Mozilla](https://wiki.mozilla.org/Security/Guidelines/OpenSSH 'Mozilla')
* [SSH/OpenSSH/Configuring](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
* [Secure Shell -- Arch Linux](https://wiki.archlinux.org/index.php/Secure_Shell 'ArchLinux')
* [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html)
* [Top 20 OpenSSH Server Best Security Practices](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html 'nixCraft')
* [Linux PAM configuration that allows or deny login via the sshd server](https://www.cyberciti.biz/tips/linux-pam-configuration-that-allows-or-deny-login-via-the-sshd-server.html 'nixCraft')
* [Secure OpenSSH Config Reference](https://calomel.org/openssh.html)
* [FAQ about setting and using TCP wrappers](http://www.cyberciti.biz/faq/tcp-wrappers-hosts-allow-deny-tutorial/)


## Bibliography
* [OpenSSH: Key Management Needs Attention](https://www.ssh.com/ssh/openssh/)
* [SSH: Secure Network Operations](https://doc.opensuse.org/documentation/leap/security/html/book.security/cha.ssh.html 'OpenSUSE')
* [OpenSSH - WikiBooks](https://en.wikibooks.org/wiki/OpenSSH)


[openssh]:https://www.openssh.com/


## Change Logs
* 2016.02.02 10:25 Tue Asia/Beijing
	* 初稿完成
* 2017.02.14 16:56 Tue Asia/Shanghai
    * 內容重構
* 2018.07.31 21:46:32 Tue Asia/Shanghai
    * 勘誤，更新，遷移到新Blog

<!-- End -->
