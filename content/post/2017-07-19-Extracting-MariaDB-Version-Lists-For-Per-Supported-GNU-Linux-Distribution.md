---
title: Extracting MariaDB Version Lists For Per Supported GNU/Linux Distribution
slug: Extracting MariaDB Version Lists For Per Supported GNU Linux Distribution
date: 2017-07-19T17:41:33+08:00
lastmod: 2019-04-26T13:28:50-04:00
draft: false
keywords: ["MySQL Variant", "MariaDB version", "Shell script"]
description: "Extracting MariaDB Version Lists For Per Supported GNU Linux Distribution"
categories:
- Data Process
tags:
- MariaDB
- Shell Script
- awk
- sed
comment: true
toc: true

---

[MariaDB][mariadb]支持多種GNU/Linux發行版，爲方便生成對應GNU/Linux發行版本的repository文件，官方提供了 [Setting up MariaDB Repositories][mariadb_repository] 頁面，操作便捷。但[MariaDB][mariadb]官方並未顯式地說明MariaDB的每一個釋出版本(如`10.4`、`10.3`、`10.2`)具體支持哪些GNU/Linux發行版本。

本文通過 [Setting up MariaDB Repositories][mariadb_repository] 頁面的HTML源碼，提取出每個MariaDB所支持的GNU/Linux發行版本具體支持的MariaDB版本信息，通過Shell Script代碼實現。

爲實現 MySQL Variants ([MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona])在GNU/Linux中的自動安裝、配置，本人通過Shell腳本提取其對各GNU/Linux發行版本的具體支持信息：

* [Extracting MariaDB Version Lists For Per Supported GNU/Linux Distribution]({{< relref "2017-07-19-Extracting-MariaDB-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}}) (本文)
* [Extracting MySQL Version Lists For Per Supported GNU/Linux Distribution]({{< relref "2017-07-20-Extracting-MySQL-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}})
* [Extracting Percona Version Lists For Per Supported GNU/Linux Distribution]({{< relref "2017-08-13-Extracting-Percona-Version-Lists-For-Per-Supported-GNU-Linux-Distribution.md" >}})

數據庫系統安裝腳本代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/MySQLVariants.sh)，腳本同時支持在Debian/Ubuntu/CentOS/Fedora/OpenSUSE/SLES等發行版中安裝[MySQL][mysql]、[MariaDB][mariadb]、[Percona][percona]。

<!--more-->

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/MySQLVariants.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/243053.js" id="asciicast-243053" async></script>


## Shell Script
整個操作過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh)，通過如下命令執行

```bash
download_tool='wget -qO-'  # curl -fsL / wget -qO-
# if need help info, specify '-h'
$download_tool https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh | bash -s -- -d mariadb
```
提取結果

```bash
MariaDB|xenial|10.4 10.3 10.2 10.1 10.0
MariaDB|trusty|10.3 10.2 10.1 10.0 5.5
MariaDB|stretch|10.4 10.3 10.2 10.1
MariaDB|sles15|10.4 10.3 10.2|sles
MariaDB|sles12|10.4 10.3 10.2 10.1 10.0|sles
MariaDB|sid|10.4 10.3 10.2 10.1|debian
MariaDB|rhel8|10.4 10.3|rhel
MariaDB|rhel7|10.4 10.3 10.2 10.1 10.0 5.5|rhel
MariaDB|rhel6|10.3 10.2 10.1 10.0 5.5|rhel
MariaDB|opensuse42|10.4 10.3 10.2 10.1|opensuse
MariaDB|opensuse15|10.4 10.3 10.2|opensuse
MariaDB|jessie|10.4 10.3 10.2 10.1 10.0
MariaDB|fedora29|10.4 10.3|fedora
MariaDB|fedora28|10.4 10.3 10.2|fedora
MariaDB|fedora27|10.2|fedora
MariaDB|cosmic|10.4 10.3 10.2
MariaDB|centos7|10.4 10.3 10.2 10.1 10.0 5.5|centos
MariaDB|centos6|10.3 10.2 10.1 10.0 5.5|centos
MariaDB|buster|10.4
MariaDB|bionic|10.4 10.3 10.2 10.1
MariaDB|artful|10.2 10.1
```

## Requirement
提取每一個MariaDB所支持的GNU/Linux發行版本具體支持的MariaDB版本列表

1. [MariaDB][mariadb]具體支持哪些GNU/Linux發行版；
2. 每一個GNU/Linux發行版具體支持MariaDB哪些版本；

[MariaDB][mariadb]在[MariaDB Package Repository Setup and Usage](https://mariadb.com/kb/en/mariadb/mariadb-package-repository-setup-and-usage/#platform-support)中列出了支持的GNU/Linux發行版及[安裝命令](https://mariadb.com/kb/en/mariadb/mariadb-package-repository-setup-and-usage/#installing-packages)。


## Analysis
分析[Setting up MariaDB Repositories][mariadb_repository]頁面的HTML源碼(快捷鍵[Ctrl+U](https://www.computerhope.com/issues/ch000746.htm "How to view the HTML source code of a web page"))，該頁面可大致分爲5部分，其分隔行爲含有如下關鍵詞的行

1. Choose a Distro
2. Choose a Release
3. Choose a Version
4. Choose a Mirror

[MariaDB][mariadb]具體支持哪些GNU/Linux發行版，可通過`/Choose a Release/，/Choose a Version/`提取；

每一個[MariaDB][mariadb]釋出版本支持哪些GNU/Linux發行版，可通過`/Choose a Version/，/Choose a Mirror/`提取；

通過`while`遍歷操作，提取出所需的數據。

## Code Snippets
以下Shell Scirpt代碼段初步提取所需的數據

### Distros Supported By MariaDB

```bash
# all support distribution by MariaDB 判断MariaDB是否支持该GNU/Linux发行版
$download_tool https://downloads.mariadb.org/mariadb/repositories| sed -r -n '/Choose a Release/,/Choose a Version/{/<\/(li|ul|div)>/d;s@^[[:space:]]*@@g;s@.*data-value="([^"]*)".*@\1@g;/^(<|[[:upper:]])/d;s@^$@@g;p}' | sed '/^$/d'
```

輸出結果爲

```bash
centos7-ppc64le--centos7
centos7-ppc64--centos7
centos7-amd64--centos7
centos6-amd64--centos6
centos6-x86--centos6
sid--sid
buster--buster
stretch--stretch
jessie--jessie
fedora29-amd64--fedora29
fedora28-amd64--fedora28
fedora27-amd64--fedora27
rhel8-amd64--rhel8
rhel7-ppc64le--rhel7
rhel7-ppc64--rhel7
rhel7-amd64--rhel7
rhel6-amd64--rhel6
rhel6-x86--rhel6
cosmic--ubuntu_cosmic
bionic--ubuntu_bionic
artful--ubuntu_artful
xenial--ubuntu_xenial
trusty--ubuntu_trusty
bionic--mint19
xenial--mint18
trusty--mint171_rebecca
trusty--mint17_qiana
opensuse15-amd64--opensuse15
opensuse42-amd64--opensuse42
sles15-amd64--sles15
sles12-ppc64le--sles12
sles12-amd64--sles12
```

### Distros For Per MariaDB Release Version

```bash
# all supported distribution lists for every MariaDB release version
$download_tool https://downloads.mariadb.org/mariadb/repositories | sed -r -n '/Choose a Version/,/Choose a Mirror/{s@^[[:space:]]*@@g;/^<[^(\/?li)]/d;p}' | awk '{if($0!~/^<\/li>/){ORS=" ";print $0}else{printf "\n"}}' | sed -r -n '/class=""/d;s@.* data-value="([^"]*)".*class="[[:space:]]*([^"]*)".*>([[:digit:].]+)[[:space:]]*\[(.*)\]@\L\3|\4|\2@g;/^[[:digit:]]/!d;p'
```

輸出結果爲

```bash
10.3|stable|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 centos6-amd64--centos6 centos6-x86--centos6 sles15-amd64--sles15 sles12-ppc64le--sles12 sles12-amd64--sles12 opensuse15-amd64--opensuse15 opensuse42-amd64--opensuse42 fedora29-amd64--fedora29 fedora28-amd64--fedora28 cosmic--ubuntu_cosmic bionic--ubuntu_bionic xenial--ubuntu_xenial trusty--ubuntu_trusty bionic--mint19 xenial--mint18 rhel8-amd64--rhel8 rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 rhel6-amd64--rhel6 rhel6-x86--rhel6 sid--sid stretch--stretch jessie--jessie  
10.4|rc|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 sles15-amd64--sles15 sles12-ppc64le--sles12 sles12-amd64--sles12 opensuse15-amd64--opensuse15 opensuse42-amd64--opensuse42 fedora29-amd64--fedora29 fedora28-amd64--fedora28 cosmic--ubuntu_cosmic bionic--ubuntu_bionic xenial--ubuntu_xenial rhel8-amd64--rhel8 rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 sid--sid buster--buster stretch--stretch jessie--jessie  
10.2|old stable|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 centos6-amd64--centos6 centos6-x86--centos6 sles15-amd64--sles15 sles12-ppc64le--sles12 sles12-amd64--sles12 opensuse15-amd64--opensuse15 opensuse42-amd64--opensuse42 fedora28-amd64--fedora28 fedora27-amd64--fedora27 cosmic--ubuntu_cosmic bionic--ubuntu_bionic artful--ubuntu_artful xenial--ubuntu_xenial trusty--ubuntu_trusty bionic--mint19 xenial--mint18 trusty--mint171_rebecca trusty--mint17_qiana rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 rhel6-amd64--rhel6 rhel6-x86--rhel6 sid--sid stretch--stretch jessie--jessie  
10.1|old stable|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 centos6-amd64--centos6 centos6-x86--centos6 sles12-ppc64le--sles12 sles12-amd64--sles12 opensuse42-amd64--opensuse42 bionic--ubuntu_bionic artful--ubuntu_artful xenial--ubuntu_xenial trusty--ubuntu_trusty bionic--mint19 xenial--mint18 trusty--mint171_rebecca trusty--mint17_qiana rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 rhel6-amd64--rhel6 rhel6-x86--rhel6 sid--sid stretch--stretch jessie--jessie  
10.0|old stable|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 centos6-amd64--centos6 centos6-x86--centos6 sles12-ppc64le--sles12 sles12-amd64--sles12 xenial--ubuntu_xenial trusty--ubuntu_trusty xenial--mint18 trusty--mint171_rebecca trusty--mint17_qiana rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 rhel6-amd64--rhel6 rhel6-x86--rhel6 jessie--jessie  
5.5|old stable|centos7-ppc64le--centos7 centos7-ppc64--centos7 centos7-amd64--centos7 centos6-amd64--centos6 centos6-x86--centos6 trusty--ubuntu_trusty trusty--mint171_rebecca trusty--mint17_qiana rhel7-ppc64le--rhel7 rhel7-ppc64--rhel7 rhel7-amd64--rhel7 rhel6-amd64--rhel6 rhel6-x86--rhel6
```


## Shell Script Snippets
以下爲Shell Script代碼段

```bash
#!/usr/bin/env bash

mariadb_repositories_page='https://downloads.mariadb.org/mariadb/repositories'
page_source=$(mktemp -t tempXXXXX.txt)
distro_lists_per_mariadb_version=$(mktemp -t tempXXXXX.txt)

# -s  FILE exists and has a size greater than zero
[[ -s "${page_source}" ]] || curl -fsL "${mariadb_repositories_page}" > "${page_source}"


# all supported distribution lists for every MariaDB release version
[[ -f "${distro_lists_per_mariadb_version}" ]] && echo '' > "${distro_lists_per_mariadb_version}"

sed -r -n '/Choose a Version/,/Choose a Mirror/{s@^[[:space:]]*@@g;/^<[^(\/?li)]/d;p}' "${page_source}" | awk '{if($0!~/^<\/li>/){ORS=" ";print $0}else{printf "\n"}}' | sed -r -n '/class=""/d;s@.* data-value="([^"]*)".*class="[[:space:]]*([^"]*)".*>([[:digit:].]+)[[:space:]]*\[(.*)\]@\L\3|\4|\2@g;/^[[:digit:]]/!d;p' | while IFS="|" read -r version types distro;do
    lists=$(echo "$distro" | sed 's@ @\n@g' | awk -F- '{a[$1]++}END{for(i in a) printf("%s ",i)}' | sed -r 's@^[[:space:]]*@@g;s@[[:space:]]*$@\n@g')
    echo "${version}|${types}|${lists}" >> "${distro_lists_per_mariadb_version}"
done

# all distribution supported by MariaDB
sed -r -n '/Choose a Release/,/Choose a Version/{/<\/(li|ul|div)>/d;s@^[[:space:]]*@@g;s@.*data-value="([^"]*)".*@\1@g;/^(<|[[:upper:]])/d;s@^$@@g;p}' "${page_source}" | awk -F- '!match($0,/^$/){a[$1]++}END{PROCINFO["sorted_in"]="@ind_str_desc";for(i in a) print i}' | while read -r line; do
    lists=$(awk -F\| 'match($NF,/'"${line}"'/){a[$1]++}END{PROCINFO["sorted_in"]="@ind_num_desc";for(i in a) printf("%s ",i)}' "${distro_lists_per_mariadb_version}" | sed -r 's@^[[:space:]]*@@g;s@[[:space:]]*$@\n@g')
    # sleep 1
    echo "${line}|${lists}"
done


# EXIT Singal Processing While Execution Finished
funcTrapEXIT(){
    unset mariadb_repositories_page
    [[ -f "${page_source}" ]] && rm -f "${page_source}"
    [[ -f "${distro_lists_per_mariadb_version}" ]] && rm -f "${distro_lists_per_mariadb_version}"
    unset page_source
    unset distro_lists_per_mariadb_version
}

trap funcTrapEXIT EXIT

# Script End
```

## Output
結果輸出

```bash
xenial|10.4 10.3 10.2 10.1 10.0
trusty|10.3 10.2 10.1 10.0 5.5
stretch|10.4 10.3 10.2 10.1
sles15|10.4 10.3 10.2
sles12|10.4 10.3 10.2 10.1 10.0
sid|10.4 10.3 10.2 10.1
rhel8|10.4 10.3
rhel7|10.4 10.3 10.2 10.1 10.0 5.5
rhel6|10.3 10.2 10.1 10.0 5.5
opensuse42|10.4 10.3 10.2 10.1
opensuse15|10.4 10.3 10.2
jessie|10.4 10.3 10.2 10.1 10.0
fedora29|10.4 10.3
fedora28|10.4 10.3 10.2
fedora27|10.2
cosmic|10.4 10.3 10.2
centos7|10.4 10.3 10.2 10.1 10.0 5.5
centos6|10.3 10.2 10.1 10.0 5.5
buster|10.4
bionic|10.4 10.3 10.2 10.1
artful|10.2 10.1
```

操作耗時

```bash
real	0m1.498s
user	0m0.101s
sys	0m0.035s
```

列表形式展示

Distro|MariaDB Version
---|---
xenial|10.4 10.3 10.2 10.1 10.0
trusty|10.3 10.2 10.1 10.0 5.5
stretch|10.4 10.3 10.2 10.1
sles15|10.4 10.3 10.2
sles12|10.4 10.3 10.2 10.1 10.0
sid|10.4 10.3 10.2 10.1
rhel8|10.4 10.3
rhel7|10.4 10.3 10.2 10.1 10.0 5.5
rhel6|10.3 10.2 10.1 10.0 5.5
opensuse42|10.4 10.3 10.2 10.1
opensuse15|10.4 10.3 10.2
jessie|10.4 10.3 10.2 10.1 10.0
fedora29|10.4 10.3
fedora28|10.4 10.3 10.2
fedora27|10.2
cosmic|10.4 10.3 10.2
centos7|10.4 10.3 10.2 10.1 10.0 5.5
centos6|10.3 10.2 10.1 10.0 5.5
buster|10.4
bionic|10.4 10.3 10.2 10.1
artful|10.2 10.1


## References
* [Setting up MariaDB Repositories][mariadb_repository]


## Change Logs
* 2017.07.19 17:39 Wed Asia/Shanghai
    * 初稿完成
* 2018.07.27 15:03:28 Fri America/Boston
    * 勘誤，更新，遷移到新Blog
* 2019.04.26 13:28 Fri America/Boston
    * 提取結果更新，增加`8.0`



[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[percona]:https://www.percona.com "The Database Performance Experts"
[mariadb]:https://mariadb.com/ "MariaDB | Enterprise Open Source Database & Data Warehouse"
[mariadb_repository]:https://downloads.mariadb.org/mariadb/repositories "MariaDB - Setting up MariaDB Repositories"

<!-- End -->
