---
title: Predefined Array Scanning Orders In AWK
slug: Predefined Array Scanning Orders In AWK
date: 2016-03-06T19:07:04+08:00
lastmod: 2018-07-31T22:10:18+08:00
draft: false
keywords: ["AxdLog", "awk"]
description: "PROCINFO[\"sorted_in\"] - Predefined Array Scanning Orders In AWK"
categories:
- Explanation
tags:
- awk
comment: true
toc: true

---


Linux運維面試中常出現諸如 **從給定的文件中提取指定字段，根據出現次數逆序顯示** 之類等試題。此問題可通過`cut`、`sort`實現。

但`awk`可實現相同功能，本文將根據Gnu Awk官方文檔中的 [8.1.6 Using Predefined Array Scanning Orders with gawk](https://www.gnu.org/software/gawk/manual/html_node/Controlling-Scanning.html#Controlling-Scanning)，討論`awk`(gawk)中的`Predefined Array Scanning Orders`（預定義數據掃描排序），這是`gawk`的一個特性，通過`PROCINFO["sorted_in"]`實現。

<!--more-->

## Example
如以下示例，按照IP出現次數進行逆向排序

```bash
[maxdsre@lemp ~]$ cat /tmp/test.txt
192.168.0.1
192.168.0.2
192.168.0.3
192.168.0.4
192.168.0.5
192.168.0.6
192.168.0.5
192.168.0.5
192.168.0.3
192.168.0.2
192.168.0.4
192.168.0.2
192.168.0.8
192.168.0.6
192.168.0.1
192.168.0.4
192.168.0.5
192.168.0.3
192.168.0.3
192.168.0.5
192.168.0.5
```

### Common Solution
常規解決方法

```bash
[maxdsre@lemp ~]$ awk '{arr[$1]++}END{for(i in arr) {print i,arr[i]}}' /tmp/test.txt | sort -t' ' -k2 -rn
192.168.0.5 6
192.168.0.3 4
192.168.0.4 3
192.168.0.2 3
192.168.0.6 2
192.168.0.1 2
192.168.0.8 1
[maxdsre@lemp ~]$
```

### Just using awk
在`awk`中添加參數`PROCINFO["sorted_in"]="@val_num_desc";`實現數組元素排序

```bash
[maxdsre@lemp ~]$ awk '{arr[$1]++}END{PROCINFO["sorted_in"]="@val_num_desc";for(i in arr) {print i,arr[i]}}' /tmp/test.txt
192.168.0.5 6
192.168.0.3 4
192.168.0.2 3
192.168.0.4 3
192.168.0.1 2
192.168.0.6 2
192.168.0.8 1
[maxdsre@lemp ~]$
```


## Predefined Array Scanning Orders

### Introductions
`Predefined Array Scanning Orders`功能主要針對`gawk`。

默認情況下，`awk`中使用for循環遍歷數組的順序是未定義的，即`awk implementation`決定了數組遍歷的順序。可通過 *數組索引* 排序，也可通過 *數組元素* 排序，`gawk`提供了2中機制來實現：

* 設置`PROCINFO["sorted_in"]`爲gawk預先定義的值
* 設置`PROCINFO["sorted_in"]`爲用戶定義的用於比較數組元素的函數

本文討論的是`predefined value`（預定義值）

### Predefined Values
可分爲3大類：`默認行爲`、`按索引排序`、`按元素排序`。`按索引排序`可細分爲 *按數值排序* 、*按字符串排序*；`按元素排序`可細分爲 *按數值排序* 、*按字符串排序*、*按數值類型排序*；


| Value | Explanation |
| :--- | :--- |
| `@unsorted` | arbitrary order(任意排序)，是awk默認行爲 |
| `@ind_num_asc` | 升序，所有數組索引被視作 **數值** 進行排序 |
| `@ind_num_desc` | 逆序，所有數組索引被視作 **數值** 進行排序 |
| `@ind_str_asc` | 升序，所有數組索引被視作 **字符串** 進行排序，數組內部索引通常視作字符串 |
| `@ind_str_desc` | 逆序，所有數組索引被視作 **字符串** 進行排序，數組內部索引通常視作字符串 |
| `@val_num_asc` | 升序，所有數組元素被視作 **數值** 進行排序，subarray（子數組）最後出現，當數值相同時，比較字符串值 |
| `@val_num_desc` | 逆序，所有數組元素被視作 **數值** 進行排序，subarray最先出現，當數值相同時，比較字符串值 |
| `@val_str_asc` | 升序，所有數組元素被視作 **字符串** 進行排序，subarray最後出現 |
| `@val_str_desc` | 逆序，所有數組元素被視作 **字符串** 進行排序，subarray最先出現 |
| `@val_type_asc` | 升序，所有數組元素按元素分配的類型進行排序，先後順序：數值、字符串、子數組 |
| `@val_type_desc` | 逆序，所有數組元素按元素分配的類型進行排序，先後順序：子數組、字符串、數值 |


### Attentions
* `PROCINFO["sorted_in"]`的值是全局的，會印象所有數組的for循環遍歷。如果想在自己的代碼中改變該值，可通過如下方式

```
#判斷並存儲sorted_in值
...
if ("sorted_in" in PROCINFO) {
    save_sorted = PROCINFO["sorted_in"]
    PROCINFO["sorted_in"] = "@val_str_desc" # or whatever
}
...

#恢復sorted_in值
if (save_sorted)
    PROCINFO["sorted_in"] = save_sorted
```

此代碼來自[Effective awk Programming, 4th Edition](http://shop.oreilly.com/product/0636920033820.do) Page182。


* `PROCINFO["sorted_in"]`的默認值是`"@unsorted"`，如果想恢復默認設置，可爲`PROCINFO["sorted_in"]`設置值爲null string(空字符串)或使用`delete`語句從數組PPROCINFO中將`"sorted_in"`刪除


## Testing
以此文件爲示例

```bash
[maxdsre@lemp ~]$ cat /tmp/sort.txt
18 1
28 lemp
38 stacker
48 8lemp
58 lemp8
lempq 1
lempb stacker
lemph lover
lempl 6lemp
lempy lemp6
61lemp 1
62lemp stacker
63lemp lover
64lemp 9lemp
65lemp lemp9
lemp18 1
lemp28 stacker
lemp38 lover
lemp48 7lemp
lemp58 lemp8
[maxdsre@lemp ~]$
```

### unsorted

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{for (i in arr) print i,arr[i]}' /tmp/sort.txt
lempq 1
18 1
lemp18 1
61lemp 1
28 lemp
lemp28 stacker
38 stacker
lemp38 lover
65lemp lemp9
48 8lemp
lemp48 7lemp
63lemp lover
lemph lover
58 lemp8
lemp58 lemp8
62lemp stacker
lempy lemp6
lempl 6lemp
64lemp 9lemp
lempb stacker
[maxdsre@lemp ~]$
```

### ind_num_asc

按索引升序，做數值處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@ind_num_asc";for (i in arr) print i,arr[i]}' /tmp/sort.txt
lemp18 1
lemp28 stacker
lemp38 lover
lemp48 7lemp
lemp58 lemp8
lempb stacker
lemph lover
lempl 6lemp
lempq 1
lempy lemp6
18 1
28 lemp
38 stacker
48 8lemp
58 lemp8
61lemp 1
62lemp stacker
63lemp lover
64lemp 9lemp
65lemp lemp9
[maxdsre@lemp ~]$
```

### ind_num_desc
按索引逆序，做數值處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@ind_num_desc";for (i in arr) print i,arr[i]}' /tmp/sort.txt
65lemp lemp9
64lemp 9lemp
63lemp lover
62lemp stacker
61lemp 1
58 lemp8
48 8lemp
38 stacker
28 lemp
18 1
lempy lemp6
lempq 1
lempl 6lemp
lemph lover
lempb stacker
lemp58 lemp8
lemp48 7lemp
lemp38 lover
lemp28 stacker
lemp18 1
[maxdsre@lemp ~]$
```

### ind_str_asc
按索引升序，做字符串處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@ind_str_asc";for (i in arr) print i,arr[i]}' /tmp/sort.txt
18 1
28 lemp
38 stacker
48 8lemp
58 lemp8
61lemp 1
62lemp stacker
63lemp lover
64lemp 9lemp
65lemp lemp9
lemp18 1
lemp28 stacker
lemp38 lover
lemp48 7lemp
lemp58 lemp8
lempb stacker
lemph lover
lempl 6lemp
lempq 1
lempy lemp6
[maxdsre@lemp ~]$
```

### ind_str_desc
按索引逆序，做字符串處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@ind_str_desc";for (i in arr) print i,arr[i]}' /tmp/sort.txt
lempy lemp6
lempq 1
lempl 6lemp
lemph lover
lempb stacker
lemp58 lemp8
lemp48 7lemp
lemp38 lover
lemp28 stacker
lemp18 1
65lemp lemp9
64lemp 9lemp
63lemp lover
62lemp stacker
61lemp 1
58 lemp8
48 8lemp
38 stacker
28 lemp
18 1
[maxdsre@lemp ~]$
```

### val_num_asc
按元素升序，做數值處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_num_asc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
28      lemp   
lempy   lemp6  
58      lemp8  
lemp58  lemp8  
65lemp  lemp9  
lemp38  lover  
63lemp  lover  
lemph   lover  
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
lempq   1      
18      1      
lemp18  1      
61lemp  1      
lempl   6lemp  
lemp48  7lemp  
48      8lemp  
64lemp  9lemp  
[maxdsre@lemp ~]$
```

### val_num_desc
按元素逆序，做數值處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
64lemp  9lemp  
48      8lemp  
lemp48  7lemp  
lempl   6lemp  
lempq   1      
18      1      
lemp18  1      
61lemp  1      
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
lemp38  lover  
63lemp  lover  
lemph   lover  
65lemp  lemp9  
58      lemp8  
lemp58  lemp8  
lempy   lemp6  
28      lemp   
[maxdsre@lemp ~]$
```

### val_str_asc
按元素升序，做字符串處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_str_asc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
lempq   1      
18      1      
lemp18  1      
61lemp  1      
lempl   6lemp  
lemp48  7lemp  
48      8lemp  
64lemp  9lemp  
28      lemp   
lempy   lemp6  
58      lemp8  
lemp58  lemp8  
65lemp  lemp9  
lemp38  lover  
63lemp  lover  
lemph   lover  
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
[maxdsre@lemp ~]$
```

### val_str_desc
按元素逆序，做字符串處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_str_desc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
lemp38  lover  
63lemp  lover  
lemph   lover  
65lemp  lemp9  
58      lemp8  
lemp58  lemp8  
lempy   lemp6  
28      lemp   
64lemp  9lemp  
48      8lemp  
lemp48  7lemp  
lempl   6lemp  
lempq   1      
18      1      
lemp18  1      
61lemp  1      
[maxdsre@lemp ~]$
```

### val_type_asc
按元素升序，做類型處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_type_asc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
lempq   1      
18      1      
lemp18  1      
61lemp  1      
lempl   6lemp  
lemp48  7lemp  
48      8lemp  
64lemp  9lemp  
28      lemp   
lempy   lemp6  
58      lemp8  
lemp58  lemp8  
65lemp  lemp9  
lemp38  lover  
63lemp  lover  
lemph   lover  
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
[maxdsre@lemp ~]$
```

### val_type_desc
按元素逆序，做類型處理

```bash
[maxdsre@lemp ~]$ awk -v FS=' ' '{arr[$1]=$2}END{PROCINFO["sorted_in"]="@val_type_desc";for (i in arr) printf "%-7s %-7s\n",i,arr[i]}' /tmp/sort.txt
lemp28  stacker
38      stacker
62lemp  stacker
lempb   stacker
lemp38  lover  
63lemp  lover  
lemph   lover  
65lemp  lemp9  
58      lemp8  
lemp58  lemp8  
lempy   lemp6  
28      lemp   
64lemp  9lemp  
48      8lemp  
lemp48  7lemp  
lempl   6lemp  
lempq   1      
18      1      
lemp18  1      
61lemp  1      
[maxdsre@lemp ~]$
```

## Extraction
只取出現次數最高的幾條數據。
如果按出現次數逆序，取前4條，設置自定義變量flag閥值

```bash
[maxdsre@lemp tmp]$ cat test.txt | awk '{arr[$1]+=1}END{flag=0;PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) if(flag<4) {print arr[i],i;flag++ }}'
6 192.168.0.5
4 192.168.0.3
3 192.168.0.2
3 192.168.0.4

[maxdsre@lemp tmp]$ cat test.txt | awk -v flag=0 '{arr[$1]+=1}END{PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) if(flag<4) {print arr[i],i;flag++ }}'
6 192.168.0.5
4 192.168.0.3
3 192.168.0.2
3 192.168.0.4

[maxdsre@lemp tmp]$ awk '{arr[$1]+=1}END{flag=0;PROCINFO["sorted_in"]="@val_num_desc";for (i in arr) if(flag<4) {print arr[i],i;flag++ }}' /tmp/test.txt
6 192.168.0.5
4 192.168.0.3
3 192.168.0.2
3 192.168.0.4
[maxdsre@lemp tmp]$
```


## Change Logs
* 2016.03.06 18:45 Sun Asia/Beijing
    * 初稿完成
* 2016.03.19 18:03 Sat Asia/Beijing
    * 添加Extract，awk中直接截取指定條數
* 2018.07.31 22:24:26 Tue Asia/Shanghai
    * 勘誤，排版，遷移到新Blog


<!-- End -->
