---
title: Extracting API Requests Counts From Nginx Access Log Per week
slug: Extracting API Requests Counts From Nginx Access Log Per week
date: 2017-12-12T19:58:34+08:00
lastmod: 2018-04-11T10:21:34-04:00
draft: false
keywords: ["awk", "sed", "Nginx"]
description: "Using sed and awk to extract API requests info from Nginx log"
categories:
- Production Case
tags:
- awk
- sed
comment: true
toc: true
---

2017年7月3日寫過一篇 [Extracting API Requests Info From Nginx Access Log]({{< relref "2017-07-03-Extracting-API-Requests-Info-From-Nginx-Access-Log.md" >}})，主要記錄從Nginx日誌中提取各自有API的請求頻次。最近一段時間，線上環境服務器的CPU、內存時常會被佔滿，導致線上服務宕掉。爲了解決該問題，其中一項舉措是要求運維這邊每週提取一次各API接口的請求數據，該任務仍有本人來操作。本文主要記錄在當前需求下，數據的提取過程。

<!--more-->

## Preface
如今再看，個人對之前那篇操作說明並不滿意，原因是操作過程耗時過長。再次進行該項工作時，不得不想辦法提高處理速度。

個人辦公電腦操作系統`Debian Stretch`，線上主機操作系統`CentOS 6.9`，整個過程主要使用命令`sed`和`awk`。`awk`只使用一個CPU核心，無法使用多核心，本人目前尚未實現如何通過`parallel`調用`awk`(命令複雜)進行多核心操作。

故而只能在實現思路上想辦法，調整後的代碼執行效率極大提高。

## Shell Script
以下是Shell腳本內容

```bash
#!/usr/bin/env bash
# Dec 12, 2017 Tue +0800

# variable setting
log_path='/data/log/api.deerbus.com_access.log'
extract_log_path=$(mktemp -t XXXXX.log)

# get date range
# last week monday
start_date=$(date --date='last monday 7 days ago' +'%d/%b/%Y')
# this monday
stop_date=$(date --date='last monday' +'%d/%b/%Y')

# increase file descriptor value
file_descriptor_val=${file_descriptor_val:-1024}
file_descriptor_val=$(ulimit -n)
ulimit -n 655360

# extract log in specified date range (a week)
start_line_no=1
[[ $(grep -c "${start_date}" "${log_path}") -gt 0 ]] && start_line_no="${start_date}"

sed -r -n '/\['"${start_line_no////\\/}"'/,/\['"${stop_date////\\/}"'/{/'"${stop_date////\\/}"'/d;/^$/d;s@-[[:space:]]@@g;s@[[:space:]]*\+[[:digit:]]+@@g;s@\"[[:upper:]]+ ([^[:space:]]*) .*@\1@g;s@(\[|\])@@g;s@ @|@g;s@([[:digit:]]{4}):@\1 @g;s@([[:digit:]]{1,2})/([[:alpha:]]+)/([[:digit:]]{4})@\1 \2 \3@g;p}' "${log_path}"  > "${extract_log_path}"

# output sample
# 58.38.100.200|10 Dec 2017 23:59:57|/buspay/buildPayParams.do

# extract specific info according differect degrees
# 1 - all api per seconds
awk -F\| 'BEGIN{OFS="|"}{arr[$2]++}END{for (i in arr) print i,arr[i]}' "${extract_log_path}" | sort -b -f -t"|" -k 2nr,2 | awk -F\| '{if($2>50){"date --date=\""$1"\" +\"%F %T\"" | getline a;printf("%s|%s\n",a,$2)}}' > /tmp/total_counts_per_second.txt

# 2 - per api per seconds
awk -F\| 'BEGIN{OFS="|"}{arr[$NF,$2]++}END{for (i in arr) {split(i,j,SUBSEP);print j[2],j[1],arr[i]}}' "${extract_log_path}" | sort -b -f -t"|" -k 3nr,3 -k 1nr,1 | awk -F\| '{if($3>10){"date --date=\""$1"\" +\"%F %T\"" | getline a;printf("%s|%s|%s\n",a,$2,$3)}}' > /tmp/total_counts_per_second.txt

# 3 - total counts for specific api
awk -F\| 'BEGIN{OFS="|"}{arr[$3]++}END{for (i in arr) print i,arr[i]}' "${extract_log_path}" | sort -b -f -t"|" -k 2nr,2 | awk -F\| '{if($2>50) print}' > /tmp/total_counts_per_api.txt


[[ -f "${extract_log_path}" ]] && /bin/rm -f "${extract_log_path}"
ulimit -n "${file_descriptor_val}"

# Script End
```

## Explanation
通過`all api per seconds`這個維度說明代碼優化思路

舊代碼

```bash
awk -F\| 'BEGIN{OFS="|"}{"date --date=\""$(NF-1)"\" +\"%F %T\"" | getline a;arr[a]++}END{for (i in arr) print i,arr[i]}' "${extract_log_path}" | sort -b -f -t"|" -k 2r,2 > /tmp/total_counts_per_second.txt
```

新代碼

```bash
awk -F\| 'BEGIN{OFS="|"}{arr[$2]++}END{for (i in arr) print i,arr[i]}' "${extract_log_path}" | sort -b -f -t"|" -k 2nr,2 | awk -F\| '{if($2>50){"date --date=\""$1"\" +\"%F %T\"" | getline a;printf("%s|%s\n",a,$2)}}' > /tmp/total_counts_per_second.txt
```

最大的區別是將在`awk`中調用系統`date`命令進行日期格式化的操作放到的命令的最後。

在`awk`中調用系統`date`命令非常消耗CPU資源，且需臨時打開大量的文件描述符(file descriptor)，須臨時提高`ulimit -n`的閾值。

進行該操作是爲了將日期從`10 Dec 2017 23:59:57`格式化成`2017-12-10 23:59:57`，將格式化後的字符串作爲`awk`數據中的索引(index/key)。因`awk`是逐行讀取目標文件，故而每讀取一行都要先進行日期的格式化操作，增加了單行數據的操作時間。而這種操作需要進行數百萬次，直接導致運行效率極大降低。

優化後的命令是將該操作放置到最後執行，理由是未經格式化的日期字符串一樣可以作爲數據的索引，只需在輸出之前進行格式化操作。這樣做的好處是，需要格式化的數據行數從數百萬降到了一萬以內，甚至更少。整體運行效率極大提升。

但因爲仍舊是使用的是CPU的一個核心，如果能夠解決`parallel`和`awk`的組合使用問題，效率必將進一步提升。


## Data Output
以下是數據展示，每個維度只列出前20條。

### all api per seconds
上一週所有api每秒請求數 (共 **6145** 條)

request time|request count
---|---
2017-12-05 11:18:46|1349
2017-12-05 11:18:45|1308
2017-12-05 11:18:55|1307
2017-12-05 11:18:36|1305
2017-12-05 11:18:44|1301
2017-12-05 11:18:53|1300
2017-12-05 11:18:40|1294
2017-12-05 11:18:51|1293
2017-12-05 11:18:30|1281
2017-12-05 11:18:33|1277
2017-12-05 11:18:32|1275
2017-12-04 14:02:06|1269
2017-12-05 11:18:35|1269
2017-12-05 11:18:38|1267
2017-12-05 11:18:37|1263
2017-12-05 11:18:28|1257
2017-12-05 11:18:31|1255
2017-12-05 11:18:57|1252
2017-12-05 11:18:52|1250
2017-12-05 11:18:54|1243
2017-12-05 11:18:41|1238
2017-12-05 11:18:56|1236
2017-12-05 11:18:48|1232
2017-12-04 14:02:08|1228
2017-12-05 11:18:47|1222



### per api per seconds　
上一週每個api每秒請求數 (共 **15647** 條)

request time|api|request count
---|---|---
2017-12-05 11:18:46|/bususer/findRouteDetail.do|1348
2017-12-05 11:18:45|/bususer/findRouteDetail.do|1308
2017-12-05 11:18:36|/bususer/findRouteDetail.do|1305
2017-12-05 11:18:55|/bususer/findRouteDetail.do|1303
2017-12-05 11:18:44|/bususer/findRouteDetail.do|1300
2017-12-05 11:18:53|/bususer/findRouteDetail.do|1296
2017-12-05 11:18:40|/bususer/findRouteDetail.do|1294
2017-12-05 11:18:51|/bususer/findRouteDetail.do|1293
2017-12-05 11:18:30|/bususer/findRouteDetail.do|1277
2017-12-05 11:18:33|/bususer/findRouteDetail.do|1274
2017-12-05 11:18:32|/bususer/findRouteDetail.do|1271
2017-12-05 11:18:35|/bususer/findRouteDetail.do|1269
2017-12-05 11:18:38|/bususer/findRouteDetail.do|1267
2017-12-05 11:18:37|/bususer/findRouteDetail.do|1263
2017-12-05 11:18:57|/bususer/findRouteDetail.do|1250
2017-12-05 11:18:52|/bususer/findRouteDetail.do|1247
2017-12-05 11:18:28|/bususer/findRouteDetail.do|1245
2017-12-05 11:18:31|/bususer/findRouteDetail.do|1241
2017-12-05 11:18:54|/bususer/findRouteDetail.do|1240
2017-12-05 11:18:41|/bususer/findRouteDetail.do|1236
2017-12-05 11:18:48|/bususer/findRouteDetail.do|1232
2017-12-05 11:18:56|/bususer/findRouteDetail.do|1232
2017-12-05 11:18:47|/bususer/findRouteDetail.do|1219
2017-12-05 11:18:27|/bususer/findRouteDetail.do|1204
2017-12-05 11:18:34|/bususer/findRouteDetail.do|1195


### total counts for specific api
上一週各個api請求總數 (共 **15647** 條)

api|request count
---|---
/driver/uploadGPS.do|475306
/bususer/getRealTimeRouteInfo.do|249580
/bususer/getBusPosition.do|247360
/bususer/getUserTickets.do|245309
/serve/optServer.do|200337
/bususer/v2/getUserInfo.do|179476
/bususer/flybusLogin.do|175028
/bususer/fuzzyQueryStopInfo.do|153243
/bususer/findRouteList.do|117275
/bususer/getContentResources.do|92414
/bususer/findRouteDetail.do|90649
/bususer/getBanner.do|69184
/bususer/myCouponList.do|49917
/bususer/findOpenDays.do|41468
/bususer/checkTicketWithPosition.do|41320
/bususer/saveToken.do|40905
/bususer/search.do|30504
/buspay/queryPayResult.do|27401
/bususer/activityNumsCheck.do|24236
/bususer/getTicektsByDate.do|24138
/bususer/routeWeather.do|21518
/bususer/getCompanyRoute.do|15481
/bususer/buyTicket.do|15359
/buspay/buildPayParams.do|13832
/bususer/getTicketDetilByShift.do|13255


## Change Logs
* 2017.12.12 19:59 Tue Asia/Shanghai
    * 初稿完成
* 2018.04.11 10:21 Wed America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
