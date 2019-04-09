---
title: A Complete Intrduction To Markdown Syntax
slug: A Complete Intrduction To Markdown Syntax
date: 2016-01-25T22:04:52+08:00
lastmod: 2018-04-11T22:20:02-04:00
draft: false
keywords: ["Markdown syntax", 'Github']
description: "A complete introduction to Markdown syntax"

categories:
- Tutorial
tags:
- Markdown
- GitHub

comment: true
toc: true

---

[Markdown][markdown]是一種輕量級的標記語言，採用純文本語法格式，文件後綴爲`.md`。[Markdown][markdown]應用廣泛，常用於撰寫[README](https://en.wikipedia.org/wiki/README 'WikiPedia')。[Jekyll][jekyll], [Hexo][hexo], [Hugo][hugo]等靜態Blog框架支持Markdown語法，[Digital Ocean][digitalocean]、[Stack Overflow][stackoverflow]等網站也支持Markdown語法。只是各家對Markdown的語法解釋不盡相同，如對`TOC`、`Diagram`、`LaTeX`的支持。

本文詳細介紹[Markdown][markdown]的語法、使用。

<!--more-->


## Intrduction
Markdown由[John Gruber](https://en.wikipedia.org/wiki/John_Gruber 'WikiPedia')於2004年開發，設計主旨爲`可讀性`，語法設計參考Email中的純文本標記。更多介紹參見[維基詞條](https://en.wikipedia.org/wiki/Markdown 'Wikipedia')。

Markdown通過`[:punct:]`(標點符號)來定義語法，同時支持部分[HTML](https://html.spec.whatwg.org/multipage/ 'WHATWG')語法。與`HTML`語法相比，Markdown的語法極爲簡潔。不過二者的語法有對應關係。對照`HTML`中相關元素學習Markdown，有助於理解並掌握Markdown。

在`HTML`語法中，元素分爲`Block Elements`(塊級元素)和`Inline Elements`(行內元素)，Markdown也有相對應的語法。

Markdown官網對於其語法格式的說明

```bash
# Basics
https://daringfireball.net/projects/markdown/basics

# Syntax
https://daringfireball.net/projects/markdown/syntax
```

本文對Markdown的介紹按照 **塊級元素** 和 **行內元素** 的分類進行，額外的功能單獨介紹。


## Block Elements syntax
塊級元素的Markdown語法涵蓋 **標題**、**段落**、**引用**、**列表**、**代碼塊**、**水平分割線** 等。

### Heading
對於標題，`HTML`中[Heading Element](https://html.spec.whatwg.org/multipage/semantics.html#the-h1,-h2,-h3,-h4,-h5,-and-h6-elements)(小標題)共有6個，從`<h1></h1>`到`<h6></h6>`，字體逐漸減小。Markdown中有對應的6個級別的標題，實現方式爲在標題文字前加上對應數量的符號`#`。如一級標題爲在文字前加一個`#`，二級標題爲在文字前加兩個`#`，依次類推。

**提醒**: 強烈建議在符號`#`與文字之間加一個空格，即以空格間隔二者。


`HTML`與Markdown語法格式的對應關係如下

HTML|Markdown
---|---
H1|`#`
H2|`##`
H3|`###`
H4|`####`
H5|`#####`
H6|`######`

HTML中`<H>`是閉合標籤，如`<H1>`必須用`</H1>`閉合，但在Markdown中，此爲可選操作，可不在文字後再添加對應個數的符號`#`。

**重要**：`# H1`爲Title(標題)，`## H2`爲SubTitle(子標題)，Markdown有額外的語法定義此二者：

1. 在文字所在行的下一行添加至少3個符號`=`即代表是 *標題* ；
2. 在文字所在行的下一行添加至少3個符號`-`即代表是 *子標題* ；

對應關係如下

HTML|Type1|Type2
---|---|---
H1|`#`|Text<br>===
H2|`##`|Text<br>---


### Paragraphs and Line Breaks
對於段落，段落間用空行間隔即可。

對於換行，通過換行符`<br />`實現。此標籤在HTML是XML寫法，爲了讓XHTML兼容HTML，也可簡寫爲`<br>`。

此爲使用Markdown語法撰寫的內容

```
I love you, my lover. <br />I miss you, every day and night.
```

Markdown渲染效果如下：

>I love you, my lover. <br />I miss you, every day and night.

### Music and Video
在HTML中，音頻、視頻是通過標籤`<audio>`, `<video>`實現，但對Markdown而言不同的平臺支持性不能不一樣。[Hexo][hexo]、[Hugo][hugo]支持通過標籤`<iframe></iframe>`實現音視頻嵌入。但GitHub到目前爲止尚不支持音視頻的嵌入，但可通過圖片鏈接形式折衷實現。

#### Hexo/Hugo
以[Youtube](https://www.youtube.com 'Youtube')中的[Adele - Rolling in the Deep](https://www.youtube.com/watch?v=rYEDA3JcQqw 'Youtube')爲例

其`<iframe>`代碼是

```html
<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/rYEDA3JcQqw?rel=0" frameborder="0" allowfullscreen></iframe>
```

Markdown渲染效果如下：

<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/rYEDA3JcQqw?rel=0" frameborder="0" allowfullscreen></iframe>


#### GitHub
對於GitHub,如果需要嵌入音、視頻，可參考如下資源：

* [Youtube videos](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#youtube-videos)
* [How to embed a video into GitHub README.md?](https://stackoverflow.com/questions/4279611/how-to-embed-a-video-into-github-readme-md 'Stack Overflow')
* [Embed a YouTube video](https://stackoverflow.com/questions/11804820/embed-a-youtube-video 'Stack Overflow')

實現方式是使用Markdown中插入圖片和超鏈接的語法，將二者結合起來。圖片可以是視頻截圖，通過點擊圖片自動跳轉到視頻的播放地址，實現 **僞** 嵌入。

對於`Youtube`而言，可通過調用`Youtube`的[API](https://developers.google.com/youtube/v3/ 'Google')獲取視頻的縮略圖。(參考自[How do I get a YouTube video thumbnail from the YouTube API?](https://stackoverflow.com/questions/2068344/how-do-i-get-a-youtube-video-thumbnail-from-the-youtube-api 'Stack Overflow'))

此處仍以[Adele - Rolling in the Deep](https://www.youtube.com/watch?v=rYEDA3JcQqw 'Youtube')爲例，在GitHub中語法如下

```markdown
[![](https://img.youtube.com/vi/rYEDA3JcQqw/maxresdefault.jpg 'Adele - Rolling in the Deep')](https://www.youtube.com/watch?v=rYEDA3JcQqw)
```

Markdown渲染效果如下

[![](https://img.youtube.com/vi/rYEDA3JcQqw/maxresdefault.jpg 'Adele - Rolling in the Deep')](https://www.youtube.com/watch?v=rYEDA3JcQqw)


### Blockquotes
對於引用，Markdown在文字前添加符號`>`表示引用。

1. 如果行之間沒有空行，則在第一行的行首添加符號`>`即可將餘下行自動添加到引用的範圍內，直到最近一個空行出現爲止；
2. 如果有多個段落需要引用，在段落間的空行中添加符號`>`可自動將下一段文字添加到引用的範圍內；
3. 也可在需要引用的每一行之前添加符號`>`，包括其中的空行。

**注意**：符號`>>`表示內部引用。

具體分析如下

#### style1
對於一段話，中間沒有空行，可在段首加上符號 `>`，即可引用整段話。

此爲使用Markdown語法撰寫的內容

```
>I love you, my lover.
I miss you, every day and night.
Today is Sun.
```

Markdown渲染效果如下

>I love you, my lover.
I miss you, every day and night.
Today is Sun.

#### style2
對於一段話，如果中間有空行，在段首加上符號`>`，只能第一次空行出現之前的內容。

此爲使用Markdown語法撰寫的內容

```
>I love you, my lover.
I miss you, every day and night.

Today is Sun.
```

Markdown渲染效果如下

>I love you, my lover.
I miss you, every day and night.

Today is Sun.

#### style3
在每行前都加上符號`>`，等效於引用一段話。

此爲使用Markdown語法撰寫的內容

```
>I love you, my lover.
>I miss you, every day and night.
```

Markdown渲染效果如下

>I love you, my lover.
>I miss you, every day and night.

#### style4
在每行前都加上符號`>`（包括空行），等效於包含空行在其中的一段話。

此爲使用Markdown語法撰寫的內容

```
>I love you, my lover.
>
>I miss you, every day and night.
```

Markdown渲染效果如下

>I love you, my lover.
>
>I miss you, every day and night.

#### style5
在每行前都加上符號`>`（不包括空行），等效於包含空行在其中的一段話。

此爲使用Markdown語法撰寫的內容

```
>I love you, my lover.

>I miss you, every day and night.
```

Markdown渲染效果如下

>I love you, my lover.

>I miss you, every day and night.

#### style6
在行首加上2個符號`>`，即`>>`，即表示內部引用。

此爲使用Markdown語法撰寫的內容

```
>To my lover
>>I love you, my lover.

>>I miss you, every day and night.
```

Markdown渲染效果如下

>To my lover
>>I love you, my lover.

>>I miss you, every day and night.


#### style7
在行首加上2個符號`>`，即`>>`，即表示內部引用。

此爲使用Markdown語法撰寫的內容

```
>To my lover
>>I love you, my lover.

>I miss you, every day and night.
```

Markdown渲染效果如下

>To my lover
>>I love you, my lover.

>I miss you, every day and night.


### Lists
在HTML中列表分爲`Ordered List`(有序列表)和`Unordered List`(無序列表)。Markdown的語法如下：

* 有序列表：使用數值，數值後有點號`.`和一個空格，再加上需要排序的字符
* 無序列表：使用星號`*`、加號`+`、減號`-`都可以

#### Ordered Lists
有序列表：數字遞增，通常只需要用數值表示即可(自動渲染爲遞增)，其後加上點號`.`和一個空格。

**注意**：點號`.`和空格必須添加，否則會被認爲不是排列。

此爲使用Markdown語法撰寫的內容

```
1. Monday
2. Tuesday
3. Wensday
4. Thursday
5. Friday
6. Staturday
7. Sunday
```

Markdown渲染效果如下

>1. Monday
2. Tuesday
3. Wensday
4. Thursday
5. Friday
6. Staturday
7. Sunday

此例子很典型，不管數值是否重複或順序，Markdown渲染出來的序號都是遞增的。如果某一行沒有添加數值格式，則Markdown會認爲其仍是上一個數值代表的數據，但會進行格式排版，與上一條數據同一豎直位置。

此爲使用Markdown語法撰寫的內容

```
1. Monday
1. Tuesday
7. Wensday
9. Thursday
Friday
18. Staturday
5. Sunday
```

Markdown渲染效果如下：

>1. Monday
1. Tuesday
7. Wensday
9. Thursday
Friday
18. Staturday
5. Sunday

#### Unordered Lists
無序列表：只需在每行之前加上`*`、`+`、`-`中任一種符號即可，需有空格間隔。

**注意**：空格必須添加，否則會被認爲不是排列。

此爲使用Markdown語法撰寫的內容

```
* Monday
* Tuesday
+ Wensday
+ Thursday
- Friday
- Staturday
- Sunday
```

Markdown渲染效果如下：

>* Monday
* Tuesday
+ Wensday
Thursday
+ Friday
- Staturday
- Sunday


### Code Blocks
代碼塊，對於代碼塊引用。在Markdown中，可通過如下2中方式實現：

1. 可使用一個`Tab`或四個空格實現。即先按一個`Tab`或四個空格，再輸入代碼；
2. 可通過三個反引號包裹起來，首行和尾行都是三個反引號；

#### style1
此爲使用Markdown語法撰寫的內容

```
	flag=1
	while $flag;do
		echo "I love you, my lover"
	done
```

Markdown渲染效果如下

>
	flag=1
	while $flag;do
		echo "I love you, my lover"
	done


#### style2
通過三個反引號包裹

此爲使用Markdown語法撰寫的內容

```
flag=1
while $flag;do
	echo "I love you, my lover"
done
```

此種形式，可以給不同語言的代碼塊加上對應語言的標記，如`html`、`bash`、`php`、`phthon`、`javascript`、`sql`等，添加位置是行首三個反引號後面，直接添加即可，這樣對應語言的語法關鍵詞會被Markdown用醒目的顏色標記出來。

如對SQL語句的渲染

```sql
drop database if exists arsenal;

-- 創建數據庫
create database if not exists arsenal
    default character set=utf8
    default collate=utf8_general_ci;

-- 進入數據庫
use arsenal;

-- 存儲用戶session信息
-- http://symfony.com/doc/current/doctrine/pdo_session_storage.html#preparing-the-database-to-store-sessions
create table if not exists sessions (
    sess_id varbinary(128) not null primary key comment 'session id',
    sess_data blob not null comment 'session data (dynamic clomumn)json 格式',
    sess_time int unsigned not null comment 'session create or update time (timestamp)',
    sess_lifetime mediumint unsigned not null comment 'session life time，單位s，默認1440s即24小時'
) engine=innodb default charset=utf8 collate=utf8_bin comment '存儲用戶session信息';
```

### HTML Entities

>Within a code block, ampersands (&) and angle brackets (< and >) are automatically converted into HTML entities. This makes it very easy to include example HTML source code using Markdown — just paste it and indent it, and Markdown will handle the hassle of encoding the ampersands and angle brackets.

在代碼塊中，符號`&`、`<`、`>`爲被自動解析爲HTML實體，Markdown支持直接加入HTML代碼，並按HTML方式解析。

此爲使用Markdown語法撰寫的內容

```
<div class="footer">
    &copy; 2004 Foo Corporation
</div>

&lt;div class="footer"&gt;
	&copy; 2004 Foo Corporation
&lt;/div&gt;
```

Markdown渲染效果如下：

><div class="footer">
    &copy; 2004 Foo Corporation
</div>
>
&lt;div class="footer"&gt;
	&copy; 2004 Foo Corporation
&lt;/div&gt;


### Tables
表格在HTML中表格需要使用`<tr><th></th><td></td></th>`形式標籤創建，Markdown中語法格式如下

此爲使用Markdown語法撰寫的內容

```
| Country    | Province     | City  |
|   :---:    |    :----     | -----:  |
| China      | Beijing      | Beiging |
| China      | Taiwan       | Taiwan   |
| China      | HongKong     | HongKong |
```

也可簡寫爲(可讀性較弱)

```
Country|Province|City
:---:|:----|-----:
China|Beijing|Beiging
China|Taiwan|Taiwan
China|HongKong|HongKong
```


Markdown渲染效果如下：

Country|Province|City
:---:|:----|-----:
China|Beijing|Beiging
China|Taiwan|Taiwan
China|HongKong|HongKong

**注意**：
1. 第二行加至少三個連接號`-`，用以定義表格，必須添加；
2. 其左右的冒號`:`則表示向左、向右、居中對齊。


### Horizonal Rules
水平分割線在HTML中用符號`<hr>`，在Markdown中可用連字符`-`、星號`*`、下劃線`_`表示，但必須至少3個字符一起使用。

type|syntax
---|---
`-`|`---`
`*`|`***`
`_`|`___`


Markdown渲染效果如下

1. 連字符`-`

---

2. 星號`*`

***

3. 下劃線`_`

___


## Inline Elements
行內元素的Markdown語法涵蓋 **超鏈接**、**圖片**、**強調**、**代碼高亮** 等。

### Links
超鏈接，常用Markdown語法如下

```
[description](url 'info')
```

會被渲染爲`[description](url 'info')`，當鼠標移動到鏈接上時，會顯示`info`信息

#### Inline-style
此爲使用Markdown語法撰寫的內容

```
[GitHub](https://github.com 'GitHub Official Site')
```

Markdown渲染效果如下

[GitHub](https://github.com 'GitHub Official Site')


#### Reference-style
將url和info通過類似錨點的形式引入

```
[GitHub][github]

[github]:https://github.com "GitHub Official Site"
```

Markdown渲染效果如下

[GitHub][github]

[github]:https://github.com "GitHub Official Site"

以下三者等效

```
[github]: https://github.com "GitHub Official Site"
[github]: https://github.com 'GitHub Official Site'
[github]: https://github.com (GitHub Official Site)
```

**注意**：在某些Markdown工具中，使用單引號形式的可能無法正常渲染。建議使用雙引號`""`或`()`。

### Images
對於圖片，Markdown的常用語法如下

```
![description](url 'info')
```

會被解釋爲`![description](url 'info')`

也可使用類似超鏈接中的`Reference-style`

此爲使用Markdown語法撰寫的內容

```markdown
![萬里長城](https://upload.wikimedia.org/wikipedia/commons/1/17/The_Great_wall_-_by_Hao_Wei.jpg '維基百科')
```

Markdown渲染效果如下

![萬里長城](https://upload.wikimedia.org/wikipedia/commons/1/17/The_Great_wall_-_by_Hao_Wei.jpg '維基百科')

### Emphasis
強調主要是 **加粗** 和 *斜體* 兩種形式，在Markdown中主要使用星號`*`和下劃線`_`兩種符號包裹，使用一個表示 *斜體*，使用兩個表示 **加粗**

強調還有一種形式 ~~刪除效果~~，也可與上面兩種組合使用，比如 ~~**刪除效果**~~

此爲使用Markdown語法撰寫的內容

```
*Italic*
_Italic_

**Bold**
__Bold__

~~strike through~~
```

Markdown渲染效果如下

*Italic*
_Italic_

**Bold**
__Bold__

~~strike through~~

### Code Highlight
代碼高亮：在Markdown中使用反引號`` ` ``包裹

此爲使用Markdown語法撰寫的內容

```
`Enter`
```

Markdown渲染效果如下

`Enter`

但有些符號，比如反引號`` ` ``本身，直接使用單個反引號包裹無法正常渲染，需使用 **雙反引號** 包裹，才能正常渲染出反引號。

## Table Of Contents
爲Markdown製作目錄索引，有些編輯器支持`[TOC]`功能，可自動生成目錄。但GitHub暫不支持`[TOC]`功能，故需手動生成目錄。如本文`Table Of Contents`部分，目錄主要針對[Heading](#heading)的6中標題，通過對其建立錨點實現頁內內容跳轉。

注意點：圓括號內的錨點內容

* 大寫字母一律替換爲小寫
* 空格接號`-`替換，連續多個空格當作一個空格看待
* 特殊字符忽略，如`""`、`%`、`#`、`(`、`)`等，但`-`不變

如下是示例

`"Hello"but% goto # school`的錨點是`hellobut-goto-school`

### Shell Script
本人撰寫了一個Shell Script用於生成TOC目錄，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/markdownTOCGeneration.sh)。

使用方式如下

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/markdownTOCGeneration.sh | bash -s --
```


## GitHub With Markdown
對於GitHub

* GitHub不支持`[TOC]`自動生成目錄，只能手動生成；
	* 對於次級目錄，GitHub不支持自動換行，在一級目錄後，類似行內元素；
* GitHub默認不支持自動解釋換行符號，需在生成的TOC目錄後添加2個空格用於標記換行(每一行都要添加)；
* GitHub無法直接插入視頻鏈接，只能通過類似插入超鏈接或圖片形式，點擊跳轉到視頻播放頁面；

### YouTube Video Thumbnail
如何獲取Youtube視頻的縮略圖
參考 [How do I get a YouTube video thumbnail from the YouTube API?](http://stackoverflow.com/questions/2068344/how-do-i-get-a-youtube-video-thumbnail-from-the-youtube-api 'stack overflow')

每個視頻生成四張縮略圖

```http
https://img.youtube.com/vi/<Youtube-Video-id>/0.jpg
https://img.youtube.com/vi/<Youtube-Video-id>/1.jpg
https://img.youtube.com/vi/<Youtube-Video-id>/2.jpg
https://img.youtube.com/vi/<Youtube-Video-id>/3.jpg
```

通常第一張是默認圖片，且是全尺寸，其餘三張是縮略圖

```http
https://img.youtube.com/vi/<Youtube-Video-id>/default.jpg
```

獲取`default.jpg`不同畫質的圖片

| quality version | url |
| :---: | :--- |
| high quality | `https://img.youtube.com/vi/<Youtube-Video-id>/hqdefault.jpg` |
| medium quality | `https://img.youtube.com/vi/<Youtube-Video-id>/mqdefault.jpg` |
| standard definition | `https://img.youtube.com/vi/<Youtube-Video-id>/sddefault.jpg` |
| maximum resolution | `https://img.youtube.com/vi/<Youtube-Video-id>/maxresdefault.jpg` |


仍以[Youtube](https://www.youtube.com 'Youtube')中的[Adele - Rolling in the Deep](https://www.youtube.com/watch?v=rYEDA3JcQqw 'Youtube')爲例。此爲使用Markdown語法撰寫的內容

```
[![](https://img.youtube.com/vi/rYEDA3JcQqw/maxresdefault.jpg 'Adele - Rolling in the Deep')](https://www.youtube.com/watch?v=rYEDA3JcQqw)
```

Markdown渲染效果如下

[![](https://img.youtube.com/vi/rYEDA3JcQqw/maxresdefault.jpg 'Adele - Rolling in the Deep')](https://www.youtube.com/watch?v=rYEDA3JcQqw)


## References
* [Daring Fireball: Markdown](http://daringfireball.net/projects/markdown/)


## Bibliography
* [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
* [Mastering Markdown](https://guides.github.com/features/mastering-markdown/)
* [14 BEST MARKDOWN EDITORS FOR LINUX](https://itsfoss.com/best-markdown-editors-linux/)


## Change logs
* 2016.01.03 17:33 Sun Asia/Beijing
    * 初稿完成
* 2017.02.24 13:46 Fri Asia/Shanghai
    * 內容結構優化，添加生成`TOC`的腳本
* 2018-04-11 22:20 Wed America/Boston
    * 勘誤，遷移到新Blog


[markdown]:https://daringfireball.net/projects/markdown/ "Markdown"
[jekyll]: https://jekyllrb.com "Jekyll is a simple, blog-aware, static site generator"
[hexo]: https://hexo.io "A fast, simple & powerful blog framework"
[hugo]: https://gohugo.io "The world’s fastest framework for building websites"
[digitalocean]: https://www.digitalocean.com "Digital Ocean"
[stackoverflow]: https://stackoverflow.com "Stack Overflow"

<!-- End -->
