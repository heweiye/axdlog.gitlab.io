---
title: gensub - AWK String-Manipulation Functions Explanation
slug: gensub AWK String-Manipulation Functions Explanation
date: 2017-02-21T16:59:06+08:00
lastmod: 2018-07-27T15:47:18-04:00
draft: false
keywords: ["AxdLog", "awk"]
description: "gensub - AWK String-Manipulation Functions Explanation"
categories:
- Explanation
tags:
- awk
comment: true
toc: true

---

在OpenSUSE Leap 42.2中使用[Gawk][gawk]的`gensub`函數時出現報錯，但相關命令在其它發行版中正常執行。覈實後發現與Gawk版本有關，出現報錯的版本是`4.1.3`，在該版本中`gensub`的使用方式與之前版本略有區別，經過測試，兼容舊版本。

<!--more-->

## Problem Occuring
`gensub`是[Gawk][gawk]的字符處理函數，通過正則匹配進行字符替換操作。其語法格式爲

```bash
gensub(regexp, replacement, how [, target]) #
```

實例如下

```bash
maxsre@jessie:~$ awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1"," ",$0)}'
4.1.1
maxsre@jessie:~$
```

此前字段`how`可以是空格`" "`，之前使用一直正常。但在OpenSUSE Leap 42.2中操作時，出現如下報錯

```
awk: cmd. line:1: (FILENAME=- FNR=1) warning: gensub: third argument `' treated as 1
```

演示過程如下

```
maxsre@linux-qtva:~> awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1"," ",$0)}'
awk: cmd. line:1: (FILENAME=- FNR=1) warning: gensub: third argument ` ' treated as 1
4.1.3
maxsre@linux-qtva:~> awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1","g",$0)}'
4.1.3
maxsre@linux-qtva:~>
```

查看Gawk版本，爲`4.1.3`。

查閱相關資料發現，在版本`4.1.3`中字段`how`需設置爲字母`g`、`G`或數字，直接使用空字符或提示如上的報錯。

參考資料如下

* [[PATCH] gawk: fix gensub usage](https://sourceware.org/ml/libc-alpha/2015-08/msg00269.html)
* [gawk 4.1.3 gensub() warning?](http://compgroups.net/comp.lang.awk/gawk-4.1.3-gensub-warning/3041407)

官方原文地址

* [9.1.3 String-Manipulation Functions](https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html)

經過測試，在`4.1.3`之前的版本中，字段`how`使用字母`g`、`G`或數字仍有效。測試如下

```bash
maxsre@jessie:~$ awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1","g",$0)}'
4.1.1
maxsre@jessie:~$

$ awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1","g",$0)}'
4.0.2
```

## Official Introduction
官方介紹如下 [link](https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html)

>Search the *target* string target for matches of the regular expression *regexp*. If *how* is a string beginning with `g` or `G` (short for “global”), then replace all matches of *regexp* with *replacement*. Otherwise, *how* is treated as a number indicating which match of regexp to replace. If no target is supplied, use `$0`. It returns the modified string as the result of the function and the original target string is *not* changed.
>
`gensub()` is a general substitution function. Its purpose is to provide more features than the standard `sub()` and `gsub() `functions.
>
`gensub()` provides an additional feature that is not available in `sub()` or `gsub()`: the ability to specify components of a regexp in the replacement text. This is done by using parentheses in the regexp to mark the components and then specifying `\N` in the replacement text, where *N* is a digit from 1 to 9. For example:

```bash
$ awk 'BEGIN{a="abc def";b=gensub(/(.+) (.+)/,"\\2 \\1", "g",a);print b}'
def abc
```

## Simple Usage Explanation
>gensub(regexp, replacement, how [, target]) #

函數`gensub`的使用說明

1. 字段`regexp`用於編寫正則表達式，支持[back reference](http://www.regular-expressions.info/backref.html)，即後向引用；
2. 需後向引用的內容用圓括號`()`包裹，在字段`replacement`中通過`\N`(N爲數值)的形式指定。使用時斜線`\`需進行轉移，即`\\`，故實際的格式是`\\N`。其中`\0`代表匹配行所在的整行數據，`\1`代表匹配行中第1個指定的後向引用，`\2`代表匹配行中第2個指定的後向引用，依次類推。
3. 字段`how`必須設置爲`g`、`G`(表示Global全局替換)或數值(表示替換第幾次出現的匹配字串)；
4. 字段`target`可手動指定，如`$1`,`$2`,`$NF`...如果不指定，則默認表示`$0`，代表匹配行整行數據。

### Production Example
以下Gawk代碼用於統計系統端口使用情況，在CentOS 6.8, 7.3中測試通過。

```bash
sudo ss -tunlp | awk 'NR>1{protocol=$1;port=gensub(/.*:(.*)/,"\\1","g",$5);service=gensub(/.*:\(\("([^,]*)".*/,"\\1","g",$NF);pid=gensub(/[^,]*,(pid=)?([^,]*),.*/,"\\2","g",$NF);printf("%4s %8s %-16s %-6s\n",protocol,port,service,pid)}' | awk '!a[$0]++' | awk '{arr[$0]=$2}BEGIN{printf("%s %4s %-15s %4s\n","Protocol","Port","Service","PID")}END{PROCINFO["sorted_in"]="@val_num_asc";for (i in arr) print i}'
```

Debian

```bash
maxsre@jessie:~$ sed -n -r '/^PRETTY_NAME=/s@PRETTY_NAME="?([^"]*)"?@\1@p' /etc/os-release
Debian GNU/Linux 8 (jessie)
maxsre@jessie:~$ awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1","g",$0)}'
4.1.1
maxsre@jessie:~$ sudo ss -tunlp | awk 'NR>1{protocol=$1;port=gensub(/.*:(.*)/,"\\1","g",$5);service=gensub(/.*:\(\("([^,]*)".*/,"\\1","g",$NF);pid=gensub(/[^,]*,(pid=)?([^,]*),.*/,"\\2","g",$NF);printf("%4s %8s %-16s %-6s\n",protocol,port,service,pid)}' | awk '!a[$0]++' | awk '{arr[$0]=$2}BEGIN{printf("%s %4s %-15s %4s\n","Protocol","Port","Service","PID")}END{PROCINFO["sorted_in"]="@val_num_asc";for (i in arr) print i}'
Protocol Port Service          PID
 tcp       22 sshd             617   
 udp       68 dhclient         9983  
 udp      123 chronyd          689   
 udp      323 chronyd          689   
 udp      631 cups-browsed     648   
 tcp      631 cupsd            2192  
 udp     1900 minissdpd        687   
 udp     5353 avahi-daemon     612   
 udp     5353 chrome           3886  
 udp    39229 avahi-daemon     612   
 udp    45187 systemd-timesyn  452   
 udp    49013 dhclient         9983  
 udp    54263 dhclient         9983  
 udp    55069 avahi-daemon     612   
maxsre@jessie:~$
```

OpenSUSE

```bash
maxsre@linux-qtva:~> sed -n -r '/^PRETTY_NAME=/s@PRETTY_NAME="?([^"]*)"?@\1@p' /etc/os-release
openSUSE Leap 42.2
maxsre@linux-qtva:~> awk --version | awk 'NR==1{print gensub(/^GNU Awk ([^,]*).*/,"\\1","g",$0)}'
4.1.3
maxsre@linux-qtva:~> sudo ss -tunlp | awk 'NR>1{protocol=$1;port=gensub(/.*:(.*)/,"\\1","g",$5);service=gensub(/.*:\(\("([^,]*)".*/,"\\1","g",$NF);pid=gensub(/[^,]*,(pid=)?([^,]*),.*/,"\\2","g",$NF);printf("%4s %8s %-16s %-6s\n",protocol,port,service,pid)}' | awk '!a[$0]++' | awk '{arr[$0]=$2}BEGIN{printf("%s %4s %-15s %4s\n","Protocol","Port","Service","PID")}END{PROCINFO["sorted_in"]="@val_num_asc";for (i in arr) print i}'
Protocol Port Service          PID
 tcp       22 sshd             883   
 udp       68 dhclient         6858  
 udp      323 chronyd          801   
 tcp      631 cupsd            870   
 udp     1082 dhclient         6858  
 udp     5353 avahi-daemon     789   
 udp    27969 dhclient         6858  
 udp    56736 avahi-daemon     789   
 udp    59534 avahi-daemon     789   
maxsre@linux-qtva:~>
```


## Change Logs
* 2017.02.21 16:16 Tue Asia/Shaghai
    * 初稿完成
* 2018.07.27 15:50:26 Fri America/Boston
    * 勘誤，遷移到新Blog


[gawk]:https://www.gnu.org/software/gawk/

<!-- End -->
