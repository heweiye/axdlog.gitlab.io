---
title: Using Sed To Reverse Character Or Line On GNU/Linux
slug: Using Sed To Reverse Character Or Line On GNU Linux
date: 2017-06-15T11:27:01+08:00
lastmod: 2018-04-12T09:42:01-04:00
draft: false
keywords: ["AxdLog", "sed"]
description: "Using GNU sed to reverse characters or lines on GNU/Linux"
categories:
- Data Process
tags:
- sed

comment: true
toc: true

---

[GNU sed][gnused]是一款文本流編輯器(stream editor)，功能強大，官方文檔 [sed, a stream editor][sedmanual]。網路上也有許多不錯的教程，[USEFUL ONE-LINE SCRIPTS FOR SED (Unix stream editor)][sed1line]便是其中之一。本文主要討論其中 *字符串逆序排列* (character reverse)和 *文本行倒置* (line reverse，功能類似`tac`命令)的實現過程。

<!--more-->

* 字符串逆序排列命令： `sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//'`；
* 文本行倒置命令：
    * `sed '1!G;h;$!d'`；
    * `sed -n '1!G;h;$p'`；


## Character Reverse
```bash
sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//'

# 改寫後形式
sed -r '/\n/!G;s@(.)(.*\n)@&\2\1@;//D;s@.@@'
```

### Command Explanation
網路上已有對該命令的解釋

* [sed: '//D' command](https://stackoverflow.com/questions/9703152/sed-d-command#9703275 'StackOverfoow')
* [Sed One-Liners Explained, Part I: File Spacing, Numbering and Text Conversion and Substitution](http://www.catonmat.net/blog/sed-one-liners-explained-part-one/)的`37. Reverse a line (emulates "rev" Unix command).`部分

相關指令定義如下：

1. `G`:  **append** *hold space* to *pattern space*；
2. `D`:  delete text in the pattern space up to the first newline；
3. `!`表是取反；
4. `;`用於分隔操作，順序執行；
5. 圓括號`()`之前的反斜線`\`表示轉義，如果使用`-r`，則無需使用`\`進行轉義；
6. `//`表示使用正則匹配，如果`//`中不包含任何內容，只是單純的`//`，則表示最近一次使用正則匹配到的內容。官方說明見：

>The empty regular expression `//` repeats the last regular expression match (the same holds if the empty regular expression is passed to the s command). Note that modifiers to regular expressions are evaluated when the regular expression is compiled, thus it is invalid to specify them together with the empty regular expression. -- [4.3 selecting lines by text matching](https://www.gnu.org/software/sed/manual/sed.html#Regexp-Addresses)


#### Step 1
`/\n/!G`表示對不含有換行符`\n`的數據行，將 *hold space* 中的內容追加到 *pattern space* ，如果 *hold space* 爲空，則相當於在每行的下一行添加了一空行。

#### Step 2
`s/\(.\)\(.*\n\)/&\2\1/`表示通過正則查找匹配內容並實現替換。`.`表示任意不爲`\n`的單個字符，`.*`表示任意多個不爲`\n`的單個字符(>=0)。`\(.\)`、`\(.*\n\)`用圓括號包裹起來表示稍後爲用到，分別對應其後的`\1`、`\2`，這種使用方式叫`back-reference`，具體見官方文檔[5.7 Back-references and Subexpressions](https://www.gnu.org/software/sed/manual/sed.html#Back_002dreferences-and-Subexpressions)。

此處字符 **`&`** 非常重要，表示最近一次使用正則匹配到的內容，即命令中的`\(.\)\(.*\n\)`(或`\1\2`)，爲匹配到的內容則仍按最開始的位置顯示。例如字符串`dcba`，第一次作用後的格式爲`dcba\ncba\nd`，經過`//D`作用，第二次的格式是`cba\nba\cd`，第二次作用後顯示的內容中，最後一個字符`d`是未用正則匹配到的內容，照常顯示，即第二次的`&\2\1`是`cba\nba\c`，輸出的內容加上未被匹配到的字符`d`，形成`cba\nba\cd`，這非常重要。

#### Step 3
`//D`是4個步驟中最爲重要的步驟，上文簡要介紹過，單純的`//`表示最近一次使用正則匹配到的內容`\(.\)\(.*\n\)`(或`\1\2`)。即`//`等同於`/\(.\)\(.*\n\)/`。字符`D`表示刪除 *pattern space* 中第一個`\n`之前的內容，如`dcba\ncba\nd`，`//`爲`/dcba\n/`，經`//D`作用後爲`cba\nd`。

`//D`會對字符串進行 **循環** 操作，操作命令即前一個命令`s/\(.\)\(.*\n\)/&\2\1/`。當無匹配內容時，則跳出開始執行下一個命令。

#### Step 4
`s/.//`表示刪除行首爲`newline`字符串的行，可理解爲是空行。


### Divided Process
構建文件`/tmp/sed.txt`，內容如下

```
4321
```

分解過程如下

count|`s/\(.\)\(.*\n\)/&\2\1/`|`//D`
---|---|---
1|`4321\n321\n4`|`321\n4`
2|`321\n21\n3`+`4`|`21\n34`
3|`21\n1\n2`+`34`|`1\n234`
4|`1\n\n1`+`234`|`\n1234`

生成的字符串`\n1234`經過`s/.//`處理得到`1234`。

實例演示

```bash
maxdsre@Stretch:~$ cat /tmp/sed.txt
dcba
4321
erSdxaM
maxdsre@Stretch:~$ sed '/\n/!G;s/\(.\)\(.*\n\)/&\2\1/;//D;s/.//' /tmp/sed.txt
abcd
1234
MaxdSre
maxdsre@Stretch:~$ sed -r '/\n/!G;s@(.)(.*\n)@&\2\1@;//D;s@.@@' /tmp/sed.txt
abcd
1234
MaxdSre
maxdsre@Stretch:~$
```


## Line Reverse
```bash
sed '1!G;h;$!d'
```
### Command Explanation
相關指令定義如下：

1. `G`:  **append** *hold space* to *pattern space*
2. `h`:  **copy** *pattern space* to *hold space*
3. `d`:  **delete** *pattern space*.  start next cycle.
4. `!`表是取反；
5. `;`用於分隔操作，順序執行。

### Divided Process
構建文件`/tmp/sed.txt`，內容如下

```
a
b
c
d
```

分解過程如下

pattern|hold|pattern|hold|pattern|hold|pattern|hold
---|---|---|---|---|---|---|---
||`1!G`|`1!G`|`h`|`h`|`$!d`|`$!d`
a| |a| |a|a| |a
b|a|b\na|a|b\na|b\na| |b\na
c|b\na|c\nb\na|b\na|c\nb\na|c\nb\na| |c\nb\na
d|c\nb\na|d\nc\nb\na|c\nb\na|d\nc\nb\na|d\nc\nb\na|d\nc\nb\na|d\nc\nb\na


完整解釋：

1. 對第一行之外的所有數據行，將當前讀取行的 *hold space* 中內容追加到 *pattern space* 中；
2. 將當前讀取行的 *pattern space* 中內容複製到 *hold space* 中；
3. 除了最後一行，清空其餘的數據行的 *pattern space*；
4. 到最後一行時，輸出 *pattern space* 中內容。

實例演示

```bash
maxdsre@Stretch:~$ cat /tmp/sed.txt
a
b
c
d
maxdsre@Stretch:~$ sed = /tmp/sed.txt | sed 'N;s@\n@ @'
1 a
2 b
3 c
4 d
maxdsre@Stretch:~$ sed '1!G;h;$!d' /tmp/sed.txt
d
c
b
a
maxdsre@Stretch:~$ sed -n '1!G;h;$p' /tmp/sed.txt
d
c
b
a
maxdsre@Stretch:~$
```


## Bibliography
* [Sed One-Liners Explained](http://www.catonmat.net/series/sed-one-liners-explained)
* [How does the command sed '1!G;h;$!d' reverse the contents of a file?](https://unix.stackexchange.com/questions/233014/how-does-the-command-sed-1ghd-reverse-the-contents-of-a-file 'StackOverflow')


## Change Logs
* 2017.06.15 11:27 Thu Asia/Shanghai
    * 初稿完成
* 2018.04.12 09:42 Wed America/Boston
    * 勘誤，遷移到新Blog


[gnused]:https://www.gnu.org/software/sed/ "GNU sed"
[sedmanual]:https://www.gnu.org/software/sed/manual/sed.html
[sed1line]:http://pement.org/sed/sed1line.txt


<!-- End -->
