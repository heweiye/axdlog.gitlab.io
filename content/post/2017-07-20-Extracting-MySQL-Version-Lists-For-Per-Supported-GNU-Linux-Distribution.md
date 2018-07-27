---
title: Extracting MySQL Version Lists For Per Supported GNU/Linux Distribution
slug: Extracting MySQL Version Lists For Per Supported GNU Linux Distribution
date: 2017-07-20T18:56:54+08:00
lastmod: 2018-07-27T15:20:50-04:00
draft: false
keywords: ["AxdLog", "MySQL version", "Shell script"]
description: "Extracting MySQL Version Lists For Per Supported GNU/Linux Distribution"
categories:
- Data Process
tags:
- MySQL
- Shell Script
- awk
- sed
comment: true
toc: true

---

[MySQL][mysql]支持RHEL/CentOS/Fedora/Debian/Ubuntu/SLES等GNU/Linux發行版，文檔頁[MySQL Repositories][mysql_repositories]列出了具體支持的發行版本，但並未顯式地說明MySQL的每一個釋出版本(如`5.6`，`5.7`，`8.0`)具體支持哪些GNU/Linux發行版本。

本文通過[MySQL Repositories][mysql_repositories]及其子頁面提取repo的安裝包地址，通過解壓後的文件提取每個MySQL所支持的GNU/Linux發行版本具體支持的MySQL版本信息，通過Shell Script代碼實現。

<!--more-->

## Shell 腳本
整個操作過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-
# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/tool/mysqlVariantsVersionAndLinuxDistroRelationTable.sh | bash -s -- -d mysql
```
提取結果

```bash
MySQL|artful|5.7 8.0
MySQL|bionic|5.7 8.0
MySQL|el6|8.0 5.7 5.6 5.5|rhel centos
MySQL|el7|8.0 5.7 5.6 5.5|rhel centos
MySQL|fc26|8.0 5.7|fedora
MySQL|fc27|8.0 5.7|fedora
MySQL|fc28|8.0 5.7|fedora
MySQL|jessie|5.6 5.7 8.0
MySQL|sles11|5.7 5.6 5.5|sles
MySQL|sles12|8.0 5.7 5.6|sles
MySQL|stretch|5.6 5.7 8.0
MySQL|trusty|5.6 5.7 8.0
MySQL|wheezy|5.6 5.7
MySQL|xenial|5.7 8.0
MySQL NDB Cluster|el6|7.6 7.5|rhel centos
MySQL NDB Cluster|el7|7.6 7.5|rhel centos
MySQL NDB Cluster|jessie|7.5 7.6
MySQL NDB Cluster|sles12|7.6 7.5|sles
MySQL NDB Cluster|stretch|7.5
MySQL NDB Cluster|trusty|7.5 7.6
MySQL NDB Cluster|wheezy|7.5 7.6
MySQL NDB Cluster|xenial|7.5 7.6
```

## Requirement
提取每一個MySQL所支持的GNU/Linux發行版本具體支持的MySQL版本列表

1. [MySQL][mysql]具體支持哪些GNU/Linux發行版；
2. 每一個GNU/Linux發行版具體支持MySQL哪些版本；

## Analysis
通過分析[MySQL Repositories][mysql_repositories]的子頁面，提取各個GNU/Linux發行版的repo安裝包的下載路徑。

* 對於RHEL，通過解壓出的`./etc/yum.repos.d/mysql-community.repo`文件提取；
* 對於SLES，通過解壓出的`./etc/zypp/repos.d/mysql-community.repo`文件提取；
* 對於Debian/Ubuntu，通過解壓出的`control.tar.gz`文件中的`config`腳本提取；

## Code Snippets
以下Shell Scirpt代碼段初步提取所需的數據

### MD5 Check
各個repo安裝包提供對應的md5校驗碼，爲安全起見，對其進行校驗。

```bash
#!/usr/bin/env bash

funcMySQLRepoMD5Check(){
    repo_type="${1:-}"

    local repo_page=${repo_page:-}
    case "${repo_type,,}" in
        yum|y )
            repo_page='https://dev.mysql.com/downloads/repo/yum/'
            ;;
        suse|sles|s)
            repo_page='https://dev.mysql.com/downloads/repo/suse/'
            ;;
        debian|ubuntu|apt|a)
            repo_page='https://dev.mysql.com/downloads/repo/apt/'
            ;;
    esac

    curl -fsL "${repo_page}" | sed -r -n '/<table/,/<\/table>/{/(button03|sub-text|md5)/!d;/style=/d;s@^[^<]*@@g;s@.*href="(.*)".*@https://dev.mysql.com\1@g;s@.*\((.*)\).*@\1@g;s@[[:space:]]*<\/td>@\n@g;s@<[^>]*>@@g;p}' | awk '{if($0!~/^$/){ORS="|";print $0}else{printf "\n"}}' | while IFS="|" read -r download_page pack_name md5_official; do
        pack_download_link=$(curl -fsL "${download_page}" | sed -r -n '/thanks/{s@.*"(.*)".*@https://dev.mysql.com\1@g;p}')
        pack_download_link="https://dev.mysql.com/get/${pack_name}"

        file_path="/tmp/${pack_name}"
        curl -fsL "${pack_download_link}" > "${file_path}"
        md5_checksum=$(md5sum "${file_path}" | awk '{print $1}')
        # md5_checksum=$(openssl dgst -md5 "${file_path}" | awk '{print $NF}')
        if [[ "${md5_official}" == "${md5_checksum}" ]]; then
            echo "File ${pack_name}, MD5 ${md5_checksum} approved"
        else
            echo "File ${pack_name}, MD5 ${md5_checksum} not approved"
        fi

        [[ -f "${file_path}" ]] && rm -f "${file_path}"

    done

}

# invoking custom function
funcMySQLRepoMD5Check 'apt'
funcMySQLRepoMD5Check 'suse'
funcMySQLRepoMD5Check 'yum'

# Script End
```

結果輸出爲

```bash
File mysql-apt-config_0.8.10-1_all.deb, MD5 5b36dd754e7752162f890206fae50931 approved
File mysql80-community-release-sles12-1.noarch.rpm, MD5 173f77e69bd1bd575306502c5e453371 approved
File mysql57-community-release-sles11-8.noarch.rpm, MD5 1fb3059ea573a8d22b0e320a84ec3dda approved
File mysql80-community-release-el7-1.noarch.rpm, MD5 739dc44566d739c5d7b893de96ee6848 approved
File mysql80-community-release-el6-1.noarch.rpm, MD5 f2befc44a4b8416864987b1686c4a72b approved
File mysql80-community-release-fc28-1.noarch.rpm, MD5 5249ce5e0a1019546cd2eb7c70961f3c approved
File mysql80-community-release-fc27-1.noarch.rpm, MD5 02b160b5c1432c75a0707d3eaaa184fc approved
File mysql80-community-release-fc26-1.noarch.rpm, MD5 c7554370ac3ef84c76211b06159ce94a approved
```


### MySQL Version Lists For RPM
適用於RHEL/SUSE

#### While Loop
通過`while`遍歷，效率較低

```bash
#!/usr/bin/env bash

funcMySQLForRPM(){
    rpm_type="${1:-}"

    local repo_page=${repo_page:-}
    local target_path=${target_path:-}
    case "${rpm_type,,}" in
        yum|y )
            repo_page='https://dev.mysql.com/downloads/repo/yum/'
            target_path='/etc/yum.repos.d/mysql-community.repo'
            ;;
        suse|sles|s)
            repo_page='https://dev.mysql.com/downloads/repo/suse/'
            target_path='/etc/zypp/repos.d/mysql-community.repo'
            ;;
    esac


    curl -fsL "${repo_page}" | sed -r -n '/<table/,/<\/table>/{/(button03|sub-text|md5)/!d;/style=/d;s@^[^<]*@@g;s@.*href="(.*)".*@https://dev.mysql.com\1@g;s@.*\((.*)\).*@\1@g;s@[[:space:]]*<\/td>@\n@g;s@<[^>]*>@@g;p}' | awk '{if($0!~/^$/){ORS="|";print $0}else{printf "\n"}}' | while IFS="|" read -r download_page pack_name md5_official; do
        distro=$(echo "${pack_name}" | sed -r -n 's@.*release-([[:alnum:]]+).*@\1@g;p')
        pack_download_link=$(curl -fsL "${download_page}" | sed -r -n '/thanks/{s@.*"(.*)".*@https://dev.mysql.com\1@g;p}')
        pack_download_link="https://dev.mysql.com/get/${pack_name}"

        file_path="/tmp/${pack_name}"
        curl -fsL "${pack_download_link}" > "${file_path}"
        md5_checksum=$(md5sum "${file_path}" | awk '{print $1}')
        # md5_checksum=$(openssl dgst -md5 "${file_path}" | awk '{print $NF}')
        if [[ "${md5_official}" != "${md5_checksum}" ]]; then
            echo "File ${pack_name}, MD5 ${md5_checksum} not approved"
        else
            # echo "File ${pack_name}, MD5 ${md5_checksum} approved"
            extract_dir="/tmp/${pack_name%%.*}"
            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            mkdir -p "${extract_dir}"
            # rpm2cpio mysql57-community-release-el7.rpm | cpio -idmv
            cd "${extract_dir}" && rpm2cpio "${file_path}" | cpio -idm 2> /dev/null
            extract_target_path="${extract_dir}${target_path}"
            [[ -s "${extract_target_path}" ]] && awk 'BEGIN{ORS=" "}match($0,/^\[mysql[[:digit:]]/){getline;print gensub(/.* ([[:digit:].]*) .*/,"\\1","g",$0)}' "${extract_target_path}" | sed 's@[[:space:]]*$@\n@;s@ @,@g;s@.*@'"${distro}"' &@'

            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            [[ -f "${file_path}" ]] && rm -f "${file_path}"

        fi

    done
}

# invoking custom function
funcMySQLForRPM 'yum'
funcMySQLForRPM 'suse'

# Script End
```


輸出結果爲

```bash
el7 5.5,5.6,5.7,8.0
el6 5.5,5.6,5.7,8.0
fc28 5.7,8.0
fc27 5.7,8.0
fc26 5.7,8.0
sles12 5.6,5.7,8.0
sles11 5.5,5.6,5.7
```

#### Parallel
通過`parallel`命令並行操作，效率較高

```bash
#!/usr/bin/env bash

funcMySQLOperationForRPM(){
    local item=${1:-}
    if [[ -n "${item}" ]]; then
        local download_page=${download_page:-}
        local pack_name=${pack_name:-}
        local md5_official=${md5_official:-}
        download_page=$(echo "${item}" | awk -F\| '{print $1}')
        pack_name=$(echo "${item}" | awk -F\| '{print $2}')
        md5_official=$(echo "${item}" | awk -F\| '{print $3}')

        distro=$(echo "${pack_name}" | sed -r -n 's@.*release-([[:alnum:]]+).*@\1@g;p')
        pack_download_link=$(curl -fsL "${download_page}" | sed -r -n '/thanks/{s@.*"(.*)".*@https://dev.mysql.com\1@g;p}')
        pack_download_link="https://dev.mysql.com/get/${pack_name}"

        file_path="/tmp/${pack_name}"
        curl -fsL "${pack_download_link}" > "${file_path}"
        md5_checksum=$(md5sum "${file_path}" | awk '{print $1}')
        # md5_checksum=$(openssl dgst -md5 "${file_path}" | awk '{print $NF}')
        if [[ "${md5_official}" != "${md5_checksum}" ]]; then
            echo "File ${pack_name}, MD5 ${md5_checksum} not approved"
        else
            # echo "File ${pack_name}, MD5 ${md5_checksum} approved"
            extract_dir="/tmp/${pack_name%%.*}"
            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            mkdir -p "${extract_dir}"
            # rpm2cpio mysql57-community-release-el7.rpm | cpio -idmv
            cd "${extract_dir}" && rpm2cpio "${file_path}" | cpio -idm 2> /dev/null
            extract_target_path="${extract_dir}${target_path}"
            [[ -s "${extract_target_path}" ]] && awk 'BEGIN{ORS=" "}match($0,/^\[mysql[[:digit:]]/){getline;print gensub(/.* ([[:digit:].]*) .*/,"\\1","g",$0)}' "${extract_target_path}" | sed 's@[[:space:]]*$@\n@;s@ @,@g;s@.*@'"${distro}"' &@'

            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            [[ -f "${file_path}" ]] && rm -f "${file_path}"

        fi

    fi
}

funcMySQLForRPM(){
    rpm_type="${1:-}"

    local repo_page=${repo_page:-}
    local target_path=${target_path:-}
    case "${rpm_type,,}" in
        yum|y )
            repo_page='https://dev.mysql.com/downloads/repo/yum/'
            target_path='/etc/yum.repos.d/mysql-community.repo'
            ;;
        suse|sles|s)
            repo_page='https://dev.mysql.com/downloads/repo/suse/'
            target_path='/etc/zypp/repos.d/mysql-community.repo'
            ;;
    esac

    export -f funcMySQLOperationForRPM
    export target_path="${target_path}"
    curl -fsL "${repo_page}" | sed -r -n '/<table/,/<\/table>/{/(button03|sub-text|md5)/!d;/style=/d;s@^[^<]*@@g;s@.*href="(.*)".*@https://dev.mysql.com\1@g;s@.*\((.*)\).*@\1@g;s@[[:space:]]*<\/td>@\n@g;s@<[^>]*>@@g;p}' | awk '{if($0!~/^$/){ORS="|";print $0}else{printf "\n"}}' | parallel -k -j 0 funcMySQLOperationForRPM 2> /dev/null
}

# invoking custom function
funcMySQLForRPM 'yum'
funcMySQLForRPM 'suse'

# Script End
```

輸出結果爲

```bash
el7 5.5,5.6,5.7,8.0
el6 5.5,5.6,5.7,8.0
fc28 5.7,8.0
fc27 5.7,8.0
fc26 5.7,8.0
sles12 5.6,5.7,8.0
sles11 5.5,5.6,5.7
```

### MySQL Version Lists For DEB
適用於Debian/Ubuntu

```bash
#!/usr/bin/env bash

funcMySQLForDEB(){
    local repo_page=${repo_page:-'https://dev.mysql.com/downloads/repo/apt/'}

    curl -fsL "${repo_page}" | sed -r -n '/<table/,/<\/table>/{/(button03|sub-text|md5)/!d;/style=/d;s@^[^<]*@@g;s@.*href="(.*)".*@https://dev.mysql.com\1@g;s@.*\((.*)\).*@\1@g;s@[[:space:]]*<\/td>@\n@g;s@<[^>]*>@@g;p}' | awk '{if($0!~/^$/){ORS="|";print $0}else{printf "\n"}}' | while IFS="|" read -r download_page pack_name md5_official; do
        pack_download_link=$(curl -fsL "${download_page}" | sed -r -n '/thanks/{s@.*"(.*)".*@https://dev.mysql.com\1@g;p}')
        pack_download_link="https://dev.mysql.com/get/${pack_name}"

        file_path="/tmp/${pack_name}"
        curl -fsL "${pack_download_link}" > "${file_path}"
        md5_checksum=$(md5sum "${file_path}" | awk '{print $1}')
        # md5_checksum=$(openssl dgst -md5 "${file_path}" | awk '{print $NF}')
        if [[ "${md5_official}" != "${md5_checksum}" ]]; then
            echo "File ${pack_name}, MD5 ${md5_checksum} not approved"
        else
            # echo "File ${pack_name}, MD5 ${md5_checksum} approved"
            extract_dir="/tmp/${pack_name%%_*}"
            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            mkdir -p "${extract_dir}"
            cd "${extract_dir}" && ar -x "${file_path}"
            [[ -f "${extract_dir}/control.tar.gz" ]] && tar xf "${extract_dir}/control.tar.gz"
            [[ -s "${extract_dir}/config" ]] && sed -r -n '/case/,/esac/{s@^[[:space:]]*@@g;s@\)@@g;/(;;|case|esac)/d;s@^[^"]*"@@g;s@(mysql-|preview)@@g;s@,?[[:space:]]*cluster.*$@@g;s@[[:space:]]*"?@@g;/(^$|\*)/d;p}' "${extract_dir}/config" | sed 'N;s@\n@ @g'

            [[ -d "${extract_dir}" ]] && rm -rf "${extract_dir}"
            [[ -f "${file_path}" ]] && rm -f "${file_path}"
        fi

    done
}

# invoking custom function
funcMySQLForDEB

# Script End
```

輸出結果爲

```bash
wheezy 5.6,5.7
jessie 5.6,5.7,8.0
stretch 5.6,5.7,8.0
trusty 5.6,5.7,8.0
xenial 5.7,8.0
artful 5.7,8.0
bionic 5.7,8.0
```


## References
* [A Quick Guide to Using the MySQL Yum Repository](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)
* [A Quick Guide to Using the MySQL APT Repository](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)
* [A Quick Guide to Using the MySQL SLES Repository](https://dev.mysql.com/doc/mysql-sles-repo-quick-guide/en/)


## Change Logs
* 2017.07.20 18:54 Thu Asia/Shanghai
    * 初稿完成
* 2018.07.27 15:28:36 Fri America/Boston
    * 勘誤，更新，遷移到新Blog


[mysql]:https://dev.mysql.com/
[mysql_repositories]:https://dev.mysql.com/downloads/repo/ "MySQL :: MySQL Repositories"

<!-- End -->
