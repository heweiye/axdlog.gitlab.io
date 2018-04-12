---
title: Building Docker Private Registry With Self-signed Certificate
date: 2016-06-06T22:06:13+08:00
lastmod: 2018-04-12T11:00:02-04:00
draft: false
keywords: ["Docker", "Private Registry", "Self-signed Certificate", "OpenSSL"]
description: "Building docker private registry with self-signed certicficate on GNU/Linux"
categories:
- Container
- Product Case
tags:
- docker
- openssl
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
contentCopyright: ""
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
---

[Docker][docker]是一款開源的，提供`Operating-system-level virtualization`(操作系統級別的虛擬化的)容器(container)產品，可實現在軟件容器中自動部署應用。本文主要記錄如何[Docker][docker]的[Registry][registry]創建私有倉庫，並通過自簽證書實現通過域名訪問私有倉庫。

<!--more-->

## Introduction
搭建Docker私有倉庫需要使用[docker/docker-registry](https://github.com/docker/docker-registry)，但該項目已經被Docker官方廢棄。

當前的 **Docker Registry 2.0** 是[docker/distribution](https://github.com/docker/distribution)。文檔說明見<https://github.com/docker/distribution/blob/master/README.md>

說明文檔也從 <https://github.com/docker/distribution/tree/master/docs> 遷移至 <https://github.com/docker/docker.github.io/tree/master/registry>。


## Preparations
### OS
根據官方文檔 [Installation on CentOS](https://docs.docker.com/engine/installation/linux/centos/)，Docker需要`64-bit`系統環境，且內核版本不低於`3.10`。

以下是本人測試環境信息：

Item|Info
---|---
OS|`CentOS Linux release 7.2.1511 (Core)`
Kernel|`3.10.0-327.13.1.el7.x86_64`

### Docker Engine
`Docker`的介紹、安裝和配置，參考最新的Blog [Installing And Configuring Docker Community Edition(CE) On GNU/Linux](https://axdlog.com/2017/installing-and-configuring-docker-community-editionce-on-gnu/linux/)。

此處假定已經安裝`docker-engine`，並且啟動該服務。

### Docker Distribution
Docker Hub的[registry][registry]中有如下說明:

>The tags >= 2 refer to the [new registry](https://github.com/docker/distribution).
>
Older tags refer to the [deprecated registry](https://github.com/docker/docker-registry).

即要想使用新的`registry`，需指定`tag`，且`>=2`，如`registry:2.4`。

下載registry鏡像時如果不加tag，則默認為老版本

```bash
docker pull registry:2
```

**注意**：截至`Apr 12, 2018`，registry的可用版本號是`2.5.2`， `2.6.2`，主版本號皆`>=2`。直接執行

```bash
docker pull registry
```

**注意**： 運行`registry`須啟動`iptables`服務，否則會報錯。

```bash
sudo yum install -y iptables-services
sudo systemctl start  iptables
sudo systemctl enable iptables
```

### Private Registry Storage Path
創建私有鏡像存儲路徑，此處以`/data/docker-registry`為例

* 創建鏡像存儲路徑
```bash
sudo mkdir -pv /data/docker-registry
```

* 將目錄屬組改為Docker
```bash
sudo chown :docker /data/docker-registry/
```

### Generate Self-signed Certificate
參考官方文檔 [Using self-signed certificates](https://github.com/docker/docker.github.io/blob/master/registry/insecure.md)進行相關操作。

此處建議以root用戶身份運行相關命令

使用`openssl`命令套件創建自簽署證書，並保存在目錄`/etc/docker/certs`目錄下，以域名`docker.axdlog.com`為例

* 創建目錄
```bash
[ ! -d /etc/docker/certs ] && mkdir -pv /etc/docker/certs
```

* 進入目錄
```bash
cd /etc/docker/certs
```

* 生成自簽證書
```bash
openssl req -x509 -newkey rsa:4096 -days 365 -nodes -sha256 -keyout registry.key -out registry.crt
```

以下是相關參數
```
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:Shanghai
Locality Name (eg, city) [Default City]:Shanghai
Organization Name (eg, company) [Default Company Ltd]:AxdLog
Organizational Unit Name (eg, section) []:AxdLog
Common Name (eg, your name or your server's hostname) []:docker.axdlog.com
Email Address []:test@axdlog.com
```

因本機IP為`192.168.0.109`，故而在`/etc/hosts`中添加
```
192.168.0.109 docker.axdlog.com
```
其它需要連接私有倉庫的主機也可添加該行記錄，用作域名解析。

### Trust The Certificate
```bash
cd /etc/docker/certs/

mkdir docker.axdlog.com:5000

cp ./registry.crt docker.axdlog.com\:5000/ca.crt

cp /etc/docker/certs/registry.crt /etc/pki/ca-trust/source/anchors/docker.axdlog.com.crt

update-ca-trust
```

操作完成後，重啟Docker服務
```bash
sudo systemctl restart docker
```

### Run Registry Container
啟動容器，使用`/var/lib/registry`而非`/tmp/registry`

```bash
docker run -d -p 5000:5000 --restart=always --name registry \
  -v /etc/docker/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key \
  -v /data/docker-registry:/var/lib/registry \
  registry:2
```

* 使用參數`-v`將本地磁盤中的`/data/docker-registry`目錄映射到容器(container)中的`/var/lib/registry`目錄
* 使用參數`-v`將本地磁盤中的`/etc/docker/certs`目錄映射到容器中的`/certs`目錄

可用過`docker ps`命令查看容器運行情況

## Testing
以鏡像`hello-world`為例

* 獲取遠程鏡像
```bash
docker pull hello-world
```

* 添加標籤
```bash
docker tag hello-world docker.axdlog.com:5000/hello-world
```

* 將新創建的鏡像push到私有倉庫
```bash
docker push docker.axdlog.com:5000/hello-world
```

以下是操作過程
```bash
[maxdsre@lemp ~]$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
4276590986f6: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:a7d7a8c072a36adb60f5dc932dd5caba8831ab53cbf016bcdd6772b3fbe8c362
Status: Downloaded newer image for hello-world:latest

[maxdsre@lemp ~]$ docker tag hello-world docker.axdlog.com:5000/hello-world

[maxdsre@lemp ~]$ docker run -d -p 5000:5000 --restart=always --name registry \
>   -v /etc/docker/certs:/certs \
>   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt \
>   -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key \
>   -v /data/docker-registry:/var/lib/registry \
>   registry:2
f3ea6cd5ef2a266538c7936c0dbaeb06d1b639a8bfd15f2296f6992ba69a3d4c

[maxdsre@lemp ~]$ docker push docker.axdlog.com:5000/hello-world
The push refers to a repository [docker.axdlog.com:5000/hello-world]
5f70bf18a086: Pushed
33e7801ac047: Pushed
latest: digest: sha256:17372d4bad2fe1355ba3dd9fe5178e2c7ea72c91529c1691e3ab5a75371135b4 size: 708

[maxdsre@lemp ~]$ ls /data/docker-registry/docker/registry/v2/repositories
hello-world

[maxdsre@lemp ~]$ docker rmi docker.axdlog.com:5000/hello-world
Untagged: docker.axdlog.com:5000/hello-world:latest

[maxdsre@lemp ~]$ docker images | grep docker.axdlog.com:5000/hello-world

[maxdsre@lemp ~]$ docker pull docker.axdlog.com:5000/hello-world
Using default tag: latest
latest: Pulling from hello-world
Digest: sha256:17372d4bad2fe1355ba3dd9fe5178e2c7ea72c91529c1691e3ab5a75371135b4
Status: Downloaded newer image for docker.axdlog.com:5000/hello-world:latest

[maxdsre@lemp ~]$ docker images | grep docker.axdlog.com:5000/hello-world
docker.axdlog.com:5000/hello-world   latest              94df4f0ce8a4        5 weeks ago         967 B
[maxdsre@lemp ~]$
```

### Other Hosts
其餘主機如果想使用該私有倉庫須進行如下操作：

進行如下操作
```bash
echo '192.168.0.109 docker.axdlog.com' >> /etc/hosts

touch /etc/pki/ca-trust/source/anchors/docker.axdlog.com.crt

# 將registry主機的/etc/docker/certs/registry.crt中內容複製到該文件中或使用scp

update-ca-trust
```

其它主機測試過程
```bash
[maxdsre@raxtone ~]$ docker pull docker.axdlog.com:5000/hello-world
Using default tag: latest
latest: Pulling from hello-world
4276590986f6: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:17372d4bad2fe1355ba3dd9fe5178e2c7ea72c91529c1691e3ab5a75371135b4
Status: Downloaded newer image for docker.axdlog.com:5000/hello-world:latest
[maxdsre@raxtone ~]$ docker run docker.axdlog.com:5000/hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

[maxdsre@raxtone ~]$
```

## Errors
遇到的報錯

### Error 1

>docker: Error response from daemon: failed to create endpoint registry on network bridge: iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5000 -j DNAT --to-destination 172.17.0.2:5000 ! -i docker0: iptables: No chain/target/match by that name. (exit status 1).

```bash
[maxdsre@lemp ~]$ docker run -d -p 5000:5000 --restart=always --name registry \
>   -v /etc/docker/certs:/certs \
>   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt \
>   -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key \
>   -v /data/docker-registry:/var/lib/registry \
>   registry:2
0a4712d7a1205f72bd27d0fa83131067b1b69539d057ac53fe0268dbe1847e72
docker: Error response from daemon: failed to create endpoint registry on network bridge: iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5000 -j DNAT --to-destination 172.17.0.2:5000 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1).
[maxdsre@lemp ~]$
```

解決方案：安裝並啟用`iptables`

### Error 2
>x509: certificate signed by unknown authority

```
[root@ptdb/data/docker-registry]
#docker push docker.axdlog.com:5000/hello-world
The push refers to a repository [docker.axdlog.com:5000/hello-world]
Get https://docker.axdlog.com:5000/v1/_ping: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "docker.axdlog.com")
```

## References
* [Deploying a registry server](https://github.com/docker/distribution/blob/master/docs/deploying.md)
* [Insecure Registry](https://github.com/docker/distribution/blob/master/docs/insecure.md)
* [Docker Instroduction and Installation on CentOS 7](https://axdlog.com/tw/Docker-Instroduction-and-Installation-on-CentOS-7/)


---
## Change Log
* 2016.06.06 22:07 Mon Asia/Shanghai
    * 初稿完成
* 2018-04-12 11:00 Thu America/Boston
    * 勘誤，遷移到新Blog

[docker]:https://www.docker.com "Docker"
[registry]: https://hub.docker.com/_/registry/

<!-- End -->
