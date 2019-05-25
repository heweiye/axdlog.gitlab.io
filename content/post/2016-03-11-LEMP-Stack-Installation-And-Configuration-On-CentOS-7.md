---
title: LEMP Stack Installation And Configuration On CentOS 7
slug: LEMP Stack Installation And Configuration On CentOS 7
date: 2016-03-11T18:31:58+08:00
lastmod: 2018-05-07T09:54:46-04:00
draft: false
keywords: ["LEMP", "PHP", "Nginx", "MySQL", "SELinux", "Firewalld", "Shell Script"]
description: "Full recording of LEMP stacker installation and configuration on CentOS 7"
categories:
- PHP
tags:
- PHP
- Nginx
- MySQL
- Shell Script

comment: true
toc: true

---

This article documents how to install, configure **LEMP** ([Nginx][nginx]、[MySQL][mysql]、[PHP][php]) stack in `CentOS 7` which `SELinux`、`Firewalld` has been enabled.

I wrote some shell script to automate operation, the project is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript "axd Shell Script"). But [PHP][php] is still need to installed manually currently.

<!--more-->

If `SELinux` is `enabled`, it may causes [Nginx][nginx]、[MySQL][mysql] not work properly. I have solved these problems in my shell script.

**Attention**: If your current user is a normal user, you need to use `sudo` to execute the following commands.


## Overview
VPS public ip `104.236.100.67`

Version info

Info | Details
:--- | :---
OS Version | `CentOS Linux release 7.4.1708 (Core)`
Kernel Version | `3.10.0-693.21.1.el7.x86_64`
PHP | `7.2.4`
Nginx | `1.14.0`
MySQL | `5.7.22`


Default config file path

Software | Configuration Files
:--- | :---
PHP | `/etc/php.ini`, `/etc/php.d/`
PHP-FPM | `/etc/php-fpm.conf`, `/etc/php-fpm.d/www.conf`
Nginx | `/etc/nginx/nginx.conf`, `/etc/nginx/conf.d/*.conf`
MySQL | `/etc/my.cnf`, `/etc/my.cnf.d`, `~/.my.cnf`


Version Check Command

```bash
# - nginx
sudo nginx -V 2>&1 | awk -v FS='/' '{print $NF;exit}'
sudo nginx -V 2>&1 | sed -r -n '1 s@.*/(.*)@\1@p'
# For Bash 4+  2>&1 | == |&

# - mysql
mysql -V | sed -r -n 's@.*Distrib (.*),.*@\1@p'

# - php
php -v 2> /dev/null | awk '{print $2;exit}'
```

Conf Path Check Command

```bash
# - nginx
sudo nginx -V 2>&1 | sed -r -n 's@.*conf-path=(.*) --error.*@\1@p'

# - mysql
mysql --help | awk '$0~/Default options/{getline;print}'
mysqladmin --help | awk '$0~/Default options/{getline;print}'

# - php
php -i 2> /dev/null | awk '$0~/^Loaded Configuration File/{print $NF}'
```

## System Initialization
System initialization is through Shell script, the code is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh), usage info

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189224.js" id="asciicast-189224" async></script>

Using flag `-Z` to choose the type of `SELinux` (`permissive/enforcing/disabled`). `Firewalld` is enabled and open SSH port by default.

The status of `SELinux`和`Firewalld`

SELinux status

```bash
# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

Firewall status

```bash
# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client
  ports: 22/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks: echo-request echo-reply timestamp-reply timestamp-request
  rich rules:

```

## Repository
These repositories are used by LEMP stack.

* [EPEL Repo](https://fedoraproject.org/wiki/EPEL)
* [REMI Repo](http://rpms.famillecollet.com) (rely on eple， use to install PHP)
* [MySQL](https://dev.mysql.com/downloads/repo/)
* [MariaDB Repo](https://downloads.mariadb.org/mariadb/repositories/)
* [Nginx Repo](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)


## Nginx
Installing and configuing [Nginx][nginx]

### Nginx Installation
Installing Nginx via shell script, including common configuration, optimization, the code is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/NginxWebServer.sh).

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/NginxWebServer.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189210.js" id="asciicast-189210" async></script>

Here choosing `mainline` version via flag `-t m`, default is `stable` version. If you wanna open web server port (80) in firewall, just specify flag `-f`.

Nginx Version Info

```bash
# nginx -v
nginx version: nginx/1.14.0
```

Firewall rule

```bash
# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client
  ports: 22/tcp 80/tcp 443/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks: echo-request echo-reply timestamp-reply timestamp-request
  rich rules:

```

The default root directory of Nginx is `/usr/share/nginx/html/`. Make sure the `owner`、`group` is `nginx`， or it may occurs error `403 Forbidden`。

```bash
sudo chown -R nginx:nginx /usr/share/nginx/html/
```

## MySQL
Installing and configuing [MySQL][mysql]

### MySQL Installation
Installing MySQL via shell script, including common configuration, optimization, the code is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/MySQLVariants.sh). This script supports distro Debian/Ubuntu/CentOS/Fedora/OpenSUSE/SLES, database variant [MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona].

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/MySQLVariants.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/243053.js" id="asciicast-243053" async></script>

Here choosing **MySQL 5.7** via flag `-t mysql`、`-v 5.7`. If you wanna MySQL server port (`3306`) in firewall, just specify flag `-f`.

Starting service if it is not started.

```bash
sudo systemctl status mysqld.service

# start service
sudo systemctl start mysqld.service
```

MySQL Version Info

```bash
# mysql --version
mysql  Ver 14.14 Distrib 5.7.22, for Linux (x86_64) using  EditLine wrapper
```

User name, password is stores in file `~/.my.cnf`， password is strong random password generated by custom function.

```bash
# cat .my.cnf
[client]
user=root
password="fEf}db)coQfvV)hQ$GX@F!Pcefi!Kt{ICw<1"

[mysql]
prompt=MySQL [\d]>\_
```

Directly logging into MySQL interactive interface via command `mysql`

```sql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.22-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> select version();
+------------+
| version()  |
+------------+
| 5.7.22-log |
+------------+
1 row in set (0.00 sec)

MySQL [(none)]> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

MySQL [(none)]>
```

### Create Remote Login User
It is not recommended to use account `root` to login remotely. Here I create a normal user `axdlog` with password `AxdLog_MySQL@2018`.

```sql
-- create new user
MySQL [(none)]> create user 'axdlog'@'%' identified by 'AxdLog_MySQL@2018';
Query OK, 0 rows affected (0.00 sec)
-- grant privilege
MySQL [(none)]> grant all on *.* to 'axdlog'@'%';
Query OK, 0 rows affected (0.00 sec)
-- flush privilege
MySQL [(none)]> flush privileges;
Query OK, 0 rows affected (0.01 sec)

MySQL [(none)]> select user,host from mysql.user where user='axdlog';
+--------+------+
| user   | host |
+--------+------+
| axdlog | %    |
+--------+------+
1 row in set (0.00 sec)

MySQL [(none)]> show grants for 'axdlog'@'%'\G
*************************** 1. row ***************************
Grants for axdlog@%: GRANT ALL PRIVILEGES ON *.* TO 'axdlog'@'%'
1 row in set (0.00 sec)

MySQL [(none)]>
```


## PHP
Installing and configuing [PHP][php]

### REMI Repo
My initialization script has install `epel` repo. If you have not installed it, using the following command to install it:

```bash
sudo yum install -y epel-release
```

Installing `REMI` repo

```bash
sudo yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

There are some repo file prefixing with `remi` in directory `/etc/yum.repos.d/` after installation is finished: `remi-php54.repo`、`remi-php70.repo`、`remi-php71.repo`、`remi-php72.repo`、`remi.repo`、`remi-safe.repo`。

The configuration directives of `PHP 5.5`、`PHP 5.6` are listed in file `remi.repo`.

Here installing the latest release version `7.2.4`, so I need to enable file `remi-php72.repo` first. What I need to do is change the value of `enabled` to `1` (default is `enabled=0` meaning disable) .

```bash
sudo sed -r -i '/^\[remi-php72\]$/,/^$/{/^enabled=/{s@^([^=]+=).*@\11@g;}}' /etc/yum.repos.d/remi-php72.repo

# update cache
sudo yum makecache fast

# extract available pack list prefixing with php72-
yum info php72-* | awk 'match($0,/^Name[[:space:]]+:/){print gensub(/^[^:]+:[[:space:]]*(.*)$/,"\\1","g",$0)}'
```

### PHP Installation
The prefix of PHP packages is `php72-` (e.g. `php72-php-mysqlnd`). If listing them one by one is too troublesome. Directive `--enablerepo` can solve this problem (pack name still is `php-mysqlnd`).

Choosing `PHP-FPM` to manage PHP, so it is need to install `php-fpm`, database drive chooses `php-mysqlnd`.

```bash
sudo yum --enablerepo=remi-php72 install -y php php-common php-devel php-fpm php-mysqlnd php-mbstring php-xml php-pecl-mcrypt php-pecl-apc php-cli php-pear php-pdo php-bcmath php-ctype php-opcache php-gd php-json
```

Listing installed modules

```bash
php --modules
```

Config file path

* `/etc/php.ini`
* `/etc/php.d/`
* `/etc/php-fpm.conf`
* `/etc/php-fpm.d/`


### PHP Configuration
Modifing PHP config file `/etc/php.ini`

```bash
# backup
sudo cp -p /etc/php.ini{,.bak}
```

Directive list

```bash
# Close Nginx file type error resolve
cgi.fix_pathinfo=0
# timezone
date.timezone = Asia/Singapore
# Disable show PHP version info
expose_php = Off
# Disable PHP short tag
short_open_tag = Off
# Disable function list
disable_functions =
```

You may consider using the following method to operate quickly

```bash
funcPHPConfiguration(){
    local l_item=${1:-}
    local l_val=${2:-}
    local l_path=${3:-'/etc/php.ini'}
    [[ -f "${l_path}" && -n "${l_item}" && -n "${l_val}" ]] && sudo sed -r -i '/^;?[[:space:]]*'"${l_item}"'[[:space:]]*=/{s@^;?[[:space:]]*([^=]+=).*$@\1 '"${l_val}"'@g;}' "${l_path}"
}

funcPHPConfiguration 'cgi.fix_pathinfo' '0'
funcPHPConfiguration 'date.timezone' "Asia/Singapore"
funcPHPConfiguration 'expose_php' 'Off'
funcPHPConfiguration 'short_open_tag' 'Off'

disable_function_list="passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname"
funcPHPConfiguration 'disable_functions' "${disable_function_list}"
```

**Attention**: [Composer][composer] relies on function `proc_open`、`proc_get_status`, using the following command to enable them

```bash
sudo sed -r -i '/disable_functions[[:space:]]*=/{s@(proc_open|proc_get_status),?@@g;}' /etc/php.ini
```

### PHP-FPM Configuration
Using the following commands to manage `PHP-FPM` service

```bash
sudo systemctl status/start/stop/enable/disable php-fpm.service
```

If the server is started successfully, `PHP-FPM` will generate a temporary sub-directory `/var/run/php-fpm/` (On Debian the directory path is `/var/run/php/`). This path will be used while setting socket path in Nginx.

**Attention**： The default user, group of directory `/var/lib/php/session/` is `root`. It is  recommended that changing them to `nginx`:

```bash
[[ -d /var/lib/php/session ]] && sudo chown -R nginx:nginx /var/lib/php/session
```

PHP-FPM configuration file `/etc/php-fpm.d/www.conf`, modifing these directives

```bash
;listen = 127.0.0.1:9000
listen = /var/run/php-fpm/php-fpm.sock

# if not specify Nginx, Nginx can not listen php-fpm.sock
;listen.owner = nobody
;listen.group = nobody
;listen.mode = 0660
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

;user = apache
user = nginx

;group = apache
group = nginx
```

You may consider using the following method to operate quickly

```bash
funcPHPConfiguration(){
    local l_item=${1:-}
    local l_val=${2:-}
    local l_path=${3:-'/etc/php-fpm.d/www.conf'}
    [[ -f "${l_path}" && -n "${l_item}" && -n "${l_val}" ]] && sudo sed -r -i '/^;?[[:space:]]*'"${l_item}"'[[:space:]]*=/{s@^;?[[:space:]]*([^=]+=).*$@\1 '"${l_val}"'@g;}' "${l_path}"
}

funcPHPConfiguration 'listen' '/var/run/php-fpm/php72-fpm.sock'
funcPHPConfiguration 'listen.owner' 'nginx'
funcPHPConfiguration 'listen.group' 'nginx'
funcPHPConfiguration 'listen.mode' '0660'
funcPHPConfiguration 'user' 'nginx'
funcPHPConfiguration 'group' 'nginx'
```

Please make sure Nginx has been installed， or PHP-FPM will fail to start. Error log info is stored in file `/var/log/messages`.

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

PHP-FPM is running now

```bash
# ls /var/run/php-fpm/php72-fpm.sock
/var/run/php-fpm/php72-fpm.sock

# systemctl status php-fpm
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-04-12 15:04:07 EDT; 4s ago
 Main PID: 29046 (php-fpm)
   Status: "Ready to handle connections"
   CGroup: /system.slice/php-fpm.service
           ├─29046 php-fpm: master process (/etc/php-fpm.conf)
           ├─29047 php-fpm: pool www
           ├─29048 php-fpm: pool www
           ├─29049 php-fpm: pool www
           ├─29050 php-fpm: pool www
           └─29051 php-fpm: pool www

Apr 12 15:04:07 centos7 systemd[1]: Starting The PHP FastCGI Process Manager...
Apr 12 15:04:07 centos7 systemd[1]: Started The PHP FastCGI Process Manager.

```

### Nginx Configuration
There are directives about PHP existed in shell script, just uncomment them.

Searching `PHP FPM Start`、`PHP FPM End` in  file `/etc/nginx/conf.d/default.conf`, uncommenting the section `location`, changing the socket address of `fastcgi_pass` to `/var/run/php-fpm/php72-fpm.sock` which is defined in file `/etc/php-fpm.d/www.conf`.

You may consider using the following method to operate quickly

```bash
# socket file path
php_fpm_socket_path='/var/run/php-fpm/php72-fpm.sock'

sudo sed -r -i '/PHP FPM Start/,/PHP FPM End/{/PHP FPM/!{s@# @@g}; /fastcgi_pass/{s@^([^:]+:)[^\;]*(.*)$@\1'"${php_fpm_socket_path}"'\2@g;};}' /etc/nginx/conf.d/default.conf
```

The final directives

```bash
# PHP FPM Start
location ~ \.php$ {
    #try_files $uri = 404;
    root   /usr/share/nginx/html;
    fastcgi_pass unix:/var/run/php-fpm/php72-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
# PHP FPM End
```

**Attention**: `$document_root` is default point to `/etc/nginx/html/`, you need to specify root path in `location ~ \.php$`.

Using `nginx -t` to check Nginx config file syntax (need `root` privilege).

```bash
# sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Important**: Must restart Nginx, PHP-FPM service to make configuration change effect. Or Nginx will fails to interpret PHP code, and the PHP file will by downloaded directly.

```bash
# restart service
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```

### phpinfo()
PHP has a function `phpinfo()` to list all PHP releated info.

Creating file `/usr/share/nginx/html/index.php`

```bash
target_path='/usr/share/nginx/html/index.php'
sudo bash -c 'echo -e "<?php\nphpinfo();\n?>" > '${target_path}''
sudo chown nginx:nginx "${target_path}"
sudo chmod 640 "${target_path}"
```
Viewing via web browser

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-03-11_LEMP_Stack_On_Centos7/2018-04-12_16-11-58_php_info.png)


## Connection Testing

### Database Connection
If you're remotely connect MySQL, please make sure directive `bind_address` in file `/etc/my.cnf` is not bind local address `127.0.0.1` or `localhost`. Or it will refuse to connect.

Local Login

```sql
MySQL [(none)]> select user(),now();
+----------------+---------------------+
| user()         | now()               |
+----------------+---------------------+
| root@localhost | 2018-04-12 15:31:32 |
+----------------+---------------------+
1 row in set (0.00 sec)

MySQL [(none)]>
```

Romote Login

```sql
┌─[maxdsre@Stretch]─[~]
└──╼ $mysql -uaxdlog -p -h 104.236.100.67 -P 3306
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.22-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> select user(),now();
+------------------------+---------------------+
| user()                 | now()               |
+------------------------+---------------------+
| axdlog@18.233.165.46   | 2018-04-12 15:39:05 |
+------------------------+---------------------+
1 row in set (0.01 sec)

MySQL [(none)]> \q
Bye
```

### Web Server
Creating file `db.php` under Nginx root directory `/usr/share/nginx/html/` (Make sure user nginx has read privilege) to test if LEMP stack is working well or not.

You may consider using the following method to operate quickly

```bash
target_path='/usr/share/nginx/html/db.php'

sudo bash -c "cat > ${target_path}" << EOF
<?php
    # Database connection test
    try {
        \$pdo = new PDO('mysql:host=localhost;port=3306','axdlog','AxdLog_MySQL@2018');
        \$pdo->exec('set names utf8');
    } catch (Exception \$e) {
        echo 'Connection Failure, Error message: '.\$e->getMessage();
    }

    \$sql = "select user,host from mysql.user";
    \$stmt = \$pdo->prepare(\$sql);
    \$stmt->execute();
    \$rows = \$stmt->fetchAll(PDO::FETCH_ASSOC);
    echo '<pre>';
    print_r(\$rows);

    # Print PHP info
    #phpinfo();
?>
EOF

sudo chown nginx:nginx "${target_path}"
sudo chmod 640 "${target_path}"
```

Details
```bash
<?php
    # Database connection test
    try {
        $pdo = new PDO('mysql:host=localhost;port=3306','axdlog','AxdLog_MySQL@2018');
        $pdo->exec('set names utf8');
    } catch (Exception $e) {
        echo 'Connection Failure, Error message: '.$e->getMessage();
    }

    $sql = "select user,host from mysql.user";
    $stmt = $pdo->prepare($sql);
    $stmt->execute();
    $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
    echo '<pre>';
    print_r($rows);

    # Print PHP info
    #phpinfo();
?>
```

Inputing url `http://{IPADDR}(:PORT)/info.php` in browser (`{IPADDR}` is target host ip address).

* Local host, specify `localhost`或`127.0.0.1`；
* Remote host, specify public ip of remote hsot， if default port is not `80`, manually specify it;

Here the url is `http://104.236.100.67/info.php`

Connecting via Shell terminal

```bash
┌─[maxdsre@Stretch]─[~]
└──╼ $curl http://104.236.100.67/info.php
Array
(
    [0] => Array
        (
            [user] => axdlog
            [host] => %
        )

    [1] => Array
        (
            [user] => mysql.session
            [host] => localhost
        )

    [2] => Array
        (
            [user] => mysql.sys
            [host] => localhost
        )

    [3] => Array
        (
            [user] => root
            [host] => localhost
        )

)
┌─[maxdsre@Stretch]─[~]
└──╼ $
```

Connecting via web browser

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-03-11_LEMP_Stack_On_Centos7/2018-04-12_16-13-39_database_conn.png)

### Nginx Log
Nginx log format in Nginx access log

>"12/Apr/2018:16:11:37 -0400" remote_addr=18.233.165.46 request_method=GET request="GET /info.php HTTP/1.1" request_length=354 status=200 bytes_sent=347 body_bytes_sent=137 http_referer="-" http_user_agent="Mozilla/5.0 (X11; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0" request_time=0.002 upstream_addr=unix:/var/run/php-fpm/php72-fpm.sock upstream_status=200 upstream_cache_status=- upstream_response_time=0.002 upstream_connect_time=0.000 upstream_header_time=0.002 msec=1523563897.481 pipe=.gzip_ratio=3.26


## Change Logs
* 2016.03.11 18:18 Fri Asia/Beijing
	* 初稿完成
* 2016.03.23 15:48 Wed Asia/Beijing
    * 內容更新，相關配置參數
* 2016.03.30 21:56 Wed Asia/Beijing
    * 添加`Mounting Partitions`
* 2016.08.19 10:41 Fri Asia/Shanghai
    * 添加參數`worker_rlimit_nofile`
* 2016.12.08 14:18 Thu Asia/Shanghai
    * MariaDB配置文件`/etc/my.cnf`參數優化
* 2016.12.20 23:18 Tue Asia/Shanghai
    * Nginx參數配置優化
* 2016.12.21 11:33 Wed Asia/Shanghai
    * 添加`Security Optimization`安全相關
* 2018.04.12 16:38 Thu America/Boston
    * 內容更新，勘誤，遷移到新Blog


[nginx]: https://www.nginx.com "High Performance Load Balancer, Web Server, & Reverse Proxy"
[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."
[percona]:https://www.percona.com/ "The Database Performance Experts"
[mariadb]:https://mariadb.org/ "One of the most popular database servers. Made by the original developers of MySQL. Guaranteed to stay open source."
[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"
[composer]: https://getcomposer.org "Dependency Manager for PHP"

<!-- End -->
