---
title: Using HAProxy and Keepalived For Nginx on CentOS 7
slug: Using HAProxy and Keepalived For Nginx on CentOS 7
date: 2016-04-11T14:52:18+08:00
lastmod: 2018-08-05T10:15:02+08:00
draft: false
keywords: ["AxdLog", "Nginx", "HAProxy", "Keepalived"]
description: "Using HAProxy and Keepalived Achieve High Availability of Load Balance For Nginx on CentOS 7"
categories:
- High Availability
tags:
- HAProxy
- Keepalived
- Nginx
comment: true
toc: true

---


本文目的是通過`HAProxy`和`Keepalived`實現Nginx的高可用(High Availability)和負載均衡(Load Balance)。

>HAProxy can run in two modes: TCP mode Layer 4 and HTTP Mode Layer 7. In Layer 4 TCP mode, HAProxy forwards the RAW TCP packets from the client to the application servers. In the Layer 7 HTTP mode, HAProxy is parsing the HTTP header before forwarding them to the application servers.


## Preparation
準備兩臺`lb`主機，三臺`nginx`主機

操作系統均爲`CentOS Linux release 7.2.1511 (Core)`

| Server | Role | IP |
| :--- | :--- | :--- |
| `nginx1` | nginx | 192.168.0.131 |
| `nginx2` | nginx | 192.168.0.132 |
| `nginx3` | nginx | 192.168.0.133 |
| `loadbalancer1` | lb | 192.168.0.141 |
| `loadbalancer2` | lb | 192.168.0.142 |


通過`HAProxy`實現負載均衡，通過`Keepalived`實現虛擬地址漂移(vrrp)

虛擬主機IP爲 `192.168.0.130`，測試時直接通過該IP訪問Nginx主機。


### Vagrantfile

測試主機使用`Vagrant`創建，以下是配置文件`Vagrantfile`

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# `Virtual IP`: 192.168.0.130

boxes = [
    { :name => :nginx1,       :role => 'nginx', :ip => '192.168.0.131' },
    { :name => :nginx2,       :role => 'nginx', :ip => '192.168.0.132' },
    { :name => :nginx3,       :role => 'nginx', :ip => '192.168.0.133' },
    { :name => :loadbalance1, :role => 'lb', :ip => '192.168.0.141' },
    { :name => :loadbalance2, :role => 'lb', :ip => '192.168.0.142' },
]

Vagrant.configure(2) do |config|
    config.vm.box = "CentOS7Minimal"

    boxes.each do |opts|
        config.vm.define opts[:name] do |config|
            config.vm.network "public_network",ip: opts[:ip],bridge: "wlp3s0"
            config.vm.host_name = "%s.vm" % opts[:name].to_s
            config.vm.provision "shell", inline: <<-SHELL
                sudo yum install -y chrony
                sudo systemctl start chronyd
                sudo systemctl enable chronyd
            SHELL
            if opts[:role] == 'nginx'
                config.vm.provision "shell", inline: <<-SHELL
                    sudo yum install -y nginx
                    sudo systemctl start nginx
                    sudo systemctl enable nginx
                SHELL
            end
            if opts[:role] == 'lb'
                config.vm.provision "shell", inline: <<-SHELL
                    sudo yum install -y haproxy keepalived
                SHELL
            end
        end
    end

end
```


## Configuations
### Hosts
修改`/etc/hosts`文件

#### LoadBalancer Hosts
`lb`主機

在`/etc/hosts`中寫入如下信息，分別是Nginx主機的IP和主機名

```
192.168.0.131 nginx1
192.168.0.132 nginx2
192.168.0.133 nginx3
```

#### Nginx Hosts
登錄各Nginx主機

在`/etc/hosts`中寫入如下信息，LB主機的IP和主機名
```
192.168.0.141 loadbalance1
192.168.0.142 loadbalance2
```

### LB
#### HAProxy
在`lb`中的每臺主機上進行配置


* `haproxy.cfg`
```
sudo cp -p /etc/haproxy/haproxy.cfg{,.bak}

sudo vim /etc/haproxy/haproxy.cfg
```

寫入如下信息

```txt
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000


#---------------------------------------------------------------------
#HAProxy Monitoring Config
#---------------------------------------------------------------------
listen haproxy3-monitoring *:8080      #Haproxy Monitoring run on port 8080
    mode http
    option forwardfor
    option httpclose
    stats enable
    stats show-legends
    stats refresh 5s
    stats uri /stats                    #URL for HAProxy monitoring
    stats realm Haproxy\ Statistics
    #此處設置登錄haproxy stats頁面的認證用戶名、密碼
    stats auth lemp:12345    #User and Password for login to the monitoring dashboard
    stats admin if TRUE
    default_backend nginxservers            #This is optionally for monitoring backend


#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:80
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             nginxservers

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend nginxservers
    balance     roundrobin
    #此處填寫Nginx主機IP
	server nginx1 192.168.0.131:80 check
	server nginx2 192.168.0.132:80 check
	server nginx3 192.168.0.133:80 check
```

`backend`的名稱可自定義，此處爲`nginxservers`，該名稱決定了`default_backend`的名稱

`backend`填入Nginx主機IP

在瀏覽器中輸入`http://HAProxyIP:8080/stats`，輸入認證用戶名、密碼即可查看Nginx主機狀態信息。


/etc/rsyslog.conf

```bash
sudo cp -p /etc/rsyslog.conf{,.bak}

#取消以下兩行的註釋
$ModLoad imudp
$UDPServerRun 514
#$UDPServerAddress 127.0.0.1
```

創建文件 `/etc/rsyslog.d/haproxy.conf`， 寫入如下信息

```
local2.=info     /var/log/haproxy-access.log    #For Access Log
local2.notice    /var/log/haproxy-info.log      #For Service Info - Backend, loadbalancer
```

執行如下命令

```bash
sudo systemctl restart rsyslog
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

#### Keepalived

* `/etc/sysctl.conf`
綁定非本機虛擬IP

```
#vim /etc/sysctl.conf

net.ipv4.ip_nonlocal_bind = 1
```

執行`sudo sysctl -p`生效


* `/etc/keepalived/keepalived.conf`

```bash
sudo cp -p /etc/keepalived/keepalived.conf{,.bak}

sudo vim /etc/keepalived/keepalived.conf

router_id #每一臺惟一
priority  #每一臺不一樣，優先級
virtual_router_id 所有節點一致
auth_pass 所有節點一致
virtual_ipaddress 所有節點一致
```


| Nodes | loadbalance1 | loadbalance2 |
| :--- | :--- | :--- |
| `router_id` | haproxy1 | haproxy2 |
| `priority` | 101 | 100 |
| `virtual_router_id` | 10 | 10 |
| `weight` | 2 | 2 |
| `interface` | enp0s8 | enp0s8 |


以下是`loadbalance1`中的配置，對應更改

```
global_defs {
   notification_email {
     example@example.com
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from haproxy1@example.com
   smtp_server stmp@example.com
   smtp_connect_timeout 30
   #路由id，每臺主機惟一
   router_id haproxy1
}

vrrp_script haproxy {
    script "killall -0 haproxy"
    interval 2
    #權重
    weight 2
}


vrrp_instance VI_1 {
    state MASTER
    #interface eth0
    #網卡接口，根據實際情況設置
    interface enp0s8
    #虛擬路由id，保持一致
    virtual_router_id 10
    #優先級，每臺主機不一樣
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 12345
    }
    #虛擬IP地址
    virtual_ipaddress {
        192.168.0.130
    }
    track_script {
        haproxy
    }
}
```


執行如下命令

```bash
sudo systemctl start keepalived
sudo systemctl enable keepalived
```


### Nginx
在每一臺Nginx主機進行如下設置

```bash
sudo mv /usr/share/nginx/html/index.html{,.bak}
```

```
#在各自主機寫入如下信息
#nginx1
echo "<h1>nginx1.loadbalance.me</h1>" | sudo tee /usr/share/nginx/html/index.html

#nginx2
echo "<h1>nginx2.loadbalance.me</h1>" | sudo tee /usr/share/nginx/html/index.html

##nginx3
echo "<h1>nginx3.loadbalance.me</h1>" | sudo tee /usr/share/nginx/html/index.html
```

執行如下命令
```
sudo systemctl start nginx
sudo systemctl enable nginx
 ```

## Testing

虛擬主機IP爲 `192.168.0.130`，測試時直接通過該IP訪問Nginx主機。

在Shell中直接執行如下命令

```bash
for ((i=0;i<9;i++)); do curl -s http://192.168.0.130 | sed 's@<[^>]*>@@g'; done
```
可獲取Nginx服務器返回信息

### Start All

所有服務啓動

loadbalance1

```bash
vagrant@loadbalance1 ~]$ ip address show dev enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ff:35:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.141/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.0.130/32 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feff:35fb/64 scope link
       valid_lft forever preferred_lft forever
[vagrant@loadbalance1 ~]$
```

loadbalance2

```bash
[vagrant@loadbalance2 ~]$ ip address show dev enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0c:55:92 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.142/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe0c:5592/64 scope link
       valid_lft forever preferred_lft forever
[vagrant@loadbalance2 ~]$
```
可看到虛擬IP現在漂移到`loadbalance1`主機上


```bash
[maxdsre@lemp ~]$ for ((i=0;i<10;i++)); do curl -s http://192.168.0.130 | sed 's@<[^>]*>@@g'; done
nginx1.loadbalance.me
nginx2.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx2.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx2.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
[maxdsre@lemp ~]$
```

可看到，所有Nginx主機都正常，說明`HAProxy`生效

登錄`http://192.168.0.141:8080/stats`查看HAProxy面板信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-04-11_HAProxy_Keepalived_Nginx/haproxy_auth.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-04-11_HAProxy_Keepalived_Nginx/haproxy_allok.png)

### Stop Nginx Host
停止一臺Nginx主機，此處停用`nginx2`，在該主機執行如下命令

```bash
sudo systemctl stop nginx
```
以模擬主機宕機

再次執行

```bash
[maxdsre@lemp ~]$ for ((i=0;i<10;i++)); do curl -s http://192.168.0.130 | sed 's@<[^>]*>@@g'; done
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
[maxdsre@lemp ~]$
```

已經沒有`nginx2.loadbalance.me`

查看HAProxy面板信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-04-11_HAProxy_Keepalived_Nginx/haproxy_nginxstop.png)

可看到`Nginx2`主機顯示紅色，說明該主機已經停止工作

### Stop LB Host
停止一臺LoadBalance主機，此處停用`loadbalance1`，在該主機執行如下命令

```bash
sudo systemctl stop keepalived
sudo systemctl stop haproxy
```
以模擬主機宕機


loadbalance1

```bash
[vagrant@loadbalance1 ~]$ ip address show dev enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:ff:35:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.141/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feff:35fb/64 scope link
       valid_lft forever preferred_lft forever
[vagrant@loadbalance1 ~]$
```

loadbalance2

```bash
[vagrant@loadbalance2 ~]$ ip address show dev enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0c:55:92 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.142/24 brd 192.168.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.0.130/32 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe0c:5592/64 scope link
       valid_lft forever preferred_lft forever
[vagrant@loadbalance2 ~]$
```

可看到虛擬IP現在漂移到`loadbalance2`主機上

```bash
[maxdsre@lemp ~]$ for ((i=0;i<10;i++)); do curl -s http://192.168.0.130 | sed 's@<[^>]*>@@g'; done
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx1.loadbalance.me
[maxdsre@lemp ~]$
```
仍能正常訪問剩餘的Nginx主機

如果將`loadbalance2`也停用，則無法再正常訪問。


### Strt Nginx Host
再次啓動`nginx2`以模擬主機上線

```bash
[maxdsre@lemp ~]$ for ((i=0;i<10;i++)); do curl -s http://192.168.0.130 | sed 's@<[^>]*>@@g'; done
nginx3.loadbalance.me
nginx2.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx2.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
nginx2.loadbalance.me
nginx1.loadbalance.me
nginx3.loadbalance.me
[maxdsre@lemp ~]$
```

說明`HAProxy`將請求正常負載到`nginx2`主機上。


關閉測試主機

```bash
[maxdsre@lemp HAProxyNginx]$ vagrant halt
==> loadbalance2: Attempting graceful shutdown of VM...
==> loadbalance1: Attempting graceful shutdown of VM...
==> nginx3: Attempting graceful shutdown of VM...
==> nginx2: Attempting graceful shutdown of VM...
==> nginx1: Attempting graceful shutdown of VM...
[maxdsre@lemp HAProxyNginx]$
```

## References
* [How to setup HAProxy as Load Balancer for Nginx on CentOS 7](https://www.howtoforge.com/tutorial/how-to-setup-haproxy-as-load-balancer-for-nginx-on-centos-7/)
* [Howto setup High-Available HAProxy with Keepalived](https://blog.laimbock.com/2014/10/01/howto-setup-high-available-haproxy-with-keepalived/)


## Change Log
* 2016.04.11 14:55 Mon Asia/Beijing
    * 初稿完成
* 2018-08-05 10:39 Sun Asia/Shanghai
    * 勘誤，排版，遷移到新Blog


<!-- End -->
