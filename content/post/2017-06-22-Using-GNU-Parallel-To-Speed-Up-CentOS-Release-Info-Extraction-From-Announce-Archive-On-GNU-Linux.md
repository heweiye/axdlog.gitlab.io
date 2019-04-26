---
title: Using GNU Parallel To Speed Up CentOS Release Info Extraction From Announce Archive On GNU/Linux
slug: Using GNU Parallel To Speed Up CentOS Release Info Extraction From Announce Archive On GNU Linux
date: 2017-06-22T16:02:39+08:00
lastmod: 2019-04-26T16:15:01-04:00
draft: false
keywords: ["sed", "awk", "GNU Parallel", "CentOS"]
description: "Using GNU Parallel To Speed Up CentOS Release Info Extraction From Announce Archive On GNU/Linux"
categories:
- Data Process
tags:
- sed
- awk
- gnu parallel
- CentOS

comment: true
toc: true

---

由於[CentOS][centos]未在其[官網][centos]列出各release版本的具體釋出時間，只能通過 **遍歷** 其[Announce Archive][announce_archive]頁面中的所有歷史文檔實現。[Announce Archive][announce_archive]頁面截至 April 2019，按月份共有 **170** 個鏈接，需逐個獲取其HTML頁面用於分析。在Shell Script實現過程中遇到一個問題：採用`while`命令逐行讀取進行操作導致 **遍歷** 操作耗時過長，將近 *180s* 。後嘗試通過[GNU Parallel][parallel]改寫腳本，成功將操作耗時控制在 *15~10s* 之內，運行時間縮短 *90%* 以上。

此爲 [主流GNU/Linux發行版的產品支持週期信息]({{< relref "2017-02-13-Life-Cycle-Info-For-GNU-Linux-Distributions.md" >}}) 的子項目。

<!--more-->

## Shell 腳本
Shell腳本託管在 [GitLab][lifecyclescript]，目前只支持[RHEL][rhel]/[CentOS][centos]、 [Debian][debian]/[Ubuntu][ubuntu]這4個發行版的信息提取。

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxLifeCycleInfo.sh | bash -s --
```

<script src="https://asciinema.org/a/189194.js" id="asciicast-189194" async></script>


## Origin Code
原始代碼如下

採用兩層`while`嵌套進行遍歷操作

```bash
#!/usr/bin/evn bash
centos_release_note=$(mktemp -t tempXXXXX.txt)
centos_announce_archive=$(mktemp -t tempXXXXX.txt)
release_note_site='https://wiki.centos.org/Manuals/ReleaseNotes'    # Release Notes for supported CentOS distributions
announce_archive_site='https://lists.centos.org/pipermail/centos-announce/'     # The CentOS-announce Archives
wiki_site='https://wiki.centos.org'

# Step 1.1 通過ReleaseNotes頁面提取各Release版本的Release note
curl -fsL "${release_note_site}" | sed -r -n '/Release Notes for CentOS [[:digit:]]/{s@<\/a>@\n@g;p}' | sed -r -n '/ReleaseNotes\/CentOS/{s@.*\"(.*)\".*@\1@g;p}' | while read line; do
    release_note_page="${wiki_site}${line}"
    release_version=${line##*'CentOS'}
    if [[ "${release_version}" == '7' ]]; then
        # Step 1.2 通過各Release版本的Release note提取準確的release版本號
        release_version=$(curl -fsL "${release_note_page}" | sed -r -n '/^<h1/{s@<[^>]*>@@g;s@ \(@.@g;s@\)@@g;s@-@ @g;s@([[:alpha:]]|[[:space:]])@@g;p}')
    fi
    # 7.1611|https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7
    # 7.1406|https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1406
    echo "$release_version|$release_note_page" >> "${centos_release_note}"
done


# Step 2.1 通過CentOS-announce Archives頁面，遍歷各個月份的Archives頁面
# June 2017|2017-June/date.html
curl -fsL "${announce_archive_site}" | sed -r -n '/Downloadable/,/\/table/{/(Thread|Subject|Author|Gzip|table|<tr>)/d;s@<\/?td>@@g;s@^[[:space:]]*@@g;/^$/d;p}' | sed -r ':a;N;$!ba;s@\n@ @g;s@<\/tr>@\n@g;' | sed -r -n '/href/{s@^[[:space:]]*@@g;s@([^:]*):.*\"(.*)\".*@\1|\2@g;p}' | while IFS="|" read -r archive_month url; do
    # https://lists.centos.org/pipermail/centos-announce/2017-June/date.html
    archive_url="${announce_archive_site}${url}"    # Month Year  Archives by date

    # Step 2.2 通過關鍵詞`Release for CentOS`篩選出各release版本對應的archive地址，提取具體時間
    # 6.9|https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html
    curl -fsL ${archive_url} | sed -r -n '/Release for CentOS( Linux |-)[[:digit:]].*x86_64$/{s@ \(@.@g;s@\)@@g;s@-@ @g;s@@@g;p}' | sed -r -n '/(Live|Minimal)/d;s@.*=\"([^"]*)\".*Release for .* ([[:digit:].]+) .*@\2|'"${archive_url%/*}"'/\1@g;p' | while IFS="|" read -r release_no release_archive_page; do
        release_date=$(curl -fsL "${release_archive_page}" | awk '$0~/<I>/{a=gensub(/.*>(.*)<.*/,"\\1","g",$0);"TZ=\"UTC\" date --date=\""a"\" +\"%F %Z\"" | getline b;print b}')    # %F %T %Z
        # 6.9|2017-04-05 UTC|https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html
        [[ "${release_no}" == '7' ]] && release_no='7.1406'
        [[ "${release_no}" == '5' ]] && release_no='5.0'
        echo "${release_no}|${release_date}"
        echo "${release_no}|${release_date}|${release_archive_page}" >> "${centos_announce_archive}"
    done
done

# Step 3 對提取到的數據按Release版本逆序排序，將Step1中獲取的release note地址合併到輸入結果中
# Markdown Output
# printf "Release Version|Release Date|Release Note\n---|---|---\n"
# Step 3.1 處理release版本排序
awk -F\| '{!a[$1]++;arr[$1]=$0}END{PROCINFO["sorted_in"]="@ind_str_desc";for (i in arr) print arr[i]}' "${centos_announce_archive}" | while IFS="|" read -r release_no release_date release_archive_page; do
    # Step 3.2 提取對應release版本的release note
    release_note_page=$(awk -F\| '$1=="'"${release_no}"'"{print $2}' "${centos_release_note}")
    printf "%s|%s\n" "${release_no}" "${release_date}"
    # printf "[%s](%s)|%s|[%s](%s)\n" "${release_no}" "${release_archive_page}" "${release_date}" "${release_note_page##*/}" "${release_note_page}"
done

[[ -f "${centos_release_note}" ]] && rm -f "${centos_release_note}"
unset centos_release_note
[[ -f "${centos_announce_archive}" ]] && rm -f "${centos_announce_archive}"
unset centos_announce_archive

# Script End
```


## Optimized Code
採用命令`parallel`實現並行操作，因暫未知曉如何在`parallel`中傳遞前一個命令中的變量，採用function函數進行操作。自定義函數須經`export -f function`定義後才能被`parallel`識別並使用。此外，供parallel使用的function，無法使用通過變量定義的文件路徑，原因未知。

```bash
#!/usr/bin/evn bash
# export -f 供parallel使用的function中，無法使用通過變量定義的文件路徑

centos_release_note=$(mktemp -t tempXXXXX.txt)
centos_announce_archive=$(mktemp -t tempXXXXX.txt)
centos_release_date=$(mktemp -t tempXXXXX.txt)
rhel_eus_date=$(mktemp -t tempXXXXX.txt)
rhel_life_cycle='https://access.redhat.com/support/policy/updates/errata/'  #Red Hat Enterprise Linux Life Cycle
release_note_site='https://wiki.centos.org/Manuals/ReleaseNotes'    # Release Notes for supported CentOS distributions
announce_archive_site='https://lists.centos.org/pipermail/centos-announce/'     # The CentOS-announce Archives
wiki_site='https://wiki.centos.org'

# Step 1.1 通過ReleaseNotes頁面提取各Release版本的Release note
funcReleaseNote(){
    curl -fsL "${release_note_site}" | sed -r -n '/Release Notes for CentOS [[:digit:]]/{s@<\/a>@\n@g;p}' | sed -r -n '/ReleaseNotes\/CentOS/{s@.*\"(.*)\".*@\1@g;p}' | while read line; do
        release_note_page="${wiki_site}${line}"
        # release_note_page_update_time=$(curl -fsL "${release_note_page}" | sed -r -n '/Last updated/{s@.*<\/strong> ([^>]*) <span class.*@\1@g;p}')  # page last update time
        release_version=${line##*'CentOS'}
        if [[ "${release_version}" == '7' ]]; then
            # Step 1.2 通過各Release版本的Release note提取準確的release版本號
            release_version=$(curl -fsL "${release_note_page}" | sed -r -n '/^<h1/{s@<[^>]*>@@g;s@ \(@.@g;s@\)@@g;s@-@ @g;s@([[:alpha:]]|[[:space:]])@@g;p}')
        fi
        # 7.1611|https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7
        # 7.1406|https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1406
        echo "$release_version|$release_note_page" >> "${centos_release_note}"
        # echo "$release_version|$release_note_page"
    done
}

# Step 1.2 通過RHEL的Life Cycle頁面提取各RHEL發行版的EUS日期
funcRedHatExtendedUpdateSupportDate(){
    curl -fsSL "${rhel_life_cycle}" | sed -r -n '/\(ends/{s@^[[:space:]]*@@g;s@(<[^>]*>|\(|\))@@g;s@ ends @|@g;s@([[:digit:]]+)st@\1@g;p}' | awk -F\| '{"date --date=\""$2"\" +\"%F\"" | getline a; printf("%s|%s\n",$1,a)}' >> "${rhel_eus_date}"
}

funcReleaseNote &
funcRedHatExtendedUpdateSupportDate &

# Step 2.1 通過CentOS-announce Archives頁面，遍歷各個月份的Archives頁面
# 通過各月份的archive頁，提取符合條件的Release信息，關鍵詞 Release for CentOS
funcSpecificReleaseInfo(){
    archive_url="${1}"
    # https://lists.centos.org/pipermail/centos-announce/2006-April/
    curl -fsL ${archive_url} | sed -r -n '/Release for CentOS( Linux |-)[[:digit:]].*x86_64$/{s@ \(@.@g;s@\)@@g;s@-@ @g;s@@@g;p}' | sed -r -n '/(Live|Minimal)/d;s@.*=\"([^"]*)\".*Release for .* ([[:digit:].]+) .*@\2|'"${archive_url}"'\1@g;p'
}

export -f funcSpecificReleaseInfo   # used for command parallel
# June 2017|2017-June/date.html
curl -fsL "${announce_archive_site}" | sed -r -n '/Downloadable/,/\/table/{/(Thread|Subject|Author|Gzip|table|<tr>)/d;s@<\/?td>@@g;s@^[[:space:]]*@@g;/^$/d;p}' | sed -r ':a;N;$!ba;s@\n@ @g;s@<\/tr>@\n@g;' | sed -r -n '/href/{s@^[[:space:]]*@@g;s@([^:]*):.*\"(.*)date.html\".*@'"${announce_archive_site}"'\2@g;p}' | parallel -k -j 0 funcSpecificReleaseInfo >> "${centos_announce_archive}"

# Step 2.2 通過各Release版本頁面提取release時間
funcReleaseDate(){
    line="$1"
    release_no=${line%%|*}
    release_archive_page=${line#*|}
    release_date=$(curl -fsL "${release_archive_page}" | awk '$0~/<I>/{a=gensub(/.*>(.*)<.*/,"\\1","g",$0);"TZ=\"UTC\" date --date=\""a"\" +\"%F %Z\"" | getline b;print b}')    # %F %T %Z
    # 6.9|2017-04-05 UTC|https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html
    [[ "${release_no}" == "7" ]] && release_no='7.1406'
    [[ "${release_no}" == "5" ]] && release_no='5.0'
    echo "${release_no}|${release_date}|${release_archive_page}"
}

export -f funcReleaseDate   # used for command parallel
cat "${centos_announce_archive}" | parallel -k -j 0 funcReleaseDate >> "${centos_release_date}"

# Step 3 對提取到的數據進行去重，並按Release版本逆序排序，將Step1中獲取的release note地址合併到輸入結果中
declare -A rhel7Arr=( ["7.1406"]='7' ["7.1503"]='7.1' ["7.1511"]='7.2' ["7.1611"]='7.3' )   # CentOS與RHEL的版本對應關係
# Step 3.1 release版本去重、排序
# printf "Release Version|Release Date|EUS Date|Release Note\n---|---|---|---\n" # Markdown Output
awk -F\| '{!a[$1]++;arr[$1]=$0}END{PROCINFO["sorted_in"]="@ind_str_desc";for (i in arr) print arr[i]}' "${centos_release_date}" | while IFS="|" read -r release_no release_date release_archive_page; do
    # Step 3.2 提取對應release版本的release note
    release_note_page=$(awk -F\| '$1=="'"${release_no}"'"{print $2}' "${centos_release_note}")
    if [[ "${release_no}" =~ ^7 ]]; then
        pattern_str=${rhel7Arr[${release_no}]}
    else
        pattern_str="${release_no}"
    fi
    eus_date=$(awk -F\| '$1=="'"${pattern_str}"'"{print $2}' "${rhel_eus_date}")

    printf "%s|%s|%s\n" "${release_no}" "${release_date}" "${eus_date}"

    # if [[ -n "${release_note_page}" ]]; then
    #     printf "[%s](%s)|%s|%s|[%s](%s)\n" "${release_no}" "${release_archive_page}" "${release_date}" "${eus_date}" "${release_note_page##*/}" "${release_note_page}"
    # else
    #     printf "[%s](%s)|%s|%s|\n" "${release_no}" "${release_archive_page}" "${release_date}" "${eus_date}"
    # fi
done

[[ -f "${centos_release_note}" ]] && rm -f "${centos_release_note}"
unset centos_release_note
[[ -f "${centos_announce_archive}" ]] && rm -f "${centos_announce_archive}"
unset centos_announce_archive
[[ -f "${centos_release_date}" ]] && rm -f "${centos_release_date}"
unset centos_release_date
[[ -f "${rhel_eus_date}" ]] && rm -f "${rhel_eus_date}"
unset rhel_eus_date

# Script End
```

## Time Cost Comparison
使用系統命令`time`對腳本耗時進行比較

### Lost Host
本地測試

原始版本耗時約`180s`，優化後耗時約`13s`

```bash
# while loop
real	2m59.756s
user	0m2.868s
sys	0m0.468s


# via parallel
real	0m12.839s
user	0m4.412s
sys	0m0.912s
```

### VPS
在AWS EC2中測試

原始版本耗時約`17s`，優化後耗時約`6s`

```bash
# while loop
real	0m17.105s
user	0m3.248s
sys	0m1.014s


# via parallel
real	0m6.339s
user	0m4.413s
sys	0m1.627s
```


## CentOS Release Info
[CentOS][centos]各release版本信息如下，通過以上腳本提取

```
Version|Release Date|EUS Date|Release Note
---|---|---|---
[7.1810](https://lists.centos.org/pipermail/centos-announce/2018-December/023082.html)|2018-12-03 UTC|2020-10-31|[CentOS7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7)
[7.1804](https://lists.centos.org/pipermail/centos-announce/2018-May/022829.html)|2018-05-10 UTC|2020-04-30|[CentOS7.1804](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1804)
[7.1708](https://lists.centos.org/pipermail/centos-announce/2017-September/022532.html)|2017-09-13 UTC|2019-08-31|[CentOS7.1708](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1708)
[7.1611](https://lists.centos.org/pipermail/centos-announce/2016-December/022172.html)|2016-12-12 UTC|2018-11-30|[CentOS7.1611](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1611)
[7.1511](https://lists.centos.org/pipermail/centos-announce/2015-December/021518.html)|2015-12-14 UTC|2017-11-30|[CentOS7.1511](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1511)
[7.1503](https://lists.centos.org/pipermail/centos-announce/2015-March/021006.html)|2015-03-31 UTC|2017-03-31|[CentOS7.1503](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1503)
[7.1406](https://lists.centos.org/pipermail/centos-announce/2014-July/020393.html)|2014-07-07 UTC||[CentOS7.1406](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1406)
[6.9](https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html)|2017-04-05 UTC||[CentOS6.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.9)
[6.8](https://lists.centos.org/pipermail/centos-announce/2016-May/021895.html)|2016-05-25 UTC||[CentOS6.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.8)
[6.7](https://lists.centos.org/pipermail/centos-announce/2015-August/021298.html)|2015-08-07 UTC|2018-12-31|[CentOS6.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.7)
[6.6](https://lists.centos.org/pipermail/centos-announce/2014-October/020709.html)|2014-10-28 UTC|2016-10-31|[CentOS6.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.6)
[6.5](https://lists.centos.org/pipermail/centos-announce/2013-December/020032.html)|2013-12-01 UTC|2015-11-30|[CentOS6.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.5)
[6.4](https://lists.centos.org/pipermail/centos-announce/2013-March/019276.html)|2013-03-09 UTC|2015-03-03|[CentOS6.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.4)
[6.3](https://lists.centos.org/pipermail/centos-announce/2012-July/018706.html)|2012-07-09 UTC|2014-06-30|[CentOS6.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.3)
[6.2](https://lists.centos.org/pipermail/centos-announce/2011-December/018335.html)|2011-12-20 UTC|2014-01-07|[CentOS6.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.2)
[6.10](https://lists.centos.org/pipermail/centos-announce/2018-July/022925.html)|2018-07-03 UTC||[CentOS6.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.10)
[6.1](https://lists.centos.org/pipermail/centos-announce/2011-December/018312.html)|2011-12-10 UTC|2013-05-31|[CentOS6.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.1)
[6.0](https://lists.centos.org/pipermail/centos-announce/2011-July/017645.html)|2011-07-10 UTC|2012-11-30|[CentOS6.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.0)
[5.9](https://lists.centos.org/pipermail/centos-announce/2013-January/019205.html)|2013-01-17 UTC|2015-03-31|[CentOS5.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.9)
[5.8](https://lists.centos.org/pipermail/centos-announce/2012-March/018478.html)|2012-03-08 UTC||[CentOS5.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.8)
[5.7](https://lists.centos.org/pipermail/centos-announce/2011-September/017727.html)|2011-09-13 UTC||[CentOS5.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.7)
[5.6](https://lists.centos.org/pipermail/centos-announce/2011-April/017282.html)|2011-04-08 UTC|2013-07-31|[CentOS5.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.6)
[5.5](https://lists.centos.org/pipermail/centos-announce/2010-May/016638.html)|2010-05-14 UTC||[CentOS5.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.5)
[5.4](https://lists.centos.org/pipermail/centos-announce/2009-October/016195.html)|2009-10-21 UTC|2011-07-31|[CentOS5.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.4)
[5.3](https://lists.centos.org/pipermail/centos-announce/2009-April/015711.html)|2009-04-01 UTC|2010-11-30|[CentOS5.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.3)
[5.2](https://lists.centos.org/pipermail/centos-announce/2008-June/014999.html)|2008-06-24 UTC|2010-03-31|[CentOS5.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.2)
[5.11](https://lists.centos.org/pipermail/centos-announce/2014-September/020601.html)|2014-09-30 UTC||[CentOS5.11](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.11)
[5.10](https://lists.centos.org/pipermail/centos-announce/2013-October/019978.html)|2013-10-19 UTC||[CentOS5.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.10)
[5.1](https://lists.centos.org/pipermail/centos-announce/2007-December/014476.html)|2007-12-02 UTC||[CentOS5.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.1)
[5.0](https://lists.centos.org/pipermail/centos-announce/2007-April/013660.html)|2007-04-12 UTC||[CentOS5.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.0)
```

渲染效果如下

Version|Release Date|EUS Date|Release Note
---|---|---|---
[7.1810](https://lists.centos.org/pipermail/centos-announce/2018-December/023082.html)|2018-12-03 UTC|2020-10-31|[CentOS7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7)
[7.1804](https://lists.centos.org/pipermail/centos-announce/2018-May/022829.html)|2018-05-10 UTC|2020-04-30|[CentOS7.1804](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1804)
[7.1708](https://lists.centos.org/pipermail/centos-announce/2017-September/022532.html)|2017-09-13 UTC|2019-08-31|[CentOS7.1708](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1708)
[7.1611](https://lists.centos.org/pipermail/centos-announce/2016-December/022172.html)|2016-12-12 UTC|2018-11-30|[CentOS7.1611](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1611)
[7.1511](https://lists.centos.org/pipermail/centos-announce/2015-December/021518.html)|2015-12-14 UTC|2017-11-30|[CentOS7.1511](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1511)
[7.1503](https://lists.centos.org/pipermail/centos-announce/2015-March/021006.html)|2015-03-31 UTC|2017-03-31|[CentOS7.1503](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1503)
[7.1406](https://lists.centos.org/pipermail/centos-announce/2014-July/020393.html)|2014-07-07 UTC||[CentOS7.1406](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1406)
[6.9](https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html)|2017-04-05 UTC||[CentOS6.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.9)
[6.8](https://lists.centos.org/pipermail/centos-announce/2016-May/021895.html)|2016-05-25 UTC||[CentOS6.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.8)
[6.7](https://lists.centos.org/pipermail/centos-announce/2015-August/021298.html)|2015-08-07 UTC|2018-12-31|[CentOS6.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.7)
[6.6](https://lists.centos.org/pipermail/centos-announce/2014-October/020709.html)|2014-10-28 UTC|2016-10-31|[CentOS6.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.6)
[6.5](https://lists.centos.org/pipermail/centos-announce/2013-December/020032.html)|2013-12-01 UTC|2015-11-30|[CentOS6.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.5)
[6.4](https://lists.centos.org/pipermail/centos-announce/2013-March/019276.html)|2013-03-09 UTC|2015-03-03|[CentOS6.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.4)
[6.3](https://lists.centos.org/pipermail/centos-announce/2012-July/018706.html)|2012-07-09 UTC|2014-06-30|[CentOS6.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.3)
[6.2](https://lists.centos.org/pipermail/centos-announce/2011-December/018335.html)|2011-12-20 UTC|2014-01-07|[CentOS6.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.2)
[6.10](https://lists.centos.org/pipermail/centos-announce/2018-July/022925.html)|2018-07-03 UTC||[CentOS6.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.10)
[6.1](https://lists.centos.org/pipermail/centos-announce/2011-December/018312.html)|2011-12-10 UTC|2013-05-31|[CentOS6.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.1)
[6.0](https://lists.centos.org/pipermail/centos-announce/2011-July/017645.html)|2011-07-10 UTC|2012-11-30|[CentOS6.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.0)
[5.9](https://lists.centos.org/pipermail/centos-announce/2013-January/019205.html)|2013-01-17 UTC|2015-03-31|[CentOS5.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.9)
[5.8](https://lists.centos.org/pipermail/centos-announce/2012-March/018478.html)|2012-03-08 UTC||[CentOS5.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.8)
[5.7](https://lists.centos.org/pipermail/centos-announce/2011-September/017727.html)|2011-09-13 UTC||[CentOS5.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.7)
[5.6](https://lists.centos.org/pipermail/centos-announce/2011-April/017282.html)|2011-04-08 UTC|2013-07-31|[CentOS5.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.6)
[5.5](https://lists.centos.org/pipermail/centos-announce/2010-May/016638.html)|2010-05-14 UTC||[CentOS5.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.5)
[5.4](https://lists.centos.org/pipermail/centos-announce/2009-October/016195.html)|2009-10-21 UTC|2011-07-31|[CentOS5.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.4)
[5.3](https://lists.centos.org/pipermail/centos-announce/2009-April/015711.html)|2009-04-01 UTC|2010-11-30|[CentOS5.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.3)
[5.2](https://lists.centos.org/pipermail/centos-announce/2008-June/014999.html)|2008-06-24 UTC|2010-03-31|[CentOS5.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.2)
[5.11](https://lists.centos.org/pipermail/centos-announce/2014-September/020601.html)|2014-09-30 UTC||[CentOS5.11](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.11)
[5.10](https://lists.centos.org/pipermail/centos-announce/2013-October/019978.html)|2013-10-19 UTC||[CentOS5.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.10)
[5.1](https://lists.centos.org/pipermail/centos-announce/2007-December/014476.html)|2007-12-02 UTC||[CentOS5.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.1)
[5.0](https://lists.centos.org/pipermail/centos-announce/2007-April/013660.html)|2007-04-12 UTC||[CentOS5.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.0)


## Change Logs
* 2017.06.22 15:57 Thu Asia/Shanghai
    * 初稿完成
* 2019.04.26 16:12 Fri America/Boston
    * 勘誤、更新，添加腳本[gnuLinuxLifeCycleInfo.sh][lifecyclescript]


[announce_archive]:https://lists.centos.org/pipermail/centos-announce/ "The CentOS-announce Archives"
[parallel]:https://www.gnu.org/software/parallel/ "GNU Parallel"
[gnulinux]:https://www.gnu.org "GNU Operating System"
[rhel]:https://www.redhat.com/en "RedHat"
[centos]:https://www.centos.org "CentOS"
[amzn]:https://aws.amazon.com/amazon-linux-ami/ "Amazon Linux AMI"
[debian]:https://www.debian.org "Debian"
[ubuntu]:https://www.ubuntu.com "Ubuntu"
[suse]:https://www.suse.com "SUSE"
[opensuse]:https://www.opensuse.org "OpenSUSE"
[lifecyclescript]:https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxLifeCycleInfo.sh

<!-- End -->
