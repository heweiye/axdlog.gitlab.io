---
title: Setting Up Oracle JDK On GNU/Linux
slug: Setting Up Oracle JDK On GNU Linux
date: 2018-04-19T15:37:16-04:00
lastmod: 2019-04-06T11:32:16-04:00
draft: false
keywords: ["Java SE", "JDK", "Java", "Shell script"]
description: "How to set up Oracle JDK on GNU/Linux via Shell script"
categories:
- GNU/Linux
tags:
- Oracle
- JDK
- Shell Script

comment: true
toc: true

---

`Java SE` is an abbreviation for `Java Platform, Standard Edition`, and `JDK` is an abbreviation for `Java SE Development Kit`. JDK provides running environment for [Java][java] applications. This article documents how to set up Oracle JDK on GNU/Linux, then implement the entire process through Shell script.

<!--more-->

**Attention**： Oracle will not post further updates of **`Java SE 8`** to its public download sites for commercial use after **`January 2019`**.

>Customers who need continued access to critical bug fixes and security fixes as well as general maintenance for Java SE 8 or previous versions can get long term support through Oracle Java SE Advanced, Oracle Java SE Advanced Desktop, or Oracle Java SE Suite. For more information, and details on how to receive longer term support for Oracle JDK 8, please see the Oracle Java SE Support Roadmap. -- http://www.oracle.com/technetwork/java/javase/overview/index.html


## Shell Script
The entire installation and configuration process has been implemented through a shell script, the code is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/OracleSEJDK.sh), usage info

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/OracleSEJDK.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189191.js" id="asciicast-189191" async></script>


## Support Life Cycle
The support life cycly of per JDK release version is list on official page [Oracle Java SE Support Roadmap](http://www.oracle.com/technetwork/java/javase/eol-135779.html)

### Java SE Public Updates
Release	| GA Date | End of Public Updates Notification | End of Public Updates
---|---|---|---
8 | Mar 2014 | Sep 2017 | January 2019 and December 2020
9 | Sep 2017 | Sep 2017 | Mar 2018
10 (18.3) | Mar 2018 | Mar 2018 | Sep 2018
11(18.9 LTS) | Sep 2018 | |

### Java SE Support Roadmap
Release | GA Date | Premier Support Until | Extended Support Until | Sustaining Support
---|---|---|---|---
6 | Dec 2006 | Dec 2015 | Dec 2018 | Indefinite
7 | Jul 2011 | Jul 2019	| Jul 2022 | Indefinite
8 | Mar 2014 | Mar 2022 | Mar 2025 | Indefinite
9 (non-LTS) | Sep 2017 | Mar 2018 | Not Available | Indefinite
10 (18.3^) (non-LTS) | Mar 2018 | Sep 2018 | Not Available | Indefinite
11 (18.9 LTS) | Sep 2018 | Sep 2023 | Sep 2026 | Indefinite
12 (19.3^ non‑LTS) | March 2019 | September 2019 | Not Available | Indefinite


## Downloading
Official download page [Java SE Downloads](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

If you wanna download JDK from [Oracle][oracle] official site, you must accept is license [Oracle Binary Code License Agreement for Java SE](http://www.oracle.com/technetwork/java/javase/terms/license/index.html) firstly. After checking `Accept License Agreement`, the page will show the following info

>Thank you for accepting the Oracle Binary Code License Agreement for Java SE; you may now download this software.

How do I download jdk package directly from official site in command line or shell script?

Reference

* [How to Install Java 9 JDK on Linux Systems](https://www.tecmint.com/install-java-jdk-jre-in-linux/)
* [P7h/jdk_download.sh](https://gist.github.com/P7h/9741922)

[Oracle][oracle] makes sure user has agree the license via directive `oraclelicense` which is setted in cookie. The complete parameter is `oraclelicense=accept-securebackup-cookie`.

Downloading in silent mode

```bash
# curl
curl -fsL -O -H "Cookie: oraclelicense=accept-securebackup-cookie" "${pack_download_link}"

# wget
wget -qO- -O "${pack_download_link##*/}" --header="Cookie: oraclelicense=accept-securebackup-cookie" "${pack_download_link}"
```

## Installation
Suffix of package is `.tar.gz`, use the following command to extract its contents to target directory

```bash
pack_path='~/Downloads/jdk-12_linux-x64_bin.tar.gz'
installation_dir='/opt/OracleSEJDK'

tar xf "${source_pack_path}" -C "${installation_dir}" --strip-components=1 2> /dev/null
```

## Configuration
File tree

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
Execution files are listed in directory `bin/`, you need to add this path into environment parameter `$PATH`.

But personal advice is put this directive under directory `/etc/profile.d/`, here assume the file is called `oracle_se_jdk.sh`.

```bash
jdk_execute_path='/etc/profile.d/oracle_se_jdk.sh'

echo "export PATH=${installation_dir}/bin:\$PATH" >> "${jdk_execute_path}"
```

### Aalternatives
If your system has more than one JDK version, you can use `alternatives` or `update-alternatives` to set default execution path for `java`.

shell code

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

## Testing
Demostration example

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


## Reference
* [Install Latest JDK on Linux Server](https://maxrohde.com/2017/11/07/install-latest-jdk-on-linux-server/)


## Change Logs
* 2018.04.19 16:48 Wed America/Boston
	* first draft
* 2018.05.14 09:08 Mon America/Boston
    * add reference
* 2018.11.29 10:28 Thu America/Boston
    * update release version
* 2019.04.06 11:32 Sat America/Boston
    * update release version to `v12`


[oracle]:https://www.oracle.com
[java]:http://www.oracle.com/technetwork/java/


<!-- End -->
