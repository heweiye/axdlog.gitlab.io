---
title: 在GNU/Linux中安裝Docker CE（社區版）全記錄
slug: Installing And Configuring Docker Community Edition CE On GNU Linux
date: 2017-03-06T13:24:08+08:00
lastmod: 2018-04-11T11:41:08-04:00
draft: false
keywords: ["Docker", "Docker CE", "Container"]
description: "本文記錄如何通過Shell腳本在GNU/Linux安裝Docker CE（社區版）。"
categories:
- Container
tags:
- Docker
comment: true
toc: true
---

[Docker][docker]是一款開源的，提供`Operating-system-level virtualization`(操作系統級別的虛擬化的)容器(container)產品，可實現在軟件容器中自動部署應用。目前有`Enterprise Edition(EE)`、`Community Edition(CE)`、`Cloud`三個變種。本文記錄在GNU/Linux中安裝、配置Docker Engine的過程，並通過Shell腳本實現。

<!--more-->

## 官方站點
以下是[Docker][docker]相關官方站點

| Site | Website
| :--- | :---
| Official Site | https://www.docker.com
| GitHub | https://github.com/docker
| Documentation | https://docs.docker.com
| Blog | https://blog.docker.com
| Twitter | https://twitter.com/docker
| Youtube | https://www.youtube.com/user/dockerrun
| Docker Hub | https://hub.docker.com
| Docker Store | https://store.docker.com

## 教程
[Docker][docker]在[Github](https://github.com)中提供了一些教程，初學者可通過此教程學習Docker的使用

* [Docker Tutorials and Labs](https://github.com/docker/labs/)
* [Docker for beginners](https://github.com/docker/labs/tree/master/beginner)

官網提供的基礎教程

* [Learn the basics of Docker](https://docs.docker.com/engine/getstarted/)
* [Docker Engine user guide](https://docs.docker.com/engine/userguide/)

若要詳細瞭解，建議閱讀Docker[官方文檔](https://docs.docker.com/)。

## 簡介
[Docker][docker]是一款操作系統級別的虛擬化產品，[Docker][docker]啓動封裝好的鏡像，運行的實例即爲容器。鏡像以精簡後的操作系統(GNU/Linux)爲基礎，安裝有運行目標應用所需的各種應用、庫。運行的容器，對於操作系統而言，像是一個個進程，但對目標應用而言，應用運行在一個“操作系統”之中。基於同一個鏡像運行的容器，容器內提供的運行環境是相同的。

[Docker][docker]利用Linux Kernel中的[control groups](https://en.wikipedia.org/wiki/Cgroups 'WikiPedia')、[namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)和[advanced multi layered unification filesystem](https://en.wikipedia.org/wiki/Aufs)等特性，實現在單個Linux實例上運行多個獨立的`容器`(container)，減少了啓動和維護[virtual machine](https://en.wikipedia.org/wiki/Virtual_machine)的開銷。 --- [Docker (software)](https://en.wikipedia.org/wiki/Docker_%28software%29 'WikiPedia')

>Docker allows you to package an application with all of its dependencies into a standardized unit for software development.

**此處插一句** [Debian](http://www.debian.org/)的聯合創建者同時也是[Docker][docker]的開發者之一 [Ian Murdock](https://en.wikipedia.org/wiki/Ian_Murdock)，於2015年12月28日去世。此爲`Docker`官方博客發佈的文章[In Memoriam: Ian Murdock](https://blog.docker.com/2015/12/ian-murdock/)，在此緬懷一下逝者。

### 架構

>Docker uses a client-server architecture. The Docker client talks to the Docker *daemon*, which does the heavy lifting of building, running, and distributing your Docker containers. Both the Docker client and the daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate via sockets or through a RESTful API.  --- [Understand the architecture](https://docs.docker.com/engine/understanding-docker/#what-is-dockers-architecture)

![architecture](https://docs.docker.com/engine/images/architecture.svg)

關於`Docker`的具體架構介紹，請移步 [Docker overview
](https://docs.docker.com/engine/docker-overview/)


### 對比虛擬機
Docker Container本質上也是`虛擬化`，但與`virtual machine`(如[Vagrant](https://en.wikipedia.org/wiki/Vagrant_%28software%29))的實現方式不同。

>Each virtual machine includes the application, the necessary binaries and libraries and an entire guest operating system - all of which may be tens of GBs in size.

每個虛擬機包含了應用程序、需要的二進制和庫文件，以及整個`guest`操作系統，整個需要數十GB的空間。

>Containers include the application and all of its dependencies, but share the kernel with other containers. They run as an isolated process in userspace on the host operating system. They’re also not tied to any specific infrastructure – Docker containers run on any computer, on any infrastructure and in any cloud.

容器包含應用程序及其所有的依賴文件，但與其它容器共享內核(kernel)，它們在宿主機(host)操作系統的用戶空間(userspace)中以獨立的進程(process)運行。它們不與任何具體的基礎架構(infrastructure)關聯。Docker容器運行在任意計算機、任意基礎架構和任意一種雲環境中。

下圖介紹了二者的區別，圖片來自[What is a Container?](https://www.docker.com/what-container)。

#### 虛擬機
舊圖  
![Virtual Machines](https://www.docker.com/sites/default/files/what-is-docker-diagram.png)

新圖  
![Virtual Machines](https://www.docker.com/sites/default/files/VM%402x.png)

#### 容器
舊圖  
![Containers](https://www.docker.com/sites/default/files/what-is-vm-diagram.png)

新圖  
![Containers](https://www.docker.com/sites/default/files/Container%402x.png)

簡言之，`Virtual Machine`虛擬的是一個完整的操作系統，而`Container`是在Linux kernel的基礎上實現獨立的運行環境。

**Containers and Virtual Machines Together**

![](https://www.docker.com/sites/default/files/containers-vms-together.png)


### 系統要求
[Docker][docker]對系統有一定要求：

1. 操作系統必須是 **64-bit**；
2. Linux Kernel版本至少是 **3.10**；
3. `iptables`的版本至少是 **1.4**；

具體見 [Install Docker CE from binaries](https://docs.docker.com/install/linux/docker-ce/binaries/)。

可通過如下命令查看系統是否爲64-bit，即`x86_64`。

```bash
uname -m
```

## Docker 產品線
[Docker][docker]目前分爲3條產品線，具體介紹見 [Install Docker](https://docs.docker.com/install/)。

1. [Docker Enterprise Edition (Docker EE)](https://www.docker.com/enterprise-edition)
2. [Docker Community Edition (Docker CE)](https://www.docker.com/community-edition)
    * Stable (release per quarter)
    * Edge (release per month)
3. [Docker Cloud](https://docs.docker.com/install/#cloud)

`Docker CE`和`Docker EE`支持的GNU/Linux發行版不盡相同，具體如下

Platform|Docker EE|Docker CE
---|:---:|:---:
RHEL|Y|
CentOS|Y|Y
Fedora||Y
Oracle Linux|Y|
Debian||Y
Ubuntu|Y|Y
SLES|Y|


>See also Docker Cloud for setup instructions for `Digital Ocean`, `Packet`, `SoftLink`, or Bring Your Own Cloud.


## 安裝
[Docker][docker]目前分`Docker EE`、`Docker CE`。

對GNU/Linux而言，`RHEL`、`Oracle Linux`、`SLES`只能安裝`Docker EE`。而安裝`Docker EE`須先註冊[Docker Store](https://store.docker.com/)。能使用`Docker CE`的只有`CentOS/Fedora`、`Debian/Ubuntu`。

本文只針對`Docker CE`，涵蓋的發行版只有CentOS/Debian/Ubuntu。

[Docker][docker]官方文檔提供了安裝說明 [Install Docker Engine](https://docs.docker.com/engine/installation/)，具體如下：

* [Get Docker for CentOS](https://docs.docker.com/install/linux/centos/)
* [Get Docker for Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
* [Get Docker for Ubuntu](https://docs.docker.com/install/linux/ubuntu/)

根據以上文檔整理出`Docker CE`支持的具體版本

Distro|Version
---|---
CentOS|7
Debian|Buster 10 (Docker CE 17.11 Edge only)
Debian|Stretch 9
Debian|Jessie 8
Debian|Wheezy 7
Ubuntu|Artful 17.10 (Docker CE 17.11 Edge and higher only)
Ubuntu|Zesty 17.04
Ubuntu|Xenial 16.04
Ubuntu|Trusty 14.04


**注意**：Linux Kernel版本必須至少爲`3.10`。


### CentOS

#### 安裝 Docker
```bash
# Remove unofficial Docker packages
yum -q makecache fast
yum -y -q remove docker docker-{client,client-latest,latest,latest-logrotate,logrotate,common,selinux,engine,engine-selinux}

# Install Docker
curl -fsSL https://download.docker.com/linux/centos/docker-ce.repo | sed -n '/ce-stable-debuginf/,$d;/^$/d;p' > /etc/yum.repos.d/docker-ce.repo

# [docker-ce-stable]
# name=Docker CE Stable - $basearch
# baseurl=https://download-stage.docker.com/linux/centos/7/$basearch/stable
# enabled=1
# gpgcheck=1
# gpgkey=https://download-stage.docker.com/linux/centos/gpg

yum -q makecache fast
yum -y -q install docker-ce
```

#### 卸載 Docker

```bash
yum -y -q remove docker-ce
rm -rf /var/lib/docker
```

執行`yum install docker-ce`提示的GPG信息

```bash
# yum -y -q install docker-ce
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch.rpm is not installed
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
setsebool:  SELinux is disabled.
libsemanage.semanage_direct_install_info: Overriding docker module at lower priority 100 with module at priority 400.
```

### Debian/Ubuntu

#### 安裝 Docker
Ubuntu與Debian的安裝過程大致相同，區別之處在於
1. 具體版本依賴的軟件不同
2. Repo配置不同(發行版名稱、codename)

整合後，操作命令如下

```bash
# remove old package
sudo apt-get -y remove docker docker-engine

versionId=$(awk '/VERSION_ID=/{print gensub(/.*"(.*)"$/,"\\1","g",$0)}' /etc/os-release)
codeName=$(awk '/VERSION=/{print gensub(/.*\((.*)\)"$/,"\\1","g",$0)}' /etc/os-release)
distroName=$(awk '/PRETTY_NAME=/{distro=gensub(/.*"([^ ]*) .*/,"\\1","g",$0);print tolower(distro)}' /etc/os-release)

case "${distroNam}" in
    debian )
        [[ "${versionId}" -gt 7 ]] && externalPack='software-properties-common gnupg2' || externalPack='python-software-properties'  # Wheezy 7
        ;;
    ubuntu )
        externalPack='software-properties-common'
        [[ "${versionId}" == '14.04' ]] && externalPack=${externalPack}" linux-image-extra-$(uname -r) linux-image-extra-virtual"   # Trusty 14.04
        ;;
esac

sudo apt-get -qy --no-install-recommends install apt-transport-https ca-certificates curl "${externalPack}"

# add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/${distroNam}/gpg | sudo apt-key add -
# sudo apt-key fingerprint 0EBFCD88

# add stable official repository
sudo bash -c "echo \"deb [arch=amd64] https://download.docker.com/linux/${distroNam} $codeName stable\" > /etc/apt/sources.list.d/docker.list"

sudo apt-get update 1> /dev/null
sudo apt-get -qy install docker-ce

unset externalPack
unset versionId
unset distroName
unset codeName
```

#### 卸載 Docker

```bash
sudo apt-get -qy purge docker-ce
sudo rm -rf /var/lib/docker
```


## 啓動 Docker 服務
安裝完成後通過如下命令啓動Docker服務

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl enable docker

# testing
sudo docker run hello-world
```

## 安裝後配置
[Docker][docker]安裝後的配置說明：

* [Post-installation steps for Linux](https://docs.docker.com/install/linux/linux-postinstall/)

### 創建非root用戶
在[Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)中有如下文字

>The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user `root` and other users can only access it using `sudo`. The docker daemon always runs as the `root` user.
>
If you don't want to use `sudo` when you use the docker command, create a Unix group called `docker` and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the `docker` group.
>
**Warning**: The `docker` group grants privileges equivalent to the `root` user. For details on how this impacts security in your system, see [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface).

翻譯如下：

Docker守護進程(daemon)綁定Unix套接字(socket)而非TCP端口(port)。默認情況下，Unix套接字的擁有者是用戶`root`和使用`sudo`權限訪問的其他用戶。Docker守護進程以`root`用戶的身份運行。

如果不想在執行docker命令的時候使用`sudo`，創建名爲`docker`的用戶組(group)，將普通用戶添加到該組中。當Docker守護進程啓動時，docker自動將Unix套接字的讀、寫權限賦予名爲`docker`的用戶組。

**提醒**： `docker`用戶組的權限與用戶`root`相同，該操作會如何影響系統安全，詳見[Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)。

>Docker Daemon Attack Surface
>
* only trusted users should be allowed to control your Docker daemon
* if you run Docker on a server, it is recommended to run exclusively Docker on the server, and move all other services within containers controlled by Docker.

簡言之：默認情況下，執行docker命令時須添加`sudo`，爲避免此情況。創建名爲`docker`的用戶組，將有次需求的用戶添加到該用戶組即可。


```bash
sudo groupadd docker
sudo usermod -aG docker $USER
# sudo gpasswd -a $USER docker
```

### 穿過防火牆訪問遠程API
[Allow access to the remote API through a firewall](https://docs.docker.com/install/linux/linux-postinstall/#allow-access-to-the-remote-api-through-a-firewall)中有如下文字

>If you run a firewall on the same host as you run Docker and you want to access the Docker Remote API from another host and remote access is enabled, you need to configure your firewall to allow incoming connections on the Docker port, which defaults to `2376` if TLS encrypted transport is enabled or `2375` otherwise.

簡言之：如果運行Docker的主機安裝有防火牆，想要從其他主機遠程訪問該主機的Docker API。須在本機中開放`2376`端口，如果通過TLS加密傳輸，則需開放`2375`端口。

## Shell 腳本
腳本託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/Docker-CE.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Docker-CE.sh | bash -s --
```


## 報錯
### image has dependent child images
在使用`docker rmi`移除tag爲`<none>`的鏡像時，出現報錯

>Error response from daemon: conflict: unable to delete 978d85d02b87 (cannot be forced) - image has dependent child images

參考[docker how can I get the list of dependent child images?](https://stackoverflow.com/questions/36584122/docker-how-can-i-get-the-list-of-dependent-child-images#answer-41195790)，在本機目錄

```bash
/var/lib/docker/image/btrfs/imagedb/content/sha256/

# docker 17.06.2-ce
/var/lib/docker/image/overlay2/imagedb/content/sha256
```

中刪除以`978d85d02b87`開頭的文件解決。

操作過程如下

```bash
# 獲取image id
maxsre@jessie:~$ docker images | awk 'match($2,/none/){print $3}'
978d85d02b87

# 移除鏡像報錯
maxsre@jessie:~$ docker rmi 978d85d02b87
Error response from daemon: conflict: unable to delete 978d85d02b87 (cannot be forced) - image has dependent child images

# 使用sudo
maxsre@jessie:~$ sudo -i

# 進入目標目錄
root@jessie:~# cd /var/lib/docker/image/btrfs/imagedb/content/sha256/

# 查找文件
root@jessie:/var/lib/docker/image/btrfs/imagedb/content/sha256# ls 978d85d02b87*
978d85d02b87aea199e4ae8664f6abf32fdea331884818e46b8a01106b114cee

# 移除文件
root@jessie:/var/lib/docker/image/btrfs/imagedb/content/sha256# rm -f 978d85d02b87aea199e4ae8664f6abf32fdea331884818e46b8a01106b114cee

# 勘驗
root@jessie:~# docker images | awk 'match($2,/none/){print $3}'
root@jessie:~#
```


## 更新日誌
* 2016.04.04 11:30 Thu Asia/Beijing
    * 初稿完成
* 2017.03.02 15:44 Thu Asia/Shanghai
    * 文檔重構
* 2017.03.03 16:49 Fri Asia/Shanghai
    * Docker官方文檔更新(分`Docker EE`、`Docker CE`、`Docker Cloud`)，文檔重構
* 2017.04.07 09:50 Fri Asia/Shanghai
    * 添加`Error Occuring`->`image has dependent child images`
* 2017.09.08 08:31 Fri Asia/Shanghai
    * 添加`/var/lib/docker/image/overlay2/imagedb/content/sha256`
* 2018.04.11 11:41 Wed America/Boston
    * 更新文檔鏈接，勘誤，遷移到新Blog

[docker]:https://www.docker.com "Docker"

<!-- End -->
