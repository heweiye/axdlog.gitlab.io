---
title: Printing The Previous Or Next Line Of The Matched Line Using Sed OR Awk
date: 2017-03-15T14:57:39-08:00
lastmod: 2018-04-11T11:09:39-04:00
draft: false
categories:
- Data Process
tags:
- awk
- sed
comment: true
toc: true
---

在GNU/Linux中，[awk(gawk)][gawk]和[sed][sed]可用於處理文本數據。本文討論的是如何使用二者提取匹配數據行的前一行或後一行數據。如果匹配數據行出現多次，則只輸出第一次匹配到的數據行。

本文採用真實案例進行討論

* (匹配行的前一行) 提取[Nginx][nginx]的最新版本號；
* (匹配行的後一行) 提取[Linux Kernel][kernel]的最新版本號；

<!--more-->

## Production Examples
從過軟件的官方網站提取所需數據，使用`curl`或`wget`下載官網HTML代碼，分別使用`awk`和`sed`從返回的HTML標籤數據中提取所需數據。

### Previous Line For Nginx
提取匹配行的前一行，以[Nginx][nginx]爲例，提取最新主線(mainline)版的版本號。版本號在匹配字符串`mainline`所在行的前一行，最新穩定版版本爲`1.13.12`(Apr 11, 2018)。

操作命令爲

```bash
# Via awk
curl -fsL https://nginx.org/ | awk '$0~/mainline/{print gensub(/.*nginx-(.*)<.*/,"\\1","g",a);exit};{a=$0}'

# Via sed
curl -fsL https://nginx.org/ | sed -n -r '/mainline/{0,/mainline/{x;s@.*nginx-(.*)<.*@\1@p;}};h'
# curl -fsL https://nginx.org/ | sed -n -r '0,/mainline/{/mainline/{x;s@.*nginx-(.*)<.*@\1@g;p};h}'
```

演示過程

```bash
# Via awk
maxsre@stretch:~$ curl -fsL https://nginx.org/ | awk '$0~/mainline/{print gensub(/.*nginx-(.*)<.*/,"\\1","g",a);exit};{a=$0}'
1.13.12
maxsre@stretch:~$

# Via sed
maxsre@stretch:~$ curl -fsL https://nginx.org/ | sed -n -r '/mainline/{0,/mainline/{x;s@.*nginx-(.*)<.*@\1@p;}};h'
1.13.12
maxsre@stretch:~$
```

HTML標籤片段如下

```html
</div><div id="content"><h2>nginx news</h2>
            <table class="news">
        <tr><td class="date"><a name="2018-04-10"></a>2018-04-10</td><td><p><a href="en/download.html">nginx-1.13.12</a>
mainline version has been released.
</p></td></tr><tr><td class="date"><a name="2018-04-03"></a>2018-04-03</td><td><p><a href="en/docs/njs_about.html">njs-0.2.0</a>
version has been released, featuring HTTP subrequest() method support
and <a href="en/docs/njs/njs_changes.html#njs0.2.0">more</a>.
</p></td></tr><tr><td class="date"><a name="2018-04-03"></a>2018-04-03</td><td><p><a href="en/download.html">nginx-1.13.11</a>
mainline version has been released.
</p></td></tr><tr><td class="date"><a name="2018-03-23"></a>2018-03-23</td><td><p><a href="https://unit.nginx.org/">unit-0.7</a>
beta version has been released with
<a href="http://mailman.nginx.org/pipermail/unit/2018-March/000040.html">
Ruby module</a>.
</p></td></tr><tr><td class="date"><a name="2018-03-20"></a>2018-03-20</td><td><p><a href="en/download.html">nginx-1.13.10</a>
mainline version has been released,
featuring the
<a href="en/docs/http/ngx_http_grpc_module.html">gRPC proxy module</a>.
</p></td></tr><tr><td class="date"><a name="2018-02-20"></a>2018-02-20</td><td><p><a href="en/download.html">nginx-1.13.9</a>
mainline version has been released.
</p></td></tr><tr><td class="date"><a name="2018-02-09"></a>2018-02-09</td><td><p><a href="https://unit.nginx.org/">unit-0.6</a>
beta version has been released with
<a href="http://mailman.nginx.org/pipermail/unit/2018-February/000034.html">
Perl module and advanced process management</a>.
</p></td></tr><tr><td class="date"><a name="2018-01-15"></a>2018-01-15</td><td><p><a href="https://unit.nginx.org/">unit-0.4</a>
beta version has been released with
<a href="http://mailman.nginx.org/pipermail/unit/2018-January/000029.html">
regression fixes</a>.
</p></td></tr><tr><td class="date"><a name="2017-12-29"></a>2017-12-29</td><td><p><a href="https://unit.nginx.org/">unit-0.3</a>
beta version has been released with HTTP keep-alive support,
latency optimizations, Python and Go improvements, and
<a href="https://unit.nginx.org/CHANGES.txt">more</a>.
</p></td></tr><tr><td class="date"><a name="2017-12-26"></a>2017-12-26</td><td><p><a href="en/download.html">nginx-1.13.8</a>
mainline version has been released.
</p></td></tr>
            </table>
```

### Next Line For Linux Kernel
提取匹配行的後一行，以[Linux Kernel][kernel]爲例，提取最新穩定(stable)版的版本號。版本號在匹配字符串`stable:`所在行的後一行，最新穩定版版本爲`4.16.1`(Apr 11, 2018)。

操作命令爲

```bash
# Via awk
curl -fsL https://www.kernel.org | awk 'match($1,/stable:/){getline;print gensub(/.*strong>(.*)<\/strong.*/,"\\1","g",$0);exit}'

# Via sed
curl -fsL https://www.kernel.org | sed -n -r '/stable:/{0,/stable:/{n;s@ @@g;s@<[^>]*>@@gp}}'
# curl -fsL https://www.kernel.org | sed -n -r '/stable:/{0,/stable:/{n;s@ @@g;s@<[^>]*>@@gp}}'
```

演示過程

```bash
# Via awk
maxsre@stretch:~$ curl -fsL https://www.kernel.org | awk 'match($1,/stable:/){getline;print gensub(/.*strong>(.*)<\/strong.*/,"\\1","g",$0);exit}'
4.16.1

# Via sed
maxsre@stretch:~$ curl -fsL https://www.kernel.org | sed -n -r '/stable:/{0,/stable:/{n;s@ @@g;s@<[^>]*>@@gp}}'
4.16.1
# longterm有多個版本
maxsre@stretch:~$ curl -fsL https://www.kernel.org | sed -n -r '/longterm:/{0,/longterm:/{n;s@ @@g;s@<[^>]*>@@gp}}'
4.14.33
maxsre@stretch:~$
```

HTML標籤片段如下

```html
        <table id="releases">
                <tr align="left">
            <td>mainline:</td>
            <td><strong>4.16</strong></td>
            <td>2018-04-01</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.16.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.16.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.16.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td> </td>
            <td>[<a href="https://git.kernel.org/torvalds/ds/v4.16/v4.15" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/torvalds/h/v4.16" title="Browse the git tree using cgit">browse</a>]  </td>
            <td> </td>
        </tr>
                <tr align="left">
            <td>stable:</td>
            <td><strong>4.16.1</strong></td>
            <td>2018-04-08</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.16.1.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.16.1.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.16.1.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td> </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.16.1/v4.16" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.16.1" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.16.1" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>stable:</td>
            <td><strong>4.15.16</strong></td>
            <td>2018-04-08</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.15.16.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.15.16.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.15.16.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/incr/patch-4.15.15-16.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.15.16/v4.15.15" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.15.16" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.15.16" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>4.14.33</strong></td>
            <td>2018-04-08</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.33.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.33.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.14.33.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/incr/patch-4.14.32-33.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.14.33/v4.14.32" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.14.33" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.14.33" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>4.9.93</strong></td>
            <td>2018-04-08</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.93.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.93.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.9.93.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/incr/patch-4.9.92-93.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.9.93/v4.9.92" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.9.93" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.9.93" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>4.4.127</strong></td>
            <td>2018-04-08</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.127.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.4.127.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.4.127.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/incr/patch-4.4.126-127.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.4.127/v4.4.126" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.4.127" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.4.127" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>4.1.51</strong></td>
            <td>2018-03-27</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.1.51.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.1.51.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/patch-4.1.51.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/incr/patch-4.1.50-51.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v4.1.51/v4.1.50" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v4.1.51" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.1.51" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>3.18.104 <span class="eolkernel" title="This release is End-of-Life">[EOL]</span></strong></td>
            <td>2018-04-10</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.18.104.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.18.104.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/patch-3.18.104.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/incr/patch-3.18.103-104.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v3.18.104/v3.18.103" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v3.18.104" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/ChangeLog-3.18.104" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>3.16.56</strong></td>
            <td>2018-03-19</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.16.56.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.16.56.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/patch-3.16.56.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/incr/patch-3.16.55-56.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v3.16.56/v3.16.55" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v3.16.56" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/ChangeLog-3.16.56" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>longterm:</td>
            <td><strong>3.2.101</strong></td>
            <td>2018-03-19</td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.2.101.tar.xz" title="Download complete tarball">tarball</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.2.101.tar.sign" title="Download PGP verification signature">pgp</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/patch-3.2.101.xz" title="Download patch to previous mainline">patch</a>] </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/incr/patch-3.2.100-101.xz" title="Download incremental patch">inc.&nbsp;patch</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/ds/v3.2.101/v3.2.100" title="View diff in cgit">view&nbsp;diff</a>] </td>
            <td>[<a href="https://git.kernel.org/stable/linux-stable/h/v3.2.101" title="Browse the git tree using cgit">browse</a>]  </td>
            <td>[<a href="https://cdn.kernel.org/pub/linux/kernel/v3.x/ChangeLog-3.2.101" title="View detailed change logs">changelog</a>] </td>
        </tr>
                <tr align="left">
            <td>linux-next:</td>
            <td><strong>next-20180411</strong></td>
            <td>2018-04-11</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td>[<a href="https://git.kernel.org/next/linux-next/h/next-20180411" title="Browse the git tree using cgit">browse</a>]  </td>
            <td> </td>
        </tr>
                </table>
```


## Analysis Of Extract Previous Line
以[Nginx][nginx]爲例，以字符串`mainline`爲匹配關鍵詞，提取匹配行的前一行數據。

### Via awk
awk的操作命令爲

```bash
curl -fsL https://nginx.org/ | awk '$0~/mainline/{print gensub(/.*nginx-(.*)<.*/,"\\1","g",a);exit};{a=$0}'
```

解釋
1. `curl -fsL https://nginx.org/`：獲取[Nginx][nginx]官網的HTML標籤；
2. `awk ''`：表示使用awk進行操作；
3. `$0~/mainline/`：`$0`表示從文本中讀取的一整行數據，`~`表示模糊匹配，`/mainline/`表示匹配含有關鍵詞`mainline`的數據行；
4. `{print gensub(/.*nginx-(.*)<.*/,"\\1","g",a);exit};{a=$0}`：`exit`表示只匹配一次就退出awk操作，`{a=$0}`和`gensub()`中的最後一個`a`表示匹配行的前一行數據； *暫不理解其實現原理*
5. `gensub(/.*nginx-(.*)<.*/,"\\1","g",a)`提取具體的版本號；
6. `print`爲awk打印命令，輸出指定的數據；

### Via sed
sed的操作命令爲

```bash
curl -fsL https://nginx.org/ | sed -n -r '/mainline/{0,/mainline/{x;s@.*nginx-(.*)<.*@\1@p;}};h'
```

使用到的選項

* `x` - Exchange the contents of the hold and pattern spaces.
* `h` - (hold) Replace the contents of the hold space with the contents of the pattern space.
* `p` - Print the pattern space.
* `-n, --quiet, --silent` - suppress automatic printing of pattern space


解釋

1. `curl -fsL https://nginx.org/`：獲取[Nginx][nginx]官網的HTML標籤；
2. `sed ''`：使用sed命令進行操作；
3. `/mainline/`：地址定界，此處使用正則(regular expression)進行匹配，匹配關鍵爲`mainline`；
4. `/mainline/{x}`:`x`表示將當前hold space和pattern space中的內容進行互換，此處只針對匹配行；
5. `h`：將當前hold space中的內容替換爲pattern space中的內容，實現hold space中的內容與pattern space中的內容一致；
6. `-r`：使用增強性正則表達式，如支持後向引用(back references)；
7. `-n`：表示不輸出當前pattern space中的內容，通常與`p`組合使用；
8. `/mainline/{x;p}`中的`p`與`-n`組合使用，表示只輸出匹配行；
9. `0,/mainline/`：只針對第一次出現的匹配數據行；
10. `s///g`: 替換操作，因使用了`-r`，此處通過後向引用提取版本號；

完整的解釋：使用sed對獲取的HTML標籤進行處理，對於匹配數據行，先使用`x`交換當前hold space和pattern space中的內容，再使用`p`和`-n`輸出當前pattern space中的內容；對所有數據行使用`h`，將所有數據行的當前hold space中的內容替換爲pattern space中的內容。經過此操作，匹配數據行的當前pattern space中的內容即前一行數據。

稍後對其處理過程進行演示。


## Analysis Of Extract Next Line
以[Linux Kernel][kernel]爲例，以字符串`stable:`爲匹配關鍵詞，提取匹配行的後一行數據。

### Via awk
awk的操作命令爲

```bash
curl -fsL https://www.kernel.org | awk 'match($1,/stable:/){getline;print gensub(/.*strong>(.*)<\/strong.*/,"\\1","g",$0);exit}'
```

解釋
1. `curl -fsL https://www.kernel.org`：獲取[Linux Kernel][kernel]官網的HTML標籤；
2. `awk ''`：表示使用awk進行操作；
3. `$0`、`$1`：awk中默認以空格爲字段(field)分隔符，`$0`表示一整行數據，`$1`表示數據行中的第一個字段(以空格爲分割符)；
4. `match($1,/stable:/)`：數據行第一個字段`$1`中含有字符串`stable:`，與`$1~/stable:/`等效；
5. `getline`：表示獲取匹配行的後一行數據，與`exit`組合使用表示只匹配一次就結束awk操作。
6. `print gensub(/.*strong>(.*)<\/strong.*/,"\\1","g",$0)`：根據HTML標籤格式提取並輸出版本號；

即提取匹配數據行的後一行數據，可通過awk的`getline`實現。

### Via sed
sed的操作命令爲

```bash
curl -fsL https://www.kernel.org | sed -n -r '/stable:/{0,/stable/{n;s@ @@g;s@<[^>]*>@@gp}}'
```

使用到的選項

* `n` - (next) If auto-print is not disabled, print the pattern space, then, regardless, replace the pattern space with the next line of input. If there is no more input then sed exits without processing any more commands.

解釋

1. `curl -fsL https://www.kernel.org`：獲取[Linux Kernel][kernel]官網的HTML標籤；
2. `sed ''`：使用sed命令進行操作；
3. `/stable:/`：地址定界，此處使用正則(regular expression)進行匹配，匹配關鍵爲`stable:`；
4. `0,/stable/`：只針對第一次出現的匹配數據行；
5. `n`: 表示將匹配行的當前pattern space中的內容，使用下一行的輸入進行替換；
6. `-r`：使用增強性正則表達式，如支持後向引用(back references)；
7. `s///g`: 替換操作，因使用了`-r`，此處通過後向引用提取版本號；


## Analysis
### Extract Previous Line Via Sed
測試數據 `/tmp/test.txt`

```
m1
mainline version
s1
stable flag
s2
stable version
s3
stable test
m2
mainline version
```

#### testing x
執行
```bash
sed '/stable/{x}' /tmp/test.txt
```
輸出爲

```
m1
mainline version
s1

s2
stable flag
s3
stable version
m2
mainline version
```

分析

line|origin|hold space|pattern space|explanation
---|---|---|---|---
1|m1||m1|
2|mainline version||mainline version|
3|s1||s1|
4|stable flag|stable flag||匹配行，前一行hold space爲空，交換後pattern space爲空
5|s2|stable flag|s2|
6|stable version|stable version|stable flag|匹配行，前一行hold space爲`stable flag`，交換後pattern space爲`stable flag`
7|s3|stable version|s3|
8|stable test|stable test|stable version|匹配行，前一行hold space爲`stable version`，交換後pattern space爲`stable version`
9|m2|stable test|m2|
10|mainline version|stable test|mainline version|

#### testing x&h
執行
```bash
sed '/stable/{x};h' /tmp/test.txt
```

輸出爲

```
m1
mainline version
s1
s1
s2
s2
s3
s3
m2
mainline version
```

分析

line|origin|hold space|pattern space|explanation
---|---|---|---|---
1|m1|m1|m1|非匹配行，將hold space中內容替換爲pattern space中內容
2|mainline version|mainline version|mainline version|
3|s1|s1|s1|
4|stable flag|s1|s1|匹配行，前一行hold space爲`s1`，交換後pattern space爲`s1`；因爲`h`，當前hold space被替換爲`s1`
5|s2|s2|s2|
6|stable version|s2|s2|匹配行，前一行hold space爲`s2`，交換後pattern space爲`s2`；因爲`h`，當前hold space被替換爲`s2`
7|s3|s3|s3|
8|stable test|s3|s3|匹配行，前一行hold space爲`s3`，交換後pattern space爲`s3`；因爲`h`，當前hold space被替換爲`s3`
9|m2|m2|m2|
10|mainline version|mainline version|mainline version

#### testing x&p&h
執行

```bash
sed -n '/stable/{x;p};h' /tmp/test.txt
```

輸出爲
```
s1
s2
s3
```

分析

line|origin|hold space|pattern space|explanation
---|---|---|---|---
1|m1|m1|m1|非匹配行不輸出，將hold space中內容替換爲pattern space中內容
2|mainline version|mainline version|mainline version|
3|s1|s1|s1|
4|stable flag|s1|s1(輸出)|匹配行，前一行hold space爲`s1`，交換後pattern space爲`s1`；因爲`h`，當前hold space被替換爲`s1`
5|s2|s2|s2|
6|stable version|s2|s2(輸出)|匹配行，前一行hold space爲`s2`，交換後pattern space爲`s2`；因爲`h`，當前hold space被替換爲`s2`
7|s3|s3|s3|
8|stable test|s3|s3(輸出)|匹配行，前一行hold space爲`s3`，交換後pattern space爲`s3`；因爲`h`，當前hold space被替換爲`s3`
9|m2|m2|m2||
10|mainline version|mainline version|mainline version|

## List Release News
### Nginx
使用如下命令獲取Release信息，以列表形式顯示

```bash
curl -fsSL https://nginx.org/ | awk '$0~/^(stable|mainline)/{$0~/stable/?type="stable":type="mainline";b=gensub(/[[:space:]]*<[^>]*>/,"","g",a);c=gensub(/nginx-/," ","g",b);printf("%s %s\n",c,type)};{a=$0}'

curl -fsSL https://nginx.org/ | awk 'BEGIN{print "Date|Version|Type\n---|---|---"}$0~/^(stable|mainline)/{$0~/stable/?type="stable":type="mainline";b=gensub(/[[:space:]]*<[^>]*>/,"","g",a);c=gensub(/nginx-/,"|","g",b);printf("%s|%s\n",c,type)};{a=$0}'
```

演示過程

```bash
maxdsre@stretch:~$ curl -fsSL https://nginx.org/ | awk '$0~/^(stable|mainline)/{$0~/stable/?type="stable":type="mainline";b=gensub(/[[:space:]]*<[^>]*>/,"","g",a);c=gensub(/nginx-/," ","g",b);printf("%s %s\n",c,type)};{a=$0}'
2018-04-10 1.13.12 mainline
2018-04-03 1.13.11 mainline
2018-03-20 1.13.10 mainline
2018-02-20 1.13.9 mainline
2017-12-26 1.13.8 mainline

maxdsre@stretch:~$ curl -fsSL https://nginx.org/ | awk 'BEGIN{print "Date|Version|Type\n---|---|---"}$0~/^(stable|mainline)/{$0~/stable/?type="stable":type="mainline";b=gensub(/[[:space:]]*<[^>]*>/,"","g",a);c=gensub(/nginx-/,"|","g",b);printf("%s|%s\n",c,type)};{a=$0}'
Date|Version|Type
---|---|---
2018-04-10|1.13.12|mainline
2018-04-03|1.13.11|mainline
2018-03-20|1.13.10|mainline
2018-02-20|1.13.9|mainline
2017-12-26|1.13.8|mainline

maxdsre@stretch:~$
```

Markdown渲染如下

Date|Version|Type
---|---|---
2018-04-10|1.13.12|mainline
2018-04-03|1.13.11|mainline
2018-03-20|1.13.10|mainline
2018-02-20|1.13.9|mainline
2017-12-26|1.13.8|mainline


### Linux Kernel
使用如下命令獲取Release信息，以列表形式顯示

```bash
curl -fsSL https://www.kernel.org | sed -r -n '/tr align="left"/,+3{s@[[:space:]]*<[^>]*>@@g;s@^$@_@g;s@:@@g;p}' | sed -r ':a;N;$!ba;s@\n@ @g;s@ \_+\ ?@\n@g;s@^_ @@'

curl -fsSL https://www.kernel.org | sed -r -n '/tr align="left"/,+3{s@[[:space:]]*<[^>]*>@@g;s@^$@_@g;s@:@@g;p}' | sed -r ':a;N;$!ba;s@\n@ @g;s@ \_+\ ?@\n@g;s@^_ @@' | sed -r '1i Type|Version|Date\n---|---|---'
```

操作過程

```bash
maxdsre@stretch:~$ curl -fsSL https://www.kernel.org | sed -r -n '/tr align="left"/,+3{s@[[:space:]]*<[^>]*>@@g;s@^$@_@g;s@:@@g;p}' | sed -r ':a;N;$!ba;s@\n@|@g;s@\|\_+\|?@\n@g;s@^_\|@@' | sed -r '1i Type|Version|Date\n---|---|---'
Type|Version|Date
---|---|---
mainline|4.16|2018-04-01
stable|4.16.1|2018-04-08
stable|4.15.16|2018-04-08
longterm|4.14.33|2018-04-08
longterm|4.9.93|2018-04-08
longterm|4.4.127|2018-04-08
longterm|4.1.51|2018-03-27
longterm|3.18.104[EOL]|2018-04-10
longterm|3.16.56|2018-03-19
longterm|3.2.101|2018-03-19
linux-next|next-20180411|2018-04-11

maxdsre@stretch:~$
```

Markdown渲染如下

Type|Version|Date
---|---|---
mainline|4.16|2018-04-01
stable|4.16.1|2018-04-08
stable|4.15.16|2018-04-08
longterm|4.14.33|2018-04-08
longterm|4.9.93|2018-04-08
longterm|4.4.127|2018-04-08
longterm|4.1.51|2018-03-27
longterm|3.18.104[EOL]|2018-04-10
longterm|3.16.56|2018-03-19
longterm|3.2.101|2018-03-19
linux-next|next-20180411|2018-04-11


## Further Reading
* [The GNU Awk User’s Guide](https://www.gnu.org/software/gawk/manual/gawk.html)
* [The GNU Sed User’s Guide](https://www.gnu.org/software/sed/manual/sed.html "sed, a stream editor")


## Change Logs
* 2017.03.15 14:54 Wed Asia/Shanghai
    * 初稿完成
* 2017.05.31 11:26 Wed Asia/Shanghai
    * 添加`List Release News`，用Markdown形式輸出
* 2018.04.11 11:09 Wed America/Boston
    * 更新軟件版本號，勘誤，遷移到新Blog


[gawk]:https://www.gnu.org/software/gawk/ "GNU awk"
[sed]:https://www.gnu.org/software/sed/ "GNU sed"
[nginx]:https://nginx.org/ "Nginx Web Server"
[kernel]:https://www.kernel.org "Linux Kernel"


<!-- End -->
