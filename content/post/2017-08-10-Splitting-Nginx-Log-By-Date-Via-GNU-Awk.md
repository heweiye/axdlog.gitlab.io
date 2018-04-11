---
title: Splitting Nginx Log By Date Via GNU Awk
date: 2017-08-10T16:40:24-08:00
lastmod: 2018-04-11T10:44:24-04:00
draft: false
categories:
- Production Case
tags:
- awk
comment: true
toc: true
---


[Nginx][nginx]在運行過程中會生成大量`access`和`error`日誌，默認情況下日誌文件大小持續增長。爲方便操作(如打開、查詢)，需對Nginx日誌進行切割，如按日期，而[Nginx][nginx]本身並不提供該功能。

網路上的方案千篇一律：利用[Nginx][nginx]的[Log Rotation][log_rotation]功能，操作日誌後，通過`USR1`信號重新打開日誌；將操作命令寫入腳本，通過cron任務定時執行。對於該種方案，本人不做任何評價。

本文主要記錄如何通過[awk][awk]實現Nginx日誌按年分、月份、周、天、小時等預定方式進行切割。

<!--more-->

## Prerequisite
[Nginx][nginx]日誌格式通過指令[log_format](https://nginx.org/en/docs/http/ngx_http_log_module.html#log_format)實現，默認的配置文件路徑爲`/etc/nginx/nginx.conf`。

詳細說明見官方文檔[Module ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html "Nginx")。


`log_format`的默認格式爲

```bash
log_format main  '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $bytes_sent '
                     '"$http_referer" "$http_user_agent" "$gzip_ratio"';
```

生成的日誌格式如下

>75.97.107.190 - - [10/Aug/2017:13:05:50 +0800] "GET /smart HTTP/1.1" 200 6452 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36"


以按日期分割爲例，通過字符串`[10/Aug/2017:13:05:50 +0800]`作爲分割日誌的依據，日期的處理通過`date`命令實現，具體見`man date`。

本文中[awk][awk]的操作命令適用於版本`3`和版本`4`。關於[awk][awk]的使用，本文不作說明，可自行查閱官方文檔[Gawk: Effective AWK Programming](https://www.gnu.org/software/gawk/manual/)。

文中的操作不做條件判斷，如在某一天之後或某一個IP或某個關鍵詞。


## Log Split
本文按照日期對日誌進行分割，假設日誌文件名稱爲`~/access.log`，結果輸出文件保存路徑爲`/tmp`，命名格式`nginx-###.log`。

### Yearly
按年份

```bash
# %Y     year
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date="a" +\"%Y\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

此處`/.* \[([^:]*):.*/`匹配的是`10/Aug/2017`；建議使用`/.* \[([^ ]*) .*/`，匹配`10/Aug/2017:13:05:50`。同時將`--date="a"`更改爲`--date=\""a"\"`以增強代碼的 **兼容性**。

修改後的代碼如下

```bash
# match 10/Aug/2017
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log

# match 10/Aug/2017:13:05:50
awk '{a=gensub(/.* \[([^ ]*) .*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017.log`

### Quarterly
按季度

```bash
#  %q     quarter of year (1..4)
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"-quarter-\"%q\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017-quarter-3.log`

### Monthly
按月份

```bash
# %b     locale's abbreviated month name (e.g., Jan)
# %m     month (01..12)
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"-\"%m\"-\"%b\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017-08-Aug.log`

### Weekly
按周

```bash
# %U week number of year, with Sunday as first day of week (00..53)
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"-week-\"%U\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017-week-32.log`


### Daily
按天

```bash
#  %F     full date; same as %Y-%m-%d
#  %a     locale's abbreviated weekday name (e.g., Sun)

awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%F\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log

# file name has weekday name
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%F\"-\"%a\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017-08-10-Thu.log`

### Hourly
按小時

```bash
# %H     hour (00..23)
awk '{a=gensub(/.* \[([^ ]*) .*/,"\\1","1",$0);a=gensub(/\:/," ","1",a);gsub(/\//,"-",a);"date --date=\""a"\" +\"%F\"-hour-\"%H\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-2017-08-10-hour-13.log`

### Per Minute
按分鐘

```bash
# %M     minute (00..59)
awk '{a=gensub(/.* \[([^ ]*) .*/,"\\1","1",$0);a=gensub(/\:/," ","1",a);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y%m%d\"-\"%H:%M\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-20170810-13:36.log`

### Per Second
按秒

```bash
# %S     second (00..60)
# %T     time; same as %H:%M:%S
awk '{a=gensub(/.* \[([^ ]*) .*/,"\\1","1",$0);a=gensub(/\:/," ","1",a);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y%m%d\"-\"%T\"" | getline b; print > "/tmp/nginx-"b".log"}' ~/access.log
```

日誌文件的命名如`nginx-20170810-13:36:09.log`


## Compression
### gzip gz
如果日誌文件已經被壓縮保存，可先解壓，在通過管道符`|`將數據傳輸給`awk`

```bash
# .gz
zcat ~/access.log.gz | awk '{a=gensub(/.* \[([^ ]*) .*/,"\\1","1",$0);a=gensub(/\:/," ","1",a);gsub(/\//,"-",a);"date --date=\""a"\" +\"%F\"-hour-\"%H\"" | getline b; print > "/tmp/nginx-"b".log"}'
```


## Example
以日誌`/var/log/baseserver_access.log`爲例，文件大小`7.9G`。

按月份分割，操作命令如下

```bash
awk '{a=gensub(/.* \[([^:]*):.*/,"\\1","1",$0);gsub(/\//,"-",a);"date --date=\""a"\" +\"%Y\"-\"%m\"-\"%b\"" | getline b; print > "/tmp/nginx-"b".log"}' /var/log/baseserver_access.log
```

生成的文件如下

```bash
-rw-r--r-- 1 root root  101M Aug 10 14:37 /tmp/nginx-2016-10-Oct.log
-rw-r--r-- 1 root root  656M Aug 10 14:37 /tmp/nginx-2016-11-Nov.log
-rw-r--r-- 1 root root  729M Aug 10 14:37 /tmp/nginx-2016-12-Dec.log
-rw-r--r-- 1 root root  650M Aug 10 14:37 /tmp/nginx-2017-01-Jan.log
-rw-r--r-- 1 root root  735M Aug 10 14:37 /tmp/nginx-2017-02-Feb.log
-rw-r--r-- 1 root root  876M Aug 10 14:37 /tmp/nginx-2017-03-Mar.log
-rw-r--r-- 1 root root  831M Aug 10 14:37 /tmp/nginx-2017-04-Apr.log
-rw-r--r-- 1 root root  937M Aug 10 14:37 /tmp/nginx-2017-05-May.log
-rw-r--r-- 1 root root 1022M Aug 10 14:37 /tmp/nginx-2017-06-Jun.log
-rw-r--r-- 1 root root  1.1G Aug 10 14:37 /tmp/nginx-2017-07-Jul.log
-rw-r--r-- 1 root root  388M Aug 10 14:37 /tmp/nginx-2017-08-Aug.log
```

通過`time`得到的操作時間如下

```bash
real	40m45.080s
user	39m59.076s
sys	0m21.479s
```

將近8G的日誌，操作耗時超過`40`分鐘，原因是awk默認只使用單核心處理器。


## Change Logs
* 2017.08.10 16:39 Thu Asia/Shanghai
    * 初稿完成
* 2018.04.11 10:44 Wed America/Boston
    * 勘誤，遷移到新Blog


[nginx]:https://www.nginx.com/ "Nginx"
[log_rotation]:https://www.nginx.com/resources/wiki/start/topics/examples/logrotation/ "Nginx"
[awk]:https://www.gnu.org/software/gawk/ "GNU awk"


<!-- End -->
