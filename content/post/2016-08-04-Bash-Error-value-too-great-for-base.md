---
title: value too great for base (error token is "08")
slug: bash error value too great for base
date: 2016-08-04T20:40:17+08:00
lastmod: 2018-08-01T22:55:42+08:00
draft: false
keywords: ["bash"]
description: "Bash error: bash error value too great for base (error token is \"08\")"
categories:
- Bash
tags:
- bash

comment: true
toc: true

---

在用Bash Shell下載[GRE](http://www.majortests.com/gre/wordlist.php)資料時，出現如下報錯

```bash
value too great for base (error token is "08")
```

查閱資料後方知與 **數字進制** 有關。

<!--more-->

默認情況下`08`以數字`0`開頭，Shell會嘗試將其解釋爲8進制(octal)，而非一個真實的數字(valid number)，導致報錯。解決方案是在變量之前添加`10#`，強制讓bash將其解釋爲10進制(decimal)

>Prepend the string "10#" to the front of your variables. That forces bash to treat them as decimal, even though the leading zero would normally make them octal. --- [Bash error: value too great for base (error token is “09”)](http://stackoverflow.com/questions/21049822/bash-error-value-too-great-for-base-error-token-is-09#answer-21050240)


## Origin
想查找GRE常用詞彙(vocabulary)，點開了[majortests.com](http://www.majortests.com/)的[GRE Word Lists](http://www.majortests.com/gre/wordlist.php)，共1500個詞彙，分爲15個`wordlist`。

>Each of the 15 wordlists contains 100 important words.

前10個屬於`Basic GRE Words`，後5個屬於`Advanced GRE Words`，提供PDF下載，鏈接形式爲

```bash
http://www.majortests.com/word-lists/word-list-01.pdf
...
...
http://www.majortests.com/word-lists/word-list-15.pdf
```

直接通過瀏覽器另存爲下載太過繁瑣且低效，故打算通過Shell Script實現。

### Requirements
想要實現的需求

1. 通過Shell Scirpt實現整個操作過程；
2. 能夠查看下載進度；
3. 下載的文件能重命名爲自己想要的形式；


### Analysis
需求分析

1. 循環可通過`for`實現
2. 下載可通過命令`curl`實現
3. 查看下載進度可通過`curl`的參數`-#`實現
4. 文件重命名可通過`curl`的參數`-o`(小寫字母`o`)實現
    * 大寫字母`O`是保留其在遠程服務器中的名字


## Procedure
以下是完整操作過程

### Testing1
第1次測試目的：實現數值從`01`到`15`的自動生成

```sh
for i in {01..15};do
    echo $i
done
```

操作過程
```sh
[testing@axdlog majortests]$ for i in {01..15};do echo $i;done
01
02
03
04
05
06
07
08
09
10
11
12
13
14
15
[testing@axdlog majortests]$
```

### Testing2
第2次測試目的：實現下載時進度條顯示

```sh
for i in {01..15};do
	curl -O# 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
done
```

操作過程

```bash
[testing@axdlog majortests]$ for i in {01..15};do
> curl -O# 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
> done
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
[testing@axdlog majortests]$ ls *.pdf
word-list-01.pdf  word-list-05.pdf  word-list-09.pdf  word-list-13.pdf
word-list-02.pdf  word-list-06.pdf  word-list-10.pdf  word-list-14.pdf
word-list-03.pdf  word-list-07.pdf  word-list-11.pdf  word-list-15.pdf
word-list-04.pdf  word-list-08.pdf  word-list-12.pdf
[testing@axdlog majortests]$
```

### Testing3 Error
第3次測試目的：下載同時實現文件的重命名操作

```bash
for i in {01..15};do
    #類似於三元運算
    [[ $i -le 10 ]] && level='basic' || level='advanced'
    # if [[ $i -le 10 ]]; then
    #     level='basic'
    # else
    #     level='advanced'
    # fi
	curl -# -o 'greWords-'$i'-'$level'.pdf' 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
done
```

操作過程

```
[testing@axdlog majortests]$ for i in {01..15};do
> [[ $i -le 10 ]] && level='basic' || level='advanced'
> curl -# -o 'greWords-'$i'-'$level'.pdf' 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
> done
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
bash: [[: 08: value too great for base (error token is "08")
######################################################################## 100.0%
bash: [[: 09: value too great for base (error token is "09")
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
[testing@axdlog majortests]$ ls *basic*
greWords-01-basic.pdf  greWords-04-basic.pdf  greWords-07-basic.pdf
greWords-02-basic.pdf  greWords-05-basic.pdf  greWords-10-basic.pdf
greWords-03-basic.pdf  greWords-06-basic.pdf
[testing@axdlog majortests]$ ls *advance*
greWords-08-advanced.pdf  greWords-12-advanced.pdf  greWords-15-advanced.pdf
greWords-09-advanced.pdf  greWords-13-advanced.pdf
greWords-11-advanced.pdf  greWords-14-advanced.pdf
[testing@axdlog majortests]$
```

出現報錯

```
bash: [[: 08: value too great for base (error token is "08")
bash: [[: 09: value too great for base (error token is "09")
```
直接導致08, 09文件重命名錯誤

#### Error Analysis

查閱資料後得知是進制原因，`08`以數字`0`開頭，Shell會嘗試將其解釋爲8進制(octal)，不是一個真實的數字(valid number)，導致報錯。

解決方案是在變量之前添加`10#`，強制讓bash將其解釋爲10進制(decimal)

>Prepend the string "10#" to the front of your variables. That forces bash to treat them as decimal, even though the leading zero would normally make them octal. --- [Bash error: value too great for base (error token is “09”)](http://stackoverflow.com/questions/21049822/bash-error-value-too-great-for-base-error-token-is-09#answer-21050240)


### Testing4
第4次測試目的：下載同時實現文件按要求重命名

```sh
for i in {01..15};do
    #類似於三元運算，在變量前添加10#
    [[ 10#$i -le 10 ]] && level='basic' || level='advanced'
	curl -# -o 'greWords-'$i'-'$level'.pdf' 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
done
```

操作過程
```sh
[testing@axdlog majortests]$ for i in {01..15};do
> [[ 10#$i -le 10 ]] && level='basic' || level='advanced'
> curl -# -o 'greWords-'$i'-'$level'.pdf' 'http://www.majortests.com/word-lists/word-list-'$i'.pdf'
> done
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
######################################################################## 100.0%
[testing@axdlog majortests]$ ls *.pdf
greWords-01-basic.pdf  greWords-06-basic.pdf  greWords-11-advanced.pdf
greWords-02-basic.pdf  greWords-07-basic.pdf  greWords-12-advanced.pdf
greWords-03-basic.pdf  greWords-08-basic.pdf  greWords-13-advanced.pdf
greWords-04-basic.pdf  greWords-09-basic.pdf  greWords-14-advanced.pdf
greWords-05-basic.pdf  greWords-10-basic.pdf  greWords-15-advanced.pdf
[testing@axdlog majortests]$
```

最終實現目標


## References
* [Shell Script Error: Value too great for base (error token is “08”) [duplicate]](http://stackoverflow.com/questions/24777597/shell-script-error-value-too-great-for-base-error-token-is-08)
* [Bash error: value too great for base (error token is “09”)](http://stackoverflow.com/questions/21049822/bash-error-value-too-great-for-base-error-token-is-09)
* [convert string into integer in bash script](http://stackoverflow.com/questions/12821715/convert-string-into-integer-in-bash-script/12821845#12821845)


## Change Logs
* 2016.08.04 10:47 Thu Asia/Shanghai
    * 初稿完成
* 2018.08.01 23：07 Wed Asia/Shanghai
    * 勘誤，排版，遷移到新Blog

<!-- End -->
