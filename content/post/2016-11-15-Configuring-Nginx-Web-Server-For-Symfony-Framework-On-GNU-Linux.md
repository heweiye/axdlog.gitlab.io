---
title: Configuring Nginx Web Server For Symfony Framework On GNU Linux
slug: Configuring Nginx Web Server For Symfony Framework On GNU Linux
date: 2016-11-15T17:18:20+08:00
lastmod: 2018-04-16T20:55:08-04:00
draft: false
keywords: ["Symfony", "PHP", 'Nginx']
description: "How to configure Nginx Web Server For Symfony Framework On GNU Linux"
categories:
- PHP
tags:
- PHP
- Nginx
- Symfony
comment: true
toc: true

---

[Symfony][symfony]是一款[PHP][php]開發框架，具體安裝、配置可參閱本人之前 [blog]({{< relref "2016-08-23-Installing-And-Setting-Up-The-Symfony-Framework-On-CentOS-7.md" >}})。本文記錄如何在[Nginx][nginx]中部署[Symfony][symfony]項目。

**說明**：文中的操作、配置基於[Symfony 3][symfony]（2016年11月），不一定適用於[Symfony 4][symfony]。此次遷移只做勘誤，不做具體配置的更新（2018年04月16日）。

<!--more-->

## Official Document
配置依據來自2分官方文檔

* Symfony - [Configuring a Web Server](https://symfony.com/doc/master/setup/web_server_configuration.html)
* Nginx - [Symfony Configuration](https://www.nginx.com/resources/wiki/start/topics/recipes/symfony/)


## Analysis
[Symfony][symfony]在其官方文檔中提供了在Nginx中的最小化配置，源碼如下

```bash
server {
    server_name domain.tld www.domain.tld;
    root /var/www/project/public;

    location / {
        # try to serve file directly, fallback to index.php
        try_files $uri /index.php$is_args$args;
    }

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php7.1-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;

        # optionally set the value of the environment variables used in the application
        # fastcgi_param APP_ENV prod;
        # fastcgi_param APP_SECRET <app-secret-id>;
        # fastcgi_param DATABASE_URL "mysql://db_user:db_pass@host:3306/db_name";

        # When you are using symlinks to link the document root to the
        # current version of your application, you should pass the real
        # application path instead of the path to the symlink to PHP
        # FPM.
        # Otherwise, PHP's OPcache may not properly detect changes to
        # your PHP files (see https://github.com/zendtech/ZendOptimizerPlus/issues/126
        # for more information).
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        # Prevents URIs that include the front controller. This will 404:
        # http://domain.tld/index.php/some-path
        # Remove the internal directive to allow URIs like this
        internal;
    }

    # return 404 for all other php files not matching the front controller
    # this prevents access to other php files you don't want to be accessible.
    location ~ \.php$ {
        return 404;
    }

    error_log /var/log/nginx/project_error.log;
    access_log /var/log/nginx/project_access.log;
}
```

說明

1. Depending on your PHP-FPM config, the `fastcgi_pass` can also be `fastcgi_pass 127.0.0.1:9000`.
2. This executes only `index.php` in the public directory. All other files ending in ".php" will be denied.
3. If you have other PHP files in your public directory that need to be executed, be sure to include them in the `location` block above.
4. After you deploy to production, make sure that you cannot access the `index.php` script (i.e. http://example.com/index.php).


## Production Deployment
主機信息 `CentOS Linux release 7.2.1511 (Core)`

Nginx信息

item|detail
---|---
配置文件路徑|`/etc/nginx/conf.d`
Web路徑|`/usr/share/nginx/html/`

在Web路徑中創建子目錄`arsenal`用於存放Symfony項目，完整路徑

```bash
/usr/share/nginx/html/arsenal
```

為Symfony項目創建Nginx配置文件，完整路徑

```bash
/etc/nginx/conf.d/arsenal.conf
```

將代碼部署到目錄/usr/share/nginx/html/arsenal後，**強烈建議** 執行如下操作

```bash
# 更改owner, group為nginx
sudo chown -R nginx:nginx /usr/share/nginx/html/arsenal
sudo chown -R nginx:nginx /var/lib/php/session
```

假設需要綁定的域名為 `as.raxtone.com`

通過`php-fpm`管理PHP，socket路徑為 `/var/run/php-fpm/php-fpm.sock`

### Configuration File
以下爲生產環境中的配置文件（設置只能公司內網訪問）

```bash
# /etc/nginx/conf.d/arsenal.conf
server {
    listen 80;
    server_name as.raxtone.com;
    root   /usr/share/nginx/html/arsenal/web;
    index index.php index.html;
    error_log /var/log/nginx/arsenal_error.log;
    access_log /var/log/nginx/arsenal_access.log;
    charset utf-8;

    location / {
        try_files $uri /app.php$is_args$args;
        allow 192.168.0.0/16;
        allow 127.0.0.1;
        deny all;
    }

    # PROD
    location ~ ^/app\.php(/|$) {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        internal;
    }

    location ~ \.php$ {
     return 404;
    }

    location ~* .(woff|eot|ttf|svg|mp4|webm|jpg|jpeg|png|gif|bmp|ico|css|js)$ {
        expires 1d;
        log_not_found off;
        access_log off;
    }

}
```

### Domain Resolution Checking
通過`dig`、`ping`、`curl`等命令查看域名解析信息

```
# dig as.raxtone.com

; <<>> DiG 9.9.4-RedHat-9.9.4-29.el7_2.4 <<>> as.raxtone.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10830
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;as.raxtone.com.			IN	A

;; ANSWER SECTION:
as.raxtone.com.		552	IN	A	192.168.4.250

;; Query time: 11 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Tue Nov 15 17:11:56 CST 2016
;; MSG SIZE  rcvd: 59

# ping -c 4 as.raxtone.com
PING as.raxtone.com (192.168.4.250) 56(84) bytes of data.
64 bytes from arsenal.raxtone.com (192.168.4.250): icmp_seq=1 ttl=63 time=0.638 ms
64 bytes from arsenal.raxtone.com (192.168.4.250): icmp_seq=2 ttl=63 time=0.326 ms
64 bytes from arsenal.raxtone.com (192.168.4.250): icmp_seq=3 ttl=63 time=0.549 ms
64 bytes from arsenal.raxtone.com (192.168.4.250): icmp_seq=4 ttl=63 time=0.536 ms

--- as.raxtone.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3000ms
rtt min/avg/max/mdev = 0.326/0.512/0.638/0.115 ms

# curl -I as.raxtone.com
HTTP/1.1 302 Found
Server: nginx
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Set-Cookie: PHPSESSID=rcd6vek80tctca5a7vfmra0ar6; path=/; HttpOnly
Cache-Control: no-cache
Location: /login
Date: Tue, 15 Nov 2016 09:12:07 GMT

```

### Snapshot
Login Page

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-22-52_login.png)

Dashboard Page

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-23-06_dashboard.png)


## References
* [Installing & Setting up the Symfony Framework](https://symfony.com/doc/master/setup.html)


## Change Logs
* 2016.11.15 17:21 Tue Asia/Shanghai
    * 初稿完成
* 2018.04.16 21:15 Mon America/Boston
    * 勘誤，遷移到新Blog


[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"
[nginx]: https://www.nginx.com "High Performance Load Balancer, Web Server, & Reverse Proxy"
[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."


<!-- End -->
