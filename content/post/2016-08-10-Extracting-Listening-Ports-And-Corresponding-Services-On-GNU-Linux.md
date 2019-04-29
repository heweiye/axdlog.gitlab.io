---
title: Extracting Listening Ports And Corresponding Services On GNU/Linux
slug: Extracting Listening Ports And Corresponding Services On GNU Linux
date: 2016-08-10T09:56:08+08:00
lastmod: 2019-04-29T10:18:42-04:00
draft: false
keywords: ["awk", "sed", "lsof"]
description: "Using Shell Script to extract ports being used and corresponding services"
categories:
- Production Case
- Data Process
tags:
- awk
- sed
- lsof
- Shell Script

comment: true
toc: true

---

最近幾天在整理公司服務器系統架構信息，其中一項是監聽端口及對應服務的整理。通過端口號可以找到對應的進程(及父进程)，再通过父进程ID找到对应的服务。

<!--more-->

## Comparsion
主要使用到的命令有`ss`、`netstat`、`ps`

在不同版本的CentOS Minimal中，命令是否預裝

OS|ss|netstat|ps
---|---|---|---
CentOS6.8| No | Yes | Yes
CentOS7| No | No | Yes

命令分屬於不同的套件

Package|Commands
---|---
[iproute](https://wiki.linuxfoundation.org/networking/iproute2) |`ip`, `rtmon`, `ss`
[net-tools](https://sourceforge.net/projects/net-tools/) |`ifconfig`, `netstat`, `route`
[procps-ng](https://sourceforge.net/projects/procps-ng/) |`ps`, `free`, `skill`, `pkill`, `pgrep`, `snice`, `tload`, `top`, `uptime`, `vmstat`, `w`, `watch`, `pwdx`

套件是否預安裝在CentOS Minimal中

OS|net-tools|iproute|procps-ng
---|---|---|---
CentOS6.8| Yes | No | No
CentOS7| No | No | Yes

以下是對套件的介紹，通過命令`yum info`即可查看

```txt

Name        : iproute
Arch        : x86_64
Version     : 3.10.0
Release     : 54.el7_2.1
Size        : 526 k
Repo        : updates/7/x86_64
Summary     : Advanced IP routing and network device configuration tools
URL         : http://kernel.org/pub/linux/utils/net/iproute2/
License     : GPLv2+ and Public Domain
Description : The iproute package contains networking utilities (ip and rtmon,
            : for example) which are designed to use the advanced networking
            : capabilities of the Linux 2.4.x and 2.6.x kernel.


Name        : net-tools
Arch        : x86_64
Version     : 2.0
Release     : 0.17.20131004git.el7
Size        : 304 k
Repo        : base/7/x86_64
Summary     : Basic networking tools
URL         : http://sourceforge.net/projects/net-tools/
License     : GPLv2+
Description : The net-tools package contains basic networking tools,
            : including ifconfig, netstat, route, and others.
            : Most of them are obsolete. For replacement check iproute package.


Name        : procps-ng
Arch        : i686
Version     : 3.3.10
Release     : 5.el7_2
Size        : 282 k
Repo        : updates/7/x86_64
Summary     : System and process monitoring utilities
URL         : https://sourceforge.net/projects/procps-ng/
License     : GPL+ and GPLv2 and GPLv2+ and GPLv3+ and LGPLv2+
Description : The procps package contains a set of system utilities that provide
            : system information. Procps includes ps, free, skill, pkill, pgrep,
            : snice, tload, top, uptime, vmstat, w, watch and pwdx. The ps
            : command displays a snapshot of running processes. The top command
            : provides a repetitive update of the statuses of running processes.
            : The free command displays the amounts of free and used memory on
            : your system. The skill command sends a terminate command (or
            : another specified signal) to a specified set of processes. The
            : snice command is used to change the scheduling priority of
            : specified processes. The tload command prints a graph of the
            : current system load average to a specified tty. The uptime command
            : displays the current time, how long the system has been running,
            : how many users are logged on, and system load averages for the
            : past one, five, and fifteen minutes. The w command displays a list
            : of the users who are currently logged on and what they are
            : running. The watch program watches a running program. The vmstat
            : command displays virtual memory statistics about processes,
            : memory, paging, block I/O, traps, and CPU activity. The pwdx
            : command reports the current working directory of a process or
            : processes.
```


故而需要先安裝相關套件

```sh
#CentOS6.x Minimal
sudo yum install -y iproute procps-ng

#CentOS7.x Minimal
sudo yum install -y net-tools iproute
```

## Shell Script
該腳本用於獲取系統正在被監聽的端口及調用對應端口的進程、應用。Shell腳本代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPortUsedInfoDetection.sh)。

使用方式如下

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPortUsedInfoDetection.sh | bash -s --
```

### Analysis
通過`lsof`命令獲取指定端口的進程id(pid)，對數據的處理主要是通過`awk`、`sed`命令。

1. 使用`awk`可更改數據展現形式
2. 使用`sed`可從目標字符串中提取指定字符串，如

```sh
sed -r 's@.*Djava.util.logging.config.file=(.*)/conf(.*)@\1@'
```

參數解釋

* `-r`： 使用正則 use extended regular expressions in the script
* `s`： 替換 substitute
* `.*`: 正則最小匹配(非貪婪匹配)
* `\1`: 指代之前的`(.*)`的內容，即要提取的內容


## Usage
使用實例

### Example 1
```bash
flying@lempstacker:~$ bash /tmp/portcheck.sh
Sorry, This Script Need root or sudo Privileges！
flying@lempstacker:~$ sudo !!
sudo bash /tmp/portcheck.sh
port     pid        service          servicePath                             
25       2018       exim4            /usr/sbin/exim4                         
68       1628       dhclient         /sbin/dhclient                          
80       6523       nginx            /usr/sbin/nginx                         
111      624        rpcbind          /sbin/rpcbind                           
123      752        chronyd          /usr/sbin/chronyd                       
323      752        chronyd          /usr/sbin/chronyd                       
631      769        cupsd            /usr/sbin/cupsd                         
631      774        cupsd            /usr/sbin/cups-browsed                  
799      624        rpcbind          /sbin/rpcbind                           
810      634        rpc.statd        /sbin/rpc.statd                         
1900     720        minissdpd        /usr/sbin/minissdpd                     
3306     1588       mysqld           /usr/sbin/mysqld                        
5353     673        avahi-daemon     avahi-daemon                            
5433     1540       postgres         /usr/lib/postgresql/9.6/bin/postgres    
10539    1628       dhclient         /sbin/dhclient                          
33524    673        avahi-daemon     avahi-daemon                            
34069    634        rpc.statd        /sbin/rpc.statd                         
38474    522        systemd-timesyn  /lib/systemd/systemd-timesyncd          
42221    634        rpc.statd        /sbin/rpc.statd                         
48296    634        rpc.statd        /sbin/rpc.statd                         
50117    673        avahi-daemon     avahi-daemon                            
50842    1628       dhclient         /sbin/dhclient                          
52959    634        rpc.statd        /sbin/rpc.statd                         
flying@lempstacker:~$
```

### Example2
開發測試環境

```sh
[root@is131 /tmp]
#bash test.sh
port     pid        service          servicePath                             
22       13267      sshd             /usr/sbin/sshd                          
80       7406       nginx            /usr/local/webmiddleware/tomcat-flycar-lgwx
80       16982      nginx            /usr/local/webmiddleware/tomcat-flycar-user
80       29586      nginx            /usr/local/webmiddleware/tomcat-flycar-user-web
80       9246       nginx            nginx                                   
80       22538      nginx            /usr/local/webmiddleware/tomcat-pmon    
80       30431      nginx            /usr/local/webmiddleware/tomcat-web-push-server
80       3875       nginx            /usr/local/webmiddleware/tomcat-flycar-order
80       15373      nginx            /usr/local/webmiddleware/tomcat-flycar-business-web
80       5784       nginx            /usr/local/webmiddleware/tomcat-baseserver
80       30154      nginx            /usr/local/webmiddleware/tomcat-flybus-weixin
80       29355      nginx            /usr/local/webmiddleware/tomcat-sgm-dealer
80       28999      nginx            /usr/local/webmiddleware/tomcat-sgm-pay
80       24198      nginx            /usr/local/webmiddleware/tomcat-customer-server
80       352        nginx            /usr/local/webmiddleware/tomcat-flybus-provider
443      6760       nginx            /usr/local/webmiddleware/tomcat-flycar-weixin
443      9246       nginx            nginx                                   
443      30154      nginx            /usr/local/webmiddleware/tomcat-flybus-weixin
2181     13333      java             /usr/local/zookeeper-3.4.6              
6000     29355      java             /usr/local/webmiddleware/tomcat-sgm-dealer
6001     29355      java             /usr/local/webmiddleware/tomcat-sgm-dealer
6002     28999      java             /usr/local/webmiddleware/tomcat-sgm-pay
6003     28999      java             /usr/local/webmiddleware/tomcat-sgm-pay
6004     29272      java             /usr/local/webmiddleware/tomcat-sgm-admin
6005     29272      java             /usr/local/webmiddleware/tomcat-sgm-admin
6391     4370       redis-server     /usr/local/redis-push/src/redis-server  
6392     4346       redis-server     /usr/local/redis/src/redis-server       
7000     352        java             /usr/local/webmiddleware/tomcat-flybus-provider
7001     352        java             /usr/local/webmiddleware/tomcat-flybus-provider
7004     30154      java             /usr/local/webmiddleware/tomcat-flybus-weixin
7005     30154      java             /usr/local/webmiddleware/tomcat-flybus-weixin
7006     17725      java             /usr/local/webmiddleware/tomcat-flybus-admin
7007     17725      java             /usr/local/webmiddleware/tomcat-flybus-admin
7008     20053      java             /usr/local/webmiddleware/tomcat-flybus-interface
7009     20053      java             /usr/local/webmiddleware/tomcat-flybus-interface
7010     31733      java             /usr/local/webmiddleware/tomcat-flybus-website
7011     31733      java             /usr/local/webmiddleware/tomcat-flybus-website
7012     28306      java             /usr/local/webmiddleware/tomcat-flybus-business-web
7013     28306      java             /usr/local/webmiddleware/tomcat-flybus-business-web
7014     28833      java             /usr/local/webmiddleware/tomcat-deer-user-provider
7015     28833      java             /usr/local/webmiddleware/tomcat-deer-user-provider
7050     15373      java             /usr/local/webmiddleware/tomcat-flycar-business-web
7051     15373      java             /usr/local/webmiddleware/tomcat-flycar-business-web
7052     28293      java             /usr/local/webmiddleware/tomcat-flycar-finance
7053     28293      java             /usr/local/webmiddleware/tomcat-flycar-finance
7056     16982      java             /usr/local/webmiddleware/tomcat-flycar-user
7057     16982      java             /usr/local/webmiddleware/tomcat-flycar-user
7060     3875       java             /usr/local/webmiddleware/tomcat-flycar-order
7061     3875       java             /usr/local/webmiddleware/tomcat-flycar-order
7062     29586      java             /usr/local/webmiddleware/tomcat-flycar-user-web
7063     29586      java             /usr/local/webmiddleware/tomcat-flycar-user-web
7064     6760       java             /usr/local/webmiddleware/tomcat-flycar-weixin
7065     6760       java             /usr/local/webmiddleware/tomcat-flycar-weixin
7068     31595      java             /usr/local/webmiddleware/tomcat-flycar-user-mobile
7069     31595      java             /usr/local/webmiddleware/tomcat-flycar-user-mobile
7070     28129      java             /usr/local/webmiddleware/tomcat-flycar-driver
7071     28129      java             /usr/local/webmiddleware/tomcat-flycar-driver
7072     4645       java             /usr/local/webmiddleware/tomcat-flycar-chargestation
7073     4645       java             /usr/local/webmiddleware/tomcat-flycar-chargestation
7074     28951      java             /usr/local/webmiddleware/tomcat-flycar-pc-dispatch
7075     28951      java             /usr/local/webmiddleware/tomcat-flycar-pc-dispatch
7076     7406       java             /usr/local/webmiddleware/tomcat-flycar-lgwx
7077     7406       java             /usr/local/webmiddleware/tomcat-flycar-lgwx
7078     29068      java             /usr/local/webmiddleware/tomcat-flycar-carm
7079     29068      java             /usr/local/webmiddleware/tomcat-flycar-carm
7080     14266      java             /usr/local/webmiddleware/tomcat-flycar-provider
7081     14266      java             /usr/local/webmiddleware/tomcat-flycar-provider
7082     28912      java             /usr/local/webmiddleware/tomcat-shorturl
7083     28912      java             /usr/local/webmiddleware/tomcat-shorturl
8000     10321      java             /usr/local/webmiddleware/tomcat-routing
8001     10321      java             /usr/local/webmiddleware/tomcat-routing
8002     30431      java             /usr/local/webmiddleware/tomcat-web-push-server
8003     30431      java             /usr/local/webmiddleware/tomcat-web-push-server
8006     19500      java             /usr/local/webmiddleware/tomcat-pushts  
8007     19500      java             /usr/local/webmiddleware/tomcat-pushts  
8008     30905      java             /usr/local/webmiddleware/tomcat-base-admin
8009     30905      java             /usr/local/webmiddleware/tomcat-base-admin
8012     14001      java             /usr/local/webmiddleware/jetty-webpush-connect
8014     5784       java             /usr/local/webmiddleware/tomcat-baseserver
8015     5784       java             /usr/local/webmiddleware/tomcat-baseserver
8016     24198      java             /usr/local/webmiddleware/tomcat-customer-server
8017     24198      java             /usr/local/webmiddleware/tomcat-customer-server
8899     19500      java             /usr/local/webmiddleware/tomcat-pushts  
17062    29586      java             /usr/local/webmiddleware/tomcat-flycar-user-web
20880    352        java             /usr/local/webmiddleware/tomcat-flybus-provider
20882    28833      java             /usr/local/webmiddleware/tomcat-deer-user-provider
27076    7406       java             /usr/local/webmiddleware/tomcat-flycar-lgwx
28899    352        java             /usr/local/webmiddleware/tomcat-flybus-provider
50000    22538      java             /usr/local/webmiddleware/tomcat-pmon    
50001    22538      java             /usr/local/webmiddleware/tomcat-pmon    
50002    22538      java             /usr/local/webmiddleware/tomcat-pmon    
53259    13333      java             /usr/local/zookeeper-3.4.6              

[root@is131 /tmp]
#
```

## Change Logs
* 2016.08.10 17:50 Wed Asia/Shanghai
    * 初稿完成
* 2016.11.01 10：04 Tue Asia/Shanghai
    * Shell腳本修改，添加`$UID`權限判斷
* 2016.12.29 17:33 Thu Asia/Shanghai
    * Shell Script代碼重構
* 2016.01.04 11:21 Wed Asia/Shanghai
    * awk版本提取代碼修改以兼容Debian系統
* 2017.02.03 12:08 Fri America/Boston
    * 添加`lsof`命令是否存在的判斷、取消PPID判斷(Debian Jessie 8.7)
* 2017.02.27 18:40 Mon Asia/Shanghai
    * 代碼重構，改用`-tuanp`指令提取端口
* 2017.04.07 14:43 Fri Asia/Shanghai
    * 代碼優化，在`while`循環中使用`IFS`，`read -r`，優化`awk`檢測代碼
* 2019.04.29 10:16 Mon America/Boston
    * 勘誤，遷移到新Blog


<!-- End -->
