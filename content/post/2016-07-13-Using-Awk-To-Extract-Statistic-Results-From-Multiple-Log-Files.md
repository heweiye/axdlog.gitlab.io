---
title: Using Awk To Extract Statistic Results From Multiple Log Files
slug: Using Awk To Extract Statistic Results From Multiple Log Files
date: 2016-07-13T12:36:37+08:00
lastmod: 2018-04-12T10:02:42-04:00
draft: false
keywords: ["awk"]
description: "Using GNU Awk to extract statistic results from multiple log files"
categories:
- Production Case
tags:
- awk

comment: true
toc: true

---

工作中遇到一個需求：從日誌文件中提取符合指定條件的數據，獲取其出現頻次(條件有多種、日誌有多個)。存有日誌的服務器是線上運行的服務器，直接在其上進行操作會對服務器造成較大壓力。故需對日誌進行轉移，而其又是內網服務器，只能通過跳板機登錄，故將相關日誌複製到跳板機中再進行操作。

個人研究後，使用`awk`命令解決，以下是操作過程和撰寫的Shell腳本。

<!--more-->

## Request
統計以`all.log`開頭的日誌文件中含有`search.do`，並符合以下幾種條件的數據的出現頻次

1. 同時含有`startLat`、`endLat`；
2. 只含有`startLat`；
3. 只含有`endLat`；

## Procedure
以下是操作過程

### Compress
使用`tar jcvf`將符合條件的日誌壓縮爲`tar.xz`格式。

```bash
[root@deerbus /project/flybus-interface/logs]
#ls -lh all.log*
-rw-r--r-- 1 root root 215M Jul 13 10:26 all.log
-rw-r--r-- 1 root root 271M Jun 23 23:59 all.log.2016-06-23
-rw-r--r-- 1 root root 423M Jun 24 23:59 all.log.2016-06-24
-rw-r--r-- 1 root root  68M Jun 25 23:59 all.log.2016-06-25
-rw-r--r-- 1 root root 134M Jun 26 23:59 all.log.2016-06-26
-rw-r--r-- 1 root root 494M Jun 27 23:59 all.log.2016-06-27
-rw-r--r-- 1 root root 472M Jun 28 23:59 all.log.2016-06-28
-rw-r--r-- 1 root root 495M Jun 29 23:59 all.log.2016-06-29
-rw-r--r-- 1 root root 495M Jun 30 23:59 all.log.2016-06-30
-rw-r--r-- 1 root root 453M Jul  1 23:59 all.log.2016-07-01
-rw-r--r-- 1 root root  71M Jul  2 23:59 all.log.2016-07-02
-rw-r--r-- 1 root root 127M Jul  3 23:59 all.log.2016-07-03
-rw-r--r-- 1 root root 467M Jul  4 23:59 all.log.2016-07-04
-rw-r--r-- 1 root root 501M Jul  5 23:59 all.log.2016-07-05
-rw-r--r-- 1 root root 472M Jul  6 23:59 all.log.2016-07-06
-rw-r--r-- 1 root root 478M Jul  7 23:59 all.log.2016-07-07
-rw-r--r-- 1 root root 456M Jul  8 23:59 all.log.2016-07-08
-rw-r--r-- 1 root root  69M Jul  9 23:59 all.log.2016-07-09
-rw-r--r-- 1 root root 137M Jul 10 23:59 all.log.2016-07-10
-rw-r--r-- 1 root root 543M Jul 11 23:59 all.log.2016-07-11
-rw-r--r-- 1 root root 518M Jul 12 23:59 all.log.2016-07-12

[root@deerbus /project/flybus-interface/logs]
#tar jcvf flybus-interface-log.tar.xz all.log*
all.log
tar: all.log: file changed as we read it
all.log.2016-06-23
all.log.2016-06-24
all.log.2016-06-25
all.log.2016-06-26
all.log.2016-06-27
all.log.2016-06-28
all.log.2016-06-29
all.log.2016-06-30
all.log.2016-07-01
all.log.2016-07-02
all.log.2016-07-03
all.log.2016-07-04
all.log.2016-07-05
all.log.2016-07-06
all.log.2016-07-07
all.log.2016-07-08
all.log.2016-07-09
all.log.2016-07-10
all.log.2016-07-11
all.log.2016-07-12

[root@deerbus /project/flybus-interface/logs]
#ls -lh flybus-interface-log.tar.xz
-rw-r--r-- 1 root root 582M Jul 13 10:57 flybus-interface-log.tar.xz

[root@deerbus /project/flybus-interface/logs]
#pwd
/project/flybus-interface/logs

[root@deerbus /project/flybus-interface/logs]
#exit
logout
Connection to deerbus closed.
```

### Transfer
在跳板機`front`上執行如下命令(192.168.92.37爲主機deerbus的ip)，將目標文件同步到跳板機中

```bash
rsync -avzP 192.168.92.37:/project/flybus-interface/logs/flybus-interface-log.tar.xz /tmp
```

操作過程

```bash
[root@front /tmp]
#rsync -avzP 192.168.92.37:/project/flybus-interface/logs/flybus-interface-log.tar.xz /tmp
receiving incremental file list
flybus-interface-log.tar.xz
   609871232 100%   18.74MB/s    0:00:31 (xfer#1, to-check=0/1)

sent 30 bytes  received 608050348 bytes  19303186.60 bytes/sec
total size is 609871232  speedup is 1.00

[root@front /tmp]
#ls -lh flybus-interface-log.tar.xz
-rw-r--r-- 1 root root 582M Jul 13 10:57 flybus-interface-log.tar.xz

[root@front /tmp]
#
```

### Decompress
在跳板機中對目標文件進行解壓操作，使用`tar jxvf`解壓，通過參數`-C`指定解壓後文件存儲路徑

```bash
[root@front /tmp]
#tar jxvf flybus-interface-log.tar.xz -C flybus-interface-log
all.log
all.log.2016-06-23
all.log.2016-06-24
all.log.2016-06-25
all.log.2016-06-26
all.log.2016-06-27
all.log.2016-06-28
all.log.2016-06-29
all.log.2016-06-30
all.log.2016-07-01
all.log.2016-07-02
all.log.2016-07-03
all.log.2016-07-04
all.log.2016-07-05
all.log.2016-07-06
all.log.2016-07-07
all.log.2016-07-08
all.log.2016-07-09
all.log.2016-07-10
all.log.2016-07-11
all.log.2016-07-12

[root@front /tmp]
#ls -lh flybus-interface-log
total 7.2G
-rw-r--r-- 1 root root 215M Jul 13 10:29 all.log
-rw-r--r-- 1 root root 271M Jun 23 23:59 all.log.2016-06-23
-rw-r--r-- 1 root root 423M Jun 24 23:59 all.log.2016-06-24
-rw-r--r-- 1 root root  68M Jun 25 23:59 all.log.2016-06-25
-rw-r--r-- 1 root root 134M Jun 26 23:59 all.log.2016-06-26
-rw-r--r-- 1 root root 494M Jun 27 23:59 all.log.2016-06-27
-rw-r--r-- 1 root root 472M Jun 28 23:59 all.log.2016-06-28
-rw-r--r-- 1 root root 495M Jun 29 23:59 all.log.2016-06-29
-rw-r--r-- 1 root root 495M Jun 30 23:59 all.log.2016-06-30
-rw-r--r-- 1 root root 453M Jul  1 23:59 all.log.2016-07-01
-rw-r--r-- 1 root root  71M Jul  2 23:59 all.log.2016-07-02
-rw-r--r-- 1 root root 127M Jul  3 23:59 all.log.2016-07-03
-rw-r--r-- 1 root root 467M Jul  4 23:59 all.log.2016-07-04
-rw-r--r-- 1 root root 501M Jul  5 23:59 all.log.2016-07-05
-rw-r--r-- 1 root root 472M Jul  6 23:59 all.log.2016-07-06
-rw-r--r-- 1 root root 478M Jul  7 23:59 all.log.2016-07-07
-rw-r--r-- 1 root root 456M Jul  8 23:59 all.log.2016-07-08
-rw-r--r-- 1 root root  69M Jul  9 23:59 all.log.2016-07-09
-rw-r--r-- 1 root root 137M Jul 10 23:59 all.log.2016-07-10
-rw-r--r-- 1 root root 543M Jul 11 23:59 all.log.2016-07-11
-rw-r--r-- 1 root root 518M Jul 12 23:59 all.log.2016-07-12

[root@front /tmp]
#
```

### Execution
數據提取過程

#### Via script

腳本執行過程，腳本內容在本文下方

```bash
[root@front /tmp]
#bash loganalyze.sh
date,totalCount,startOnlyCount,endOnlyCount
2016-07-13,401,96,6
2016-06-23,809,188,10
2016-06-24,1059,225,19
2016-06-25,291,74,11
2016-06-26,749,136,2
2016-06-27,1164,224,14
2016-06-28,1157,214,14
2016-06-29,1263,196,14
2016-06-30,1316,176,5
2016-07-01,868,154,5
2016-07-02,322,87,13
2016-07-03,749,153,17
2016-07-04,1109,179,27
2016-07-05,1170,238,24
2016-07-06,1132,191,20
2016-07-07,1147,209,29
2016-07-08,964,153,5
2016-07-09,299,56,7
2016-07-10,703,131,10
2016-07-11,1415,231,26
2016-07-12,1306,248,23

[root@front /tmp]
#
```

#### Via tail
使用`tail`動態查看

```bash
[root@front /tmp]
#tail -F flybus-interface-log.txt
tail: cannot open 'flybus-interface-log.txt' for reading: No such file or directory
tail: 'flybus-interface-log.txt' has become accessible
date,2016-07-13,total,401,startOnly,96,endOnly,6
date,2016-06-23,total,809,startOnly,188,endOnly,10
date,2016-06-24,total,1059,startOnly,225,endOnly,19
date,2016-06-25,total,291,startOnly,74,endOnly,11
date,2016-06-26,total,749,startOnly,136,endOnly,2
date,2016-06-27,total,1164,startOnly,224,endOnly,14
date,2016-06-28,total,1157,startOnly,214,endOnly,14
date,2016-06-29,total,1263,startOnly,196,endOnly,14
date,2016-06-30,total,1316,startOnly,176,endOnly,5
date,2016-07-01,total,868,startOnly,154,endOnly,5
date,2016-07-02,total,322,startOnly,87,endOnly,13
date,2016-07-03,total,749,startOnly,153,endOnly,17
date,2016-07-04,total,1109,startOnly,179,endOnly,27
date,2016-07-05,total,1170,startOnly,238,endOnly,24
date,2016-07-06,total,1132,startOnly,191,endOnly,20
date,2016-07-07,total,1147,startOnly,209,endOnly,29
date,2016-07-08,total,964,startOnly,153,endOnly,5
date,2016-07-09,total,299,startOnly,56,endOnly,7
date,2016-07-10,total,703,startOnly,131,endOnly,10
date,2016-07-11,total,1415,startOnly,231,endOnly,26
date,2016-07-12,total,1306,startOnly,248,endOnly,23
^C

[root@front /tmp]
#
```


## Scripts
此爲本人撰寫的腳本，同時實現數據的提取、處理、顯示，輸出結果可直接複製入Excel文件中

```bash
#!/usr/bin/env bash
#Usage: 从日志文件all.log*中提取含有'search.do'，并统计以下几种情形出现次数
#1. 同时含有'startLat', `endLat`
#2. 只含有'startLat'
#3. 只含有'endLat'

#Author:
#E-mail:
#Date: 2016.07.13 12:51 Wed

#日志存放路径
sourceDir='/tmp/flybus-interface-log'

#计算结果存储文件路径
targetFile='/tmp/flybus-interface-log.txt'

if [[ -d "${sourceDir}" ]]; then
    cd ${sourceDir}
else
    echo '日志存放路径不存在！'
    exit
fi

#运算过程
for i in `ls all.log*`;do
    #通过日志名称获取对应日期
    if [[ "$i" = 'all.log' ]]; then
        date=`date +%F`
    else
        date=${i##*.}
    fi

    #使用awk提取相关数据并存储到指定文件中
    awk -v varDate=${date} '$0~/search.do/{
        if($0~/"startLat"/&&$0~/"endLat"/){total++}
        else if($0~/"startLat"/&&$0!~/"endLat"/){start++}
        else if($0!~/"startLat"/&&$0~/"endLat"/){end++}
    }END{
        printf("date,%s,total,%d,startOnly,%d,endOnly,%d\n",varDate,total,start,end)
    }' $i >> ${targetFile}
done

#使用awk对运算后的结果进行处理展示
awk -v FS=',' 'BEGIN{printf("date,totalCount,startOnlyCount,endOnlyCount\n")}{printf("%s,%d,%d,%d\n",$2,$4,$6,$8)}' ${targetFile}
```


## Results
以下是輸出的最終結果

date|totalCount|startOnlyCount|endOnlyCount
---|:---:|:---:|:---:
2016-06-23|809|188|10
2016-06-24|1059|225|19
2016-06-25|291|74|11
2016-06-26|749|136|2
2016-06-27|1164|224|14
2016-06-28|1157|214|14
2016-06-29|1263|196|14
2016-06-30|1316|176|5
2016-07-01|868|154|5
2016-07-02|322|87|13
2016-07-03|749|153|17
2016-07-04|1109|179|27
2016-07-05|1170|238|24
2016-07-06|1132|191|20
2016-07-07|1147|209|29
2016-07-08|964|153|5
2016-07-09|299|56|7
2016-07-10|703|131|10
2016-07-11|1415|231|26
2016-07-12|1306|248|23
2016-07-13|401|96|6


## Change Logs
* 2016.07.13 20:38 Wed Asia/Shanghai
    * 初稿完成
* 2018.04.12 10:02 Thu America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
