---
title: 在CentOS 7中配置LEMP開發環境全記錄
slug: LEMP Stack Installation And Configuration On CentOS 7
date: 2016-03-11T18:31:58+08:00
lastmod: 2018-05-07T09:54:46-04:00
draft: false
keywords: ["AxdLog", "LEMP", "PHP", "Nginx", "MySQL", "SELinux", "Firewalld", "Shell Script"]
description: "如何在CentOS 7中安裝、配置LEMP開發環境"
categories:
- PHP
tags:
- PHP
- Nginx
- MySQL

comment: true
toc: true

---

本文記錄如何在已啓用`SELinux`、`Firewalld`防火牆的`CentOS 7`系統中安裝、配置 **LEMP** ([Nginx][nginx]、[MySQL][mysql]、[PHP][php])套件。

本人寫了一些Shell腳本用以實現操作的自動化，項目代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript "axd Shell Script")。已實現 系統初始化，[Nginx][nginx]、[MySQL][mysql]的腳本化操作，但[PHP][php]暫時仍需要手動安裝。

<!--more-->

通常如果`SELinux`設爲`enabled`，會導致[Nginx][nginx]、[MySQL][mysql]無法正常使用，本人已在Shell腳本中處理好相關問題。

**注意**：如果是普通用戶，則須通過`sudo`執行相關命令。


## 概要
VPS主機外網地址 `104.236.100.67`

版本信息

Info | Details
:--- | :---
OS Version | `CentOS Linux release 7.4.1708 (Core)`
Kernel Version | `3.10.0-693.21.1.el7.x86_64`
PHP | `7.2.4`
Nginx | `1.14.0`
MySQL | `5.7.22`


配置文件默認安裝路徑

Software | Configuration Files
:--- | :---
PHP | `/etc/php.ini`, `/etc/php.d/`
PHP-FPM | `/etc/php-fpm.conf`, `/etc/php-fpm.d/www.conf`
Nginx | `/etc/nginx/nginx.conf`, `/etc/nginx/conf.d/*.conf`
MySQL | `/etc/my.cnf`, `/etc/my.cnf.d`, `~/.my.cnf`


版本檢查

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

配置文件路徑檢查

```bash
# - nginx
sudo nginx -V 2>&1 | sed -r -n 's@.*conf-path=(.*) --error.*@\1@p'

# - mysql
mysql --help | awk '$0~/Default options/{getline;print}'
mysqladmin --help | awk '$0~/Default options/{getline;print}'

# - php
php -i 2> /dev/null | awk '$0~/^Loaded Configuration File/{print $NF}'
```

## 系統初始化
系統初始化操作通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189224.js" id="asciicast-189224" async></script>

通過指令`-Z`設置`SELinux`的類型(`permissive/enforcing/disabled`)，默認啓用`Firewalld`防火牆並開放SSH端口。

系統初始化後`SELinux`和`Firewalld`防火牆的狀態

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

## 軟件源
安裝各組件需要用到的Repo

* [EPEL Repo](https://fedoraproject.org/wiki/EPEL)
* [REMI Repo](http://rpms.famillecollet.com) (依賴epel，用於安裝PHP)
* [MySQL](https://dev.mysql.com/downloads/repo/)
* [MariaDB Repo](https://downloads.mariadb.org/mariadb/repositories/)
* [Nginx Repo](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)


## Nginx
[Nginx][nginx]安裝、配置

### 安裝 Nginx
Nginx通過Shell腳本安裝，已包含常規的設置、優化，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/NginxWebServer.sh)。

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/NginxWebServer.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189210.js" id="asciicast-189210" async></script>

此處安裝`mainline`版本，故指定參數`-t m`，默認安裝的是`stable`版本。若要在防火牆中開放Web服務器端口(80)，只需在Shell腳本中指定參數`-f`即可。

Nginx 版本信息

```bash
# nginx -v
nginx version: nginx/1.14.0
```

防火牆規則

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

Nginx的Root目錄默認爲`/usr/share/nginx/html/`，確保`owner`、`group`都是`nginx`，否則可能會出現`403 Forbidden`。

```bash
sudo chown -R nginx:nginx /usr/share/nginx/html/
```

## MySQL
[MySQL][mysql]安裝、配置

### 安裝 MySQL
MySQL通過Shell腳本安裝，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/MySQLVariants.sh)，腳本同時支持在Debian/Ubuntu/CentOS/Fedora/OpenSUSE/SLES等發行版中安裝[MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona]。

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/MySQLVariants.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189202.js" id="asciicast-189202" async></script>

此處安裝 **MySQL 5.7**，故在Shell腳本中指定參數`-t mysql`、`-v 5.7`。如需在防火牆中開啓端口(默認爲`3306`)，加上參數`-f`即可。

如果服務未啓動，執行如下命令啓動服務

```bash
sudo systemctl status mysqld.service

# start service
sudo systemctl start mysqld.service
```

MySQL 版本信息

```bash
# mysql --version
mysql  Ver 14.14 Distrib 5.7.22, for Linux (x86_64) using  EditLine wrapper
```

用戶名、密碼保存在文件`~/.my.cnf`，密碼是通過自定義函數隨機生成的強密碼。

```bash
# cat .my.cnf
[client]
user=root
password="fEf}db)coQfvV)hQ$GX@F!Pcefi!Kt{ICw<1"

[mysql]
prompt=MySQL [\d]>\_
```

通過命令`mysql`直接登入MySQL交互界面

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

### 創建遠程登錄用戶
對於數據庫而言，不建議使用`root`賬戶進行遠程登錄操作。此處以創建用戶`axdlog`爲例，密碼`AxdLog_MySQL@2018`。

```sql
-- 創建用戶
MySQL [(none)]> create user 'axdlog'@'%' identified by 'AxdLog_MySQL@2018';
Query OK, 0 rows affected (0.00 sec)
-- 授權
MySQL [(none)]> grant all on *.* to 'axdlog'@'%';
Query OK, 0 rows affected (0.00 sec)
-- 刷新權限
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
[PHP][php]安裝、配置

### 配置 REMI 倉庫
初始化腳本已經安裝了`epel`源，如果還沒有安裝，須先安裝，可通過如下命令安裝：

```bash
sudo yum install -y epel-release
```

通過如下命令安裝`REMI`

```bash
sudo yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

安裝完成後，目錄`/etc/yum.repos.d/`中出現如下若干以`remi`開頭的repo：`remi-php54.repo`、`remi-php70.repo`、`remi-php71.repo`、`remi-php72.repo`、`remi.repo`、`remi-safe.repo`。

其中`PHP 5.5`、`PHP 5.6`的配置信息在文件`remi.repo`中。

此處安裝最新版本`7.2.4`，故選擇啓用`remi-php72.repo`，將該文件中`[remi-php72]`中的`enabled`設置爲`1`即可。(這些文件默認是禁用的，即`enabled=0`)

```bash
sudo sed -r -i '/^\[remi-php72\]$/,/^$/{/^enabled=/{s@^([^=]+=).*@\11@g;}}' /etc/yum.repos.d/remi-php72.repo

# 更新緩存
sudo yum makecache fast

# 獲取以 php72- 開頭的可用安裝包列表
yum info php72-* | awk 'match($0,/^Name[[:space:]]+:/){print gensub(/^[^:]+:[[:space:]]*(.*)$/,"\\1","g",$0)}'
```

### 安裝 PHP
PHP相關安裝包都是以`php72-`爲前綴(如`php72-php-mysqlnd`)，逐個指定太過麻煩，可通過`--enablerepo`參數指定目標repo來規避該問題(包名仍指定`php-mysqlnd`即可)。

PHP使用`PHP-FPM`來管理，故需安裝`php-fpm`，數據庫驅動使用`php-mysqlnd`。

安裝命令如下

```bash
sudo yum --enablerepo=remi-php72 install -y php php-common php-devel php-fpm php-mysqlnd php-mbstring php-xml php-pecl-mcrypt php-pecl-apc php-cli php-pear php-pdo php-bcmath php-ctype php-opcache php-gd php-json
```

查看已安裝的模塊

```bash
php --modules
```

相關配置文件路徑

* `/etc/php.ini`
* `/etc/php.d/`
* `/etc/php-fpm.conf`
* `/etc/php-fpm.d/`


### 配置 PHP
PHP配置文件 `/etc/php.ini`，進行修改

```bash
# backup
sudo cp -p /etc/php.ini{,.bak}
```

修改 `/etc/php.ini`

```bash
#關閉Nginx文件類型錯誤解析
cgi.fix_pathinfo=0
#設置時區
date.timezone = Asia/Singapore
#禁止顯示PHP版本信息
expose_php = Off
#支持PHP短標籤
short_open_tag = Off
#禁用的功能函數
disable_functions =
```

可通過如下方式進行快速修改

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

**注意**：[Composer][composer]依賴函數`proc_open`、`proc_get_status`，需從`disable_functions`的列表中移除。執行如下命令即可啓用：

```bash
sudo sed -r -i '/disable_functions[[:space:]]*=/{s@(proc_open|proc_get_status),?@@g;}' /etc/php.ini
```

### 配置 PHP-FPM
通過如下命令管理`PHP-FPM`服務

```bash
sudo systemctl status/start/stop/enable/disable php-fpm.service
```

服務啓動後，會在目錄`/var/run/`中生成臨時目錄`/var/run/php-fpm/`(Debian中生成的目錄名爲`/var/run/php/`)，設置`socket`監聽時會用到該路徑。

**注意**：目錄`/var/lib/php/session/`默認的屬主、屬組爲`root`，建議更改爲`nginx`，執行如下命令更改

```bash
[[ -d /var/lib/php/session ]] && sudo chown -R nginx:nginx /var/lib/php/session
```

PHP-FPM配置文件 `/etc/php-fpm.d/www.conf`，修改相關指令

```bash
;listen = 127.0.0.1:9000
listen = /var/run/php-fpm/php-fpm.sock

#如果不指定爲nginx，無法正常監聽php-fpm.sock
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

通過自定義函數修改

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

啓動PHP-FPM服務 (務必確認Nginx已經安裝，否則PHP-FPM無法正常啓動)。如果啓動失敗，可在日誌文件`/var/log/messages`查看錯誤信息。

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

PHP-FPM服務已成功啓動

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

### 配置 Nginx
Shell腳本中已配置有PHP相關指令，取消註釋即可。

打開文件 `/etc/nginx/conf.d/default.conf`，找到`PHP FPM Start`、`PHP FPM End`部分，將`location`部分的指令取消註釋，同時將`fastcgi_pass`的socket地址更改爲上文在文件`/etc/php-fpm.d/www.conf`的路徑，即 `/var/run/php-fpm/php72-fpm.sock`。

可通過如下命令修改

```bash
# socket file path
php_fpm_socket_path='/var/run/php-fpm/php72-fpm.sock'

sudo sed -r -i '/PHP FPM Start/,/PHP FPM End/{/PHP FPM/!{s@# @@g}; /fastcgi_pass/{s@^([^:]+:)[^\;]*(.*)$@\1'"${php_fpm_socket_path}"'\2@g;};}' /etc/nginx/conf.d/default.conf
```

最終格式如下

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

**注意**: `$document_root`默認指向`/etc/nginx/html/`，需在`location ~ \.php$`指定root路徑

使用`nginx -t`檢測配置文件是否有語法錯誤(需要`root`權限)

```bash
# sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**重要**：配置修改後，務必重啓Nginx、PHP-FPM服務，否則會 **出現Web服務器無法解釋執行PHP代碼，直接將目標PHP文件下載到本地** 的情況。

執行如下命令重啓服務

```bash
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```

### phpinfo()
PHP的詳細配置信息可通過函數`phpinfo()`查看。

通過如下命令創建文件`/usr/share/nginx/html/index.php`

```bash
target_path='/usr/share/nginx/html/index.php'
sudo bash -c 'echo -e "<?php\nphpinfo();\n?>" > '${target_path}''
sudo chown nginx:nginx "${target_path}"
sudo chmod 640 "${target_path}"
```

通過Web瀏覽器查看

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-03-11_LEMP_Stack_On_Centos7/2018-04-12_16-11-58_php_info.png)


## 連接測試

### 數據庫連接
測試MySQL能否正常連接，如果是遠程連接，確保`/etc/my.cnf`中指令`bind_address`是否綁定本地地址(`bind_address = 127.0.0.1`)，如果是，則無法成功連接。

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

### Web 服務器
在Nginx的Root目錄`/usr/share/nginx/html/`下創建`db.php`文件（確保Nginx對文件有讀權限），測試LEMP能否正常工作，包括PHP和數據庫連接。

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

文件內代碼如下

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

在瀏覽器地址欄中輸入 `http://{IPADDR}(:PORT)/info.php`，其中的`{IPADDR}`即主機IP地址

* 如果是本機，則可用`localhost`或`127.0.0.1`；
* 如果是遠程主機，則使用遠程主機的地址，如果不是`80`端口，則須手動指定端口；

此處遠程訪問地址爲 `http://104.236.100.67/info.php`

通過Shell終端連接

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

通過瀏覽器訪問

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-03-11_LEMP_Stack_On_Centos7/2018-04-12_16-13-39_database_conn.png)

### Nginx 日誌
Nginx中訪問日誌格式

>"12/Apr/2018:16:11:37 -0400" remote_addr=18.233.165.46 request_method=GET request="GET /info.php HTTP/1.1" request_length=354 status=200 bytes_sent=347 body_bytes_sent=137 http_referer="-" http_user_agent="Mozilla/5.0 (X11; Linux x86_64; rv:59.0) Gecko/20100101 Firefox/59.0" request_time=0.002 upstream_addr=unix:/var/run/php-fpm/php72-fpm.sock upstream_status=200 upstream_cache_status=- upstream_response_time=0.002 upstream_connect_time=0.000 upstream_header_time=0.002 msec=1523563897.481 pipe=.gzip_ratio=3.26


## 更新日誌
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
