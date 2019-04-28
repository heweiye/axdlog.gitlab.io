---
title: Using Shell Script To Generate Markdown TOC On GNU/Linux
slug: Using Shell Script To Generate Markdown TOC On GNU Linux
date: 2016-01-29T19:20:30+08:00
lastmod: 2019-04-28T14:18:18-04:00
draft: false
keywords: ["markdown", "toc"]
description: "Using Shell Script To Generate TOC For Markdown File On GNU/Linux"
categories:
- Markdown
- Data Process
tags:
- markdown
- Shell Script
comment: true
toc: true

---

本人日常使用[Markdown][markdown]進行筆記書寫，創建TOC(目錄)可起到內容索引的作用，但有些編輯器或平臺(eg:GitHub)並不支持TOC，故需要手動創建TOC目錄。~~當筆記內容量很大時，這項工作耗時很長，很影響學習效率。在對如何創建TOC有一定心得後，想通過Shell Script來實現自動生成TOC。~~ 爲提高效率，使用Shell Script自動生成TOC。

關於[Markdown][markdown]的語法介紹，可參閱本人的 『[A Complete Intrduction To Markdown Syntax]({{< relref "2016-01-25-A-Complete-Introduction-To-Markdown-Syntax.md" >}})』。

<!--more-->

## Shell Script
代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/markdownTOCGeneration.sh)。

使用方式如下

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/markdownTOCGeneration.sh | bash -s --


# Usage:
#     script [options] ...
#     script | bash -s -- [options] ...
# Markdown TOC Generating On GNU/Linux!
#
# [available option]
#     -h          --help, show help info
#     -q          --quiet, quiet mode, don't output anything to screen
#     -f file     --file path specified
#     -t type     --type name (git|gitlab|github|hexo), default is git
#     -o order    --order list type (ul|ol), default is 'ol'
```

默認爲Git格式，目錄採用數字列表格式。

FILEPATH 建議使用絕對路徑，如果其中含有空格，須用雙引號`""`將其包裹。

## Thinking
實現思路

* 使用`$1`判斷輸入的文件路徑是否存在
* 讀取源文件，獲取以`#`開頭的行，通過管道符`|`輸出給`while`循環
* 獲取去除`#`後的數據並進行處理
* 將處理好的數據進行拼接，生成所需的TOC目錄

對字符串的處理主要通過bash字符串處理工具，去除特殊標點符號對TOC目錄的影響(Markdown中的TOC對某些特殊符號採取忽略策略)。


## Example
此處以本人的blog [A Complete Intrduction To Markdown Syntax]({{< relref "2016-01-25-A-Complete-Introduction-To-Markdown-Syntax.md" >}}) 爲例。

### Source
索引列表如下

```txt
## Intrduction
## Block Elements syntax
### Heading
### Paragraphs and Line Breaks
### Music and Video
#### Hexo/Hugo
#### GitHub
### Blockquotes
#### style1
#### style2
#### style3
#### style4
#### style5
#### style6
#### style7
### Lists
#### Ordered Lists
#### Unordered Lists
### Code Blocks
#### style1
#### style2
### HTML Entities
### Tables
### Horizonal Rules
## Inline Elements
### Links
#### Inline-style
#### Reference-style
### Images
### Emphasis
### Code Highlight
## Table Of Contents
### Shell Script
## GitHub With Markdown
### YouTube Video Thumbnail
## References
## Bibliography
## Change logs
```

### TOC Generate
生成的TOC如下

#### ol
```txt
1. [Intrduction](#intrduction)  
2. [Block Elements syntax](#block-elements-syntax)  
2.1 [Heading](#heading)  
2.2 [Paragraphs and Line Breaks](#paragraphs-and-line-breaks)  
2.3 [Music and Video](#music-and-video)  
2.3.1 [Hexo/Hugo](#hexohugo)  
2.3.2 [GitHub](#github)  
2.4 [Blockquotes](#blockquotes)  
2.4.1 [style1](#style1)  
2.4.2 [style2](#style2)  
2.4.3 [style3](#style3)  
2.4.4 [style4](#style4)  
2.4.5 [style5](#style5)  
2.4.6 [style6](#style6)  
2.4.7 [style7](#style7)  
2.5 [Lists](#lists)  
2.5.1 [Ordered Lists](#ordered-lists)  
2.5.2 [Unordered Lists](#unordered-lists)  
2.6 [Code Blocks](#code-blocks)  
2.6.1 [style1](#style1-1)  
2.6.2 [style2](#style2-1)  
2.7 [HTML Entities](#html-entities)  
2.8 [Tables](#tables)  
2.9 [Horizonal Rules](#horizonal-rules)  
3. [Inline Elements](#inline-elements)  
3.1 [Links](#links)  
3.1.1 [Inline-style](#inline-style)  
3.1.2 [Reference-style](#reference-style)  
3.2 [Images](#images)  
3.3 [Emphasis](#emphasis)  
3.4 [Code Highlight](#code-highlight)  
4. [Table Of Contents](#table-of-contents)  
4.1 [Shell Script](#shell-script)  
5. [GitHub With Markdown](#github-with-markdown)  
5.1 [YouTube Video Thumbnail](#youtube-video-thumbnail)  
6. [References](#references)  
7. [Bibliography](#bibliography)  
8. [Change logs](#change-logs)  
```

#### ul

```txt
1. [Intrduction](#intrduction)  
2. [Block Elements syntax](#block-elements-syntax)  
    * [Heading](#heading)  
    * [Paragraphs and Line Breaks](#paragraphs-and-line-breaks)  
    * [Music and Video](#music-and-video)  
        * [Hexo/Hugo](#hexohugo)  
        * [GitHub](#github)  
    * [Blockquotes](#blockquotes)  
        * [style1](#style1)  
        * [style2](#style2)  
        * [style3](#style3)  
        * [style4](#style4)  
        * [style5](#style5)  
        * [style6](#style6)  
        * [style7](#style7)  
    * [Lists](#lists)  
        * [Ordered Lists](#ordered-lists)  
        * [Unordered Lists](#unordered-lists)  
    * [Code Blocks](#code-blocks)  
        * [style1](#style1-1)  
        * [style2](#style2-1)  
    * [HTML Entities](#html-entities)  
    * [Tables](#tables)  
    * [Horizonal Rules](#horizonal-rules)  
3. [Inline Elements](#inline-elements)  
    * [Links](#links)  
        * [Inline-style](#inline-style)  
        * [Reference-style](#reference-style)  
    * [Images](#images)  
    * [Emphasis](#emphasis)  
    * [Code Highlight](#code-highlight)  
4. [Table Of Contents](#table-of-contents)  
    * [Shell Script](#shell-script)  
5. [GitHub With Markdown](#github-with-markdown)  
    * [YouTube Video Thumbnail](#youtube-video-thumbnail)  
6. [References](#references)  
7. [Bibliography](#bibliography)  
8. [Change logs](#change-logs)  
```

---
## Change Log
* 2016.01.29 18:00 Fri Asia/Beijing
    * 初稿完成
* 2016.02.02 21:58 Tue Asia/Beijing
    * 使用case函數實現按目錄層級編號
* 2016.08.11 16:11 Thu Asia/Shanghai
    * 解決重複出現的標題的索引問題
    * 解決代碼塊中符號`#`被腳本視爲標題的問題
* 2017.02.22 11:44 Wed Asia/Shanghai
    * 代碼優化、添加使用格式說明、添加爲Git/Hexo添加對應格式索引、支持數字、非數字列表目錄
* 2017.04.11 17:57 Tue Asia/Shanghai
    * 解決符號`.`,`/`在awk中引起報錯，sed中使用字符類處理特殊符號
* 2019.04.28 14:16 Sun America/Boston
    * 勘誤、更新，遷移到新Blog


[markdown]:https://daringfireball.net/projects/markdown/ "Markdown"

<!-- End -->
