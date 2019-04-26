---
title: 在GNU/Linux中安裝配置Oracle JDK
slug: Setting Up Oracle JDK On GNU Linux
date: 2018-04-19T15:37:16-04:00
lastmod: 2019-04-26T15:30:16-04:00
draft: false
keywords: ["Java SE", "JDK", "Java", "Shell script"]
description: "如何在GNU/Linux中安裝Oracle JDK，並通過Shell腳本實現整個操作過程。"
categories:
- GNU/Linux
tags:
- Oracle
- JDK
- Shell Script

comment: true
toc: true

---

`Java SE`是`Java Platform, Standard Edition`的縮寫，而`JDK`是`Java SE Development Kit`的縮寫，JDK爲[Java][java]應用提供環境支持。本文記錄如何在GNU/Linux中安裝、配置 Oracle JDK，並通過Shell腳本實現整個過程。

Oracle JDK許可已於`April 16, 2019`更新，詳見 [Oracle Technology Network License Agreement for Oracle Java SE](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html)。

<!--more-->

## Oracle JDK License
~~從2019年1月起，[Oracle][oracle]將不再對普通用戶(非商用)提供 **Java SE 8** 的更新支持。~~

<!-- ~~Customers who need continued access to critical bug fixes and security fixes as well as general maintenance for Java SE 8 or previous versions can get long term support through Oracle Java SE Advanced, Oracle Java SE Advanced Desktop, or Oracle Java SE Suite. For more information, and details on how to receive longer term support for Oracle JDK 8, please see the Oracle Java SE Support Roadmap. -- http://www.oracle.com/technetwork/java/javase/overview/index.html ~~ -->


### Important Oracle JDK License Update

>The Oracle JDK License has changed for releases starting April 16, 2019.
>
>The new [Oracle Technology Network License Agreement for Oracle Java SE](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html) is substantially different from prior Oracle JDK licenses. The new license permits certain uses, such as personal use and development use, at no cost -- but other uses authorized under prior Oracle JDK licenses may no longer be available. Please review the terms carefully before downloading and using this product. An FAQ is available [here](https://www.oracle.com/technetwork/java/javase/overview/oracle-jdk-faqs.html).
>
>Commercial license and support is available with a low cost [Java SE Subscription](https://www.oracle.com/java/java-se-subscription.html).
>
>Oracle also provides the latest OpenJDK release under the open source [GPL License](https://openjdk.java.net/legal/gplv2+ce.html) at [jdk.java.net](https://jdk.java.net/).

Oracle JDK 12

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_Oracle_JDK/2019-04-26_15-10-26.png)

Oracle JDK 11

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_Oracle_JDK/2019-04-26_15-10-37.png)

Oracle JDK 8

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_Oracle_JDK/2019-04-26_15-10-49.png)


###  Oracle Java SE License
[Oracle Technology Network License Agreement for Oracle Java SE](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_Oracle_JDK/2019-04-26_15-24_Oracle_Java_SE_License.png)


## Shell Script
整個安裝、配置過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/OracleSEJDK.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/OracleSEJDK.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189191.js" id="asciicast-189191" async></script>


## 支持週期
[Oracle][oracle]對各JDK版本的支持週期，詳見官方文檔 [Oracle Java SE Support Roadmap](http://www.oracle.com/technetwork/java/javase/eol-135779.html)

### Java SE 公共更新
Release	| GA Date | End of Public Updates Notification | End of Public Updates
---|---|---|---
8 | Mar 2014 | Sep 2017 | January 2019 and December 2020
9 | Sep 2017 | Sep 2017 | Mar 2018
10 (18.3) | Mar 2018 | Mar 2018 | Sep 2018
11(18.9 LTS) | Sep 2018 | |

### Java SE 支持路線圖
Release | GA Date | Premier Support Until | Extended Support Until | Sustaining Support
---|---|---|---|---
6 | Dec 2006 | Dec 2015 | Dec 2018 | Indefinite
7 | Jul 2011 | Jul 2019	| Jul 2022 | Indefinite
8 | Mar 2014 | Mar 2022 | Mar 2025 | Indefinite
9 (non-LTS) | Sep 2017 | Mar 2018 | Not Available | Indefinite
10 (18.3^) (non-LTS) | Mar 2018 | Sep 2018 | Not Available | Indefinite
11 (18.9 LTS) | Sep 2018 | Sep 2023 | Sep 2026 | Indefinite
12 (19.3^ non‑LTS) | March 2019 | September 2019 | Not Available | Indefinite


## 下載
JDK官方下載頁 [Java SE Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

[Oracle][oracle]對下載做了設置，必須得同意其許可 [Oracle Binary Code License Agreement for Java SE](http://www.oracle.com/technetwork/java/javase/terms/license/index.html) 才能下載。勾選`Accept License Agreement`後，會出現如下信息

>Thank you for accepting the Oracle Binary Code License Agreement for Java SE; you may now download this software.

這爲通過命令行或Shell腳本下載設置了障礙。參考

* [How to Install Java 9 JDK on Linux Systems](https://www.tecmint.com/install-java-jdk-jre-in-linux/)
* [P7h/jdk_download.sh](https://gist.github.com/P7h/9741922)

[Oracle][oracle]在cookie中設置了指令`oraclelicense`，通過它判斷用戶是否已同意其許可協議，完整參數爲`oraclelicense=accept-securebackup-cookie`。

可通過如下命令實現非交互操作

```bash
# curl
curl -fsL -O -H "Cookie: oraclelicense=accept-securebackup-cookie" "${pack_download_link}"

# wget
wget -qO- -O "${pack_download_link##*/}" --header="Cookie: oraclelicense=accept-securebackup-cookie" "${pack_download_link}"
```

## 安裝
安裝包後綴`.tar.gz`，可通過如下命令解壓並安裝到目標目錄

```bash
pack_path='~/Downloads/jdk-12_linux-x64_bin.tar.gz'
installation_dir='/opt/OracleSEJDK'

tar xf "${source_pack_path}" -C "${installation_dir}" --strip-components=1 2> /dev/null
```

## 配置
解壓後，文件目錄信息如下

```bash
┌─[maxdsre@Stretch]─[/opt/OracleSEJDK]
└──╼ $tree -L 1

├── bin
├── conf
├── include
├── jmods
├── legal
├── lib
├── man
└── release

7 directories, 1 file
┌─[maxdsre@Stretch]─[/opt/OracleSEJDK]
└──╼ $
```

### $PATH
可執行文件存放在`bin/`目錄中，須將該路徑添加到環境變量`$PATH`中。

個人建議將配置信息存放在目錄`/etc/profile.d/`中，此處假設文件名爲`oracle_se_jdk.sh`

```bash
jdk_execute_path='/etc/profile.d/oracle_se_jdk.sh'

echo "export PATH=${installation_dir}/bin:\$PATH" >> "${jdk_execute_path}"
```

### Aalternatives
如果系統安裝有多個版本的JDK，可通過`alternatives`或`update-alternatives`爲`java`設置默認的可執行路徑。

代碼實現

```bash
funcAlternativeModeConfiguration(){
    local jdkcommand="${1:-}"
    local jdkaction="${2:-'install'}"
    local jdkdir="${3:-"${installation_dir}"}"

    if [[ -n "${jdkdir}" && -n "${alternative_command}" ]]; then
        local jdkcommand_path
        jdkcommand_path="${jdkdir}/bin/${jdkcommand}"

        case "${jdkaction}" in
            install )
                if [[ -s "${jdkcommand_path}" ]]; then
                    ${alternative_command} --install /usr/bin/"${jdkcommand}" "${jdkcommand}" "${jdkcommand_path}" 99 &> /dev/null
                    ${alternative_command} --set "${jdkcommand}" "${jdkcommand_path}" &> /dev/null
                fi
                ;;
            remove|* )
                if [[ -s "${jdkcommand_path}" ]]; then
                    ${alternative_command} --remove "${jdkcommand}" "${jdkcommand_path}" &> /dev/null
                fi
                ;;
        esac
    fi
}

alternative_command='alternatives'
# alternative_command='update-alternatives'

funcAlternativeModeConfiguration 'java' 'remove'
# funcAlternativeModeConfiguration 'javac' 'remove'
# funcAlternativeModeConfiguration 'jar' 'remove'
```

## 測試
演示示例

```bash
┌─[maxdsre@Xenial]─[~]
└──╼ $which java
/usr/bin/java
┌─[maxdsre@Xenial]─[~]
└──╼ $java -version
java version "12" 2019-03-19
Java(TM) SE Runtime Environment (build 12+33)
Java HotSpot(TM) 64-Bit Server VM (build 12+33, mixed mode, sharing)
┌─[maxdsre@Xenial]─[~]
└──╼ $update-alternatives --display java
java - manual mode
  link best version is /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
  link currently points to /opt/OracleSEJDK/bin/java
  link java is /usr/bin/java
  slave java.1.gz is /usr/share/man/man1/java.1.gz
/opt/OracleSEJDK/bin/java - priority 99
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java - priority 1081
  slave java.1.gz: /usr/lib/jvm/java-8-openjdk-amd64/jre/man/man1/java.1.gz
┌─[maxdsre@Xenial]─[~]
└──╼ $
```


## 引用
* [Install Latest JDK on Linux Server](https://maxrohde.com/2017/11/07/install-latest-jdk-on-linux-server/)


## 更新日誌
* 2018.04.19 16:48 Wed America/Boston
	* 初稿完成
* 2018.05.14 09:08 Mon America/Boston
    * 添加 reference
* 2018.11.29 10:28 Thu America/Boston
    * 更新釋出版本
* 2019.04.06 11:32 Sat America/Boston
    * 更新釋出版本至`v12`
* 2019.04.26 15:30 Fri America/Boston
    * Oracle 更新 JDK [許可](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html)


[oracle]:https://www.oracle.com
[java]:http://www.oracle.com/technetwork/java/


<!-- End -->
