---
title: Connecting VNC Server Via SSH Tunnel On CentOS 7
slug: Connecting VNC Server Via SSH Tunnel On CentOS 7
date: 2017-01-02T18:13:14+08:00
lastmod: 2018-08-01T22:36:18+08:00
draft: false
keywords: ["SSH", "VNC"]
description: "Connecting VNC Server Via SSH Tunnel On CentOS 7"
categories:
- SSH
tags:
- ssh
- vnc
comment: true
toc: true

---

[Virtual Network Computing](https://en.wikipedia.org/wiki/Virtual_Network_Computing 'WikiPedia')(VNC)是基於[Remote Framebuffer (RFB)](https://en.wikipedia.org/wiki/RFB_protocol)協議([RFC6143](https://tools.ietf.org/html/rfc6143))的圖形化桌面共享系統，可通過網路，控制遠程主機的桌面。

本文記錄在CentOS7.x中安裝、配置VNC Server([tigervnc](http://tigervnc.org/))，通過VNC連接遠程主機的圖形化桌面，並通過創建SSH Tunnel實現加密通信。

<!--more-->

## Introduction
### VNC

Virtual Network Computing (VNC)的RFC編號是[7869](https://tools.ietf.org/html/rfc7869)。

>In computing, `Virtual Network Computing` (VNC) is a graphical desktop sharing system that uses the Remote Frame Buffer protocol (RFB) to remotely control another computer. It transmits the keyboard and mouse events from one computer to another, relaying the graphical screen updates back in the other direction, over a network. -- <https://en.wikipedia.org/wiki/Virtual_Network_Computing>

### TigerVNC
官方介紹

>`TigerVNC` is a high-performance, platform-neutral implementation of VNC (Virtual Network Computing), a client/server application that allows users to launch and interact with graphical applications on remote machines. TigerVNC provides the levels of performance necessary to run 3D and video applications, and it attempts to maintain a common look and feel and re-use components, where possible, across the various platforms that it supports. TigerVNC also provides extensions for advanced authentication methods and TLS encryption. -- <http://tigervnc.org/>

RedHat官方文檔介紹

>`TigerVNC` (Tiger Virtual Network Computing) is a system for graphical desktop sharing which allows you to remotely control other computers.
>
>`TigerVNC` works on the client-server principle: a **server** shares its output (vncserver) and a **client** (vncviewer) connects to the server. -- <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-TigerVNC.html>

`TigerVNC`採用server/client架構，在server端安裝`tigervnc-server`，在client端安裝`tightvnc`或其它vncviewer。


## Preparation
本文所有操作在[DigitalOcean](https://www.digitalocean.com/)的VPS中進行，默認是`root`用戶，為方便使用普通用戶賬戶登錄的用戶，在相關命令前添加`sudo`指令。

VPS相關信息如下

item|detail
---|---
OS|`CentOS Linux release 7.3.1611 (Core)`
Kernel|`3.10.0-514.2.2.el7.x86_64`
IP|`192.241.240.132`

通過SSH連接該主機，命令如下

```bash
ssh -C -c aes256-cbc root@192.241.240.132
```

使用VNC是為了連接圖形化桌面，故需在VPS中安裝圖形化卓名，此處選擇`GNOME Desktop`。執行如下命令進行安裝

```bash
sudo yum groupinstall -y "GNOME Desktop"
```

VNC軟件選擇[tigervnc](http://tigervnc.org/)，安裝、配置過程參考RedHat官方文檔 [CHAPTER 13. TIGERVNC](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-TigerVNC.html 'RedHat')。


## TigerVNC Server
### Installing
安裝`tigervnc`，可通過如下命令查看相關安裝包

```bash
sudo yum info tigervnc* | awk '$1=="Name"{print $NF}'
```

此處只安裝`tigervnc-server`，為演示方便添加普通用戶`tigervnc`，*display_number* 設置為`1`。

執行如下命令安裝`tigervnc-server`

```bash
sudo yum install -y tigervnc-server
# 查看服務管理文件
ls /lib/systemd/system/vncserver@.service
```

執行如下命令創建普通用戶`tigervnc`
```bash
sudo useradd tigervnc
echo 'tigervnc2017' | sudo passwd --stdin tigervnc
```

### Attention
>Unlike in previous Red Hat Enterprise Linux distributions, `TigerVNC` in Red Hat Enterprise Linux 7 uses the `systemd` system management daemon for its configuration. The */etc/sysconfig/vncserver* configuration file has been replaced by */etc/systemd/system/vncserver@.service*. -- <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-TigerVNC.html>

查看文件

```bash
/lib/systemd/system/vncserver@.service
```

其內容如下

```bash
# The vncserver service unit file
#
# Quick HowTo:
# 1. Copy this file to /etc/systemd/system/vncserver@.service
# 2. Edit /etc/systemd/system/vncserver@.service, replacing <USER>
#    with the actual user name. Leave the remaining lines of the file unmodified
#    (ExecStart=/usr/sbin/runuser -l <USER> -c "/usr/bin/vncserver %i"
#     PIDFile=/home/<USER>/.vnc/%H%i.pid)
# 3. Run `systemctl daemon-reload`
# 4. Run `systemctl enable vncserver@:<display>.service`
#
# DO NOT RUN THIS SERVICE if your local area network is
# untrusted!  For a secure way of using VNC, you should
# limit connections to the local host and then tunnel from
# the machine you want to view VNC on (host A) to the machine
# whose VNC output you want to view (host B)
#
# [user@hostA ~]$ ssh -v -C -L 590N:localhost:590M hostB
#
# this will open a connection on port 590N of your hostA to hostB's port 590M
# (in fact, it ssh-connects to hostB and then connects to localhost (on hostB).
# See the ssh man page for details on port forwarding)
#
# You can then point a VNC client on hostA at vncdisplay N of localhost and with
# the help of ssh, you end up seeing what hostB makes available on port 590M
#
# Use "-nolisten tcp" to prevent X connections to your VNC server via TCP.
#
# Use "-localhost" to prevent remote VNC clients connecting except when
# doing so through a secure tunnel.  See the "-via" option in the
# `man vncviewer' manual page.


[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
ExecStart=/usr/sbin/runuser -l <USER> -c "/usr/bin/vncserver %i"
PIDFile=/home/<USER>/.vnc/%H%i.pid
ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

[Install]
WantedBy=multi-user.target
```

註釋部分敘述了配置過程及通過SSH端口轉發實現加密通信。

**重要**：在複製文件`/etc/systemd/system/vncserver@.service`時，涉及到 *display_number*，該參數可指定具體的數值，如`1`、`2`、`3`等。VNC Server默認端口是`5900`，如果指定了 *display_number*，則VNC Server最終的監聽端口號為`5900 + display_number`。比如：指定 *display_number* 為1，則最終的監聽端口號為`5901`；指定 *display_number* 為2，則最終的監聽端口號為`5902`，依次類推。監聽端口可通過如下命令查看

```bash
sudo ss -tnlp | grep -i vnc
#或
sudo netstat -tnlpe | grep -i vnc
```

>The default port of VNC server is 5900. To reach the port through which a remote desktop will be accessible, sum the default port and the user's assigned display number. For example, for the second display: 2 + 5900 = 5902. -- <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sec-vnc-viewer.html>


此處指定 *display_number* 為`1`。

### Configuring
執行如下命令進行配置

```bash
#複製服務管理文件
sudo cp -f /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service

#替換文件中的<USER>為目標用戶
#user: tigervnc
sudo sed -i '/^[^#]/s@<USER>@tigervnc@' /etc/systemd/system/vncserver@:1.service
#current user
# sudo sed -i '/^[^#]/s@<USER>@'"$USER"'@' /etc/systemd/system/vncserver@:1.service

#更改.service文件後須執行該操作
sudo systemctl daemon-reload

#切換到用戶tigervnc
sudo su tigervnc
#設置訪問VNC桌面時的密碼，此處密碼設置為`axdlog2017`
vncpasswd

#啟動服務
sudo systemctl start vncserver@:1
# sudo systemctl start vncserver@:1.service

#設置為開機啟動
sudo systemctl enable vncserver@:1.service
```

查看端口

```bash
#ss -tnlp | grep -i vnc
LISTEN     0     5         *:5901        *:*        users:(("Xvnc",pid=5046,fd=10))
LISTEN     0     128       *:6001        *:*        users:(("Xvnc",pid=5046,fd=1))
LISTEN     0     5        :::5901        :::*       users:(("Xvnc",pid=5046,fd=11))
LISTEN     0     128      :::6001        :::*       users:(("Xvnc",pid=5046,fd=0))
```

## VNC Viewer
### Installing
對於client端而言，只需安裝vncviewer即可，可選擇的軟件有`tigervnc`、`xtightvncviewer`、`xvnc4viewer`等，也可選擇[RealVnc](https://www.realvnc.com/)，或GNOME Desktop中的`vinagre`(Applications-->Utilities-->Remote Desktop Viewer)。

```bash
#CentOS
sudo yum install -y tigervnc

#Debian
sudo apt-get install -y xtightvncviewer
sudo apt-get install -y xvnc4viewer
```


## Connecting Test
### vncviewer
命令格式如下
```bash
vncviewer address:display_number
```

執行
```bash
vncviewer -Shared -FullColour 192.241.240.132:1
# -ViewOnly -FullScreen
```

操作截圖

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/1vncviewer-2017-01-02_17:05:14.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/1vncviewer-2017-01-02_17:07:39.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/1vncviewer-2017-01-02_17:09:48.png)


### Real vncviewer
在如下頁面下載RealVNC Viewer

```http
https://www.realvnc.com/download/viewer/
```
此處下載

```bash
VNC-Viewer-6.0.1-Linux-x64.gz
```
解壓至目錄`/tmp`中，執行
```bash
chmod u+x VNC-Viewer-6.0.1-Linux-x64
./VNC-Viewer-6.0.1-Linux-x64
```

操作截圖
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/2realvnc-2017-01-02_17:13:58.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/2realvnc-2017-01-02_17:14:11.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/2realvnc-2017-01-02_17:14:23.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/2realvnc-2017-01-02_17:14:57.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/2realvnc-2017-01-02_17:16:16.png)

### vinagre
如何打開`Applications`-->`Utilities`-->`Remote Desktop Viewer`

操作截圖
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/3vinagre-2017-01-02_17:16:57.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/3vinagre-2017-01-02_17:17:17.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/3vinagre-2017-01-02_17:17:38.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/3vinagre-2017-01-02_17:22:04.png)


## SSH Localhost Forwarding
設置SSH本地端口轉發，參考RedHat官方文檔[10.4. MORE THAN A SECURE SHELL](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-ssh-beyondshell.html 'RedHat')中相關章節，命令格式如下

```bash
ssh -L local-port:remote-hostname:remote-port username@hostname
```

此處將其改寫為

```bash
#本地端口設置為`9876`
ssh -C -c aes256-cbc -f -N -g -L 9876:localhost:5901 tigervnc@192.241.240.132
```

要求輸入用戶`tigervnc`的密碼。

將目標主機的`5901`端口通過SSH Tunnel轉發到本地的`9876`端口。

執行

```bash
vncviewer localhost:9876
```

正常打開遠程主機桌面。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/4sshtunnel-2017-01-02_17:34:44.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/4sshtunnel-2017-01-02_17:36:48.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/4sshtunnel-2017-01-02_17:37:20.png)

此時執行

```bash
vncviewer -Shared -FullColour 192.241.240.132:1
```
仍能打開目標主機桌面。

### Only Allow From LocalHost
出於安全考慮，設置VNC Server只允許VPS本地`localhost`訪問。創建SSH Tunnel後，就能直接訪問

修改文件
```bash
#/etc/systemd/system/vncserver@:1.service
#更改
ExecStart=/usr/sbin/runuser -l tigervnc -c "/usr/bin/vncserver %i"
#為
ExecStart=/usr/sbin/runuser -l tigervnc -c "/usr/bin/vncserver -localhost %i"

#重新載入服務
sudo systemctl daemon-reload
sudo systemctl restart vncserver@:1
```

修改後，直接連接報錯

```bash
flying@maxdsre:~$ vncviewer -Shared -FullColour 192.241.240.132:1

VNC Viewer Free Edition 4.1.1 for X - built Apr  2 2015 21:51:06
Copyright (C) 2002-2005 RealVNC Ltd.
See http://www.realvnc.com for information on VNC.

Mon Jan  2 17:49:20 2017
 main:        unable to connect to host: Connection refused (111)
flying@maxdsre:~$
```

通過SSH Tunnel可正常連接

```bash
vncviewer localhost:9876
```

操作截圖

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/5sshtunnel-localhost-2017-01-02_17:52:05.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/5sshtunnel-localhost-2017-01-02_17:52:26.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-02-vncserver_via_ssh/5sshtunnel-localhost-2017-01-02_18:01:32.png)


## References
* [Install & Configure VNC Server in CentOS 7 and RHEL 7](http://www.linuxtechi.com/install-configure-vnc-server-centos-7-rhel-7/)
* [How To Install and Configure VNC Remote Access for the GNOME Desktop on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-remote-access-for-the-gnome-desktop-on-centos-7 'DigitalOcean')


## Change Logs
* 2017.01.02 17:54 Mon Asia/Shanghai
    * 初稿完成
* 2018-08-01 22:46 Wed Asia/Shanghai
    * 勘誤，排版，遷移到新Blog


<!-- End -->
