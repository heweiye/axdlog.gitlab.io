---
title: Extracting API Requests Info From Nginx Access Log
slug: Extracting API Requests Info From Nginx Access Log
date: 2017-07-03T11:43:21+08:00
lastmod: 2018-04-11T09:59:08-04:00
draft: false
keywords: ["AxdLog", "awk", "sed", "Nginx"]
description: "Using sed and awk to extract API requests info from Nginx log"
categories:
- Production Case
tags:
- awk
- sed
comment: true
toc: true
---

運維部接到研發經理的需求，從Nginx日誌中提取某一天API接口調用信息，並根據指定的維度進行彙總。因日誌文件在線上生產環境中，且單個文件大小達數GB，爲了減少操作對生產環境的穩定性造成影響。將初步提取的數據下載到本地環境，進行進一步的彙總分析。

以前是將提取的數據導入MySQL數據庫，通過SQL進行提取分析，雖方便但數據導入過程過於漫長。本文
操作主要使用`sed`、`awk`命令，並大量使用了一些高級特性，如`sed`的添加行號、[字符类][character_class]、正则、[後向引用][backreference]；`awk`的getline、[預定義數組掃描排序][controlling_scanning]、[多維數組][multidimensional]等，算是後積薄發的一次展現。

<!--more-->

**注意**：本文只記錄需求實現過程，對於關鍵文件，只顯示文件名，不列出路徑。

## Requirements
### Target
提取[飛路巴士][deerbus]於2017年06月26日的API接口調用數據，具體需求爲：

1. 每秒請求數量最多的Top10時間段及數量，精確到秒；
2. 每秒請求頻次最高的接口及數量Top10；
3. 執行最慢的SQL及耗時、發生時間；

### Data Source
具體的API域名爲

* api.deerbus.com
* api.feilu66.com

通過域名在Nginx配置文件中提取到[飛路巴士][deerbus]相關服務對應的訪問日誌，具體信息如下

API Domain|Path|Size
---|---|---
**api.deerbus.com** |`api.deerbus.com_access.log`|`5.5G`
**api.feilu66.com** |`api.feilu66.com_access.log`|`1.7G`


### Nginx Log Format
指令`log_format`在Nginx配置文件中的定義如下

```
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"
```

**注意**：當某個指定的值爲空時，用字符`-`表示其內容。

提取日誌`api.deerbus.com_access.log`中的一條數據用於分析

>223.104.213.12 - - [30/Jun/2017:13:27:56 +0800] "POST /bususer/nearbyRoute.do HTTP/1.1" 200 2703 "-" "%E9%A3%9E%E8%B7%AF%E5%B7%B4%E5%A3%AB/3.6.0 CFNetwork/758.0.2 Darwin/15.0.0"

分析如下

parameter|value
---|---
$remote_addr|`223.104.213.12`
$remote_user|`-`
[$time_local]|`[30/Jun/2017:13:27:56 +0800]`
$request|`"POST /bususer/nearbyRoute.do HTTP/1.1"`
$status|`200`
$body_bytes_sent|`2703`
$http_referer|`-`
$http_user_agent|`"%E9%A3%9E%E8%B7%AF%E5%B7%B4%E5%A3%AB/3.6.0 CFNetwork/758.0.2 Darwin/15.0.0"`
$http_x_forwarded_for|


需要使用到的指令有`$remote_addr`、`[$time_local]`、`$request`，此處僅以`$remote_addr`、`[$time_local]`二者確定一次API接口的訪問請求。

初步提取出的數據格式爲`remote_addr|time_local|api`

>223.104.213.12|30 Jun 2017:13:27:56|/bususer/nearbyRoute.do

#### Outlier Data
Nginx日誌文件中存在格式異常的數據，如

>223.104.212.38 - - [26/Jun/2017:223.104.212.76 - - [26/Jun/2017:07:25:18 +0800] "POST /serve/driver/getInfo.do HTTP/1.1" 200 761 "-" "okhttp/3.1.2"

數據處理時，須考慮對異常數據對操作命令的影響。


## Data Extraction
將目標日期當天所有符合條件的API接口調用數據提取出來，存儲到臨時文件中作進一步分析用。此處定義相關臨時文件路徑：

api domain|temp file
---|---
**api.deerbus.com** |`/tmp/api_deerbus.txt`
**api.feilu66.com** |`/tmp/api_feilu66.txt`

### Command
提取命令如下

```bash
# api.deerbus.com
sed -r -n '/\[26\/Jun\/2017:/{s@-[[:space:]]@@g;s@[[:space:]]*\+[[:digit:]]+@@g;s@\"[[:upper:]]+ ([^[:space:]]*) .*@\1@g;s@(\[|\])@@g;s@ @|@g;s@([[:digit:]]{4}):@\1 @g;s@([[:digit:]]{1,2})/([[:alpha:]]+)/([[:digit:]]{4})@\1 \2 \3@g;p}' /PATH/api.deerbus.com_access.log > /tmp/api_deerbus.txt
```

通過`sed`的[字符類][character_class]、[後向引用][backreference]、正則表達式的配合使用實現。

`api.feilu66.com`的日誌提取同上。

### Data Format
提取出的數據格式如下

```bash
#head -n 2 /tmp/api_deerbus.txt
101.90.253.174|26 Jun 2017 00:00:00|/bususer/search.do
180.159.44.125|26 Jun 2017 00:00:00|/bususer/search.do

#head -n 2 /tmp/api_feilu66.txt
58.33.50.235|26 Jun 2017 00:00:08|/serve/driver/uploadGPS.do
183.193.190.67|26 Jun 2017 00:00:09|/serve/driver/uploadGPS.do
```

提取操作相關信息

temp file|lines|size|time consuming
---|---|---|---
**/tmp/api_deerbus.txt** |392,712|23M|53.983s
**/tmp/api_feilu66.txt** |39,100|2.6M|14.411s


## Analyzing
**注意**：

1. 此處的分析不作 `去重`處理；
2. 爲避免過度佔用線上主機的CPU資源，將文件下載至個人工作電腦(`ThinkPad E450`)中進行操作；

### Requests Per Second
按時間(精度爲秒)，每一秒的API接口請求數

```bash
# api.deerbus.com
awk -F\| 'BEGIN{OFS="|"}{"date --date=\""$(NF-1)"\" +\"%F %T\"" | getline a;arr[a]++}END{PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) print i,arr[i] }' api_deerbus.txt > /tmp/deerbus_total_counts_per_second.txt
```

通過`awk`實現，此處使用`$(NF-1)`是爲了消除異常數據對操作的影響。

`api.feilu66.com`的日誌提取同上。


提取操作相關信息

temp file|lines|time consuming
---|---|---
/tmp/deerbus_total_counts_per_second.txt|52,302|12m31.617s
/tmp/feilu66_total_counts_per_second.txt|25,364|3m48.103s


### Requests Per Second For Specific API
按具體接口每秒被調用的次數，接口名和請求時間二者確定一次請求。

```bash
# api.deerbus.com
awk -F\| 'BEGIN{OFS="|"}{"date --date=\""$(NF-1)"\" +\"%F %T\"" | getline a;arr[$NF,a]++}END{PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) {split(i,j,SUBSEP);print j[2],j[1],arr[i]}}' api_deerbus.txt > /tmp/deerbus_specific_counts_per_second.txt
```

通過`awk`的[多維數組][multidimensional]實現2個維度確定一條數據。

`api.feilu66.com`的日誌提取同上。


提取操作相關信息

temp file|lines|time consuming
---|---|---
/tmp/deerbus_specific_counts_per_second.txt|210,889|11m30.479s
/tmp/feilu66_specific_counts_per_second.txt|34,705|3m26.213s


## Data Output
最終數據展示，此處提取前Top20。其中的排列需要通過`sed = file | sed 'N;s/\n/|/g'`實現。

### Requests Per Second

#### deerbus
No.|time|requests
---|---|---
1|08:00:02|2854
2|07:25:24|108
3|07:25:17|105
4|07:25:23|103
5|07:25:22|97
6|07:25:25|92
7|07:25:18|87
8|07:25:19|86
9|07:25:29|85
10|07:25:26|82
11|07:25:27|82
12|07:25:38|82
13|07:25:28|80
14|07:25:30|79
15|07:25:33|78
16|07:25:37|78
17|07:25:20|77
18|07:25:21|77
19|07:25:41|77
20|07:25:31|72


#### feilu66
No.|time|requests
---|---|---
1|19:30:02|49
2|13:10:01|27
3|12:00:00|25
4|12:00:01|25
5|13:10:00|23
6|19:30:03|22
7|07:00:19|19
8|07:56:46|16
9|09:29:18|16
10|15:35:43|16
11|15:34:25|16
12|07:56:56|15
13|15:35:54|15
14|14:16:23|14
15|11:07:38|14
16|07:59:00|14
17|07:00:20|14
18|15:32:34|14
19|10:19:13|14
20|10:21:56|13


### Requests Per Second For Specific API
#### deerbus

No.|time|api|requests
---|---|---|---
1|08:00:02|/driver/uploadGPS.do|864
2|08:00:02|/bususer/getRealTimeRouteInfo.do|297
3|08:00:02|/bususer/getBusPosition.do|296
4|08:00:02|/bususer/getUserTickets.do|202
5|08:00:02|/serve/optServer.do|185
6|08:00:02|/bususer/checkTicketWithPosition.do|133
7|08:00:02|/bususer/getUserInfo.do|129
8|08:00:02|/bususer/flybusLogin.do|118
9|08:00:02|/bususer/search.do|98
10|08:00:02|/bususer/findRouteList.do|84
11|08:00:02|/bususer/getContentResources.do|76
12|08:00:02|/bususer/getBanner.do|65
13|08:00:02|/bususer/saveToken.do|51
14|08:00:02|/bususer/myCouponList.do|45
15|08:00:02|/bususer/findOpenDays.do|37
16|14:38:17|/bususer/fuzzyQueryStopInfo.do|32
17|07:25:17|/bususer/getUserTickets.do|29
18|07:25:18|/bususer/getUserTickets.do|28
19|08:00:02|/bususer/findRouteDetail.do|24
20|07:25:23|/serve/optServer.do|22


#### deerbus

No.|time|api|requests
---|---|---|---
1|19:30:02|/serve/getNewsBriefList.do|42
2|13:10:01|/serve/getNewsBriefList.do|25
3|12:00:00|/serve/getNewsBriefList.do|22
4|12:00:01|/serve/getNewsBriefList.do|22
5|19:30:03|/serve/getNewsBriefList.do|21
6|13:10:00|/serve/getNewsBriefList.do|20
7|07:00:19|/serve/getNewsBriefList.do|18
8|09:29:18|/vehchile/getCarNum.do|16
9|15:34:25|/vehchile/getCarNum.do|16
10|11:07:38|/vehchile/getCarNum.do|14
11|15:35:43|/vehchile/getCarNum.do|14
12|15:35:54|/vehchile/getCarNum.do|14
13|15:32:34|/vehchile/getCarNum.do|14
14|10:19:13|/vehchile/getCarNum.do|14
15|14:16:23|/vehchile/getCarNum.do|14
16|15:32:24|/vehchile/getCarNum.do|13
17|10:19:03|/vehchile/getCarNum.do|13
18|10:21:56|/vehchile/getCarNum.do|13
19|07:00:20|/serve/getNewsBriefList.do|13
20|12:09:22|/vehchile/getCarNum.do|13


## SQL Slow Query
SQL慢查詢日誌來自生成數據庫，因涉及到數據庫表字段，此處不列出具體的SQL語句，只顯示執行時間。

通過關鍵詞`Time: 170626`提取。

```sql
# Time: 170626  7:53:47
# Query_time: 4.589281  Lock_time: 0.000688 Rows_sent: 90  Rows_examined: 5456950

# Time: 170626  7:53:50
# Query_time: 4.300753  Lock_time: 0.000414 Rows_sent: 90  Rows_examined: 5456952

# Time: 170626  7:53:51
# Query_time: 4.318564  Lock_time: 0.000609 Rows_sent: 90  Rows_examined: 5456952

# Time: 170626  9:09:06
# Query_time: 3.083190  Lock_time: 0.000229 Rows_sent: 1  Rows_examined: 1342780

# Time: 170626  9:09:09
# Query_time: 3.300643  Lock_time: 0.000252 Rows_sent: 18  Rows_examined: 1337464

# Query_time: 3.300645  Lock_time: 0.000252 Rows_sent: 18  Rows_examined: 1337464
SET timestamp=1498439349;

# Query_time: 3.303412  Lock_time: 0.000165 Rows_sent: 30  Rows_examined: 1344612
SET timestamp=1498439349;

# Time: 170626  9:10:02
# Query_time: 4.495894  Lock_time: 0.000265 Rows_sent: 1  Rows_examined: 2702204

# Time: 170626  9:10:06
# Query_time: 4.442528  Lock_time: 0.000155 Rows_sent: 30  Rows_examined: 2702360

# Time: 170626 15:49:39
# Query_time: 3.037750  Lock_time: 0.000758 Rows_sent: 1  Rows_examined: 816958

# Time: 170626 17:57:06
# Query_time: 3.962839  Lock_time: 0.000207 Rows_sent: 1  Rows_examined: 1825767

# Time: 170626 22:29:46
# Query_time: 3.916867  Lock_time: 0.000170 Rows_sent: 1  Rows_examined: 1827446
```

雖然兼任公司的DBA，但手中擁有的權限少得可憐。在面對開發人員的SQL語句時，有一種無力感。

## Change Logs
* 2017.07.03 11:42 Mon Asia/Shanghai
    * 初稿完成
* 2018.04.11 09:59 Wed America/Boston
    * 勘誤，遷移到新Blog


[deerbus]:http://www.deerbus.com "飛路巴士"
[character_class]:https://www.gnu.org/software/sed/manual/html_node/Character-Classes-and-Bracket-Expressions.html "Character Class"
[backreference]:https://www.gnu.org/software/sed/manual/html_node/Back_002dreferences-and-Subexpressions.html "Back-references and Subexpressions"
[multidimensional]:https://www.gnu.org/software/gawk/manual/html_node/Multidimensional.html "Multidimensional Arrays"
[controlling_scanning]:https://www.gnu.org/software/gawk/manual/html_node/Controlling-Scanning.html "Controlling Array Traversal"

<!-- End -->
