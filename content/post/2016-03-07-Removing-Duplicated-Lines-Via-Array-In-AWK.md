---
title: Removing Duplicated Lines Via Array In AWK
slug: Removing Duplicated Lines Via Array In AWK
date: 2016-03-07T11:38:52+08:00
lastmod: 2018-07-31T21:55:18+08:00
draft: false
keywords: ["AxdLog", "awk"]
description: "!a[$0]++ - Removing Duplicated Lines In AWK"
categories:
- Explanation
tags:
- awk
comment: true
toc: true

---

在awk中可通過`!a[$0]++`刪除重複行，本文對其進行具體分析，以瞭解其實現原理。

<!--more-->

## Introductions
在`awk`中

* `$0`代表整行數據，`$1`代表每行數據使用分隔符分割後的第一個字段，`$2`代表代表第二個字段，依次類推，而`NF`即代表每行的字段數；
* `array[index­expression]`代表數組，定義數組不用預先定義數組長度，`index­expression`是索引，可以是數值或字符串；
* `!`是邏輯操作符，取反，是`非`的意思；
* `++`是賦值操作符，先運算，後賦值(自增1)；
* `bool`判斷爲`false`的有：數字`0`、空字符串`""`、未定義變量（根據使用情況，對應轉換爲數字`0`、空字符串`""`）
* 默認的action(動作)是`{print $0}`，打印整行，不指定即代表打印整行

舉例說明

```bash
[maxdsre@lemp ~]$ ps -ef | awk -v FS=' ' '$1~/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}END{for(i in user){split(i,j,SUBSEP);print j[1],user[j[1],"count"],user[j[1],"sum"]}}' | sort
apache 5 9145
apache 5 9145
avahi 2 1470
avahi 2 1470
chrony 1 734
chrony 1 734
colord 1 1863
colord 1 1863
dbus 1 703
dbus 1 703
maxdsre 80 250833
maxdsre 80 250833
libstor+ 1 701
libstor+ 1 701
mysql 1 1518
mysql 1 1518
nobody 1 1790
nobody 1 1790
polkitd 1 854
polkitd 1 854
postfix 2 3311
postfix 2 3311
root 156 107248
root 156 107248
rtkit 1 711
rtkit 1 711
zabbix 33 184938
zabbix 33 184938
[maxdsre@lemp ~]$
```

## Inverse Derivation
嘗試從結果推導過程

抽出以下數據進行分析

```bash
apache 5 9145
apache 5 9145
```

`awk`的默認的action(動作)是`{print $0}`（打印整行），不指定即代表打印整行。

暫不清楚打印的是第幾行，但可以確定必須打印一行。

### Assumption
對第一行而言，`a[$0]`以整行數據爲數組`a`的index，因爲未給定具體元素值，屬於未定義變量，其默認值可能是數值`0`或空字符串`""`，故`a[$0]`本身的bool判斷是`false`。

假設先執行`++`再執行`！`，則先自增變成1，再取反變成0，則第一行的bool判斷是`false`，第一行不打印

在第一行不打印的前提下，可推得打印的是第二行。

第一行的`a[$0]`因自增變成1，在第二行順序進行`++`和`！`，第二行的bool判斷是`false`，即第二行不打印，得到的結果是兩行數據都不輸出，這與必須打印一行的前提矛盾。

故而可以確定是先執行`!`再執行`++`（先取反變成1，再自增變成2？？？），第一行的bool判斷是`true`，第一行打印。

綜上所述，可得到以下結論

1. `!`和`++`的執行順序是，先執行`!`再執行`++`
2. 打印第一行


根據只打印第一行，可得到`第一行的bool判斷結果爲true，而第二行的bool判斷結果爲false`的結果

則可復原操作過程

1. 對第一行，`a[$0]`的bool值是false，取反後變成1，bool值是true，打印，自增後變成2
2. 對第二行，`a[$0]`是2，取反後後變成0，bool值是false，不打印，自增後變成1

這是暫時的推論，尚不知是否正確，需要驗證。

### Verification
驗證

```bash
[maxdsre@lemp ~]$ awk 'BEGIN{a=0;print !a++,a}'
1 1
[maxdsre@lemp ~]$ awk 'BEGIN{a=1;print !a++,a}'
0 2
[maxdsre@lemp ~]$ awk 'BEGIN{a=2;print !a++,a}'
0 3
[maxdsre@lemp ~]$ awk 'BEGIN{a=3;print !a++,a}'
0 4
[maxdsre@lemp ~]$ awk 'BEGIN{a=4;print !a++,a}'
0 5
[maxdsre@lemp ~]$
```

* `a=0`時，`!a++`是1，bool值true，`a`變成1
* `a=1`時，`!a++`是0，bool值false，`a`變成2
* `a=2`時，`!a++`是0，bool值false，`a`變成3
* `a=3`時，`!a++`是0，bool值false，`a`變成4
* `a=4`時，`!a++`是0，bool值false，`a`變成5

分析

* `a=0`時，`!a++`變成1，可理解爲是因爲`!`取反後得到1，而`a`未變成2而是1，則說明`!a`和`a++`是分開的，`!a`的值判斷整行是否未true，`a++`的值是`a`當前的值，互不影響。（`!`和`++`的先後執行順序，仍可通過假設推導得出是先執行`!`，後執行`++`）
* 根據第一條分析結果來分析`a=1`，`!a`取反變成0，整行bool值爲false，`a++`將`a`的值變成2
* `a=2`，`!a`取反變成0，整行bool值爲false，`a++`將`a`的值變成3
* `a=3`，`!a`取反變成0，整行bool值爲false，`a++`將`a`的值變成4
* `a=4`，`!a`取反變成0，整行bool值爲false，`a++`將`a`的值變成5


### Conclusion
根據假設和驗證，可得出以下結論：

1. `!`和`++`的執行順序是先執行`!a`再執行`a++`，但二者互不影響
2. `!a`和`a++`中用於計算的`a`是初始`a`值
3. `!a`決定了整行的bool判斷，`a++`決定了計算後`a`的值
4. 打印第一行


```bash
[maxdsre@lemp ~]$ ps -ef | awk -v FS=' ' '$1~/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}BEGIN{print " username | count | sum\n---------------------------"}END{for(i in user){split(i,j,SUBSEP);printf "%-10s %-8d %-8d\n",j[1],user[j[1],"count"],user[j[1],"sum"]}}END{print "============================"}' | awk '!a[$0]++'
 username | count | sum
---------------------------
maxdsre     79       260273  
rtkit      1        711     
colord     1        1863    
postfix    2        9866    
root       154      134370  
apache     5        31260   
chrony     1        734     
libstor+   1        701     
avahi      2        1470    
zabbix     33       184938  
polkitd    1        854     
dbus       1        703     
nobody     1        1790    
mysql      1        1518    
============================
[maxdsre@lemp ~]$
```

## References
* [How does awk '!a[$0]++' work?](http://unix.stackexchange.com/questions/159695/how-does-awk-a0-work)
* [awk '!a[$0]++'去重原理分析](http://sapser.github.io/shell/2014/08/07/awk-remove-duplicate-analyze)


## Change Logs
* 2016.03.07 11:30 Mon Asia/Beijing
    * 初稿完成
* 2018.07.31 22:07:26 Tue Asia/Shanghai
    * 勘誤，排版，遷移到新Blog


[gawk]:https://www.gnu.org/software/gawk/

<!-- End -->
