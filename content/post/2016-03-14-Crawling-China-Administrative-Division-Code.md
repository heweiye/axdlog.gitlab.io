---
title: Crawling China Administrative Division Code
slug: Crawling China Administrative Division Code
date: 2016-03-14T00:01:59+08:00
lastmod: 2018-04-22T20:22:59-04:00
draft: false
keywords: ["AxdLog", "Web Crawling", "SED", "AWK", "MySQL", "Shell script"]
description: "How to use shell script to crawl China administrative division code from Center National Bureau of Statistics of China, then importing row datas into MySQL database."
categories:
- Web Crawling
tags:
- sed
- awk
- MySQL
- Shell Script

comment: true
toc: true
autoCollapseToc: true
contentCopyright: ""
mathjax: false

---

本文記錄如何從[中華人民共和國國家統計局][nbs]官網提取中國大陸地區的行政區劃信息，數據入庫，並通過Shell腳本實現整個過程。項目代碼託管在[GitHub](https://github.com/MaxdSre/Shell-script-web-crawler/tree/master/project/CADC)，包含Shell腳本、數據庫數據表、文本數據、從MySQL中導出的SQL數據。

<!--more-->

## Prerequisite
Short Name | Name | English Name
---|---|---
ROC | 中華民國 | Republic of China
PRC | 中華人民共和國 | People's Republic of China

數據來源 [中華人民共和國國家統計局][nbs]

* [統計用區劃和城鄉劃分代碼](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/)
* [統計用區劃代碼和城鄉劃分代碼編制規則](http://www.stats.gov.cn/tjsj/tjbz/200911/t20091125_8667.html)


根據最新的[2016年](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html)版信息，中國目前有`34`個省級行政區，包括`23`個省（含臺灣省）、`5`個自治區、`4`個直轄市、`2`個特別行政區（香港特別行政區、澳門特別行政區）。但 *區劃代碼和城鄉劃分代碼* 不包括（臺灣省、香港特別行政區、澳門特別行政區）。


中國大陸地區按地理位置可分為六塊

Geographical Division | Provice Lists
:--- | :---
`1` 華北 | 北京市、天津市、河北省、山西省、內蒙古自治區
`2` 東北 | 遼寧省、吉林省、黑龍江省
`3` 華東 | 上海市、江蘇省、浙江省、安徽省、福建省、江西省、山東省
`4` 中南 | 河南省、河北省、湖北省、廣東省、廣西壯族自治區、海南省
`5` 西南 | 重慶市、四川省、貴州省、雲南省、西藏自治區
`6` 西北 | 陝西省、甘肅省、青海省、寧夏回族自治區、新疆維吾爾族自治區

各省份簡稱

region|code|name|short name
---|---|---|---
華北|11|北京市|京
華北|12|天津市|津
華北|13|河北省|冀
華北|14|山西省|晉
華北|15|內蒙古自治區|蒙
東北|21|遼寧省|遼
東北|22|吉林省|吉
東北|23|黑龍江省|黑
華東|31|上海市|滬
華東|32|江蘇省|蘇
華東|33|浙江省|浙
華東|34|安徽省|皖
華東|35|福建省|閩
華東|36|江西省|贛
華東|37|山東省|魯
中南|41|河南省|豫
中南|42|湖北省|鄂
中南|43|湖南省|湘
中南|44|廣東省|粵
中南|45|廣西壯族自治區|桂
中南|46|海南省|瓊
西南|50|重慶市|渝
西南|51|四川省|川
西南|52|貴州省|黔
西南|53|雲南省|滇
西南|54|西藏自治區|藏
西北|61|陝西省|陝
西北|62|甘肅省|甘
西北|63|青海省|青
西北|64|寧夏回族自治區|寧
西北|65|新疆維吾爾自治區|新
 | |臺灣|臺
 | |香港特別行政區|港
 | |澳門|澳


## Rules Explanation
規則依據來自 [統計用區劃代碼和城鄉劃分代碼編制規則](http://www.stats.gov.cn/tjsj/tjbz/200911/t20091125_8667.html)。

>為規範統計用區劃代碼和城鄉劃分代碼，建立各項普查、全面統計、抽樣調查、專項調查統一使用的《統計用區劃代碼和城鄉劃分代碼庫》，特制定本規則。

**統計用區劃代碼** 和 **城鄉劃分代碼** 分為2段共`17`位，前者由第1~12位代碼構成，後者由13~17位代碼構成，其中第13~14位為 *城鄉屬性代碼* ，第16~17為 *城鄉分類代碼* 。

### Statistical Division Code
>統計用區劃代碼基本長度為`12`位，省、地、縣、鄉四級代碼不足12位用`0`補足。

`統計用區劃代碼`共12位，行政區劃按行政級別共分為5級。

| Division Level | Position | Length |English | Explanation |
| :--- | :--- | :--- | :--- | :--- |
| `省級` | 1~2 | 2 | Province | 包含省、直轄市、自治區 |
| `地市級` | 3~4 | 2 |City | 各省、自治區的地級市，各直轄市的市轄區 |
| `縣級` | 5～6 | 3 | Country | 各直轄市的市轄區，各地級市所轄的區、市、縣 |
| `鄉級` | 7～9 | 3 | Town | 各街道辦事處或鄉鎮 |
| `村級` | 10～12 | 3 | Village | 各社區居委會 、各村委會 |


**縣以上行政區劃代碼由1～6位代碼組成**。在統計工作中，各級統計部門不編制縣以上行政區劃代碼，統一採用《中華人民共和國行政區劃代碼》國家標準。

**縣以下區劃代碼由7～12位代碼組成**，包括 *鄉級代碼* 和 *村級代碼* 兩部分。

Level|Code Range|Position|Length|Explanation
---|---|---|---|---
鄉級|`001～099`|7~9|3|街道
鄉級|`100～199`|7~9|3|鎮
鄉級|`200～399`|7~9|3|鄉
鄉級|`400～599`|7~9|3|類似鄉級單位 (民政部門未確認的開發區、工礦區、農場等)
村級|`001~199`|10~12|3|居民委員會
村級|`200~399`|10~12|3|村民委員會
村級|`400~499`|10~12|3|類似居民委員會(不含498代碼)
村級|`500~599`|10~12|3|類似村民委員會(不含598代碼)
村級|`498`|10~12|3|虛擬社區 (街道、鎮以及類似鄉級單位的開發區、科技園區、工業園區、工礦區、高校園區、科研機構園區等)
村級|`598`|10~12|3|虛擬生活區 (鄉以及類似鄉級單位的農、林、牧、漁場和其它農業活動區域)


>凡民政部門確認的街道、鎮、鄉，按照國家標準《縣級以下行政區劃代碼編制規則》（GB/T 10114—2003）編制，其鄉級代碼為001～399；民政部門未確認的開發區、工礦區、農場等類似鄉級單位，鄉級代碼為400～599。


### Urban And Rural Classification Code
`城鄉劃分代碼`共5位，含有 **城鄉屬性**、**城鄉分類** 2個維度。

Postion|Length|Explanation
---|---|---
13~14 | 2 | **城鄉屬性代碼**
15~17 | 3 | **城鄉分類代碼**

#### Attributes Code

>城鄉屬性代碼由第13、14位代碼組成。其中：第13位表示鄉級屬性，第14位表示村級屬性。

>鄉級屬性代碼表示街道、鎮、鄉以及類似鄉級單位的鄉級屬性。鄉級屬性代碼用1～3數字表示。

>村級屬性代碼表示居民委員會（社區）、村民委員會以及類似村級單位的村級屬性。村級屬性代碼用1～9數字表示。

Level|Code Num|Position|Length|Explanation
---|---|---|---|---
鄉級|`1`|13|1|縣級政府駐地
鄉級|`2`|13|1|連接的鄉級區域
鄉級|`3`|13|1|其他鄉級區域
村級|`1`|14|1|鄉級政府駐地
村級|`2`|14|1|完全連接的村級地域
村級|`3`|14|1|部分連接的村級地域
村級|`4`|14|1|與其他區、市完全連接的村級地域
村級|`5`|14|1|與其他區、市部分連接的村級地域
村級|`6`|14|1|與其他鎮完全連接的村級地域
村級|`7`|14|1|與其他鎮部分連接的村級地域
村級|`8`|14|1|特殊地域
村級|`9`|14|1|其他村級地域


>在城鄉屬性代碼中，特殊地域僅指以下兩種情況：
* 在類似居委會中，常住人口達到或超過3000人的開發區、工礦區、大專院校、科研單位等居民生活區域。
* 在類似村委會中，常住人口達到或超過3000人，且非農產業從業人員達到70%的農、林、牧、漁場及其他以農業活動為主的區域。
當類似居委會、類似村委會不滿足上述要求，也不滿足代碼`1～7`的編制要求時,村級屬性一律編`9`。

>農、林、牧、漁場
農、林、牧、漁場場部的村級屬性代碼一律編`1`，與場部連接的下屬生產單位，村級屬性代碼編`2`或`3`，與場部不連接的下屬生產單位，村級屬性代碼編`9`。當常住人口達到3000人，非農產業從業人員達到70%時，不連接的下屬生產單位的村級屬性代碼編`8`。

>只有一個村級單位的村級屬性
凡鄉級單位下只有一個村級單位，無論是實際存在還是虛擬的，其對應的村級屬性代碼一律編`1`。

>不連接的居民委員會
當居民委員會與政府駐地不連接時，如果有農業用地，村級屬性代碼編`9`；如果沒有農業用地（或沒有明確的地域），村級屬性代碼編`8`。


#### Classification Code
城鄉分類代碼

通常由`3`位數字構成：
* 首位是`1`表示 **城鎮**；
* 首位是`2`表示 **鄉村**；

>城鄉分類代碼由第15～17位代碼組成。第15位為`1`，表示城鎮；第15位為`2`，表示鄉村。

Code Num|Position|Length|Explanation
---|---|---|---
`111`|15～17|3|主城區
`112`|15～17|3|城鄉結合區
`121`|15～17|3|鎮中心區域
`122`|15～17|3|鎮鄉結合區
`123`|15～17|3|特殊區域
`210`|15～17|3|鄉中心區
`220`|15～17|3|村莊


## Web Crawling Methods
通過命令`curl`或`wget`抓取html頁面，使用命令`sed`、`awk`對數據進行處理。

直接使用`curl`抓取頁面，返回的數據出現中文亂碼，考慮是編碼問題。使用`curl -fsL URL | grep charset`返回的數據中有`charset=gb2312`，即頁面採用`GB2312`編碼。

解決方案是使用`iconv`命令進行編碼轉換，格式
```
iconv -f fromcode -t tocode [file ...]
```

使用命令
```bash
iconv -l | grep -Ei '^(gb|utf)'
```
返回的編碼集中有`GB2312`、`GBK`、`UTF8`、`UTF-8`。經過測試源編碼可使用`GB2312`或`GBK`，目標編碼可使用`UTF8`或`UTF-8`。

即使用`curl -fsL URL | iconv -f GBK -t UTF-8`。


>-s quiet靜默模式 --retry 重試次數 --retry-delay 間隔時間 -x 代理 -o保存路徑 ipaddr 代理IP port 代理端口

使用代理進行抓取操作，如 `curl -fsL --retry 5 --retry-delay 5 -x ipaddr:port | iconv -f GBK -t UTF-8`

使用`sed -r 's@<[^>]*>@@g;'`去除HTML頁面中標籤。


## Official Page Analysis

對 [2016年統計用區劃代碼和城鄉劃分代碼(](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html) 相關頁面進行分析。

HTML標籤class類

* 省級: `<tr class='provincetr'></tr>`
* 地級: `<tr class='citytr'></tr>`
* 縣級: `<tr class='countytr'></tr>`
* 鄉級: `<tr class='towntr'></tr>`
* 村級: `<tr class='villagetr'></tr>`


### URL Analysis
**首頁地址**
```http
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html
```

**省級列表地址**
```http
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13.html
```
`13`是省級區劃代碼，如河北省`13`，對應替換為其它省份代碼

**地級列表地址**
```http
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/1306.html
```
`13`是河北省，`1306`是保定市，格式是`2位省份代碼/地級市代碼`

**縣級列表地址**
```http
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/06/130637.html
```
`13`是河北省，`06`在河北省下代表保定，`130637`是河北省保定市博野縣，格式是`2位省份代碼/2位所屬地級市代碼/縣級代碼`

**鄉級列表地址**
```http
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/06/37/130637100.html
```
`13`是河北省，`06`在河北省下代表保定，`s`在保定下代表博野縣，`130637100`河北省保定市博野縣博野鎮，顯示具體村莊列表信息，格式是`2位省份代碼/2位所屬地級市代碼/2位縣級代碼/鄉級代碼`


### Crawler Codes
首頁地址: `index.html`

```bash
curl -fsL http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html | iconv -f GBK -t UTF-8 2> /dev/null | sed -n "/class='provincetr'/{s@'@@g;s@<td>@\n@g;p}" | sed -r -n '/href/{s@.*href=([[:digit:]]+)[^>]*>([^<]*)<.*@\1|\2|&@g;p}'
```

省級列表地址: `13.html`

```bash
curl -fsL http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13.html | iconv -f GBK -t UTF-8 | sed -r -n "/class='citytr'/{s@</tr>@\n@g;s@<\/td>@ @g;s@['\''|\"]@@g;s@[[:space:]]*<a href=[^>]*>([[:digit:]]+)<\/a>[[:space:]]@\1@g;s@(href=)([^>]+)@\1>\2<@g;s@<[^>]*>@ @g;s@[[:blank:]]+@ @g;p}" | sed -r -n '/^[[:space:]]*$/d;s@^[[:space:]]*@@g;s@[[:space:]]*$@@g;p'
```

地級列表地址: `13/1306.html`

```bash
curl -fsL http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/1306.html  | iconv -f GBK -t UTF-8 | sed -r -n "/class='countytr'/{s@</tr>@\n@g;s@<\/td>@ @g;s@['\''|\"]@@g;s@[[:space:]]*<a href=[^>]*>([[:digit:]]+)<\/a>[[:space:]]@\1@g;s@(href=)([^>]+)@\1>\2<@g;s@<[^>]*>@ @g;s@[[:blank:]]+@ @g;p}" | sed -r -n '/^[[:space:]]*$/d;s@^[[:space:]]*@@g;s@[[:space:]]*$@@g;p'
```

縣級列表地址: `13/06/130637.html`

```bash
curl -fsL http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/06/130637.html  | iconv -f GBK -t UTF-8 | sed -r -n "/class='towntr'/{s@</tr>@\n@g;s@<\/td>@ @g;s@['\''|\"]@@g;s@[[:space:]]*<a href=[^>]*>([[:digit:]]+)<\/a>[[:space:]]@\1@g;s@(href=)([^>]+)@\1>\2<@g;s@<[^>]*>@ @g;s@[[:blank:]]+@ @g;p}" | sed -r -n '/^[[:space:]]*$/d;s@^[[:space:]]*@@g;s@[[:space:]]*$@@g;p'
```

鄉級列表地址: `13/06/37/130637100.html`

```bash
curl -fsL http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/13/06/37/130637100.html  | iconv -f GBK -t UTF-8 | sed -r -n "/class='villagetr'/{s@</tr>@\n@g;s@<\/td>@ @g;s@<[^>]*>@@g;p}" | sed -r '/^[[:digit:]]+/!d;s@[[:blank:]]*$@@g'
```

## Code Design Architecture
代碼架構設計

按照 `省級` --> `地級` --> `縣級` --> `鄉級` --> `村級` 層級,逐級操作。為提高操作效率，通過命令`parallel`進行多進程操作。數據寫入文本文件，待數據抓取完畢後，導入數據庫。

為簡化操作，不配置代理IP。


共3個核心文件

```sh
┌─[maxdsre@Stretch]─[/tmp/CADC]
└──╼ $tree
.
├── cadc_database_table.sql
├── cadc_crawler.sh
└── cadc_import_database.sh

0 directories, 3 files
┌─[maxdsre@Stretch]─[/tmp/CADC]
└──╼ $
```

file|explanation
---|---
cadc_extraction.sh | 數據抓取腳本
cadc_database_table.sql| 數據表建表語句
cadc_import_database.sh | 數據導入至數據庫腳本


### SQL Design
文件`cadc_database_table.sql`， 存放數據表建表語句，共5張表，關聯表之間有外間約束。

```sql
-- 數據庫名 cadc
drop database if exists cadc;
create database if not exists cadc character set=utf8 collate=utf8_general_ci;
use cadc

-- province省、直轄市、自治區
drop table if exists province;
create table if not exists province (
    id tinyint unsigned not null auto_increment primary key comment '省級列表自增id',
    name char(30) not null comment '省份名稱',
    code tinyint unsigned not null comment '行政區劃代碼(共12位)，前2位指定，第1-2位為省級代碼',
    region enum('華北','東北','華東','中南','西南','西北') not null comment '省所屬地理區域:1華北、2東北、3華東、4中南(華中,華南)、5西南、6西北',
    createTime timestamp default current_timestamp comment '數據入庫時間 YYYY-MM-DD HH:MM:SS',
    updateTime timestamp null on update current_timestamp comment '數據更新時間 YYYY-MM-DD HH:MM:SS',
    key in_name (name),
    unique key in_code (code)
)engine=innodb default charset=utf8 collate=utf8_general_ci comment='中國大陸地區省份列表';
-- alter table province add index in_name (name);

--  city地市級
drop table if exists city;
create table if not exists city (
    id smallint unsigned not null auto_increment primary key comment '地級市列表自增id',
    province_id tinyint unsigned not null comment '省級id，對應表province，外鍵約束',
    name char(60) not null comment '地級市名稱',
    code char(12) not null comment '行政區劃代碼(共12位)，由前4位指定，第3-4位為地級代碼',
    createTime timestamp default current_timestamp comment '數據入庫時間 YYYY-MM-DD HH:MM:SS',
    updateTime timestamp null on update current_timestamp comment '數據更新時間 YYYY-MM-DD HH:MM:SS',
    key in_name (name(15)),
    key in_fk_province (province_id),
    unique key in_code (code),
    constraint fk_province_city foreign key (province_id) references province(id) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment='中國大陸地區地級市列表';
-- alter table city add constraint fk_province_city foreign key(province_id) references province(id) on update cascade;

--  country縣級
drop table if exists country;
create table if not exists country (
    id smallint unsigned not null auto_increment primary key comment '縣級列表自增id',
    city_id smallint unsigned not null comment '地級市id，對應表city，外鍵約束',
    name char(60) not null comment '縣級市名稱',
    code char(12) not null comment '行政區劃代碼(共12位)，由前6位指定，第5-6位為縣級代碼',
    createTime timestamp default current_timestamp comment '數據入庫時間 YYYY-MM-DD HH:MM:SS',
    updateTime timestamp null on update current_timestamp comment '數據更新時間 YYYY-MM-DD HH:MM:SS',
    key in_name (name(15)),
    key in_fk_city (city_id),
    unique key in_code (code),
    constraint fk_city_country foreign key (city_id) references city(id) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment='中國大陸地區縣級列表';

--  town鄉級
drop table if exists town;
create table if not exists town (
    id mediumint unsigned not null auto_increment primary key comment '鄉鎮列表自增id',
    coutnry_id smallint unsigned not null comment '縣級市id，對應表country，外鍵約束',
    name char(60) not null comment '鄉鎮名稱',
    code char(12) not null comment '行政區劃代碼(共12位)，由前9位指定，第7-9位為鄉級代碼',
    code_type enum('街道','鎮','鄉','類似鄉級單位') null comment '鄉級代碼分類(第7-9位指定)，1為街道 001~099，2為鎮 100~199，3為鄉 200~399, 4為類似鄉級單位 400~599',
    createTime timestamp default current_timestamp comment '數據入庫時間 YYYY-MM-DD HH:MM:SS',
    updateTime timestamp null on update current_timestamp comment '數據更新時間 YYYY-MM-DD HH:MM:SS',
    key in_name (name(15)),
    key in_fk_country (coutnry_id),
    unique key in_code (code),
    constraint fk_country_town foreign key (coutnry_id) references country(id) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment='中國大陸地區鄉鎮列表';

--  village村級
drop table if exists village;
create table if not exists village (
    id mediumint unsigned not null auto_increment primary key comment '村級列表自增id',
    town_id mediumint unsigned not null comment '鄉鎮id，對應表town，外鍵約束',
    name char(60) not null comment '村莊名稱',
    code char(12) not null comment '行政區劃代碼(共12位)，第10-12位為村級代碼',
    code_type enum('居民委員會', '村民委員會', '類似居民委員會','類似村民委員會','498虛擬社區','598虛擬社區') not null comment '村級代碼分類(第10-12位指定)，: 1 居民委員會 001~199, 2 村民委員會 200~399, 3 類似居民委員會（不含498代碼） 400~499, 4 類似村民委員會（不含598代碼） 500~599, 5 虛擬社區 498, 6 虛擬社區 598',
    -- town_attr enum('縣級政府駐地','連接的鄉級區域','其他鄉級區域') not null comment '城鄉屬性代碼鄉級屬性(由第13位決定): 1 縣級政府駐地, 2 連接的鄉級區域, 3 其他鄉級區域',
    -- village_attr enum('鄉級政府駐地','完全連接的村級地域','部分連接的村級地域','與其他區、市完全連接的村級地域','與其他區、市部分連接的村級地域','與其他鎮完全連接的村級地域','與其他鎮部分連接的村級地域','特殊地域','其他村級地域') not null comment '城鄉屬性代碼村級屬性(由第14位決定): 1 鄉級政府駐地, 2 完全連接的村級地域, 3 部分連接的村級地域, 4 與其他區、市完全連接的村級地域, 5 與其他區、市部分連接的村級地域, 6 與其他鎮完全連接的村級地域, 7 與其他鎮部分連接的村級地域, 8 特殊地域, 9 其他村級地域',
    classification enum('主城區','城鄉結合區','鎮中心區','鎮鄉結合區','特殊地區','鄉中心區','村莊') not null comment '城鄉分類,由第15-17位代碼指定,頁面中為3個獨立的數字(第15位代碼指定城鄉分類，1為城鎮，2為鄉村)：主城區 111，城鄉結合區 112，鎮中心區 121，鎮鄉結合區 122，特殊地區 123，鄉中心區 210，村莊 220',
    createTime timestamp default current_timestamp comment '數據入庫時間 YYYY-MM-DD HH:MM:SS',
    updateTime timestamp null on update current_timestamp comment '數據更新時間 YYYY-MM-DD HH:MM:SS',
    key in_name (name(15)),
    key in_fk_town (town_id),
    unique key in_code (code),
    constraint fk_town_village foreign key (town_id) references town(id) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment='中國大陸地區村級列表';
```

## Data Presentation

year|province|city|country|town|village
---|---|---|---|---|---
2013|31|345|3136|43854|694691
2014|31|346|3139|40053|670427
2015|31|346|3137|39928|667511
2016|31|344|3133|42866|666612

```sql
MySQL [(none)]> show databases like 'cadc%';
+------------------+
| Database (cadc%) |
+------------------+
| cadc_2013        |
| cadc_2014        |
| cadc_2015        |
| cadc_2016        |
+------------------+
4 rows in set (0.00 sec)

MySQL [(none)]> use cadc_2016
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [cadc_2016]> show tables;
+---------------------+
| Tables_in_cadc_2016 |
+---------------------+
| city                |
| country             |
| province            |
| town                |
| village             |
+---------------------+
5 rows in set (0.01 sec)

MySQL [cadc_2016]> select count(distinct(a.id)) as province,count(distinct(b.id)) as city,count(distinct(c.id)) as country,count(distinct(d.id)) as town,count(distinct(e.id)) as village from province a
    -> left join city b on a.id=b.province_id
    -> left join country c on b.id=c.city_id
    -> left join town d on c.id=d.coutnry_id
    -> left join village e on d.id=e.town_id;
+----------+------+---------+-------+---------+
| province | city | country | town  | village |
+----------+------+---------+-------+---------+
|       31 |  344 |    3133 | 42866 |  666612 |
+----------+------+---------+-------+---------+
1 row in set (0.63 sec)

MySQL [cadc_2016]>
```

### Interesting Name
```bash
database_name='cadc_2016'

mysql -D "${database_name}" -se "select a.name province, b.name city, c.name country, d.name as town, e.name as village from province a left join city b on a.id=b.province_id left join country c on b.id=c.city_id left join town d on c.id=d.coutnry_id left join village e on d.id=e.town_id where e.name like '佛祖%';"
```

以`佛祖`二字開頭的村子

province|city|country|town|village
---|---|---|---|---
四川省|內江市|東興區|同福鎮|佛祖巖村村委會
湖北省|武漢市|江夏區|佛祖嶺街道辦事處|佛祖嶺村村委會
湖北省|武漢市|江夏區|佛祖嶺街道辦事處|佛祖嶺社區一區居民委員會
湖北省|武漢市|江夏區|佛祖嶺街道辦事處|佛祖嶺社區三區居民委員會
湖北省|武漢市|江夏區|佛祖嶺街道辦事處|佛祖嶺社區二區居民委員會
湖北省|武漢市|江夏區|佛祖嶺街道辦事處|佛祖嶺社區四區居民委員會
湖南省|郴州市|臨武縣|鎮南鄉|佛祖村村委會
四川省|綿陽市|遊仙區|太平鎮|佛祖村民委員會
四川省|南充市|南部縣|碾埡鄉|佛祖溝村村委會
廣東省|清遠市|清城區|橫荷街道辦事處|佛祖社區居委會


以`菩薩`二字開頭的村子

province|city|country|town|village
---|---|---|---|---
山西省|臨汾市|蒲縣|黑龍關鎮|菩薩凹村委會
河南省|南陽市|淅川縣|荊紫關鎮|菩薩堂村民委員會
河北省|邢臺市|內丘縣|南賽鄉|菩薩嶺村委會
河北省|保定市|淶水縣|趙各莊鎮|菩薩峪村村委會
河北省|石家莊市|井陘縣|辛莊鄉|菩薩崖村委會
遼寧省|丹東市|東港市|菩薩廟鎮|菩薩廟村委會
山東省|青島市|膠州市|三里河街道辦事處|菩薩廟村委會
河北省|衡水市|武強縣|武強鎮|菩薩村委會
四川省|南充市|高坪區|御史鄉|菩薩村村民委員會
陝西省|商洛市|鎮安縣|月河鎮|菩薩殿村委會
四川省|資陽市|樂至縣|佛星鎮|菩薩灣村村民委員會
四川省|涼山彝族自治州|會東縣|堵格鎮|菩薩箐村村民委員會
北京市|市轄區|昌平區|流村鎮|菩薩鹿村委會


以`閻王`二字開頭的村子

province|city|country|town|village
---|---|---|---|---
四川省|南充市|儀隴縣|瓦子鎮|閻王坡村委會
四川省|南充市|嘉陵區|大通鎮|閻王溝村村民委員會
山東省|萊蕪市|萊城區|羊裡鎮|閻王石村委會


以`神仙`二字開頭的村子

province|city|country|town|village
---|---|---|---|---
湖南省|婁底市|漣源市|渡頭塘鎮|神仙侖村
湖北省|黃岡市|浠水縣|關口鎮|神仙衝村委會
湖南省|長沙市|瀏陽市|集裡街道|神仙坳社區居委會
湖南省|永州市|道縣|仙子腳鎮|神仙頭村委會
湖南省|婁底市|新化縣|溫塘鎮|神仙嶺村委會
湖南省|永州市|零陵區|七裡店街道|神仙嶺社區
河南省|南陽市|南召縣|四棵樹鄉|神仙崖村委會
貴州省|銅仁市|石阡縣|龍塘鎮|神仙廟村委會
北京市|市轄區|通州區|於家務回族鄉|神仙村委會
四川省|成都市|武侯區|芳草街道辦事處|神仙樹社區居委會
湖南省|常德市|桃源縣|陬市鎮|神仙橋村委會
湖南省|永州市|東安縣|白牙市鎮|神仙橋村村委會
四川省|成都市|金堂縣|轉龍鎮|神仙橋村民委員會
四川省|瀘州市|瀘縣|太伏鎮|神仙橋社區居民委員會
吉林省|延邊朝鮮族自治州|汪清縣|天橋嶺鎮|神仙洞村委會
河南省|鄭州市|新密市|尖山風景區管理委員會|神仙洞村委會
湖南省|永州市|江華瑤族自治縣|白芒營鎮|神仙洞村委會
廣東省|汕頭市|潮南區|仙城鎮|神仙裡村委會



## Problems
* 文末`^M`出現，使用`tr -s "\r\n" "\n"`將其去除
* Shell腳本中自定義函數必須先定義，而後才能調用，位置有先後。將函數返回值賦值給變量，可使用'`function_name`'實現，反引號包裹函數名


## Change Log
* 2016.03.14 00:01 Mon Asia/Beijing
    * 初稿完成
* 2016.03.17 20:34 Thu Asia/Beijing
    * 數據表設計優化
* 2016.11.11 09:10 Fri Asia/Shanghai
    * 數據源更新、數據表結構優化、代碼優化
* 2018.04.22 20:20 Sun America/Boston
    * 勘誤，重構，更新，遷移到新Blog


[nbs]:http://www.stats.gov.cn

<!-- end -->
