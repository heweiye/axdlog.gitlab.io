---
title: Multidimensional Arrays in awk
slug: Using Multidimensional Arrays in awk
date: 2016-03-06T19:07:04+08:00
lastmod: 2018-07-31T22:26:18+08:00
draft: false
keywords: ["awk"]
description: "Multidimensional Arrays in awk"
categories:
- Explanation
tags:
- awk
comment: true
toc: true

---

本人於 Jan 20, 2016 開始研究過`awk`多維數組的使用，操作參考[Arnold Robbins](http://www.skeeve.com/)撰寫的[Effective awk Programming, 4th Edition](http://shop.oreilly.com/product/0636920033820.do)，`Chapter 8. Arrays in awk`下的`Multidimensional Arrays`部分，另見於Gnu Awk官方文檔 [8.5 Multidimensional Arrays](https://www.gnu.org/software/gawk/manual/html_node/Multidimensional.html#Multidimensional)。

<!--more-->

## Introductions
### Single-dimensional Array
>An array is a table of values called `elements`. The elements of an array are distinguished by their indices. Indices may be either numbers or strings. --Page172

>The `awk` language provides one-dimensional arrays for storing groups of related strings or numbers. Every `awk` array must have a name. Array names have the same syntax as variable names; any valid variable name would also be a valid array name. But one name cannot be used in both ways (as an array and as a variable) in the same `awk` program.  --Page173

>Arrays in `awk` superficially resemble arrays in other programming languages, but there are fundamental differences. In `awk`, it isn’t necessary to specify the size of an array before starting to use it. Additionally, any number or string, not just consecutive integers, may be used as an array index. --Page173

`awk`中，數組長度不用預先定義，數組索引可以是數值或字符串

>A reference to an element that does not exist automatically creates that array element, with the null string as its value. --page176

### Multidimensional Array
>A multidimensional array is an array in which an element is identified by a sequence of indices instead of a single index. For example, a two-dimensional array requires two indices. The usual way (in many languages, including `awk`) to refer to an element of a two-dimensional array named grid is with grid[x,y] . --page185

>Multidimensional arrays are supported in `awk` through concatenation of indices into one string. `awk` converts the indices into strings and concatenates them together, with a **separator** between them. This creates a single string that describes the values of the separate indices. The combined string is used as a single index into an ordinary, one-dimensional array. The separator used is the value of the built-in variable `SUBSEP` .--page185

>The default value of `SUBSEP` is the string `\034` , which contains a nonprinting character that is unlikely to appear in an awk program or in most input data. The usefulness of choosing an unlikely character comes from the fact that index values that contain a string matching `SUBSEP` can lead to combined strings that are ambiguous. --page186

>There is no special for statement for scanning a “multidimensional” array. There cannot be one, because, in truth, awk does not have multidimensional arrays or elements—there is only *a multidimensional way of accessing an array*. --page187

>However, if your program has an array that is always accessed as multidimensional, you can get the effect of scanning it by combining the scanning for statement with the built-in `split()` function. It works in the following manner: --page187

```
for (combined in array) {
    split(combined, separate, SUBSEP)
    ...
}
```

>This sets the variable combined to each concatenated combined index in the array, and splits it into the individual indices by breaking it apart where the value of `SUBSEP` appears. The individual indices then become the elements of the array `separate` .--page187

其中`separate[1]`對應第一個indice，`separate[2]`對應第二個indice，依次類推。

### Attentions
`awk`中測試元素是否存在，假設數組名`arr`，查選的索引名爲`lemp`

不可以使用`if (arr["lemp"] != "")`形式，如果不存在索引名爲`lemp`的值，則`awk`會在數組`arr`中自動創建索引名爲`lemp`，值爲null。

可以使用`indx in array`形式，如`if (lemp in arr)`

刪除使用 `delete array[index-expression]`形式

`awk`中的多維數組本質是一維數組，通過separator（分隔符）將`indice`聯結成一個字符串，使用時再還原出來。而分隔符就是內置變量`SUBSEP`的值，默認是`\034`。

使用內置函數`split()`將`indice`從字符串中還原。


## Example
需求：列出`ps -ef`中UID出現的次數並分別對PID值求和

想要實現的效果是這種格式

```bash
 username | count | sum
---------------------------
maxdsre     77       960724  
rtkit      1        716     
colord     1        1860
postfix    2        25721   
root       154      450382  
apache     10       29567
...
...
...  
zabbix     33       105059  
dbus       1        732     
============================
```

### Test1

```bash
[maxdsre@lemp ~]$ ps -ef | awk -v FS=' ' '$1~/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}END{for(i in user){split(i,j,SUBSEP);print j[1],user[j[1],"count"],user[j[1],"sum"]}}' | sort
apache 10 29567
apache 10 29567
avahi 2 1418
avahi 2 1418
chrony 1 741
chrony 1 741
colord 1 1860
colord 1 1860
dbus 1 732
dbus 1 732
maxdsre 78 988545
maxdsre 78 988545
libstor+ 1 698
libstor+ 1 698
mysql 1 1516
mysql 1 1516
nobody 1 1787
nobody 1 1787
polkitd 1 773
polkitd 1 773
postfix 2 25721
postfix 2 25721
root 156 505086
root 156 505086
rtkit 1 716
rtkit 1 716
zabbix 33 105059
zabbix 33 105059
[maxdsre@lemp ~]$
```

發現結果出現重複


### Test2
直接打印`j[1],j[2]`

```bash
[maxdsre@lemp ~]$ ps -ef | awk '/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}END{for(i in user){split(i,j,SUBSEP);print j[1],j[2]}}' | sort
apache count
apache sum
avahi count
avahi sum
chrony count
chrony sum
colord count
colord sum
dbus count
dbus sum
maxdsre count
maxdsre sum
libstor+ count
libstor+ sum
mysql count
mysql sum
nobody count
nobody sum
polkitd count
polkitd sum
postfix count
postfix sum
root count
root sum
rtkit count
rtkit sum
zabbix count
zabbix sum
[maxdsre@lemp ~]$
```

發現`j[2]`分別代表了`count`和`sum`，故對應的UID會出現2次。

`awk`中是對`indice`進行拼接組成字符串，不能對值進行該操作。暫時沒有想到有效的解決方案，使用`uniq`去重


### Test3

```bash
[maxdsre@lemp ~]$ ps -ef | awk -v FS=' ' '$1~/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}END{for(i in user){split(i,j,SUBSEP);print j[1],user[j[1],"count"],user[j[1],"sum"]}}' | sort | uniq
apache 10 29567
avahi 2 1418
chrony 1 741
colord 1 1860
dbus 1 732
maxdsre 79 1016953
libstor+ 1 698
mysql 1 1516
nobody 1 1787
polkitd 1 773
postfix 2 25721
root 154 450433
rtkit 1 716
zabbix 33 105059
[maxdsre@lemp ~]$
```
但這樣沒有辦法加表頭和表尾。

### Test4
使用`!a[$0]++`

```bash
[maxdsre@lemp ~]$ ps -ef | awk -v FS=' ' '$1~/^[^UID]/{user[$1,"count"]+=1;user[$1,"sum"]+=$2}BEGIN{print " username | count | sum\n---------------------------"}END{for(i in user){split(i,j,SUBSEP);printf "%-10s %-8d %-8d\n",j[1],user[j[1],"count"],user[j[1],"sum"]}}END{print "============================"}' | awk '!a[$0]++'
 username | count | sum
---------------------------
maxdsre     81       1076787
rtkit      1        716     
colord     1        1860    
postfix    2        25721   
root       155      482698  
apache     10       29567   
chrony     1        741     
libstor+   1        698     
avahi      2        1418    
zabbix     33       105059  
polkitd    1        773     
dbus       1        732     
nobody     1        1787    
mysql      1        1516    
============================
[maxdsre@lemp ~]$
```
成功實現去重，但使用了2次`awk`命令，感覺不是很優雅，且還未實現按指定字段排序，暫且先這樣。

關於`!a[$0]++`的含義，稍後會整理一篇Blog介紹。


## References
* [How does awk '!a[$0]++' work?](http://unix.stackexchange.com/questions/159695/how-does-awk-a0-work)

## Change Logs
* 2016.03.06 22:55 Sun Asia/Beijing
    * 初稿完成
* 2018.07.31 22:31:26 Tue Asia/Shanghai
    * 勘誤，排版，遷移到新Blog

<!-- End -->
