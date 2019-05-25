---
title: Using Docker Container To Auto Deploy Blog To GitHub Pages
slug: Using Docker Container To Auto Deploy Blog To GitHub Pages
date: 2016-06-16T09:34:39+08:00
lastmod: 2019-04-28T14:39:02-04:00
draft: false
keywords: ["Hexo", "Github", "Node.js", "Git"]
description: "Using Docker Container To Auto Deploy Blog To GitHub Pages."

categories:
- Blog
- Container
tags:
- Hexo
- GitHub
- Docker

comment: true
toc: true

---

本文主要記錄如何使用Docker創建安裝有[Hexo][hexo]的鏡像，並通啓動過容器實現Blog的正常發佈。[Hexo][hexo]研究過程見本人blog 『[Using Hexo To Deploy Blog To GitHub Pages On GNU/Linux]({{< relref "2016-01-21-Using-Hexo-To-Deploy-Blog-To-GitHub-Pages-On-GNU-Linux.md" >}})』。


<!--more-->

## Prerequisites

本人通過鏡像[debian](https://hub.docker.com/_/debian/ 'Docker')構建定製化鏡像。

1. Linux操作系統(CentOS7,Debian Jessie);
2. 安裝Docker，~~安裝過程參考 [Docker Instroduction and Installation on CentOS 7](https://lempstacker.com/tw/Docker-Instroduction-and-Installation-on-CentOS-7/)~~；
3. 下載鏡像[debian](https://hub.docker.com/_/debian/ 'Docker')作爲基礎鏡像；

目的：基於Docker構建[Hexo](https://www.haomwei.com/technology/maupassant-hexo.html)環境，`source`，`themes`等目錄通過`volumn`的形式掛載。


## Dockerfile

**注意**： 因`npm`出現`WARN`提示導致此Dockerfile未能成功構建，僅作思路參考

本定製化鏡像基於`debian`系統構建，該系統使用的是`apt-get`命令，也可基於[centos](https://hub.docker.com/_/centos/)鏡像構建，根據個人喜好而定。

需要額外安裝[Node.js](https://nodejs.org/en/)。

```dockerfile
FROM debian:latest

MAINTAINER LempStacker.com

USER root

RUN mkdir -p /root/blog

WORKDIR /root/blog

RUN \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get dist-upgrade -y && \
    apt-get install -y xz-utils curl git

RUN \
    git config --global user.name "USERNAME" && \
    git config --global user.email "EMAIL" && \
    git config --global core.autocrlf input && \
    git config --global push.default simple && \
    git config --list && \
    ssh-agent -s && \
    eval $(ssh-agent -s) && \
    ssh-add /root/.ssh/id_ed25519 && \
    ssh -o StrictHostKeyChecking=no -T git@github.com

# https://superuser.com/questions/476512/how-do-i-permanently-reset-the-time-zone-in-debian
RUN \
    bash -c 'echo "Asia/Shanghai" > /etc/timezone' && \
    dpkg-reconfigure -f noninteractive tzdata &> /dev/null

RUN \
    mkdir -p /opt/nodejs && \
    curl -s -o /usr/local/src/node-linux-x64.tar.xz $(curl -sL https://nodejs.org/en/download/current/ | sed -n -r '/Linux Binaries \(x86\/x64\)/,+2s@.*href="(.*)".*@\1@p' | sed '$!d') && \
    tar xf /usr/local/src/node-linux-x64.tar.xz -C /opt/nodejs --strip-components=1 && \
    # ln -fs /opt/nodejs/include /usr/local/include/nodejs && \
    # echo -e "/opt/nodejs/lib" > /etc/ld.so.conf.d/nodejs.conf && \
    # ldconfig -v &> /dev/null && \
    # echo "export PATH=$PATH:/opt/nodejs/bin" >> /root/.bashrc && \
    # . /root/.bashrc
    ln -fs /opt/nodejs/bin/node /usr/local/bin/ && \
    ln -fs /opt/nodejs/bin/npm /usr/local/bin/

RUN \
    npm config set user 0 && \
    npm config set unsafe-perm true && \
    npm install hexo-cli -g && \
    hexo init && \
    npm install && \
    npm install hexo-generator-cname --save && \
    npm install hexo-generator-sitemap --save && \
    npm install hexo-generator-search --save && \
    #npm install hexo-generator-feed --save && \
    npm install hexo-deployer-git --save && \
    #npm un hexo-renderer-marked --save && \
    #npm i hexo-renderer-markdown-it --save

RUN \
    git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant && \
    npm install hexo-renderer-jade@0.3.0 --save && \
    npm install hexo-renderer-sass --save

RUN \
    apt-get purge -y curl xz-utils && \
    apt-get autoremove -y && \
    apt-get clean all && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/* && \
    history -c && \

VOLUME ["/root/blog/source"]

VOLUME ["/root/blog/themes"]

EXPOSE 4000

CMD ["/bin/bash"]
```

因爲本人使用主題是[Maupassant](https://www.haomwei.com/technology/maupassant-hexo.html)，需要執行如下操作

```bash
$ git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
$ npm install hexo-renderer-jade@0.3.0 --save
$ npm install hexo-renderer-sass --save
```

但在安裝`hexo-renderer-sass`時出現報錯，造成無法正常構建鏡像。

構建命令如下

```bash
#在Dockerfile所在目錄執行
docker build -t NewImageName .

#在其它目錄下執行
docker build -t NewImageName -f /PATH/Dockerfile .
```

*註*：此中的`NewImageName`是要生成的鏡像名，可以改成其它名字。


## Manual Operation
因無法通過Dockerfile自動構建鏡像，此處採用`曲線`方式進行構建，步驟
1. 以交互模式進入`debian`容器
2. 在容器中進行相關操作
3. 退出容器後，通過`docker commit`命令基於更改後的容器構建鏡像

SSH密鑰對文件在宿主機上生成後，採用volume形式掛在到容器中。文件權限設置如下

file|attribute
---|---
`.ssh/`|700
`id_ed25519`|600
`id_ed25519.pub`|644
`known_hosts`|644

**注意**： 須將公鑰文件`id_ed25519.pub`的內容複製到Github帳號的`SSH keys`中保存。

**重要**：因爲Blog部署在Github上，故而必須在容器中進行Git配置，操作過程參考 [Compile Install And Configure Git On CentOS7](https://lempstacker.com/tw/Compile-Install-And-Configure-Git-On-CentOS7/)。如果不進行該操作，執行`hexo deploy`時會報錯。


### Run node Container

```bash
docker images | grep debian

docker run -ti -h hexo --name hexo -p 4000:4000 -v ~/Documents/HexoBlog/LempStacker.com/.ssh:/root/.ssh -w /root debian:latest bash
```

### Install Software
`hexo`環境安裝在目錄`/boot/blog`中

進入容器後，依次執行如下命令

```bash
#更換 sources (中國大陸地區)
mv /etc/apt/sources.list{,.bak}

tee /etc/apt/sources.list <<-'EOF'
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
EOF

#更新操作系統
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y

#安裝相關軟件
apt-get install -y xz-utils curl git

#配置Git
git config --global user.name "YOUR USERNAME"
git config --global user.email "YOUT EMAIL"
git config --global core.autocrlf input
git config --global push.default simple

git config --list
ssh-agent -s
eval $(ssh-agent -s)
ssh-add /root/.ssh/id_ed25519
ssh -o StrictHostKeyChecking=no -T git@github.com

#更改時區
bash -c 'echo "Asia/Shanghai" > /etc/timezone'
dpkg-reconfigure -f noninteractive tzdata &> /dev/null


#下載、配置Node.js
cd /usr/local/src && rm -rf ./*
curl -s -o /usr/local/src/node-linux-x64.tar.xz $(curl -sL https://nodejs.org/en/download/current/ | sed -r -n '/linux-x64/{s@.*href="([^"]*)".*@\1@g;p}')

mkdir -pv /opt/nodejs
tar xf /usr/local/src/node-linux-x64.tar.xz -C /opt/nodejs --strip-components=1

ln -fs /opt/nodejs/include /usr/local/include/nodejs
bash -c 'echo -e "/opt/nodejs/lib" > /etc/ld.so.conf.d/nodejs.conf'
ldconfig -v &> /dev/null
# bash -c 'echo "export PATH=$PATH:/opt/nodejs/bin" > /etc/profile.d/nodejs.sh'
# source /etc/profile.d/nodejs.sh
bash -c 'echo "export PATH=$PATH:/opt/nodejs/bin" >> ~/.bashrc'
source ~/.bashrc

#創建blog路徑
mkdir -p /root/blog
cd /root/blog

#安裝hexo及相關插件
npm config set user 0
npm config set unsafe-perm true

npm install hexo-cli -g
hexo init
npm install

npm install hexo-generator-cname --save
npm install hexo-generator-sitemap --save
npm install hexo-generator-search --save
npm install hexo-deployer-git --save

#下載主題
git clone -q https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-jade@0.3.0 --save
npm install hexo-renderer-sass --save

#清理系統以減少空間使用
apt-get purge -y curl xz-utils
apt-get autoremove -y
apt-get clean all
rm -rf /var/lib/apt/lists/*
rm -rf /usr/local/src/*
rm -rf /tmp/*
history -c
```

操作完成後，執行`exit`退出容器

### Docker Commit
執行

```bash
#commit後的hexo是容器的名稱，由docker run的參數--name指定
docker commit hexo hexo:latest
```

構建名爲`hexo`，標籤爲`latest`的定製化鏡像。

除了通過容器名稱，還可通過容器id操作，可通過如下命令獲取容器id

```bash
docker ps -a | awk '$NF~/hexo/{print $1}'
```

此處爲`ad34f420052a`

定製化鏡像構建命令如下

```
docker commit ad34f420052a hexo:latest
```

構建定製化鏡像，除了`docker commit`，也可通過`export`、`import`命令實現，命令如下

```bash
docker export hexo > hexo.tar
docker import - hexo:latest < hexo.tar
```

### Running Container
因爲Blog文件目錄`source`，主題文件`themes`，CNMAE文件`CNAME`，hexo配置文件`_config.yml`在宿主機(本機)，通過volume形式掛載。

文件路徑

```bash
~/Documents/HexoBlog/LempStacker.com
```

執行如下命令啓動新容器

```
docker run -ti --rm -p 4000:4000 \
-v ~/Documents/HexoBlog/LempStacker.com/_config.yml:/root/blog/_config.yml \
-v ~/Documents/HexoBlog/LempStacker.com/source:/root/blog/source \
-v ~/Documents/HexoBlog/LempStacker.com/themes:/root/blog/themes \
-v ~/Documents/HexoBlog/LempStacker.com/CNAME:/root/blog/CNAME \
-v ~/Documents/HexoBlog/LempStacker.com/.ssh:/root/.ssh \
-w /root/blog \
--name hexoblog -h lempstacker hexo:latest bash
```

以交互模式進入容器，進行操作。

建議在命令最後添加`bash`指定，否則可能會出現如下報錯

>docker: Error response from daemon: No command specified.

爲方便操作，可在文件`~/.bashrc`中設置命令別名(`alias`)

```bash
hexo_dir='~/Documents/HexoBlog/LempStacker.com'
alias hexoblog="docker run -ti --rm -p 4000:4000 -v ${hexo_dir}/_config.yml:/root/blog/_config.yml -v ${hexo_dir}/source:/root/blog/source -v ${hexo_dir}/themes:/root/blog/themes -v ${hexo_dir}/CNAME:/root/blog/CNAME -v ${hexo_dir}/.ssh:/root/.ssh -w /root/blog --name hexoblog -h lempstacker hexo:latest bash"
```

## Hexo Operations
文件操作直接在本地更改，但是hexo的相關命令必須在容器中執行。

以下是相關命令

命令|說明
---|---
hexo n|創建新blog
hexo g|生成靜態文件
hexo s|啓動本地服務器
hexo d|發佈到網站
hexo c|清除緩存文件

具體命令，見官方文檔 [Commands](https://hexo.io/docs/commands.html)

常用的操作命令

```bash
#clean cache
hexo clean all

#local server
hexo s

#deploy to github.com
hexo g && hexo d
```

## Error Occuring
### npm install hexo-cli -g
報錯如下

```
#npm install hexo-cli -g
/opt/nodejs/bin/hexo -> /opt/nodejs/lib/node_modules/hexo-cli/bin/hexo

> dtrace-provider@0.8.1 install /opt/nodejs/lib/node_modules/hexo-cli/node_modules/dtrace-provider
> node scripts/install.js


> hexo-util@0.6.0 postinstall /opt/nodejs/lib/node_modules/hexo-cli/node_modules/hexo-util
> npm run build:highlight

npm ERR! Linux 3.16.0-4-amd64
npm ERR! argv "/opt/nodejs/bin/node" "/opt/nodejs/bin/npm" "run" "build:highlight"
npm ERR! node v7.8.0
npm ERR! npm  v4.2.0
npm ERR! path /root/.npm/_logs
npm ERR! code EACCES
npm ERR! errno -13
npm ERR! syscall scandir

npm ERR! Error: EACCES: permission denied, scandir '/root/.npm/_logs'
npm ERR!  { Error: EACCES: permission denied, scandir '/root/.npm/_logs'
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'scandir',
npm ERR!   path: '/root/.npm/_logs' }
npm ERR!
npm ERR! Please try running this command again as root/Administrator.
glob error { Error: EACCES: permission denied, scandir '/root/.npm/_logs'
  errno: -13,
  code: 'EACCES',
  syscall: 'scandir',
  path: '/root/.npm/_logs' }

> hexo-util@0.6.0 build:highlight /opt/nodejs/lib/node_modules/hexo-cli/node_modules/hexo-util
> node scripts/build_highlight_alias.js > highlight_alias.json

npm ERR! Linux 3.16.0-4-amd64
npm ERR! argv "/opt/nodejs/bin/node" "/opt/nodejs/bin/npm" "run" "build:highlight"
npm ERR! node v7.8.0
npm ERR! npm  v4.2.0

npm ERR! Callback called more than once.
npm ERR!
npm ERR! If you need help, you may report this error at:
npm ERR!     <https://github.com/npm/npm/issues>

npm ERR! Please include the following file with any support request:
npm ERR!     /root/.npm/_logs/2017-04-10T06_43_17_446Z-debug.log
/opt/nodejs/lib
└── (empty)

npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@^1.0.0 (node_modules/hexo-cli/node_modules/chokidar/node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.1: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})
npm ERR! Linux 3.16.0-4-amd64
npm ERR! argv "/opt/nodejs/bin/node" "/opt/nodejs/bin/npm" "install" "hexo-cli" "-g"
npm ERR! node v7.8.0
npm ERR! npm  v4.2.0
npm ERR! code ELIFECYCLE
npm ERR! errno 243

npm ERR! hexo-util@0.6.0 postinstall: `npm run build:highlight`
npm ERR! Exit status 243
npm ERR!
npm ERR! Failed at the hexo-util@0.6.0 postinstall script 'npm run build:highlight'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the hexo-util package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     npm run build:highlight
npm ERR! You can get information on how to open an issue for this project with:
npm ERR!     npm bugs hexo-util
npm ERR! Or if that isn't available, you can get their info via:
npm ERR!     npm owner ls hexo-util
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     /root/.npm/_logs/2017-04-10T06_43_20_889Z-debug.log
```

根據[Execute npm install hexo-cli -g promt ERR (root user)](https://github.com/hexojs/hexo/issues/2505#issuecomment-294123974)中提供的方案，先執行

```bash
npm config set user 0
npm config set unsafe-perm true
```

解決。



## Change Log
* 2016.06.16 20:55 Thu Asia/Shanghai
    * 初稿完成
* 2016.07.07 08:17 Thu Asia/Shanghai
    * docker啓動容器命令添加`-w`參數
* 2016.08.05 12：40 Fri Asia/Shanghai
    * 完善`Running Container`中相關命令
* 2016.12.21 14:17 Wed Asia/Shanghai
    * 重新製作hexo鏡像，新增時區更改
* 2017.01.31 11:37 Tue America/Boston
    * 文章重構，重新編寫鏡像構建命令
* 2017.04.17 09:41 Mon Asia/Shanghai
    * 解決`npm install hexo-cli -g`報錯
* 2017.05.16 14:13 Tue America/Boston
    * 更新`maupassant`主題配置命令
* 2019.04.28 14:43 Sun America/Boston
    * 勘誤，遷移到新Blog

[hexo]:https://hexo.io "A fast, simple & powerful blog framework"

<!-- End -->
