---
title: Installing And Configuring Docker Community Edition(CE) On GNU/Linux
slug: Installing And Configuring Docker Community Edition CE On GNU Linux
date: 2017-03-06T13:24:08+08:00
lastmod: 2018-04-11T11:41:08-04:00
draft: false
keywords: ["AxdLog", "Docker", "Docker CE", "Container"]
description: "How to use shell script to install and configure Docker Community Edition(CE) on GNU/Linux"
categories:
- Container
tags:
- Docker
comment: true
toc: true
---

[Docker][docker] is a open source container program that performs operating-system-level virtualization. [Docker][docker] currently provides three products `Enterprise Edition(EE)`、`Community Edition(CE)` and `Cloud`. This article documents how to install and configure Docker CE in GNU/Linux, then implement the entire process through Shell script.

<!--more-->

## Official Site
Relevant official site of [Docker][docker]

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

## Tutorials
[Docker][docker] provides some tutorials in its [Github](https://github.com) repository. If you're a newbie, it may be useful for you to learn [Docker][docker].

* [Docker Tutorials and Labs](https://github.com/docker/labs/)
* [Docker for beginners](https://github.com/docker/labs/tree/master/beginner)

Official tutorials of [Docker][docker]

* [Learn the basics of Docker](https://docs.docker.com/engine/getstarted/)
* [Docker Engine user guide](https://docs.docker.com/engine/userguide/)

If you wanna learn more, please read its [official document site](https://docs.docker.com).

## Introduction
[What is Docker?](https://www.docker.com/what-docker)

### Architecture

>Docker uses a client-server architecture. The Docker client talks to the Docker *daemon*, which does the heavy lifting of building, running, and distributing your Docker containers. Both the Docker client and the daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate via sockets or through a RESTful API.  --- [Understand the architecture](https://docs.docker.com/engine/understanding-docker/#what-is-dockers-architecture)

![architecture](https://docs.docker.com/engine/images/architecture.svg)

More info about Docker, please read [Docker overview
](https://docs.docker.com/engine/docker-overview/)


### VS Virtual Machine
Docker Container is a methos of virtualization, but it is different from `virtual machine`, such as [Vagrant](https://en.wikipedia.org/wiki/Vagrant_%28software%29).

virtual machine

>Each virtual machine includes the application, the necessary binaries and libraries and an entire guest operating system - all of which may be tens of GBs in size.

container

>Containers include the application and all of its dependencies, but share the kernel with other containers. They run as an isolated process in userspace on the host operating system. They’re also not tied to any specific infrastructure – Docker containers run on any computer, on any infrastructure and in any cloud.

The following pictures are from [What is a Container?](https://www.docker.com/what-container).

#### Virtual Machines
Old picture

![Virtual Machines](https://www.docker.com/sites/default/files/what-is-docker-diagram.png)

New picture  
![Virtual Machines](https://www.docker.com/sites/default/files/VM%402x.png)

#### Containers
Old picture

![Containers](https://www.docker.com/sites/default/files/what-is-vm-diagram.png)

New picture

![Containers](https://www.docker.com/sites/default/files/Container%402x.png)

**Containers and Virtual Machines Together**

![](https://www.docker.com/sites/default/files/containers-vms-together.png)


### OS requirements
1. OS must be **64-bit**；
2. Linux kernel version at least **3.10**；
3. `iptables` version at least **1.4**；

More details in [Install Docker CE from binaries](https://docs.docker.com/install/linux/docker-ce/binaries/)。

The following can be used to check if the running system is 64-bit (`x86_64`).

## Docker Product
[Docker][docker] currently has 3 products, more details in [Install Docker](https://docs.docker.com/install/)。

1. [Docker Enterprise Edition (Docker EE)](https://www.docker.com/enterprise-edition)
2. [Docker Community Edition (Docker CE)](https://www.docker.com/community-edition)
    * Stable (release per quarter)
    * Edge (release per month)
3. [Docker Cloud](https://docs.docker.com/install/#cloud)

`Docker CE` and `Docker EE` supper different distributions.

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


## Installation
For `RHEL`、`Oracle Linux`、`SLES`, it can only install `Docker EE` which needs to register [Docker Store](https://store.docker.com/) firstly.

This document is focus on `Docker CE` which is just support `CentOS/Fedora`、`Debian/Ubuntu`.

[Docker][docker] provides official installation document [Install Docker Engine](https://docs.docker.com/engine/installation/):

* [Get Docker for CentOS](https://docs.docker.com/install/linux/centos/)
* [Get Docker for Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
* [Get Docker for Ubuntu](https://docs.docker.com/install/linux/ubuntu/)

Complete distro release version which are supported by `Docker CE`

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

### CentOS

#### Install Docker
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

#### Uninstall Docker

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

#### Install Docker
The difference between Ubuntu and Debian

1. different package dependencies
2. different distribution name, codename


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

#### Uninstall Docker

```bash
sudo apt-get -qy purge docker-ce
sudo rm -rf /var/lib/docker
```


## Start Docker Daemon
Start docker service

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl enable docker

# testing
sudo docker run hello-world
```

## Post-installation Configuration
[Post-installation steps for Linux](https://docs.docker.com/install/linux/linux-postinstall/)

### Manage Docker As A Non-root User
[Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)

>The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user `root` and other users can only access it using `sudo`. The docker daemon always runs as the `root` user.
>
If you don't want to use `sudo` when you use the docker command, create a Unix group called `docker` and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the `docker` group.
>
**Warning**: The `docker` group grants privileges equivalent to the `root` user. For details on how this impacts security in your system, see [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface).


Docker Daemon Attack Surface
>
* only trusted users should be allowed to control your Docker daemon
* if you run Docker on a server, it is recommended to run exclusively Docker on the server, and move all other services within containers controlled by Docker.


```bash
sudo groupadd docker
sudo usermod -aG docker $USER
# sudo gpasswd -a $USER docker
```

### Access Remote API Through A Firewall
[Allow access to the remote API through a firewall](https://docs.docker.com/install/linux/linux-postinstall/#allow-access-to-the-remote-api-through-a-firewall)

>If you run a firewall on the same host as you run Docker and you want to access the Docker Remote API from another host and remote access is enabled, you need to configure your firewall to allow incoming connections on the Docker port, which defaults to `2376` if TLS encrypted transport is enabled or `2375` otherwise.


## Shell Script
Shell script is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/Docker-CE.sh), usage info

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Docker-CE.sh | bash -s --
```


## Error Occuring
### image has dependent child images
Fail to use command `docker rmi` to remove image whose tag name is `<none>`

>Error response from daemon: conflict: unable to delete 978d85d02b87 (cannot be forced) - image has dependent child images

[docker how can I get the list of dependent child images?](https://stackoverflow.com/questions/36584122/docker-how-can-i-get-the-list-of-dependent-child-images#answer-41195790)

```bash
/var/lib/docker/image/btrfs/imagedb/content/sha256/

# docker 17.06.2-ce
/var/lib/docker/image/overlay2/imagedb/content/sha256
```

Solving it via deleting file begins with `978d85d02b87`

```bash
# get image id
maxsre@jessie:~$ docker images | awk 'match($2,/none/){print $3}'
978d85d02b87

# remove image prompt error
maxsre@jessie:~$ docker rmi 978d85d02b87
Error response from daemon: conflict: unable to delete 978d85d02b87 (cannot be forced) - image has dependent child images

# use sudo
maxsre@jessie:~$ sudo -i

# enter target directory
root@jessie:~# cd /var/lib/docker/image/btrfs/imagedb/content/sha256/

# find file
root@jessie:/var/lib/docker/image/btrfs/imagedb/content/sha256# ls 978d85d02b87*
978d85d02b87aea199e4ae8664f6abf32fdea331884818e46b8a01106b114cee

# remove
root@jessie:/var/lib/docker/image/btrfs/imagedb/content/sha256# rm -f 978d85d02b87aea199e4ae8664f6abf32fdea331884818e46b8a01106b114cee

# verification
root@jessie:~# docker images | awk 'match($2,/none/){print $3}'
root@jessie:~#
```


## Change Logs
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
