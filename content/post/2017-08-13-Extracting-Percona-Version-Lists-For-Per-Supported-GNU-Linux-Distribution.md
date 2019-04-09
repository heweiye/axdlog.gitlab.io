---
title: Extracting Percona Version Lists For Per Supported GNU/Linux Distribution
slug: Extracting Percona Version Lists For Per Supported GNU Linux Distribution
date: 2017-08-13T21:32:34+08:00
lastmod: 2018-07-27T15:20:50-04:00
draft: false
keywords: ["Percona version", "Shell script"]
description: "Extracting Percona Version Lists For Per Supported GNU/Linux Distribution"
categories:
- Data Process
tags:
- Percona
- Shell Script
- awk
- sed
comment: true
toc: true
---

[Percona][percona]支持RHEL/CentOS/Debian/Ubuntu等GNU/Linux發行版，在[Installing Percona Server from Repositories](https://www.percona.com/doc/percona-server/LATEST/installation.html#installing-percona-server-from-repositories)中列出了具體支持的發行版本，但並未顯式地說明[Percona Server for MySQL][percona]的每一個釋出版本(如`5.6`，`5.7`)具體支持哪些GNU/Linux發行版本。

本文通過[Percona Software Downloads][percona_software_downloads]提取各個可供下載的Percona版本的下載頁鏈接，再通過下載頁的HTML代碼提取各版本具體支持的GNU/Linux發行版，最後通過Shell Script代碼實現。

<!--more-->

**重要**：[Percona][percona]官方已經停止對`RHEL 5`和`Ubuntu 12.04 LTS`的支持，詳細說明見[Platform End of Life (EOL) Announcement for RHEL 5 and Ubuntu 12.04 LTS](https://www.percona.com/blog/2017/07/31/platform-end-of-life-announcement-rhel-5-ubuntu-12-04-lts/)。

>RHEL 5 was EOL as of March 31st, 2017 and Ubuntu 12.04 LTS was end of life as of April 28th, 2017.

## Shell 腳本
整個操作過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-
# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh | bash -s -- -d percona
```

提取結果

```bash
Percona Server|zesty|5.5
Percona Server|xenial|5.7 5.6 5.5
Percona Server|wheezy|5.7 5.6 5.5
Percona Server|trusty|5.7 5.6 5.5
Percona Server|stretch|5.7 5.6 5.5
Percona Server|squeeze|5.1
Percona Server|rhel7|5.7 5.6 5.5|rhel centos
Percona Server|rhel6|5.7 5.6 5.5 5.1|rhel centos
Percona Server|rhel5|5.1|rhel centos
Percona Server|precise|5.1
Percona Server|lucid|5.1
Percona Server|jessie|5.7 5.6 5.5
Percona Server|bionic|5.7 5.6 5.5
Percona Server|artful|5.7 5.6 5.5
Percona XtraDB Cluster|xenial|5.7 5.6
Percona XtraDB Cluster|wheezy|5.7 5.6 5.5
Percona XtraDB Cluster|trusty|5.7 5.6 5.5
Percona XtraDB Cluster|stretch|5.7 5.6
Percona XtraDB Cluster|rhel7|5.7 5.6 5.5|rhel centos
Percona XtraDB Cluster|rhel6|5.7 5.6 5.5|rhel centos
Percona XtraDB Cluster|rhel5|5.5|rhel centos
Percona XtraDB Cluster|precise|5.5
Percona XtraDB Cluster|jessie|5.7 5.6
Percona XtraDB Cluster|bionic|5.7 5.6
Percona XtraDB Cluster|artful|5.7 5.6
```

## Requirement
提取每一個[Percona][percona]所支持的GNU/Linux發行版本具體支持的[Percona][percona]版本列表
1. [Percona][percona]具體支持哪些GNU/Linux發行版；
2. 每一個GNU/Linux發行版具體支持[Percona][percona]哪些版本；


##　Analysis
通過分析[Percona Software Downloads][percona_software_downloads]頁面，提取[Percona][percona]目前提供下載的版本號及對應的下載頁鏈接。

```bash
curl -fsL https://www.percona.com/downloads/ | sed -r -n '/A drop-in replacement for MySQL/,/<\/div>/{/<a/{s@.* href="([^"]*)".*>Download ([[:digit:].]+).*@\2 https://www.percona.com\1@g;p}}'

# 5.7 https://www.percona.com/downloads/Percona-Server-LATEST/
# 5.6 https://www.percona.com/downloads/Percona-Server-5.6/LATEST/
# 5.5 https://www.percona.com/downloads/Percona-Server-5.5/LATEST/
# 5.1 https://www.percona.com/downloads/Percona-Server-5.1/LATEST/
```

通過下載頁鏈接，提取具體支持的GNU/Linux發行版。

```bash
# 5.7
curl -fsL https://www.percona.com/downloads/Percona-Server-LATEST/ | sed -r -n '/Select Software Platform/{s@redhat/@rhel@g;s@<\/option>@\n@g;p}' | sed -r -n 's@.*/([^"]*)"[[:space:]]*>.*@\1@g;/binary|source|select|^$/d;p'

# wheezy
# jessie
# stretch
# rhel6
# rhel7
# trusty
# xenial
# artful
# bionic


# 多行轉換爲一行
sed -r ':a;N;$!ba;s@\n@ @g;'
# wheezy jessie stretch rhel6 rhel7 trusty xenial artful bionic
```

## Code Snippets
通過兩種方式實現：`while`循環和`parallel`並行操作。

### Original
利用`while`循環，較爲耗時

```bash
#!/usr/bin/env bash

download_tool=${download_tool:-'curl -fsL'}
version_list_info=$(mktemp -t tempXXXXX.txt)

# - version|distro_list
$download_tool https://www.percona.com/downloads/ | sed -r -n '/A drop-in replacement for MySQL/,/<\/div>/{/<a/{s@.* href="([^"]*)".*>Download ([[:digit:].]+).*@\2 https://www.percona.com\1@g;p}}' | while read -r version url; do
    distro_list=${distro_list:-}
    distro_list=$($download_tool $url | sed -r -n '/Select Software Platform/{s@redhat/@rhel@g;s@<\/option>@\n@g;p}' | sed -r -n 's@.*/([^"]*)"[[:space:]]*>.*@\1@g;/binary|source|select|^$/d;p' | sed -r ':a;N;$!ba;s@\n@ @g;')
    echo "${version}|${distro_list}" >> "${version_list_info}"
done

# cat "${version_list_info}"

# 5.7|wheezy jessie stretch rhel6 rhel7 trusty xenial artful bionic
# 5.6|wheezy jessie stretch rhel6 rhel7 trusty xenial artful bionic
# 5.5|wheezy jessie stretch rhel6 rhel7 trusty xenial zesty artful bionic
# 5.1|squeeze rhel5 rhel6 lucid precise

# - distro|version_list
sed -r 's@.*\|@@g' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;s@ @\n@g' | awk '!a[$0]++' | while read -r distro; do
    version_list=$(awk -F\| 'match($NF,/[[:space:]]*rhel6[[:space:]]*/){print $1}' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;')
    echo "$distro|$version_list"
done

# wheezy|5.7 5.6 5.5 5.1
# jessie|5.7 5.6 5.5 5.1
# stretch|5.7 5.6 5.5 5.1
# rhel6|5.7 5.6 5.5 5.1
# rhel7|5.7 5.6 5.5 5.1
# trusty|5.7 5.6 5.5 5.1
# xenial|5.7 5.6 5.5 5.1
# artful|5.7 5.6 5.5 5.1
# bionic|5.7 5.6 5.5 5.1
# zesty|5.7 5.6 5.5 5.1
# squeeze|5.7 5.6 5.5 5.1
# rhel5|5.7 5.6 5.5 5.1
# lucid|5.7 5.6 5.5 5.1
# precise|5.7 5.6 5.5 5.1

[[ -f "${version_list_info}" ]] && rm -f "${version_list_info}"
unset version_list_info
```

### Parallel
利用`parallel`命令並行操作，較節省時間

```bash
#!/usr/bin/env bash

download_tool=${download_tool:-'curl -fsL'}
version_list_info=$(mktemp -t tempXXXXX.txt)
export version_list_info="${version_list_info}"

# - version|distro_list
funcDistroListForPerVersion(){
    # $1 = 5.7 https://www.percona.com/downloads/Percona-Server-LATEST/
    local item=${1:-}
    if [[ -n "${item}" ]]; then
        local version=${version:-}
        local version_url=${version_url:-}
        local distro_list=${distro_list:-}
        version="${item%% *}"
        version_url="${item##* }"
        distro_list=$($download_tool $version_url | sed -r -n '/Select Software Platform/{s@redhat/@rhel@g;s@<\/option>@\n@g;p}' | sed -r -n 's@.*/([^"]*)"[[:space:]]*>.*@\1@g;/binary|source|select|^$/d;p' | sed -r ':a;N;$!ba;s@\n@ @g;')
        echo "${version}|${distro_list}" >> "${version_list_info}"
    fi
}

export -f funcDistroListForPerVersion
export download_tool="${download_tool}"
$download_tool https://www.percona.com/downloads/ | sed -r -n '/A drop-in replacement for MySQL/,/<\/div>/{/<a/{s@.* href="([^"]*)".*>Download ([[:digit:].]+).*@\2 https://www.percona.com\1@g;p}}' | parallel -k -j 0 funcDistroListForPerVersion 2> /dev/null

cat "${version_list_info}"

# 5.6|wheezy jessie stretch rhel6 rhel7 trusty xenial artful bionic
# 5.1|squeeze rhel5 rhel6 lucid precise
# 5.7|wheezy jessie stretch rhel6 rhel7 trusty xenial artful bionic
# 5.5|wheezy jessie stretch rhel6 rhel7 trusty xenial zesty artful bionic


# - distro|version_list
funcVersionListPerDistro(){
    local distro=${1:-}
    local version_list=${version_list:-}
    version_list=$(awk -F\| 'match($NF,/[[:space:]]*'"${distro}"'[[:space:]]*/){print $1}' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;')
    echo "$distro|$version_list"
}

export version_list_info="${version_list_info}"
export -f funcVersionListPerDistro
sed -r 's@.*\|@@g' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;s@ @\n@g' | awk '!a[$0]++' | parallel -k -j 0 funcVersionListPerDistro 2> /dev/null

# wheezy|5.6 5.7 5.5
# jessie|5.6 5.7 5.5
# stretch|5.6 5.7 5.5
# rhel6|5.6 5.1 5.7 5.5
# rhel7|5.6 5.7 5.5
# trusty|5.6 5.7 5.5
# xenial|5.6 5.7 5.5
# artful|5.6 5.7 5.5
# bionic|5.6 5.7 5.5
# squeeze|5.1
# rhel5|5.1
# lucid|5.1
# precise|5.1

[[ -f "${version_list_info}" ]] && rm -f "${version_list_info}"
unset version_list_info
```

## Performance Comparison
使用bash內置命令`time`對操作進行計時

```bash
# Original Time
real	0m4.429s
user	0m0.136s
sys	0m0.024s

# Parallel Time
real	0m2.456s
user	0m0.544s
sys	0m0.160s
```

可以看到使用`parallel`，操作耗時節省接近一半時間。


## Reference
* [Installing Percona Server on Debian and Ubuntu](https://www.percona.com/doc/percona-server/LATEST/installation/apt_repo.html)
* [Installing Percona Server on Red Hat Enterprise Linux and CentOS](https://www.percona.com/doc/percona-server/LATEST/installation/yum_repo.html)


## Change Logs
* 2017.08.13 21:26 Sun Asia/Shanghai
    * 初稿完成
* 2018.07.27 15:39:42 Fri America/Boston
    * 勘誤，更新，遷移到新Blog


[percona]:https://www.percona.com/ "Percona"
[percona_software_downloads]:https://www.percona.com/downloads/ "Percona"


<!-- End -->
