---
title: Analying The Change Frequency Of Active Users Tokens From Tomcat Logs
slug: Analying The Change Frequency Of Active Users Tokens From Tomcat Logs
date: 2016-12-05T07:43:08+08:00
lastmod: 2018-04-11T09:37:08-04:00
draft: false
keywords: ["awk", "sed"]
description: "Using sed and awk to analyze the change frequency of active users' token from Tomcat log"
categories:
- Production Case
tags:
- awk
- sed
- Shell Script
- MySQL
comment: true
toc: true
---

接到Java開發人員的需求,從Tomcat日誌中提取最近一個月查詢、購票活躍度最高的5位iOS用戶的token變更頻率。日誌文件在線上生產服務器中，單個文件高達十數GB，無法使用`vim`打開，只能下載到本機後再進行數據提取操作，以下是完整的記錄。

<!--more-->

## Requirements
需求整理如下

### Target
提取最近一個月(2016.11.01~30, 2016.12.01)查詢、購票活躍度最高的5位iOS用戶的token變更頻率

### Data Source
#### Active Users' Info
活躍用戶信息查詢，SQL語句由Java開發人員提供 (SQL語句似乎有問題,沒有判斷是否屬於iOS用戶)

```sql
select a.* from t_bus_member a join (select member_id,count(1) sum from t_bus_ticket where create_time >= '2016-11-1' and create_time <'2016-12-1' and ticket_status = 2 group by member_id order by sum desc limit 5) b on a.id = b.member_id\G
```

其中含有需要的設備ID (t_bus_member.mac_id)

#### Data Source File
日誌文件

```
/PATH1/flybus-provider/logs/all.log*
/PATH2/tomcat-flybus-provider/logs/catalina.out
```

### Filter Parameters
日誌在線上生產服務器中，日誌格式

```txt
flybus-provider [2016-12-01 00:07:39] 1480408150655-2091 com.raxtone.utils.spring.logger.LoggerAspect:48
INFO : (LoggerAspect.java:48)    请求方法：void com.raxtone.flybus.rpc.rmi.user.IPushTSRmiService.sendTokenToSerer(TokenParam,String,Integer,Integer)，请求参数：[{"cityId":0,"token":"80275d72bebcbf9c0d9d5c4f62c7b15e90c6795c5a4a58af79f4f57f47a7590f"},"cb0eeb6990335140b8f65fb3fc9a3f522de48b35",11,100]
```

按如下要求在日誌文件中查詢匹配數據

1. 含有關鍵詞 **`sendTokenToSerer`**；
2. 同時含有設備ID，示例中的`cb0eeb6990335140b8f65fb3fc9a3f522de48b35`；

提取其中的日期，設備ID，token值，在示例中分別是

item|details
---|---
date|`2016-12-01 00:07:39`
mac_id|`cb0eeb6990335140b8f65fb3fc9a3f522de48b35`
token|`80275d72bebcbf9c0d9d5c4f62c7b15e90c6795c5a4a58af79f4f57f47a7590f`


### Output Format
結果輸出格式 `時間|設備編號|token`

>2016-12-01 00:07:39|cb0eeb6990335140b8f65fb3fc9a3f522de48b35|80275d72bebcbf9c0d9d5c4f62c7b15e90c6795c5a4a58af79f4f57f47a7590f`


## Analysis

### Step1 Active Users' Info
查詢、購票活躍度最高的5爲iOS用戶的設備ID由開發人員提供的SQL語句獲取。

### Step2 Download Log Files
原始日誌文件總大小達16GB，在生產服務器上進行數據提取操作會對線上業務造成影響；通過`sftp`將相關日誌拉取到本地進行處理，保存路徑爲`/tmp/flybusProvider`。

```bash
[flying@lemp flybusProvider]$ ls -lh
total 16G
-rw-r--r-- 1 flying flying 100M Dec  2 16:34 all.log
-rw-r--r-- 1 flying flying 105M Dec  2 16:34 all.log.2016-11-29
-rw-r--r-- 1 flying flying 211M Dec  2 16:34 all.log.2016-11-30
-rw-r--r-- 1 flying flying 208M Dec  2 16:35 all.log.2016-12-01
-rw-r--r-- 1 flying flying  16G Dec  2 16:33 catalina.out
[flying@lemp flybusProvider]$ pwd
/tmp/flybusProvider
[flying@lemp flybusProvider]$ ls -lh
total 16G
-rw-r--r-- 1 flying flying 100M Dec  2 16:34 all.log
-rw-r--r-- 1 flying flying 105M Dec  2 16:34 all.log.2016-11-29
-rw-r--r-- 1 flying flying 211M Dec  2 16:34 all.log.2016-11-30
-rw-r--r-- 1 flying flying 208M Dec  2 16:35 all.log.2016-12-01
-rw-r--r-- 1 flying flying  16G Dec  2 16:33 catalina.out
[flying@lemp flybusProvider]$ du -sh
16G	.
[flying@lemp flybusProvider]$
```

### Step3 Merge Line
日誌中請求日期和請求體在相鄰的兩行中，須將其合併爲一行，只對指定時間範圍內的數據進行操作，使用命令`awk`及其中的`getline`方法實現；


原始數據格式

```txt
flybus-provider [2016-11-30 00:45:32] 1480408150487-2087 com.raxtone.utils.spring.logger.LoggerAspect:48
INFO : (LoggerAspect.java:48)    请求方法：void com.raxtone.flybus.rpc.rmi.user.IPushTSRmiService.sendTokenToServer(TokenParam,String,Integer,Integer)，请求参数
：[{"cityId":0,"token":"843edf6bfd3df2b448f58b762a2e6c06cfae05b0c44ac088edce81dd50c5d4bc"},"bed8a36e77253e1e16972e73b2c8743def6356e7",11,100]
flybus-provider [2016-11-30 00:45:32] 1480408150487-2087 PushTSLogger:119
```

期望的數據格式(將兩行合併成一行)
```
flybus-provider [2016-11-30 00:45:32] 1480408150487-2087 com.raxtone.utils.spring.logger.LoggerAspect:48 INFO : (LoggerAspect.java:48)    请求方法：void com.raxtone.flybus.rpc.rmi.user.IPushTSRmiService.sendTokenToServer(TokenParam,String,Integer,Integer)，请求参数：[{"cityId":0,"token":"843edf6bfd3df2b448f58b762a2e6c06cfae05b0c44ac088edce81dd50c5d4bc"},"bed8a36e77253e1e16972e73b2c8743def6356e7",11,100]
```

### Step4 Filtering Via Keywords
通過關鍵詞`sendTokenToServer`和`token`過濾，提取含有該2個關鍵詞的行，通過命令`awk`實現。

#### Extract Specific Data
1. `token`字段通過命令`sed`提取
2. `設備ID`、`日期`字段通過命令`awk`提取

將提取的數據寫入新文件中

### Step5 Load Data Into MySQL DBMS
創建數據表，將提取的數據寫入數據庫。


## Operation Precedures
### Active Users' Info
從生產數據庫主庫查詢

```sql
17:56:15 flybus> select a.id,client_id,a.mac_id from t_bus_member a join (select member_id,count(1) sum from t_bus_ticket where create_time >= '2016-11-1' and create_time <'2016-12-1' and ticket_status = 2 group by member_id order by sum desc limit 5) b on a.id = b.member_id\G
*************************** 1. row ***************************
             id: 32177
      client_id: 171
         mac_id: a58c41716de3c98b0b9b6a44f03442c803d61d78
*************************** 2. row ***************************
             id: 37081
      client_id: 171
         mac_id: 96262d20d58519e2ea6ea87f8cb0a5f7653fcc40
*************************** 3. row ***************************
             id: 26003
      client_id: 171
         mac_id: e444928b471124ce31609f992ac0853cbf2a0b8c
*************************** 4. row ***************************
             id: 37323
      client_id: 171
         mac_id: 88befa42adb6745cc9c19e18d689f0913290c4d2
*************************** 5. row ***************************
             id: 36479
      client_id: 169
         mac_id: 12a352b35e513dc5f341ec35aad12ff
5 rows in set (0.95 sec)

17:56:25 flybus>
```

提取到的数据(设备ID)为

```
a58c41716de3c98b0b9b6a44f03442c803d61d78
96262d20d58519e2ea6ea87f8cb0a5f7653fcc40
e444928b471124ce31609f992ac0853cbf2a0b8c
88befa42adb6745cc9c19e18d689f0913290c4d2
12a352b35e513dc5f341ec35aad12ff
```

### Download Source Log Files
通過創建SSH Tunnel穿透 **跳板主機** 直連內網主機，利用命令`sftp`將相關日誌文件下載到本地。

### Merge Two Lines Into One Line
將指定時間範圍內的數據行，兩行合併一行，其行首分別是`flybus-provider`、`INFO`。

具體命令爲

```bash
awk '$1~/flybus-provider/&&($2~/2016-11-[[:digit:]]{2}/||$2~/2016-12-01/){printf $0;getline;print}' /tmp/flybusProvider/* > /tmp/mergeline.txt
```

提取出的數據文件大小

```bash
[flying@lemp ~]$ ls -lh /tmp/mergeline.txt
-rw-rw-r-- 1 flying flying 7.7G Dec  2 17:40 /tmp/mergeline.txt
# 數據行數
[flying@lemp ~]$ awk 'END{print NR}' /tmp/mergeline.txt
9257670
[flying@lemp ~]$
```

### Filter Data Via Keywords
關鍵詞爲`sendTokenToServer`和`token`，提取同時含有的數據行

具體命令爲

```bash
awk '$0~/sendTokenToServer/&&$0~/"token"/{printf("%s\n",$0)}' /tmp/mergeline.txt > /tmp/haskeywords.txt
```

提取出的數據文件大小

```bash
[flying@lemp ~]$ ls -lh /tmp/haskeywords.txt
-rw-rw-r-- 1 flying flying 9.7M Dec  5 11:54 /tmp/haskeywords.txt
# 數據行數
[flying@lemp ~]$ awk 'END{print NR}' /tmp/haskeywords.txt
25835
[flying@lemp ~]$
```

### Extract Specific Field
文件`/tmp/haskeywords.txt`數據格式爲

```
flybus-provider [2016-11-29 16:23:15] 1480407795138-737 com.raxtone.utils.spring.logger.LoggerAspect:48 INFO : (LoggerAspect.java:48)    请求方法：void com.raxtone.flybus.rpc.rmi.user.IPushTSRmiService.sendTokenToServer(TokenParam,String,Integer,Integer)，请求参数：[{"cityId":0,"token":"3d34356980904e8348107081b64112eac0852f3851dd71537fa74d0e9d2fbf41"},"36416",11,100]
```

通過命令
```bash
sed -r 's@(\[|\]|\")@@g'
```
去除日期兩側的方括號`[`、`]`以及數據行中的雙引號`"`。

處理後的格式如下
```
flybus-provider 2016-11-29 16:23:15 1480407795138-737 com.raxtone.utils.spring.logger.LoggerAspect:48 INFO : (LoggerAspect.java:48)    请求方法：void com.raxtone.flybus.rpc.rmi.user.IPushTSRmiService.sendTokenToServer(TokenParam,String,Integer,Integer)，请求参数：{cityId:0,token:3d34356980904e8348107081b64112eac0852f3851dd71537fa74d0e9d2fbf41},36416,11,100
```

#### Extract change_date
通過命令
```bash
awk '{printf("%s %s\n",$2,$3)}'
```

提取變更日期，輸出爲
```bash
2016-11-29 16:23:15
```

測試過程
```bash
[flying@lemp ~]$ head -1 /tmp/haskeywords.txt  | sed -r 's@(\[|\]|\")@@g' | awk '{printf("%s %s\n",$2,$3)}'
2016-11-29 16:23:15
[flying@lemp ~]$
```

#### Extract token
通過命令
```bash
sed -r -n 's@.*token:(.*)}.*@\1@p'
```
提取token，輸出爲
```bash
3d34356980904e8348107081b64112eac0852f3851dd71537fa74d0e9d2fbf41
```

測試過程
```bash
[flying@lemp ~]$ head -1 /tmp/haskeywords.txt  | sed -r 's@(\[|\]|\")@@g' | sed -r -n 's@.*token:(.*)}.*@\1@p'
3d34356980904e8348107081b64112eac0852f3851dd71537fa74d0e9d2fbf41
[flying@lemp ~]$
```

#### Extract mac_id
通過命令
```bash
awk -v FS=',' '{print $(NF-2)}'
```
提取設備ID，輸出爲
```bash
36416
```

測試過程
```bash
[flying@lemp ~]$ head -1 /tmp/haskeywords.txt  | sed -r 's@(\[|\]|\")@@g' | awk -v FS=',' '{print $(NF-2)}'
36416
[flying@lemp ~]$
```

### Write Info New File
將其取出的數據寫入新文件，具體命令如下

```bash
resultfile='/tmp/resultfile.txt'
[[ -f "$resultfile" ]] && rm -f "$resultfile"
touch "$resultfile"

flag=0
while read line; do
    strip_character=$(echo "$line" | sed -r 's@(\[|\]|\")@@g')
    change_date=$(echo "$strip_character" | awk '{printf("%s %s\n",$2,$3)}')
    token=$(echo "$strip_character" | sed -r -n 's@.*token:(.*)}.*@\1@p')
    mac_id=$(echo "$strip_character" | awk -v FS=',' '{print $(NF-2)}')
    let flag+=1
    printf "%d|%s|%s|%s\n" "$flag" "$mac_id" "$token" "$change_date" >> "$resultfile"
done < /tmp/haskeywords.txt
```

提取出的數據文件大小

```bash
[flying@lemp ~]$ ls -lh /tmp/resultfile.txt
-rw-rw-r-- 1 flying flying 2.7M Dec  5 12:17 /tmp/resultfile.txt
# 數據行數
[flying@lemp ~]$ awk 'END{print NR}' /tmp/resultfile.txt
25835
# 數據格式
[flying@lemp ~]$ awk 'END{print $0}' /tmp/resultfile.txt
25835|27441|6c565066e2637c6b2b53d2d0e4dae6d4ef21856cc9187ab9d61111692a077c2b|2016-12-01 23:54:41
[flying@lemp ~]$
```

### Load Data Into MySQL DBMS
將提取出的數據寫入數據表

1. 創建數據表

```sql
-- 刪除存在的同名數據庫
drop database if exists userTokenAnalysis;

-- 創建數據庫
create database if not exists userTokenAnalysis
    default character set=utf8
    default collate=utf8_general_ci;

-- 進入數據庫
use userTokenAnalysis;

-- 創建數據表details
create table if not exists details (
    id mediumint unsigned not null auto_increment primary key comment 'primary id',
    mac_id char(40) not null comment 'mac id',
    token char(64) not null comment 'token length 64',
    change_date timestamp null default null comment 'change date',
    storage_time timestamp not null default current_timestamp comment 'item format YYYY-MM-DD HH:MM:SS'
)engine=innodb default charset=utf8 collate=utf8_general_ci comment 'token mac_id change_date';

-- 創建索引
create index details_mac_id on details (mac_id) comment 'index mac_id';
```

2. 寫入數據庫

```bash
mysql -e "load data local infile '/tmp/resultfile.txt' into table userTokenAnalysis.details fields terminated by '|' lines terminated by '\n';"
```


## Complete Shell Script
將上述在本機進行的操作合併成Shell Script，內容如下

```bash
#!/bin/bash
#writer: lempstacker
#date: 2016.12.05 12:17 Mon Asia/Shanghai

awk '$1~/flybus-provider/&&($2~/2016-11-[[:digit:]]{2}/||$2~/2016-12-01/){printf $0;getline;print}' /tmp/flybusProvider/* > /tmp/mergeline.txt

awk '$0~/sendTokenToServer/&&$0~/"token"/{printf("%s\n",$0)}' /tmp/mergeline.txt > /tmp/haskeywords.txt

mysql -e "
drop database if exists userTokenAnalysis;

create database if not exists userTokenAnalysis
    default character set=utf8
    default collate=utf8_general_ci;

use userTokenAnalysis;

create table if not exists details (
    id mediumint unsigned not null auto_increment primary key comment 'primary id',
    mac_id char(40) not null comment 'mac id',
    token char(64) not null comment 'token length 64',
    change_date timestamp null default null comment 'change date',
    storage_time timestamp not null default current_timestamp comment 'item format YYYY-MM-DD HH:MM:SS'
)engine=innodb default charset=utf8 collate=utf8_general_ci comment 'token mac_id change_date';

create index details_mac_id on details (mac_id) comment 'index mac_id'
"

resultfile='/tmp/resultfile.txt'
[[ -f "$resultfile" ]] && rm -f "$resultfile"
touch "$resultfile"

flag=0
while read line; do
    strip_character=$(echo "$line" | sed -r 's@(\[|\]|\")@@g')
    change_date=$(echo "$strip_character" | awk '{printf("%s %s\n",$2,$3)}')
    token=$(echo "$strip_character" | sed -r -n 's@.*token:(.*)}.*@\1@p')
    mac_id=$(echo "$strip_character" | awk -v FS=',' '{print $(NF-2)}')
    let flag+=1
    printf "%d|%s|%s|%s\n" "$flag" "$mac_id" "$token" "$change_date" >> "$resultfile"
done < /tmp/haskeywords.txt

unset flag

mysql -e "load data local infile '/tmp/resultfile.txt' into table userTokenAnalysis.details fields terminated by '|' lines terminated by '\n';"

# rm -f /tmp/mergeline.txt
# rm -f /tmp/haskeywords.txt
# rm -f /tmp/resultfile.txt

#Script End
```

## Query
數據查詢，可分爲2種

* 直接操作數據文件`/tmp/resultfile.txt`；
* 在數據庫中執行select查詢；

### Directly Search File
提取出現次數最高的10個設備ID，並按照出現次數的多寡逆序排列

```bash
[flying@lemp ~]$ awk -v FS='|' '{arr[$2]++}END{flag=0;PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) if(flag < 10){print i,arr[i];flag++}}' /tmp/resultfile.txt
36416 198
29718 160
31385 158
37493 154
31246 153
28326 149
21157 146
488133db68064f8aba9ba08c8ef4e746687568b6 140
35985 140
22032 136
[flying@lemp ~]$
```

提取出現次數最高的10個設備ID(字符長度大於5)，並按照出現次數的多寡逆序排列
```bash
[flying@lemp ~]$ awk -v FS='|' '{if(length($2)>5){arr[$2]++}}END{flag=0;PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) if(flag < 10) {print i,arr[i];flag++}}' /tmp/resultfile.txt
488133db68064f8aba9ba08c8ef4e746687568b6 140
d41d8cd98f00b204e9800998ecf8427eaf861266 110
fad24ff33c1c935aeb4355c047b6780ea5862b1f 99
bed8a36e77253e1e16972e73b2c8743def6356e7 99
d41d8cd98f00b204e9800998ecf8427e7c0e84ed 98
b64fe0e416eecd61bdea35379643846d308f6b97 98
933abe66f2a3ac125f722bf44eb566488aad0c0f 92
c3aabfe68db99085463de99f0756874108e90479 92
ae6b851b36b11836c11b86c3d24b2037f32376fa 89
afcab9731bd52208a64a0bf15a9ef75b6bcb1ccc 87
[flying@lemp ~]$
```

查詢用戶設備ID爲`88befa42adb6745cc9c19e18d689f0913290c4d2`的數據，格式按`日期 設備ID token`排列

```bash
[flying@lemp ~]$ awk -v FS='|' '$2~/88befa42adb6745cc9c19e18d689f0913290c4d2/{printf "%s %s %s\n",$NF,$2,$3}' /tmp/resultfile.txt
2016-12-01 17:50:56 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
2016-12-01 18:18:58 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
2016-12-01 18:35:12 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
2016-12-01 17:50:56 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
2016-12-01 18:18:58 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
2016-12-01 18:35:12 88befa42adb6745cc9c19e18d689f0913290c4d2 31aa8d0c86a2974e700b2e1c7a1c3d1a9ddf36326cafbc63c5164bdb1ce27a39
[flying@lemp ~]$
```

### Use Select Statements
查詢獨立的設備ID數
```sql
MariaDB [userTokenAnalysis]> select count(distinct(mac_id)) as counts from details;
+--------+
| counts |
+--------+
|   1817 |
+--------+
1 row in set (0.03 sec)

MariaDB [userTokenAnalysis]>
```

查詢各設備ID出現的次數(只取10條)

```sql
MariaDB [userTokenAnalysis]> select mac_id,count(id) as counts from details group by mac_id order by counts desc limit 10;
+------------------------------------------+--------+
| mac_id                                   | counts |
+------------------------------------------+--------+
| 36416                                    |    198 |
| 29718                                    |    160 |
| 31385                                    |    158 |
| 37493                                    |    154 |
| 31246                                    |    153 |
| 28326                                    |    149 |
| 21157                                    |    146 |
| 35985                                    |    140 |
| 488133db68064f8aba9ba08c8ef4e746687568b6 |    140 |
| 22032                                    |    136 |
+------------------------------------------+--------+
10 rows in set (0.11 sec)

MariaDB [userTokenAnalysis]>
```

查詢指定的用戶的設備ID出現的次數

```sql
MariaDB [userTokenAnalysis]> select mac_id,count(id) as counts from details where mac_id in ('a58c41716de3c98b0b9b6a44f03442c803d61d78','96262d20d58519e2ea6ea87f8cb0a5f7653fcc40','e444928b471124ce31609f992ac0853cbf2a0b8c','88befa42adb6745cc9c19e18d689f0913290c4d2','12a352b35e513dc5f341ec35aad12ff') group by mac_id order by counts desc;
+------------------------------------------+--------+
| mac_id                                   | counts |
+------------------------------------------+--------+
| a58c41716de3c98b0b9b6a44f03442c803d61d78 |     50 |
| 88befa42adb6745cc9c19e18d689f0913290c4d2 |      6 |
| 96262d20d58519e2ea6ea87f8cb0a5f7653fcc40 |      5 |
+------------------------------------------+--------+
3 rows in set (0.04 sec)

MariaDB [userTokenAnalysis]>
```

查詢設備ID`a58c41716de3c98b0b9b6a44f03442c803d61d78`各token出現的次數

```sql
MariaDB [userTokenAnalysis]> select token,count(token) as counts from details where mac_id='a58c41716de3c98b0b9b6a44f03442c803d61d78' group by token desc;
+------------------------------------------------------------------+--------+
| token                                                            | counts |
+------------------------------------------------------------------+--------+
| 12f8416a809e895caaba4903a3dcd916c0a0ea18fb135f968601d842c09964ed |     50 |
+------------------------------------------------------------------+--------+
1 row in set (0.04 sec)

MariaDB [userTokenAnalysis]>
```

在數據庫中將查詢到的數據提取到外部文件中

文件默認不存在,如存在會報錯

>ERROR 1086 (HY000): File '/tmp/mysqlextractdata.txt' already exists

```sql
MariaDB [(none)]> select change_date,mac_id,token into outfile '/tmp/mysqlextractdata.txt' fields terminated by ' ' lines terminated by '\n' from userTokenAnalysis.details order by mac_id;
Query OK, 25835 rows affected (0.09 sec)

MariaDB [(none)]> exit
Bye
```

```bash
[flying@lemp ~]$ awk 'END{print NR}' /tmp/mysqlextractdata.txt
25835
[flying@lemp ~]$
```

在Bash Shell中執行SQL將查詢數據導入到文件中

```bash
[flying@lemp ~]$ mysql -Bse "select change_date,mac_id,token from userTokenAnalysis.details where mac_id in ('a58c41716de3c98b0b9b6a44f03442c803d61d78','96262d20d58519e2ea6ea87f8cb0a5f7653fcc40','e444928b471124ce31609f992ac0853cbf2a0b8c','88befa42adb6745cc9c19e18d689f0913290c4d2','12a352b35e513dc5f341ec35aad12ff') order by mac_id;" > /tmp/mysqlextractdata.txt
[flying@lemp ~]$ awk 'END{print NR}' /tmp/mysqlextractdata.txt
61
[flying@lemp ~]$
```


## References
* [LOAD DATA INFILE Syntax](https://dev.mysql.com/doc/refman/5.7/en/load-data.html)
* [SELECT ... INTO Syntax](https://dev.mysql.com/doc/refman/5.7/en/select-into.html)


## Change Logs
* 2016.12.05 15:29 Mon Asia/Shanghai
    * 初稿完成
* 2018.04.11 09:37 Wed America/Boston
    * 勘誤，遷移到新Blog
