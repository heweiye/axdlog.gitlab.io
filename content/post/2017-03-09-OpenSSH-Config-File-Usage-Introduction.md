---
title: OpenSSH Config File Usage Introduction
slug: OpenSSH Config File Usage Introduction
date: 2017-03-09T00:01:52+08:00
lastmod: 2018-07-27T14:26:18-04:00
draft: false
keywords: ["AxdLog", "ssh", "port forward", "x forward"]
description: "OpenSSH Config File Usage Introduction"
categories:
- Secure Shell
tags:
- ssh
comment: true
toc: true

---

[OpenSSH][openssh]是使用SSH協議進行遠程連接的工具，當前最新版本是[OpenSSH 7.7](https://www.openssh.com/txt/release-7.7)，於 **April 03, 2018** 釋出。

[OpenSSH][openssh]可分爲 *client* 和 *server* 端。其中 *client* 的配置文件有２個，一個是全局配置文件`/etc/ssh/ssh_config`，另一個是用戶配置文件`~/.ssh/config`，默認後者中的設置會覆蓋前者(全局配置文件)。通過配置`~/.ssh/config`，可實現SSH的自定義操作。

本文主要介紹`~/.ssh/config`的配置、使用，實驗主機爲[Digital Ocean][digitalocean]的VPS主機。

<!--more-->

## Preparation
在[Digital Ocean][digitalocean]中創建2台VPS，系統選用`Debian Stretch 9.5`，一台作爲跳板機，一台作爲內網主機，只能通過跳板機訪問，通過SSH keygen進行認證連接。

VPS信息如下

Hostname|Intern IP|Public IP|Port
---|---|---|---
front|`10.128.29.79`|`162.243.63.4`|22
secret|`10.128.24.198`|`162.243.86.165`|22

在主機`front`中創建ssh genkey，key的類型選擇`ed25519`，將公鑰中的內容添加到主機`secret`的`~/.ssh/authorized_key`中。此處假設已經進行防火牆配置，只允許通過主機`front`訪問主機`secret`。

爲方便進行實驗，將在主機`front`生成的密鑰對保存到本地主機`/tmp`目錄下。

測試過程如下

```bash
maxdsre@Stretch:~$ ssh front
Last login: Thur Jul  26 22:05:54 2018 from 116.235.185.156
root@front:~# ssh root@10.128.24.198
Last login: Thur Jul  26 22:13:19 2018 from 10.128.29.79
root@secret:~# exit
logout

Connection to 10.128.24.198 closed.
root@front:~#
```


## Introduction
`ssh_config`的說明可通過如下命令查看

```bash
man ssh_config
```

也可參閱文檔 [SSH Config File for OpenSSH Client](https://www.ssh.com/ssh/config/ "ssh.com")

通過`~/.ssh/config`配置的主機，可以通過`ssh`、`sftp`等命令進行連接。

### Parameters List
詳細配置參數清單如下

parameter|detail
---|---
Host|
Match|
AddKeysToAgent|
AddressFamily|
BatchMode|
BindAddress|
CanonicalDomains|
CanonicalizeFallbackLocal|
CanonicalizeHostname|
CanonicalizeMaxDots|
CanonicalizePermittedCNAMEs|
CertificateFile|
ChallengeResponseAuthentication|
CheckHostIP|
Cipher|
Ciphers|
ClearAllForwardings|
Compression|
CompressionLevel|
ConnectionAttempts|
ConnectTimeout|
ControlMaster|
ControlPath|
ControlPersist|
DynamicForward|
EnableSSHKeysign|
EscapeChar|
ExitOnForwardFailure|
FingerprintHash|
ForwardAgent|
ForwardX11|
ForwardX11Timeout|
ForwardX11Trusted|
GatewayPorts|
GlobalKnownHostsFile|
GSSAPIAuthentication|
GSSAPIKeyExchange|
GSSAPIClientIdentity|
GSSAPIServerIdentity|
GSSAPIDelegateCredentials|
GSSAPIRenewalForcesRekey|
GSSAPITrustDns|
HashKnownHosts|
HostbasedAuthentication|
HostbasedKeyTypes|
HostKeyAlgorithms|
HostKeyAlias|
HostName|
IdentitiesOnly|
IdentityAgent|
IdentityFile|
IgnoreUnknown|
Include|
IPQoS|
KbdInteractiveAuthentication|
KbdInteractiveDevices|
KexAlgorithms|
LocalCommand|
LocalForward|
LogLevel|
MACs|
NoHostAuthenticationForLocalhost|
NumberOfPasswordPrompts|
PasswordAuthentication|
PermitLocalCommand|
PKCS11Provider|
Port|
PreferredAuthentications|
Protocol|
ProxyCommand|
ProxyJump|
ProxyUseFdpass|
PubkeyAcceptedKeyTypes|
PubkeyAuthentication|
RekeyLimit|
RemoteForward|
RequestTTY|
RevokedHostKeys|
RhostsRSAAuthentication|
RSAAuthentication|
SendEnv|
ServerAliveCountMax|
ServerAliveInterval|
StreamLocalBindMask|
StreamLocalBindUnlink|
StrictHostKeyChecking|
TCPKeepAlive|
Tunnel|
TunnelDevice|
UpdateHostKeys|
UsePrivilegedPort|
User|
UserKnownHostsFile|
VerifyHostKeyDNS|
VisualHostKey|
XAuthLocation|

### Format
在文件`~/.ssh/config`中配置的格式如下

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	Port 22
	User root

Host ...
    ...
    ...
    ...

Host *
    Protocol 2
    StrictHostKeyChecking no
	HashKnownHosts yes
	UserKnownHostsFile /dev/null
```

以`Host`開頭，配額選項以`key val`形式設置，縮進1個Tab(或4個空格)。

`Host *`表示通用配置，適用於`~/.ssh/config`所配置的主機。`Host front`則表示針對主機`front`進行配置，如果通用配置和特定主機中的配置選項重複，則自動忽略通用配置中的選項。


## Simple Login
使用SSH登錄遠程主機，需要提供遠程主機的IP、端口號、用戶名、密碼或key。則`ssh_config`中對應的參數選項如下

item|para
---|---
IP|`HostName`
端口號|`Port`
用戶名|`User`
key|`IdentityFile`

在最簡配置中，只要指定這4個選項即可。

如主機`front`的配置

```bash
Host front
	HostName 162.243.63.4
	Port 22
	User root
	IdentityFile ~/.ssh/id_ed25519
```

處於安全等考慮，還需進行其他參數的配置。

### Common Configuration
其中某些參數的配置，可參考Mozilla的[Security/Guidelines/OpenSSH](https://wiki.mozilla.org/Security/Guidelines/OpenSSH#Configuration)。

如果想通過socket進行通信，可使用指令`ControlMaster`、`ControlPath`、`ControlPersist`，創建目錄`~/.ssh/sockets/`存放生成的socket文件。

ControlMaster auto
ControlPath ~/.ssh/sockets/%r@%h:%p
#ControlPath ~/.ssh/sockets/%r@%h
ControlPersist 300

通用配置如下

```bash
Host *
	Protocol 2
	Port 22
	User root
	StrictHostKeyChecking no
	VerifyHostKeyDNS no
	AddressFamily inet
	HashKnownHosts yes
	UserKnownHostsFile /dev/null
	LogLevel QUIET
	IdentityFile ~/.ssh/id_ed25519
	ServerAliveCountMax 5
	ServerAliveInterval 120
	ControlMaster auto
	ControlPath ~/.ssh/sockets/%r@%h:%p
	#ControlPath ~/.ssh/sockets/%r@%h
	ControlPersist 300
   	#PreferredAuthentications gssapi-with-mic,hostbased,publickey,keyboard-interactive,password
   	PreferredAuthentications publickey,keyboard-interactive,password
	HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-ed25519,ssh-rsa,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256
	KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
	MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
	Compression yes
	#Cipher aes256-ctr
	Cipher aes256-gcm@openssh.com
	Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
```

## Visit Intern Host Via Front
配置通過主機`front`訪問內網主機`secret`(10.128.24.198)，通過參數`ProxyCommand`實現

>In the command string, any occurrence of `%h` will be substituted by the host name to connect, `%p` by the port, and `%r` by the remote user name.

在`ProxyCommand`中，`%h`指代要連接的主機名，`%p`指代端口號，`%r`指代用戶名，SSH會自動進行替換。

### Method 1
方式1通過`ssh front -W %h:%p %r`實現

**注意**

1. ssh後的 *front* 是跳板主機的主機名稱，即`Host front`中的`front`；
2. 主機`secret`指定的key是主機`front`連接`secret`時使用的key；

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519

Host secret
	HostName 162.243.86.165
	IdentityFile /tmp/id_ed25519
    ProxyCommand ssh front -W %h:%p %r
```

演示過程如下

```bash
maxdsre@Stretch:~$ ssh secret

Last login: Thur Jul  26 22:16:03 2018 from 10.128.29.79
root@secret:~# exit
logout

maxdsre@Stretch:~$
```

### Method 2 nc
方式2通過`ssh front nc %h %p %r`實現，但是有前提，命令中的`nc`必須已經安裝在主機`front`中，否則會報錯，無法正常連接。


```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519

Host secret
	HostName 162.243.86.165
	IdentityFile /tmp/id_ed25519
    ProxyCommand ssh front nc %h %p %r
```

### Method 3 ProxyJump
[OpenSSH 7.3](https://www.openssh.com/txt/release-7.3)中添加了`ProxyJump`選項，通過該選項可實現與`ProxyCommand`相同的功能，但設置更爲簡單。

**注意**：要想使用`ProxyJump`，須確保OpenSSH的版本至少爲`7.3`。

具體實例可參見[What is new in OpenSSH 7.4 (in RHEL 7.4)?](https://access.redhat.com/blogs/766093/posts/3051131)中的示例。

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519

Host secret
	HostName 162.243.86.165
	IdentityFile /tmp/id_ed25519
	ProxyJump front
```


## Port Forward
端口轉發分爲`LocalForward`、`RemoteForward`、`DynamicForward`這3種。

在主機`front`中進行操作

### LocalForward
在主機`front`安裝Nginx，在瀏覽器中輸入外網IP`162.243.63.4`即能訪問Web內容，默認爲80端口。通過本地端口轉發，將主機`front`的 *80* 端口轉發到本機的 *9999* 端口，實現通過訪問本地`127.0.0.1:9999`獲取主機`front`中的Web頁面，即80端口顯示的內容。

配置如下內容

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	LocalForward 9999 127.0.0.1:80
```

執行

```bash
ssh -fNg front
```

使用`ss -tnl`可查看到本機已經在監聽`9999`端口。

**注意**：僅指定`-fNg`即可，不要添加`L`、`R`、`D`。如果設置爲`-fNgL`，會出現報錯
>Bad local forwarding specification

提取Head信息比對

```bash
# 162.243.63.4
maxdsre@Stretch:~$ curl -I 162.243.63.4
HTTP/1.1 200 OK
Server: nginx/1.15.2
Date: Thur, 26 Jul 2018 15:12:04 GMT
Content-Type: text/html
Content-Length: 867
Last-Modified: Thur, 26 Jul 2018 15:06:10 GMT
Connection: keep-alive
ETag: "58c01de2-363"
Accept-Ranges: bytes

# 127.0.0.1:9999
maxdsre@Stretch:~$ curl -I 127.0.0.1:9999
HTTP/1.1 200 OK
Server: nginx/1.15.2
Date: Thur, 26 Jul 2018 15:12:14 GMT
Content-Type: text/html
Content-Length: 867
Last-Modified: Thur, 26 Jul 2018 15:06:10 GMT
Connection: keep-alive
ETag: "58c01de2-363"
Accept-Ranges: bytes

maxdsre@Stretch:~$
```

### RemoteForward
本機安裝有Nginx，在瀏覽器中輸入`127.0.0.1`即能訪問Web內容，默認爲80端口。通過遠程端口轉發，將本機的 *80* 端口轉發到遠程主機`front`的 *7777* 端口。實現在主機`front`中通過訪問本地`127.0.0.1:7777`獲取本機中的Web頁面，即80端口顯示的內容。

**注意**：使用主機`front`進行端口轉發，須在主機`front`的`/etc/ssh/sshd_config`中將選項`GatewayPorts`設置爲`yes`

```bash
GatewayPorts yes
```

然後重啓sshd服務

```bash
systemctl restart ssh
```

配置如下內容

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	RemoteForward 7777 127.0.0.1:80
```

**注意**：`RemoteForward`的第一個參數爲遠程主機`front`要啓用的端口號，第二個參數`127.0.0.1:80`爲本機的端口信息。

執行

```bash
ssh -fNg front
```

可在主機`front`中看到已經監聽端口`7777`。

提取Head信息比對

```bash
# local host
maxdsre@Stretch:~$ curl -I 127.0.0.1:80
HTTP/1.1 200 OK
Server: nginx/1.15.2
Date: Thur, 26 Jul 2018 15:49:06 GMT
Content-Type: text/html
Content-Length: 640
Last-Modified: Tue, 07 Mar 2017 00:32:08 GMT
Connection: keep-alive
ETag: "58bdff88-280"
Accept-Ranges: bytes

maxdsre@Stretch:~$


# front host
root@front:~# curl -I 127.0.0.1:7777
HTTP/1.1 200 OK
Server: nginx/1.15.2
Date: Thur, 26 Jul 2018 15:47:56 GMT
Content-Type: text/html
Content-Length: 640
Last-Modified: Tue, 07 Mar 2017 00:32:08 GMT
Connection: keep-alive
ETag: "58bdff88-280"
Accept-Ranges: bytes

root@front:~#
```

### DynamicForward
動態端口轉發，創建SOCKS，在本機監聽`6666`端口

配置如下內容

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	DynamicForward 127.0.0.1:6666
```

執行

```bash
ssh -fNg front
```
使用`ss -tnl`可查看到本機已經在監聽`6666`端口。


## X Forwarding

>`X` is a popular window system for Unix workstations, and one of its best features is its transparency. Using `X`, you can run remote X applications that open their windows on your local display (and vice versa, running local applications on remote displays). Unfortunately, the inter-machine communication is insecure and wide open to snoopers. But there's good news: SSH X forwarding makes the communication secure by tunneling the X protocol. -- https://docstore.mik.ua/orelly/networking_2ndEd/ssh/ch09_03.htm

`X`是一種通信協議，通過該協議，可將遠程主機中的應用在本地窗口中打開，通信默認是不安全的，通過 **SSH X Forwarding** 可確保通信安全。

因`X`協議在認證請求連接的客戶端時，需要對客戶端進行身份認證，方式可分爲兩種`Host-based X`和`Key-based X`。後者通過程序`xauth`維護X的認證key，通常保存在文件`~/.Xauthority`中。

### Step 1 xauth
故首先須確保在本地和遠程主機中安裝了`xauth`。

### Step 2 sshd_config
然後在配置文件`/etc/ssh/sshd_config`中進行如下配置

```bash
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
```
修改完成後重啓sshd服務。

如果`X11UseLocalhost no`爲`no`，則會出現如下報錯

```bash
/usr/bin/xauth: (stdin):1:  bad display name "Malachite:10.0" in "remove" command
/usr/bin/xauth: (stdin):2:  bad display name "Malachite:10.0" in "add" command
```

其中`Malachite:10.0`的含義，可閱讀如下說明：

>A central concept of `X` is the *display*, an abstraction for the screen managed by an X server. When an X client is invoked, it needs to know which display to use. Displays are named by strings of the form *HOST:n.v*, where:
>
* `HOST` is the name of the machine running the X server controlling the display.
* `n` is the *display number*, an integer, usually 0. X allows for multiple displays controlled by a single server; additional displays are numbered 1, 2, and so on.
* `v` is the *visual number*, another integer. A visual is a virtual display. X supports multiple virtual displays on a single, physical display. If there's only one virtual display (which is the most common scenario), you omit the ".v", and the default is visual 0.  -- https://docstore.mik.ua/orelly/networking_2ndEd/ssh/ch09_03.htm


### Step 3 DISPLAY
如果不使用X轉發，但想使使用遠程主機(通過SSH登錄)中的X，則必須手動設置變量`DISPLAY`，說明如下

>SSH sets the `DISPLAY` variable automatically only if X forwarding is in effect. If you don't use X forwarding but want to use X on a remote machine you logged into via SSH, remember that **you have to set the DISPLAY variable yourself**. You should only do this when the both machines are on the same, trusted network, as the X protocol by itself is quite insecure.	-- https://docstore.mik.ua/orelly/networking_2ndEd/ssh/ch09_03.htm

在本機(X client)中進行如下設置

```bash
export DISPLAY=:0.0
```

也可将其写入文件`~/.bashrc`中。

### Step 4 ssh -Y
操作完成後即可通過`ssh -X`或`ssh -Y`登錄遠程主機，啓用含有圖形化界面的程序，即可看到效果(在本地跳出新窗口，內容爲遠程主機中的圖形化程序)。

>-X      Enables X11 forwarding
-Y      Enables trusted X11 forwarding.

如果按照以上操作执行后仍报错，可退出当前用户登录，重新登入后尝试。


## The Onion Router
The Onion Router簡稱[Tor](https://www.torproject.org/)，是一個匿名網絡。可通過`LocalForward`進行端口轉發，在本機連入Tor網絡，實現網路匿名訪問。

在VPS中安裝`tor`，服務啓用後，監聽 *9050* 端口。

配置如下

```bash
Host tor
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	LocalForward 3333 127.0.0.1:9050
```

執行

```bash
ssh -fNg tor
```

操作成功後，會在本機啓用 *3333* 端口，通過 *127.0.0.1:3333* 可連入Tor網絡。

如果需要獲取本機的真實外網IP，可通過如下命令查看

```bash
dig +short myip.opendns.com @resolver1.opendns.com
```

該命令參考於 [Command for determining my public IP?](https://askubuntu.com/questions/95910/command-for-determining-my-public-ip#answer-145017)。


## ~/.ssh/config
完整配置文件內容如下

```bash
Host front
	HostName 162.243.63.4
	IdentityFile ~/.ssh/id_ed25519
	# LocalForward 9999 127.0.0.1:80
	# RemoteForward 7777 127.0.0.1:80
	# DynamicForward 127.0.0.1:6666

Host secret
	HostName 162.243.86.165
	IdentityFile /tmp/id_ed25519
    ProxyCommand ssh front -W %h:%p %r


Host *
	Protocol 2
	Port 22
	User root
	StrictHostKeyChecking no
	VerifyHostKeyDNS no
	AddressFamily inet
	HashKnownHosts yes
	UserKnownHostsFile /dev/null
	LogLevel QUIET
	IdentityFile ~/.ssh/id_ed25519
	ServerAliveCountMax 5
	ServerAliveInterval 120
	ControlMaster auto
	ControlPath ~/.ssh/sockets/%r@%h:%p
	ControlPersist 180
   	#PreferredAuthentications gssapi-with-mic,hostbased,publickey,keyboard-interactive,password
   	PreferredAuthentications publickey,keyboard-interactive,password
	HostKeyAlgorithms ssh-ed25519-cert-v01@openssh.com,ssh-rsa-cert-v01@openssh.com,ssh-ed25519,ssh-rsa,ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,ecdsa-sha2-nistp256-cert-v01@openssh.com,ecdsa-sha2-nistp521,ecdsa-sha2-nistp384,ecdsa-sha2-nistp256
	KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
	MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com
	Compression yes
	Cipher aes256-gcm@openssh.com
	Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
```

## References
* [How To Configure Custom Connection Options for your SSH Client](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client)
* [OpenSSH Config File Examples](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)
* [SSH ProxyCommand example: Going through one host to reach another server](https://www.cyberciti.biz/faq/linux-unix-ssh-proxycommand-passing-through-one-host-gateway-server/)
* [Simplify Your Life With an SSH Config File](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)
* [Limiting Exposure via SSH ProxyJump](https://storrgie.epiphyte.network/limiting-exposure-via-ssh-proxyjump/)
* [set-up X11 Forwarding over ssh](https://stackoverflow.com/questions/19589844/set-up-x11-forwarding-over-ssh#answer-23033038)
* [SSH XForwarding fails - xauth bad display name](https://unix.stackexchange.com/questions/112217/ssh-xforwarding-fails-xauth-bad-display-name#answer-112218)


## Bibliography
* [OpenSSH Manual Pages](https://www.openssh.com/manual.html)
* [doc/TorifyHOWTO/ssh](https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/ssh)


## Change Logs
* 2017.03.08 23:59 Wed Asia/Shanghai
    * 初稿完成
* 2018.07.27 14:48:32 Fri America/Boston
    * 勘誤，更新，遷移到新Blog


[openssh]:https://www.openssh.com "OpenSSH"
[digitalocean]:https://www.digitalocean.com "Digital Ocean"

<!-- End -->
