---
title: Cleaning Text Data Which Delimiter Is Special Character Using Sed & Awk
date: 2018-03-09T01:26:21-04:00
lastmod: 2018-04-11T10:31:21-04:00
categories:
- Production Case
tags:
- awk
- sed
toc: true
---

需要對一份招聘相關的數據樣本(160多萬行)進行數據分析，妻子在用Python處理時出現報錯。經過分析,原因出在`分隔符`、`換行符`等特殊符號上，此外還有一條數據分散爲兩行的情況。本人通過對異常數據進行分析，整理出特殊情況類別，在GNU/Linux系統下用`sed`和`awk`對數據進行初步清洗，實現預期要求。

<!--more-->

## Analysis

1. 數據樣本有近160多萬行，有11個字段;
2. 字段間分隔符爲製表空格符`\t`(vim中通過`set list`顯示爲`^I`，可通過 *Ctrl+V+I* 組合鍵輸出)；
3. 回車換行符`\r\n`(vim中顯示爲`^M`，可通過 *Ctrl+V+M* 組合鍵輸出)，該字符後有斜線`\`，表示後一行與該行本應該是一行，須將其合併爲一行；
4. 有多個`\^I`連續在一起或夾雜在字符串中，經過分析，可將其視爲空格；

樣本中示例

origin item|usge
---|---
`\^I^I`|`^I`
`Jonathan\^IBartlett`|`Jonathan IBartletts`
`\^I\^I^I`|`^I`
`\^I\^I\^I^I`|`^I`

源文件分析

```bash
# file /tmp/members.tsv
/tmp/members.tsv: UTF-8 Unicode text

# wc /tmp/members.tsv
  1607521  22258307 165505405 /tmp/members.tsv

# 按分隔符`^I`切分， 左側是出現次數，右側是每一行的字段數
# awk -F"   " '{print NF}' /tmp/members.tsv | sort | uniq -c
1607506 11
      9 12
      1 13
      1 14
      2 4
      2 8

# awk 'END{print NR}' /tmp/members.tsv
1607521
```

## Processing
完整操作命令

* 源文件 /tmp/members.tsv
* 臨時文件 /tmp/members_temp.tsv
* 處理後文件 /tmp/members_new.tsv


```bash
# 通過sed的`N`特性，合併含有 ^M\ 的行及後一行， 將結果輸出到臨時文件
sed -r -n '/^M\\$/{N;s@\n@@g;s@^M\\@@g;p};s@[[:blank:]]*$@@g;p' /tmp/members.tsv > /tmp/members_temp.tsv

awk -F"   " 'NF!=11{print $0}' /tmp/members_temp.tsv | sed -r -n 's@\\     @ @g;p' > /tmp/members_new.tsv

awk -F"   " 'NF!=11{print $0}' /tmp/members_temp.tsv | sed -r -n 's@\\     @ @g;p' > /tmp/members_new.tsv
```

### Step 1
去除`^M\`，將之後行與該行合併爲一行

```bash
sed -r -n '/^M\\$/{N;s@\n@@g;s@^M\\@@g;p};s@[[:blank:]]*$@@g;p' /tmp/members.tsv > /tmp/members_temp.tsv
```

```bash
awk -F"   " '{print NF}' /tmp/members_temp.tsv | sort | uniq -c
1607510 11
      9 12
      1 13
      1 14
```

### Step 2
提取字段不是11個的，單獨處理，將`\^I`轉換爲空格，寫入文件

```bash
awk -F"   " 'NF!=11{print $0}' /tmp/members_temp.tsv | sed -r -n 's@\\     @ @g;p' > /tmp/members_new.tsv
```

```bash
awk -F"   " '{print NF}' /tmp/members_new.tsv | sort | uniq -c
     11 11
```

此處會出現一個情況：被替換掉的`\^I`變成了空格，會導致分隔符兩邊有空格，此處暫不做處理。

### Step 3
提取members_temp.tsv中字段爲11個的數據到/tmp/members_new.tsv中

```bash
awk -F"   " 'NF==11{print}' /tmp/members_temp.tsv >> /tmp/members_new.tsv
```

```bash
awk -F"   " '{print NF}' /tmp/members_new.tsv | sort | uniq -c
1607521 11
```

### Setp 4
校驗

```bash
wc -l /tmp/members.tsv /tmp/members_new.tsv
  1607521 /tmp/members.tsv
  1607521 /tmp/members_new.tsv
  3215042 total
```

## Data Example
通過命令
```bash
awk -F"   " 'NF!=11{print $0}' /tmp/members_temp.tsv
```

提取出的異常數據

```
11175037^I2012-01-16 12:25:10^Iyahoo.com^Imakke\^I^I\N^I\N^I60000^IBachelor^I2001^I0.0891283^IAdmin Assistant Job$
11294840^I2012-01-24 10:06:25^Isbcglobal.net^Ikim\^I^ILIVONIA^IMI^I48154^IAssociate^I1984^I0.169424^Inursing home application$
12267559^I2012-04-05 20:28:20^IYahoo.com^IRoberta\^I^INEWTON UPPER FALLS^IMA^I02464^ISome College^I1974^I0.280936^Irent-a-center careers$
14597145^I2012-08-22 01:18:02^Iameritrade.com^IJonathan\^IBartlett^ISAN DIEGO^ICA^I92101^ISome College^I2009^I0.220914^IHiring$
14884873^I2012-09-05 17:45:41^Igmail.com^ISabrina\^I^IPORT SAINT LUCIE^IFL^I34953^IGED^I2008^I0.266815^Imcdonalds career$
14901409^I2012-09-06 12:58:19^Iyahoo.com^IRegina\^I\^I^ITUSCALOOSA^IAL^I35401^IHS^I1992^I0.217793^Iwalmart application$
14913467^I2012-09-06 18:00:01^Iyahoo.com^IKaimba\^I^ILAS VEGAS^INV^I89147^ISome HS^I2013^I0.119687^Isubway.com$
15001091^I2012-09-11 13:08:26^Ihotmail.com^I \^Ikiramat^I\N^I\N^I25000^IMaster^I1991^I0.116966^Ijobs opportunities$
15073984^I2012-09-13 18:01:59^Ilive.com^Isameh\^I^IASTORIA^INY^I11102^ISome HS^I1989^I0.096791^Iwalmart application$
15198033^I2012-09-19 10:17:00^Iyahoo.com^ITyrone\^I\^I\^I^IJACKSONVILLE^IFL^I32209^ISome College^I1999^I0.313548^IKelly Services Job$
15225560^I2012-09-20 10:31:29^Iyahoo.com^Isameh\^I^IASTORIA^INY^I11102^IHS^I1997^I0.236401^Iwalmart application$
```

經過處理後的數據

```
11175037^I2012-01-16 12:25:10^Iyahoo.com^Imakke ^I\N^I\N^I60000^IBachelor^I2001^I0.0891283^IAdmin Assistant Job$
11294840^I2012-01-24 10:06:25^Isbcglobal.net^Ikim ^ILIVONIA^IMI^I48154^IAssociate^I1984^I0.169424^Inursing home application$
12267559^I2012-04-05 20:28:20^IYahoo.com^IRoberta ^INEWTON UPPER FALLS^IMA^I02464^ISome College^I1974^I0.280936^Irent-a-center careers$
14597145^I2012-08-22 01:18:02^Iameritrade.com^IJonathan Bartlett^ISAN DIEGO^ICA^I92101^ISome College^I2009^I0.220914^IHiring$   
14884873^I2012-09-05 17:45:41^Igmail.com^ISabrina ^IPORT SAINT LUCIE^IFL^I34953^IGED^I2008^I0.266815^Imcdonalds career$
14901409^I2012-09-06 12:58:19^Iyahoo.com^IRegina  ^ITUSCALOOSA^IAL^I35401^IHS^I1992^I0.217793^Iwalmart application$
14913467^I2012-09-06 18:00:01^Iyahoo.com^IKaimba ^ILAS VEGAS^INV^I89147^ISome HS^I2013^I0.119687^Isubway.com$
15001091^I2012-09-11 13:08:26^Ihotmail.com^I  kiramat^I\N^I\N^I25000^IMaster^I1991^I0.116966^Ijobs opportunities$
15073984^I2012-09-13 18:01:59^Ilive.com^Isameh ^IASTORIA^INY^I11102^ISome HS^I1989^I0.096791^Iwalmart application$
15198033^I2012-09-19 10:17:00^Iyahoo.com^ITyrone   ^IJACKSONVILLE^IFL^I32209^ISome College^I1999^I0.313548^IKelly Services Job$
15225560^I2012-09-20 10:31:29^Iyahoo.com^Isameh ^IASTORIA^INY^I11102^IHS^I1997^I0.236401^Iwalmart application$
```

結果比對

```bash
# awk -F"   " '{print NF}' /tmp/111 | sort | uniq -c
      9 12
      1 13
      1 14

# awk -F"   " '{print NF}' /tmp/222 | sort | uniq -c
     11 11
```

## Change Log
* 2018.03.08 12:21 Thu America/Boston
    * 初稿完成
* 2018.04.11 10:31 Wed America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
