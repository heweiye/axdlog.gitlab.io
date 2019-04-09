---
title: Merge Multiple Lines Into One Line Via Specific String On GNU/Linux
slug: Merge Multiple Lines Into One Line Via Specific String On GNU Linux
date: 2016-09-22T10:49:06+08:00
lastmod: 2018-07-27T13:52:16-04:00
draft: false
keywords: ["awk", "sed"]
description: "Using sed and awk to merge multiple lines into one line on GNU/Linux"
categories:
- Data Process
tags:
- awk
- sed
comment: true
toc: true

---

處理文本數據時經常會遇到這種情況：單條記錄(`Record`)的各維度數據分散在相鄰的多行中，而非在一行中。`Record`之間以一相同的數據行作爲分隔標誌，比如`...`或`,,,`之類的符號。在GNU/Linux中可通過`awk`和`sed`實現多行合併爲一行，即每行爲一個`Record`。本文記錄具體實現方式和解釋說明。

<!--more-->

## System Info
操作系統信息

Item|Details
---|---
OS|`Debian GNU/Linux 9.5 (stretch)`
Kernel|`4.9.0-7-amd64`

軟件信息

Software|Version
---|---
[awk](https://www.gnu.org/software/gawk/ 'GNU awk')|`GNU Awk 4.1.4`
[sed](https://www.gnu.org/software/sed/ 'GNU sed')|`sed (GNU sed) 4.4`

**注**： 在`*nix`系統中，換行符號默認爲`\n`。

## Data Preparation
測試數據準備，創建文件`/tmp/data.txt`，其內容如下

```txt
1
2
3
4
...
aa
bb
cc
...
Mon
Tue
Wed
...
Jan
Feb
Mar
Asia
Africa
Europe
...
192.168.1.1
192.168.1.2
192.168.1.3
192.168.1.4
...
AxdLog
is my
personal
blog
...
```

數據行`...`爲各`Record`之間的分隔標誌，需將其處理成如下格式

```txt
1 2 3 4
aa bb cc
Mon Tue Wed
Jan Feb Mar Asia Africa Europe
192.168.1.1 192.168.1.2 192.168.1.3 192.168.1.4
AxdLog is my personal blog
```

## Via awk
通過`awk`實現該需求

### Command
以下是操作命令

```bash
awk '{if($0!~/^[.]+/){ORS=" ";print $0}else{printf "\n"}}' /tmp/data.txt
```

### Operation Procedure
操作過程

```bash
maxdsre@Stretch:~$ awk '{ if($0!~/^[.]+/){ORS=" ";print $0}else{printf "\n"} }' /tmp/data.txt
1 2 3 4
aa bb cc
Mon Tue Wed
Jan Feb Mar Asia Africa Europe
192.168.1.1 192.168.1.2 192.168.1.3 192.168.1.4
AxdLog is my personal blog
maxdsre@Stretch:~$ awk '{ if($0!~/^[.]+/){ORS=",";print $0}else{printf "\n"} }' /tmp/data.txt
1,2,3,4,
aa,bb,cc,
Mon,Tue,Wed,
Jan,Feb,Mar,Asia,Africa,Europe,
192.168.1.1,192.168.1.2,192.168.1.3,192.168.1.4,
AxdLog,is my,personal,blog ,
maxdsre@Stretch:~$ awk '{ if($0!~/^[.]+/){ORS="|";print $0}else{printf "\n"} }' /tmp/data.txt
1|2|3|4|
aa|bb|cc|
Mon|Tue|Wed|
Jan|Feb|Mar|Asia|Africa|Europe|
192.168.1.1|192.168.1.2|192.168.1.3|192.168.1.4|
AxdLog|is my|personal|blog |
maxdsre@Stretch:~$
```

可以看到，該命令可設置不同的分隔符號，這樣做的好處是可以明顯區分含有空格的維度數據，如`AxdLog|is my|personal|blog |`，默認的`AxdLog is my personal blog`則無法區分。

如果要去除每行末尾的`|`或`,`，可藉助`sed`實現，比如

```bash
maxdsre@Stretch:~$ awk '{ if($0!~/^[.]+/){ORS=",";print $0}else{printf "\n"} }' /tmp/data.txt | sed -r 's@,$@@g'
1,2,3,4
aa,bb,cc
Mon,Tue,Wed
Jan,Feb,Mar,Asia,Africa,Europe
192.168.1.1,192.168.1.2,192.168.1.3,192.168.1.4
AxdLog,is my,personal,blog
maxdsre@Stretch:~$ awk '{ if($0!~/^[.]+/){ORS="|";print $0}else{printf "\n"} }' /tmp/data.txt | sed -r 's@\|$@@g'
1|2|3|4
aa|bb|cc
Mon|Tue|Wed
Jan|Feb|Mar|Asia|Africa|Europe
192.168.1.1|192.168.1.2|192.168.1.3|192.168.1.4
AxdLog|is my|personal|blog
maxdsre@Stretch:~$
```

### Explanation
命令解釋

```bash
awk '{ if($0!~/^[.]+/){ORS=" ";print $0}else{printf "\n"} }' /tmp/data.txt
```

1. 通過`awk`中的條件判斷`if`進行條件分析，if語句需包裹在`'{ }'`中；
2. 參數`$0`代表整行數據，`~`代表模式匹配，`!`代表取反；
4. `/^[.]+/`是正則表達式，代表以逗點`.`開頭，且逗點至少有一個，此正則表達式用於匹配分隔各`Record`的數據行；
5. `ORS`代表 *output record seperator* (輸出換行符號)，默認是`\n`，故`ORS=" "`的含義是將輸出換行符更換爲空格(`" "`)；
6. `print $0`表示輸出整行數據；
7. `printf "\n"`表示輸出換行符號`\n`；

**整個命令的含義** 即：

* 通過判斷每一行數據是否爲分隔各`Record`的數據行：
    * 如果 *不是*，則說明是`Record`的維度數據，將輸出分隔符號從默認的`\n`更換爲空格(`" "`)並將其打印，實現同一`Record`下各維度數據的拼接；
    * 如果 *是*，則說明是分隔各`Record`的數據行，需將其刪除或隱藏，再以此位置爲基準，設置各`Record`之間的換行符`\n`，通過`printf "\n"`直接打印換行符`\n`；

最終實現預期效果。


## Via sed
通過`sed`實現

### Command
以下是操作命令

```bash
sed -r ':a;N;$!ba;s@\n@ @g;s@ [.]+[[:space:]]?@\n@g;' /tmp/data.txt

xargs -a /tmp/data.txt | sed -r 's@ [.]+[[:space:]]?@\n@g;'
```

單純使用`sed`的命令參考自[Command Line Magic](https://twitter.com/climagic 'Twitter')的[twitter](https://twitter.com/climagic/status/778598536195690496)。

### Operation Procedure
操作過程

```bash
maxdsre@Stretch:~$ xargs -a /tmp/data.txt | sed -r 's@ [.]+[[:space:]]?@\n@g;'
1 2 3 4
aa bb cc
Mon Tue Wed
Jan Feb Mar Asia Africa Europe
192.168.1.1 192.168.1.2 192.168.1.3 192.168.1.4
AxdLog is my personal blog

maxdsre@Stretch:~$ sed -r ':a;N;$!ba;s@\n@ @g;s@ [.]+[[:space:]]?@\n@g;' /tmp/data.txt
1 2 3 4
aa bb cc
Mon Tue Wed
Jan Feb Mar Asia Africa Europe
192.168.1.1 192.168.1.2 192.168.1.3 192.168.1.4
AxdLog is my personal blog

maxdsre@Stretch:~$ sed -r ':a;N;$!ba;s@\n@|@g;s@[|][.]+[|]?@\n@g;' /tmp/data.txt
1|2|3|4
aa|bb|cc
Mon|Tue|Wed
Jan|Feb|Mar|Asia|Africa|Europe
192.168.1.1|192.168.1.2|192.168.1.3|192.168.1.4
AxdLog|is my|personal|blog

maxdsre@Stretch:~$ sed -r ':a;N;$!ba;s@\n@,@g;s@[,][.]+[,]?@\n@g;' /tmp/data.txt
1,2,3,4
aa,bb,cc
Mon,Tue,Wed
Jan,Feb,Mar,Asia,Africa,Europe
192.168.1.1,192.168.1.2,192.168.1.3,192.168.1.4
AxdLog,is my,personal,blog

maxdsre@Stretch:~$
```

使用`sed`會導致最後多處一行空行，仍可通過`sed`將其去除

```bash
maxdsre@Stretch:~$ sed -r ':a;N;$!ba;s@\n@,@g;s@[,][.]+[,]?@\n@g;' /tmp/data.txt | sed '/^$/d'
1,2,3,4
aa,bb,cc
Mon,Tue,Wed
Jan,Feb,Mar,Asia,Africa,Europe
192.168.1.1,192.168.1.2,192.168.1.3,192.168.1.4
AxdLog,is my,personal,blog
maxdsre@Stretch:~$
```

### Explanation

原理與使用`awk`的思路類似，具體分析參見 [How can I replace a newline (\n) using sed?](http://stackoverflow.com/questions/1251999/how-can-i-replace-a-newline-n-using-sed#answer-7697604)。

命令解釋

```bash
sed -r ':a;N;$!ba;s@\n@ @g;s@ [.]+[[:space:]]?@\n@g;' /tmp/data.txt
```

1. 選項`-r`表示使用擴展性正則(extended regular expressions);
2. `:a`表示設置名稱為`a`的label，之後的`b`表示無條件判斷自動跳轉到設置的label；
3. `N`表示將新讀取的行添加(append)入 *pattern space*;
4. `$!ba`表示如果不是最後一行，則分支(branch)`ba`跳轉到label `a`;
5. `s@\n@ @g;`表示將換行符號`\n`替換為空格，`s`表替換，`g`為flag，表全局;
6. `s@ [.]+[[:space:]]?@\n@g;`表示將各Record的分隔符替換為換行符`\n`;


>n N    Read/append the next line of input into the pattern space.

**注意**： 使用`xargs`和`sed`的方法弊病很大，只能用空格做默認分隔符，如果使用其它符號做分隔符則無法實現。


## Tutorials
以下是`sed`相關的教程

* [sed, a stream editor - GNU](http://www.gnu.org/software/sed/manual/sed.html)
* [Linux and Unix sed command](http://www.computerhope.com/unix/used.htm)
* [Sed Tutorial](https://www.tutorialspoint.com/sed/ 'Tutorials Point')
* [sed tutorial](http://www.panix.com/~elflord/unix/sed.html)
* [Learning Linux Commands: sed](https://linuxconfig.org/learning-linux-commands-sed)
* [Sed - An Introduction and Tutorial by Bruce Barnett](http://www.grymoire.com/Unix/Sed.html)
* [Tutorials/Sed - Debian Wiki](https://wiki.debian.org/Tutorials/Sed 'Debian')


### Bibliography

* [LFCS: How to use GNU 'sed' Command to Create, Edit, and Manipulate files in Linux – Part 1](http://www.tecmint.com/sed-command-to-create-edit-and-manipulate-files-in-linux/ 'Tecmint')
* [15 Useful 'sed' Command Tips and Tricks for Daily Linux System Administration Tasks](http://www.tecmint.com/linux-sed-command-tips-tricks/ 'Tecmint')
* [The Basics of Using the Sed Stream Editor to Manipulate Text in Linux](https://www.digitalocean.com/community/tutorials/the-basics-of-using-the-sed-stream-editor-to-manipulate-text-in-linux 'Digital Ocean')
* [The Basics of Using the Sed Stream Editor to Manipulate Text in Linux](https://www.digitalocean.com/community/tutorials/the-basics-of-using-the-sed-stream-editor-to-manipulate-text-in-linux#tutorial_series_41 'Digital Ocean')
* [Is there a basic tutorial for grep, awk and sed?](http://unix.stackexchange.com/questions/2434/is-there-a-basic-tutorial-for-grep-awk-and-sed)
* [Turning multiple lines into one line with comma separated (Perl/Sed/AWK)](http://stackoverflow.com/questions/15758814/turning-multiple-lines-into-one-line-with-comma-separated-perl-sed-awk)

### Sed Tips and Tricks
在[The Geek Stuff](http://www.thegeekstuff.com/)中有一個[Sed Tips and Tricks](http://www.thegeekstuff.com/tag/sed-tips-and-tricks/)系列教程

1. [Unix Sed Tutorial: Printing File Lines using Address and Patterns](http://www.thegeekstuff.com/2009/09/unix-sed-tutorial-printing-file-lines-using-address-and-patterns/ 'The Geek Stuff')
2. [Unix Sed Tutorial: Delete File Lines Using Address and Patterns](http://www.thegeekstuff.com/2009/09/unix-sed-tutorial-delete-file-lines-using-address-and-patterns/ 'The Geek Stuff')
3. [Unix Sed Tutorial: Find and Replace Text Inside a File Using RegEx](http://www.thegeekstuff.com/2009/09/unix-sed-tutorial-replace-text-inside-a-file-using-substitute-command/ 'The Geek Stuff')
4. [Unix Sed Tutorial: How To Write to a File Using Sed](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-how-to-write-to-a-file-using-sed/ 'The Geek Stuff')
5. [Unix Sed Tutorial: How To Execute Multiple Sed Commands](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-how-to-execute-multiple-sed-commands/ 'The Geek Stuff')
6. [Unix Sed Tutorial: Multi-Line File Operation with 6 Practical Examples](http://www.thegeekstuff.com/2009/11/unix-sed-tutorial-multi-line-file-operation-with-6-practical-examples/ 'The Geek Stuff')
7. [Unix Sed Tutorial: Append, Insert, Replace, and Count File Lines](http://www.thegeekstuff.com/2009/11/unix-sed-tutorial-append-insert-replace-and-count-file-lines/ 'The Geek Stuff')
8. [Unix Sed Tutorial : 7 Examples for Sed Hold and Pattern Buffer Operations](http://www.thegeekstuff.com/2009/12/unix-sed-tutorial-7-examples-for-sed-hold-and-pattern-buffer-operations/ 'The Geek Stuff')
9. [Unix Sed Tutorial: Advanced Sed Substitution Examples](http://www.thegeekstuff.com/2009/10/unix-sed-tutorial-advanced-sed-substitution-examples/ 'The Geek Stuff')
10. [Unix Sed Tutorial: 6 Examples for Sed Branching Operation](http://www.thegeekstuff.com/2009/12/unix-sed-tutorial-6-examples-for-sed-branching-operation/ 'The Geek Stuff')

---
## Change Logs
* 2016.09.22 18:48 Thu Asia/Shanghai
    * 初稿完成
* 2016.11.23 17:05 Wed Asia/Shanghai
    * 爲`sed`命令添加解釋說明鏈接[How can I replace a newline (\n) using sed?](http://stackoverflow.com/questions/1251999/how-can-i-replace-a-newline-n-using-sed#answer-7697604)
* 2017.01.05 14:26 Thu Asia/Shanghai
    * 為`sed`的label使用添加解釋
* 2018.07.27 14:15:42 Fri America/Boston
    * 勘誤，遷移到新Blog


<!-- End -->
