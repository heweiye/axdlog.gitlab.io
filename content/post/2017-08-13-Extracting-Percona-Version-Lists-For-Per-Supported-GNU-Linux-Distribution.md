---
title: Extracting Percona Version Lists For Per Supported GNU/Linux Distribution
slug: Extracting Percona Version Lists For Per Supported GNU Linux Distribution
date: 2017-08-13T21:32:34+08:00
lastmod: 2019-04-26T13:10:50-04:00
draft: false
keywords: ["MySQL Variant", "Percona version", "Shell script"]
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


[Percona][percona]支持RHEL/CentOS/Debian/Ubuntu等GNU/Linux發行版，在 [Installing Percona Server from Repositories](https://www.percona.com/doc/percona-server/LATEST/installation.html#installing-percona-server-from-repositories) 中列出了具體支持的發行版本，但並未顯式地說明[Percona Server for MySQL][percona]的每一個釋出版本(如`8.0`、`5.6`、`5.7`)具體支持哪些GNU/Linux發行版本。

本文通過 [Percona Software Downloads][percona_software_downloads] 提取各個可供下載的Percona版本的下載頁鏈接，再通過下載頁的HTML代碼提取各版本具體支持的GNU/Linux發行版，最後通過Shell Script代碼實現。

<!--more-->

爲實現 MySQL Variants ([MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona])在GNU/Linux中的自動安裝、配置，本人通過Shell腳本提取其對各GNU/Linux發行版本的具體支持信息：

* [Extracting MariaDB Version Lists For Per Supported GNU/Linux Distribution]({{< relref "2017-07-19-Extracting-MariaDB-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}})
* [Extracting MySQL Version Lists For Per Supported GNU/Linux Distribution]({{< relref "2017-07-20-Extracting-MySQL-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}})
* [**Extracting Percona Version Lists For Per Supported GNU/Linux Distribution**]({{< relref "2017-08-13-Extracting-Percona-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}})

數據庫系統安裝腳本代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/MySQLVariants.sh)，腳本同時支持在Debian/Ubuntu/CentOS/Fedora/OpenSUSE/SLES等發行版中安裝[MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona]。

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/MySQLVariants.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/243053.js" id="asciicast-243053" async></script>


## Announcement
* [Platform End of Life (EOL) Announcement for RHEL 5 and Ubuntu 12.04 LTS](https://www.percona.com/blog/2017/07/31/platform-end-of-life-announcement-rhel-5-ubuntu-12-04-lts/)

>RHEL 5 was EOL as of March 31st, 2017 and Ubuntu 12.04 LTS was end of life as of April 28th, 2017.

* [End of Life Announcement for Query Analyzer and MySQL Configuration Generator](https://www.percona.com/blog/2019/04/22/end-of-life-query-analyzer-and-mysql-configuration-generator/)

>Percona will no longer be offering the free services of Query Analyzer and MySQL Configuration Generator hosted at [tools.percona.com](https://www.percona.com/software/database-tools/percona-toolkit).  We made this decision based on observed low usage numbers, outdated technology, and lack of support for MySQL 5.7+, MongoDB, and PostgreSQL.


## Shell Script
整個操作過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh)，通過如下命令執行

```bash
download_tool='wget -qO-'  # curl -fsL / wget -qO-
# if need help info, specify '-h'
$download_tool https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh | bash -s -- -d percona
```

提取結果

```bash
Percona Server|xenial|8.0 5.7 5.6 5.5
Percona Server|trusty|5.7 5.6 5.5
Percona Server|stretch|8.0 5.7 5.6 5.5
Percona Server|squeeze|5.1
Percona Server|rhel7|8.0 5.7 5.6 5.5|rhel centos
Percona Server|rhel6|8.0 5.7 5.6 5.5 5.1|rhel centos
Percona Server|rhel5|5.1|rhel centos
Percona Server|precise|5.1
Percona Server|lucid|5.1
Percona Server|jessie|5.7 5.6 5.5
Percona Server|cosmic|8.0 5.7 5.6
Percona Server|bionic|8.0 5.7 5.6 5.5
Percona XtraDB Cluster|xenial|5.7 5.6
Percona XtraDB Cluster|wheezy|5.5
Percona XtraDB Cluster|trusty|5.7 5.6 5.5
Percona XtraDB Cluster|stretch|5.7 5.6
Percona XtraDB Cluster|rhel7|5.7 5.6 5.5|rhel centos
Percona XtraDB Cluster|rhel6|5.7 5.6 5.5|rhel centos
Percona XtraDB Cluster|rhel5|5.5|rhel centos
Percona XtraDB Cluster|precise|5.5
Percona XtraDB Cluster|jessie|5.7 5.6
Percona XtraDB Cluster|cosmic|5.7 5.6
Percona XtraDB Cluster|bionic|5.7 5.6
```

## Requirement
提取每一個[Percona][percona]所支持的GNU/Linux發行版本具體支持的[Percona][percona]版本列表

1. [Percona][percona]具體支持哪些GNU/Linux發行版；
2. 每一個GNU/Linux發行版具體支持[Percona][percona]哪些版本；


## Analysis
通過分析[Percona Software Downloads][percona_software_downloads]頁面，提取[Percona][percona]目前提供下載的版本號及對應的下載頁鏈接。

```bash
$download_tool https://www.percona.com/downloads/ | sed -r -n '/A drop-in replacement for MySQL/,/<\/div>/{/<a/{s@.* href="([^"]*)".*>Download ([[:digit:].]+).*@\2 https://www.percona.com\1@g;p}}'

# 8.0 https://www.percona.com/downloads/Percona-Server-LATEST/
# 5.7 https://www.percona.com/downloads/Percona-Server-5.7/LATEST/
# 5.6 https://www.percona.com/downloads/Percona-Server-5.6/LATEST/
# 5.5 https://www.percona.com/downloads/Percona-Server-5.5/LATEST/
# 5.1 https://www.percona.com/downloads/Percona-Server-5.1/LATEST/
```

通過下載頁鏈接，提取具體支持的GNU/Linux發行版。

```bash
# Latest 8.0
$download_tool https://www.percona.com/downloads/Percona-Server-LATEST/ | sed -r -n '/Select Software Platform/{s@redhat/@rhel@g;s@<\/option>@\n@g;p}' | sed -r -n 's@.*/([^"]*)"[[:space:]]*>.*@\1@g;/binary|source|select|^$/d;p'

# stretch
# rhel6
# rhel7
# xenial
# bionic
# cosmic

# 多行轉換爲一行
sed -r ':a;N;$!ba;s@\n@ @g;'
# stretch rhel6 rhel7 xenial bionic cosmic
```

## Code Snippets
通過兩種方式實現：`while`循環和`parallel`並行操作。

### Original
利用`while`循環，較爲耗時

```bash
#!/usr/bin/env bash

download_tool=${download_tool:-'wget -qO-'}  # curl -fsL
version_list_info=$(mktemp -t tempXXXXX.txt)

# - version|distro_list
$download_tool https://www.percona.com/downloads/ | sed -r -n '/A drop-in replacement for MySQL/,/<\/div>/{/<a/{s@.* href="([^"]*)".*>Download ([[:digit:].]+).*@\2 https://www.percona.com\1@g;p}}' | while read -r version url; do
    distro_list=${distro_list:-}
    distro_list=$($download_tool $url | sed -r -n '/Select Software Platform/{s@redhat/@rhel@g;s@<\/option>@\n@g;p}' | sed -r -n 's@.*/([^"]*)"[[:space:]]*>.*@\1@g;/binary|source|select|^$/d;p' | sed -r ':a;N;$!ba;s@\n@ @g;')
    echo "${version}|${distro_list}" >> "${version_list_info}"
done

cat "${version_list_info}"
# 8.0|stretch rhel6 rhel7 xenial bionic cosmic
# 5.7|jessie stretch rhel6 rhel7 trusty xenial bionic cosmic
# 5.6|jessie stretch rhel6 rhel7 trusty xenial bionic cosmic
# 5.5|jessie stretch rhel6 rhel7 trusty xenial bionic
# 5.1|squeeze rhel5 rhel6 lucid precise

# - distro|version_list
sed -r 's@.*\|@@g' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;s@ @\n@g' | awk '!a[$0]++' | while read -r distro; do
    version_list=$(awk -F\| 'match($NF,/[[:space:]]*rhel6[[:space:]]*/){print $1}' "${version_list_info}" | sed -r ':a;N;$!ba;s@\n@ @g;')
    echo "$distro|$version_list"
done

# stretch|8.0 5.7 5.6 5.5 5.1
# rhel6|8.0 5.7 5.6 5.5 5.1
# rhel7|8.0 5.7 5.6 5.5 5.1
# xenial|8.0 5.7 5.6 5.5 5.1
# bionic|8.0 5.7 5.6 5.5 5.1
# cosmic|8.0 5.7 5.6 5.5 5.1
# jessie|8.0 5.7 5.6 5.5 5.1
# trusty|8.0 5.7 5.6 5.5 5.1
# squeeze|8.0 5.7 5.6 5.5 5.1
# rhel5|8.0 5.7 5.6 5.5 5.1
# lucid|8.0 5.7 5.6 5.5 5.1
# precise|8.0 5.7 5.6 5.5 5.1

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
# 5.5|jessie stretch rhel6 rhel7 trusty xenial bionic
# 5.1|squeeze rhel5 rhel6 lucid precise
# 8.0|stretch rhel6 rhel7 xenial bionic cosmic
# 5.6|jessie stretch rhel6 rhel7 trusty xenial bionic cosmic
# 5.7|jessie stretch rhel6 rhel7 trusty xenial bionic cosmic


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

# jessie|5.5 5.6 5.7
# stretch|5.5 8.0 5.6 5.7
# rhel6|5.5 5.1 8.0 5.6 5.7
# rhel7|5.5 8.0 5.6 5.7
# trusty|5.5 5.6 5.7
# xenial|5.5 8.0 5.6 5.7
# bionic|5.5 8.0 5.6 5.7
# squeeze|5.1
# rhel5|5.1
# lucid|5.1
# precise|5.1
# cosmic|8.0 5.6 5.7


[[ -f "${version_list_info}" ]] && rm -f "${version_list_info}"
unset version_list_info
```

## Performance Comparison
使用bash內置命令`time`對操作進行計時

```bash
# Original Time
real	0m3.334s
user	0m0.090s
sys	0m0.023s

# Parallel Time
real	0m1.730s
user	0m0.443s
sys	0m0.128s
```

可以看到使用`parallel`，操作耗時節省接近一半時間。


## Reference
* [Installing Percona Server on Debian and Ubuntu](https://www.percona.com/doc/percona-server/LATEST/installation/apt_repo.html)
* [Installing Percona Server on Red Hat Enterprise Linux and CentOS](https://www.percona.com/doc/percona-server/LATEST/installation/yum_repo.html)


## Change Logs
* 2017.08.13 21:26 Sun Asia/Shanghai
    * 初稿完成
* 2018.07.27 15:39 Fri America/Boston
    * 勘誤，更新，遷移到新Blog
* 2019.04.26 13:09 Fri America/Boston
    * 提取結果更新，增加`8.0`



[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[percona]:https://www.percona.com "The Database Performance Experts"
[mariadb]:https://mariadb.com/ "MariaDB | Enterprise Open Source Database & Data Warehouse"
[percona_software_downloads]:https://www.percona.com/downloads/ "Percona"

<!-- End -->
