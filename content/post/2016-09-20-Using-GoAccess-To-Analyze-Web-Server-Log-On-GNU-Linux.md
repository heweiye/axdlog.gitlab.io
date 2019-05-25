---
title: Using GoAccess To Analyze Web Server Log On GNU/Linux
slug: Using GoAccess To Analyze Web Server Log On GNU Linux
date: 2016-09-20T10:29:12+08:00
lastmod: 2019-04-29T10:42:20-04:00
draft: false
keywords: ["awk", "sed", "sysctl"]
description: "Using GoAccess To Analyze Web Server Log On GNU/Linux"
categories:
- Production Case
tags:
- GoAccess

comment: true
toc: true

---


[GoAccess][goaccess]是一款開源的實時Web日誌分析器，可在`*nix`系統的終端(`terminal`)或通過瀏覽器與查看者(viewer)進行交互。

<!--more-->

## Causality
收到研發經理的需求：**統計某具體時間段內某服務請求API接口的數據**。

統計緯度有

1. 按各接口的總請求次數分類彙總；
2. 各接口每分鐘的請求數，取出請求數最多的20個API接口；

數據展示有指定的格式。

[Nginx](https://www.nginx.com/)訪問日誌中的[數據格式](http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format)如下

```txt
114.114.114.114 - - [07/Sep/2016:18:23:22 +0800] "POST /bususer/flybusLogin.do HTTP/1.1" 200 0 "-" "okhttp/3.1.2"
```

對應的Nginx的`log_format`為

```txt
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"
```

其中的`/bususer/flybusLogin.do`即為請求的API接口,指定時間範圍內的數據有 **772,528** 條。

本人的處理方案是：

1. 使用`vim`提取指定時間範圍內的數據存儲到臨時文件中；(也可通過`sed`進行)
2. 使用`scp`將該臨時文件從遠程服務器中下載到本地；
3. 使用`awk`、`bash`進行數據的清洗、提取；
4. 將提取的數據存入MariaDB數據庫中；
    * 另一方案：將提取的數據存入另一臨時文件中；
5. 數據入庫完成後，通過SQL語句進行數據分析彙總；

前兩步手動進行，之後的操作通過撰寫Shell Script執行。

數據清洗、提取、入庫 **全程耗時將近5個小時**，嚴重影響到工作效率。

後聽主管說有一款名為`GoAccess`的工具可以處理Nginx日誌，並有圖表展示，故而有了本文。

以下是個人電腦配置信息

item|detail
---|---
OS|`CentOS Linux release 7.2.1511 (Core)`
Kernel|`3.10.0-327.28.3.el7.x86_64`
CPU|`Intel(R) Core(TM) i5-5200U CPU @ 2.20GHz`
Memory|`8GB`

本文的相關操作也在該機上進行


## Introduction
`GoAccess`用C語言開發，代碼託管在[GitHub](https://github.com/allinurl/goaccess)，[演示頁面](https://rt.goaccess.io)。

其在[官網網站](https://goaccess.io/)的介紹如下：

>GoAccess is an open source real-time web log analyzer and interactive viewer that runs in a terminal in `*nix` systems or through your browser.
>
>It provides fast and valuable HTTP statistics for system administrators that require a visual server report on the fly.

關於其特性及支持的Web日誌格式，具體可見 [鏈接](https://github.com/allinurl/goaccess/blob/master/README.md)

常見問題見 [鏈接](https://goaccess.io/faq)。


## Installation
`GoAccess`目前的最新版本是 **`1.0.2`**，其官方的安裝文檔見 [GoAccess - Downloads](https://goaccess.io/download)，以下相關內容皆來自該文檔。

### Dependence
編譯安裝`GoAccess`，其依賴文件只需`ncurses`即可

>The only dependency is `ncurses`.

Distro|NCurses|GeoIP (optional)|Tokyo Cabinet (optional)
---|---|---|---
Ubuntu/Debian|libncursesw5-dev|libgeoip-dev|libtokyocabinet-dev
Fedora/RHEL/CentOS|ncurses-devel|geoip-devel|tokyocabinet-devel
Arch Linux|ncurses|geoip|[compile from source](https://goaccess.io/faq#tokyo-src)
Gentoo|sys-libs/ncurses|dev-libs/geoip|dev-db/tokyocabinet
Slackware|ncurses|GeoIP|tokyocabinet

>You may need to install tools like `gcc`, `make`, etc for compiling/building software from source. e.g., `base-devel`, `build-essential`, `"Development Tools"`.

安裝依賴包

```bash
#CentOS
sudo yum install -y epel-release
sudo yum install -y gcc make ncurses ncurses-devel geoip geoip-devel tokyocabinet tokyocabinet-devel

#Debian
sudo apt-get install libncursesw5-dev libgeoip-dev libtokyocabinet-dev
```

### Via Compile Installation
編譯安裝步驟見 [鏈接](https://github.com/allinurl/goaccess#installation)。

編譯安裝時如果`./configure`未指定`--prefix`、`--sysconfdir`等參數，則默認執行如下操作：

```bash
# 創建目錄 /usr/local/bin
/usr/bin/mkdir -p '/usr/local/bin'
# 將可執行文件 goaccess 複製到該目錄中
/usr/bin/install -c goaccess '/usr/local/bin'

# 創建目錄 /usr/local/etc
/usr/bin/mkdir -p '/usr/local/etc'
# 將文件 goaccess.conf 複製到該目錄中並賦予644權限
/usr/bin/install -c -m 644 config/goaccess.conf '/usr/local/etc'

# 創建目錄 /usr/local/share/doc/goaccess
/usr/bin/mkdir -p '/usr/local/share/doc/goaccess'
# 將相關文件複製到該目錄中
/usr/bin/install -c -m 644 resources/tpls.html resources/css/app.css resources/css/bootstrap.min.css resources/css/fa.min.css resources/js/app.js resources/js/charts.js resources/js/d3.v3.min.js resources/js/hogan.min.js '/usr/local/share/doc/goaccess'

# 創建目錄 /usr/local/share/man/man1
/usr/bin/mkdir -p '/usr/local/share/man/man1'
# 將文件 goaccess.1 複製到該目錄中並賦予644權限
/usr/bin/install -c -m 644 goaccess.1 '/usr/local/share/man/man1'
```

即相關目錄為`/usr/local/bin`、`/usr/local/etc`、`/usr/local/share/doc/goaccess`、`/usr/local/share/man/man1`

如果需要卸載已經編譯安裝好的`GoAccess`，執行命令

```bash
sudo make uninstall
```
即可，具體的操作為
```bash
cd '/usr/local/bin' && rm -f goaccess
cd '/usr/local/etc' && rm -f goaccess.conf
cd '/usr/local/share/doc/goaccess' && rm -f tpls.html app.css bootstrap.min.css fa.min.css app.js charts.js d3.v3.min.js hogan.min.js
cd '/usr/local/share/man/man1' && rm -f goaccess.1
```

#### Configure Options
Multiple options can be used to configure `GoAccess`. For a complete up-to-date list of configure options, run `./configure --help`

* `--enable-debug`: Compile with debugging symbols and turn off compiler optimizations. Default is disabled
* `--enable-utf8`: Compile with wide character support. Ncursesw is required.
* `--enable-geoip`: Compile with GeoLocation support. MaxMind's GeoIP is required. Default is disabled
* `--enable-tcb=<memhash|btree>`: Compile with Tokyo Cabinet storage support. *memhash* will utilize Tokyo Cabinet's on-memory hash database. *btree* will utilize Tokyo Cabinet's on-disk B+ Tree database. Default is disabled
* `--disable-zlib`: Disable zlib compression on B+ Tree database.
* `--disable-bzip`: Disable bzip2 compression on B+ Tree database.
* `--with-getline`: Use GNU `getline()` to parse full line requests instead of a fixed size buffer of 4096.

其餘選項
* `--disable-option-checking`: ignore unrecognized --enable/--with options
* `--disable-FEATURE`: do not include FEATURE (same as --enable-FEATURE=no)
* `--enable-FEATURE[=ARG]`: include FEATURE [ARG=yes]
* `--enable-silent-rules`: less verbose build output (undo: "make V=1")
* `--disable-silent-rules`: verbose build output (undo: "make V=0")
* `--enable-dependency-tracking`: do not reject slow dependency extractors
* `--disable-dependency-tracking`: speeds up one-time build


執行`yum info tokyocabinet`可看到如下信息
>Tokyo Cabinet is a library of routines for managing a database. It is the successor of QDBM. Tokyo Cabinet runs very fast. For example, the time required to store 1 million records is 1.5 seconds for a hash database and 2.2 seconds for a B+ tree database. Moreover, the database size is very small and can be up to 8EB. Furthermore, the scalability of Tokyo Cabinet is great.

故而編譯配置選項選擇如下：

```bash
# 數據存儲在內存中
--enable-tcb=memhash

# 數據存儲在磁盤中
--enable-tcb=btree
```

**注意**：數據存儲在磁盤中要比存儲在內存中耗時。

>A dataset of about 52M hits (12GB size) is parsed in 20 mins (in-memory), 60 mins (on-disk storage). -- https://goaccess.io/faq


此處指定可執行文件路徑`/usr/local`，配置文件路徑`/etc/goaccess`，安裝包臨時存放路徑`/tmp`。

```bash
# Way 1: Directly Download

## 通過curl
curl -# -o /tmp/goaccess.tar.gz http://tar.goaccess.io/goaccess-1.0.2.tar.gz
cd /tmp && tar -xzf goaccess.tar.gz -C /tmp
## 通過wget
# wget -q http://tar.goaccess.io/goaccess-1.0.2.tar.gz -P /tmp
# cd /tmp && tar -xzf goaccess-1.0.2.tar.gz -C /tmp

mv goaccess-1.0.2 goaccess
cd ./goaccess


# Way 2: Build from GitHub (Development)
git clone https://github.com/allinurl/goaccess.git /tmp #下載到臨時目錄中
cd /tmp/goaccess
autoreconf -fiv


# compile
# ./configure --prefix=/usr/local --sysconfdir=/etc/goaccess --enable-geoip --enable-utf8
./configure --prefix=/usr/local --sysconfdir=/etc/goaccess --enable-geoip --enable-utf8 --enable-tcb=memhash --with-getline
make -j 4   #Specifies the number of jobs (commands) to run simultaneously.
sudo make install
```

以下是安裝後的信息
```bash
[flying@lempstacker ~]$ which goaccess
/usr/local/bin/goaccess
[flying@lempstacker ~]$ ls /etc/goaccess/goaccess.conf
/etc/goaccess/goaccess.conf
[flying@lempstacker ~]$ goaccess -V
GoAccess - 1.0.2.
For more details visit: http://goaccess.io
Copyright (C) 2009-2016 by Gerardo Orellana
[flying@lempstacker ~]$
```

### Via Package Manager Installation
通過各GNU/Linux發行版的包管理器安裝，具體見 [鏈接](https://github.com/allinurl/goaccess#distributions)。

在CentOS中可通過如下命令安裝
```bash
sudo yum install -y epel-release
sudo yum install -y goaccess #在epel倉庫中
```

Debian/Ubuntu 可通過官方倉庫安裝最新版本的`GoAccess`

```bash
echo "deb https://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install goaccess
```

>Note: .deb packages in the official repo are available through *https* as well. You may need to install `apt-transport-https`.


## Command Line / Config Options

以下是命令參數 [鏈接](https://github.com/allinurl/goaccess#command-line--config-options)

>The following options can be supplied to the command or specified in the
configuration file. If specified in the configuration file, long options need
to be used without prepending `--`.
>
>Removing the query string with `-q` can greatly decrease memory consumption, especially on timestamped requests.


## Configuration
`GoAccess`官方文檔[Manual Page](https://goaccess.io/man)的[CUSTOM LOG/DATE FORMAT](https://goaccess.io/man#custom-log)部分對`GoAccess`的配置文件及`time-format`、`date-format`、`log-format`的格式進行了簡要說明。

其配置文件路徑可`%sysconfdir%/goaccess.conf`或`~/.goaccessrc`，其中`%sysconfdir%`可以是`/etc/`、`/usr/etc/`或`/usr/local/etc/`。

>The configuration file resides under: `%sysconfdir%/goaccess.conf` or `~/.goaccessrc`
Note `%sysconfdir%` is either `/etc/`, `/usr/etc/` or `/usr/local/etc/`

通常只需要設置`time-format`、`date-format`、`log-format`三個參數即可，可在`%sysconfdir%/goaccess.conf`中修改，也可直接寫入`~/.goaccessrc`。

**注**：本文的配置文件路徑為`/etc/goaccess/goaccess.conf`。

如果直接使用初始配置文件(未設置`time-format`、`date-format`、`log-format`)，執行

```bash
goaccess -f /tmp/api_access.log
```

會出現如下提示

```txt
+-----------------------------------------------------------+
| Log Format Configuration                                  |
| [SPACE] to toggle - [ENTER] to proceed                    |
|                                                           |
| [ ] NCSA Combined Log Format                              |
| [ ] NCSA Combined Log Format with Virtual Host            |
| [ ] Common Log Format (CLF)                               |
| [ ] Common Log Format (CLF) with Virtual Host             |
| [ ] W3C                                                   |
| [ ] Squid Native Format                                   |
|                                                           |
| Log Format - [c] to add/edit format                       |
|                                                           |
|                                                           |
| Date Format - [d] to add/edit format                      |
|                                                           |
|                                                           |
| Time Format - [t] to add/edit format                      |
|                                                           |
+-----------------------------------------------------------+
```

進行手動選擇、設置。

在配置文件`%sysconfdir%/goaccess.conf`的對應位置按如下格式進行修改 或 直接寫入`~/.goaccessrc`
```
# Apache/NGINX's log formats below.
time-format %H:%M:%S

# Apache/NGINX's log formats below.
date-format %d/%b/%Y

# NCSA Combined Log Format
log-format %h %^[%d:%t %^] "%r" %s %b "%R" "%u"
```

再次執行`goaccess -f /tmp/api_access.log`，`GoAccess`自動開始分析(parse)日誌中數據，分析完成後即能顯示圖表數據，如何在Shell窗口中使用快捷按鍵進行交互，可參考其官方文檔中的 [INTERACTIVE KEYS
](https://goaccess.io/man#interactive-keys)部分。


### time-format

**time-format** 的格式`%T`或 `%H:%M:%S`，如果時間戳(timestamp)設置了微秒(microsecond)，則必須設置`%f`。具體可通過命令`man strftime`查看。

>The time-format variable followed by a space, specifies the log format date containing any combination of regular characters and special format specifiers. They all begin with a percentage (`%`) sign. See `man strftime`. `%T` or `%H:%M:%S`.
>
>Note: If a timestamp is given in microseconds, %f must be used as time-format

* `%H`: The hour as a decimal number using a 24-hour clock (range 00 to 23).
* `%M`: The minute as a decimal number (range 00 to 59).
* `%S`: The second as a decimal number (range 00 to 60).  (The range is up to 60 to allow for occasional leap seconds.)

### date-format
具體可通過命令`man strftime`查看。

>The date-format variable followed by a space, specifies the log format date containing any combination of regular characters and special format specifiers. They all begin with a percentage (`%`) sign. See `man strftime`.
>
Note: If a timestamp is given in microseconds, `%f` must be used as date-format

* `%b`: The abbreviated month name according to the current locale.
* `%d`: The day of the month as a decimal number (range 01 to 31).
* `%Y`: The year as a decimal number including the century.

### log-format

>The log-format variable followed by a space or `\t` for tab-delimited, specifies the log format string.

具體指令解釋說明見[CUSTOM LOG/DATE FORMAT](https://goaccess.io/man#custom-log)。


## Usage
使用示例，以日誌文件`/tmp/api_access.log`為例

### Shell Windows
直接使用`-f`參數可在Shell窗口中生成圖表數據

執行
```bash
goaccess -f /tmp/api_access.log
```

截圖如下：

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-51-42.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-10.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-18.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-25.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-28.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-33.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/shell_window_2016-09-20_17-52-37.png)


### Browser
使用`sed`命令將指定時間段內的日誌存儲到臨時文件中，使用`goaccess`將分析好的數據寫入html文件通過瀏覽器打開
```bash
sed -n '/02\/Sep\/2016/,/19\/Sep\/2016:12/ p' /tmp/api_access.log > /tmp/access.log

goaccess -f /tmp/access.log -o ~/Desktop/report.html
# goaccess -f /tmp/access.log > ~/Desktop/report.html

google-chrome ~/Desktop/report.html
```

截圖如下：

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-09-34.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-09-51.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-09-58.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-10-08.png)


![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-10-17.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2016-09-20_GoAccess/browser_2016-09-20_18-10-28.png)


本文只進行簡單的測試，更詳細的示例參見 <https://github.com/allinurl/goaccess#usage>

## Conclusion
`GoAccess`功能確實強大，對於體積很大的日誌處理起來很方便。但對一些具體的需求並不能滿足，比如我接到的需求：

* 各接口每分鐘的請求數，取出請求數最多的20個API接口

在不動用ELK的前提下，似乎只能自己寫工具處理。

此處附上本人寫的腳本

## Shell Script
創建SQL數據表

```sql
create database if not exists log_analysis
    default character set=utf8
    default collate=utf8_general_ci;

use log_analysis;

create table if not exists log_analysis (
    id int unsigned not null auto_increment primary key comment '自增id',
    ip char(20) not null comment 'ip地址',
    access_time timestamp not null comment '訪問接口時間,格式 YYYY-MM-DD HH:MM:SS',
    method enum('POST','GET') default 'POST' comment 'api訪問methond',
    api_url char(60) not null comment 'api 地址',
    response_code char(3) not null comment '響應碼',
    key `log_ip` (`ip`),
    key `log_access_time` (`access_time`),
    key `log_method` (`method`),
    key `log_api_url` (`api_url`),
    key `log_response_code` (`response_code`)
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '接口訪問數據統計表';
```

```bash
#!/bin/bash

log_path='/tmp/analysis.txt'
# log_path='/tmp/test.txt'
target_file=`mktemp -t tempXXXXX.txt`
# 數據格式 116.228.89.242 - - [07/Sep/2016:18:23:22 +0800] "POST /bususer/flybusLogin.do HTTP/1.1" 200 0 "-" "okhttp/3.1.2"

while read line; do
    ip=`echo "$line" | awk '{print $1}'`
    api_url=`echo "$line" | awk '{print $7}'`
    response_code=`echo "$line" | awk '{print $9}'`

    # 日期時間處理 access_time
    timetemp=`echo "$line" | awk '{print $4}'`
    timetemp=${timetemp#*[}
    hms=${timetemp#*:}
    ymdtemp=${timetemp%%:*}
    ymdtemp=${ymdtemp/Sep/09}
    ymd=$(echo $ymdtemp | awk -v FS='/' '{printf("%s-%s-%s",$3,$2,$1)}')
    access_time=$ymd' '$hms

    # method處理 method
    methodtemp=`echo "$line" | awk '{print $6}'`
    method=${methodtemp#*\"}

    mysql -e "insert into log_analysis.log_analysis set ip='$ip',access_time='$access_time',method='$method',api_url='$api_url',response_code='$response_code';"

    # echo "$ymd $hms $hm $api_url $ip $response_code $method" >> "$target_file"

    unset ip
    unset api_url
    unset response_code
    unset timetemp
    unset hms
    unset ymdtemp
    unset ymd
    unset access_time
    unset methodtemp
    unset method
    # echo "$response_code"

    # awk '{print $1}' "$line"
done<"$log_path"

# End Script
```

數據提取語句

```bash
mysql -D log_analysis -Bse "select api_url as 'API',count(id) as Frequency from log_analysis group by api_url order by Frequency desc;" | awk '{printf("%s —— %s\n",$1,$2)}'

mysql -D log_analysis -Bse "select api_url, count(id) as num, date_format(access_time, '%Y年%m月%d日 %H時%i分') as accessTime from log_analysis group by api_url,date_format(access_time, '%Y年%m月%d日 %H時%i分') order by num desc limit 20;" | awk '{printf("%s —— %s —— %s\n",$1,$2,$3)}'
```


## Bibliography
* [GoAccess (A Real-Time Apache and Nginx) Web Server Log Analyzer](http://www.tecmint.com/goaccess-a-real-time-apache-and-nginx-web-server-log-analyzer/ 'Tecmint')
* [使用 GoAccess 分析 Nginx 網頁記錄檔，即時監控伺服器狀態](https://blog.gtwang.org/linux/analysing-nginx-logs-using-goaccess/)
* [GoAccess - 圖形化 Web Log 的分析程式](https://blog.longwin.com.tw/2015/12/goaccess-visual-web-log-analysis-2015/)
* [使用GoAccess分析Nginx日誌以及sed/awk手動分析實踐](https://wsgzao.github.io/post/goaccess/)


## Change Logs
* 2016.09.20 18:26 Tue Asia/Shanghai
    * 初稿完成
* 2019.04.29 10:56 Mon America/Boston
    * 勘誤，遷移到新Blog


[goaccess]:https://goaccess.io


<!-- End -->
