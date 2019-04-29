---
title: Generating Available Random Port Number On GNU/Linux
slug: Generating Available Random Port Number On GNU Linux
date: 2016-09-09T02:50:38+08:00
lastmod: 2019-04-29T10:38:20-04:00
draft: false
keywords: ["awk", "sed", "sysctl"]
description: "Using Shell Script To Generated Available Random Port Number On GNU/Linux"
categories:
- Production Case
- Data Process
tags:
- awk
- sed
- sysctl
- Shell Script

comment: true
toc: true

---

因個人工作需要，寫了一個穿透 **跳板機主機** 直接連接內網服務器主機的Shell腳本，用於直接連接內網主機，執行`ssh`或`sftp`命令。其中涉及到 **隨機生成的未被佔用的端口** ，本文主要介紹本人的處理思路及其實現方式。值得一提的是，在該腳本中成功實現了函數自身的遞歸(`recursive`)調用。

<!--more-->

該腳本在 `CentOS Linux release 7.2.1511 (Core)` 系統下撰寫。


## Personal Requests
以下是個人需求

1. 生成隨機數
2. 該隨機數必須大於`1024`(前1024個端口號只能root用戶使用)
2. 該隨機數必須在當前系統允許的端口號範圍內
3. 該隨機數必須不能佔用文件`/etc/services`中被分配的端口
4. 該隨機數必須是未被佔用/使用的端口號


## Thinking
1. 在命令`sysctl -a`的輸出結果中查詢`net.ipv4.ip_local_port_range`獲取當前系統允許的端口號範圍，使用`awk`提取數據；
2. 使用命令`ss -tunl`或`netstat -tupln`查看當前系統被監聽/佔用的端口信息，使用`awk`提取數據；
3. 使用命令`sed -n -r 's@.*[[:space:]]+([0-9]+)/.*@\1@p' /etc/services`從文件`/etc/services`中提取端口數值（含重複數值）
4. 通過讀取文件`/dev/urandom`中內容獲取隨機信息，
5. 在自定義函數內部進行判斷，是否符合預期要求，通過函數遞歸調用實現生成符合要求的隨機數

使用`cksum`命令獲取隨機數，可使用如下命令格式

```bash
# print the first K bytes of each  file;
head -c K /dev/urandom| cksum

# print  the first K lines instead of the first 10;
head -n K /dev/urandom| cksum
```

**注**： 對於默認端口分配，可查看文件`/etc/services`，也可訪問[Service Name and Transport Protocol Port Number Registry](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) 查看詳細介紹。


## Shell Script
Shell腳本代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxRandomAvailablePortGeneration.sh)。

使用方式如下

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxRandomAvailablePortGeneration.sh | bash -s --
```


## Usage
命令格式`bash FILEPATH/BASH_NAME `，建議使用絕對路徑，如果其中含有空格，須用雙引號`""`將其包裹。

此处假设Shell脚本的路径为`/tmp/port.sh`

### Executation
通過`for`循環生成多個隨機端口

```bash
$ for ((i=0;i<15;i++));do sudo bash /tmp/port.sh; done
Newly Generated Port NO. is 59612
Newly Generated Port NO. is 44238
Newly Generated Port NO. is 64898
Newly Generated Port NO. is 30770
Newly Generated Port NO. is 57741
Newly Generated Port NO. is 20995
Newly Generated Port NO. is 25898
Newly Generated Port NO. is 37325
Newly Generated Port NO. is 29044
Newly Generated Port NO. is 42332
Newly Generated Port NO. is 40999
Newly Generated Port NO. is 30149
Newly Generated Port NO. is 38957
Newly Generated Port NO. is 7215
Newly Generated Port NO. is 41652
$
```

### Executation Procedure
以下是腳本的具體執行過程

```bash
$ bash -x /tmp/port.sh
+ [[ 1000 -ne 0 ]]
+ printf '對不起，執行該腳本須擁有root或sudo權限！\n'
對不起，執行該腳本須擁有root或sudo權限！
+ exit 1
$ sudo bash -x /tmp/port.sh
+ [[ 0 -ne 0 ]]
+ c_red='\e[31;1m'
+ c_blue='\e[34m'
+ c_end='\e[0m'
++ sysctl -a
++ awk '$0~/ip_local_port_range/{print $(NF-1)}'
+ lport_from=1024
++ sysctl -a
++ awk '$0~/ip_local_port_range/{print $NF}'
+ lport_to=65000
++ mktemp -t tempXXXXX.txt
+ tempPortUsedlist=/tmp/tempsVOEH.txt
++ which ss
+ '[' '!' -z /sbin/ss ']'
+ ss -tunl
+ awk '$1~/udp|tcp/{print $(NF-1)}'
+ awk -v FS=: '$0!~/^*$/{arr[$NF]++}END{for(i in arr)print i}'
+ sed -n -r 's@.*[[:space:]]+([0-9]+)/.*@\1@p' /etc/services
+ awk '!a[$0]++{if($0>1024){print $0}}'
++ generate_random_port 1024 65000
++ local port_from=1024
++ local port_to=65000
+++ head -n 9 /dev/urandom
+++ cksum
+++ awk '{print $1}'
++ random_num=2778524653
++ port=34653
+++ grep -c -w 34653 /tmp/tempsVOEH.txt
++ [[ 0 -gt 0 ]]
++ [[ 34653 -lt 1024 ]]
++ echo 34653
+ generated_port=34653
+ printf '\e[34mNewly Generated Port NO. is\e[0m \e[31;1m34653\e[0m\n'
Newly Generated Port NO. is 34653
+ unset c_red
+ unset c_blue
+ unset c_end
+ unset lport_from
+ unset lport_to
+ unset usedPorts
+ unset generated_port
+ rm -f /tmp/tempsVOEH.txt
$
```


## References
* [shell实例浅谈之三产生随机数七种方法](http://blog.csdn.net/taiyang1987912/article/details/39997303)


## Change Logs
* 2016.09.09 10:47 Fri Asia/Shanghai
    * 初稿完成
* 2016.11.01 11:58 Tue Asia/Shanghai
    * 脚本重构，添加执行权限判断、文件`/etc/services`中端口过滤等
* 2016.12.21 15:52 Wed Asia/Shanghai
    * 代碼優化，更改判斷命令是否存在的方式
* 2019.04.29 10:38 Mon America/Boston
    * 勘誤，遷移到新Blog


<!-- End -->
