---
title: Life Cycle Info For Mainstream GNU Linux Distributions
slug: Product Support Life Cycle Info For Mainstream GNU Linux Distributions
date: 2017-02-13T13:40:36+08:00
lastmod: 2018-07-11T11:12:06-04:00
draft: false
keywords: ["AxdLog", "Life Cycle", "Shell script"]
description: "Life Cycle Info For GNU Linux Distributions"
categories:
- GNU/Linux
tags:
- Shell Script
- Life Cycle

comment: true
toc: true

---

[GNU/Linux][gnulinux] is a open source operating system, it consists of [Linux kernel](linuxkernel) and a piece of programs. It has many distributions, [RHEL][rhel]/[CentOS][centos]、 [Debian][debian]/[Ubuntu][ubuntu]、[SUSE][suse]/[OpenSUSE][opensuse] are mainstream distributions. In order to extract the product support life cycle info, I wrote a [Shell script][lifecyclescript] to extract relevant info from their official site.

<!--more-->

[GNU/Linux][gnulinux] has 3 major mainstream distributions: [RHEL][rhel]、[Debian][debian]、[SUSE][suse]。[RHEL][rhel]、[SUSE][suse] are maintenanceed by commercial corporation, [Debian][debian] is maintenanceed by community, but [Ubuntu][ubuntu] (the derivative of [Debian][debian]) is maintenanceed by commercial corporation.


## Official Documents
RedHat

* [Red Hat Enterprise Linux Life Cycle](https://access.redhat.com/support/policy/updates/errata/)
* [Red Hat Enterprise Linux Release Dates](https://access.redhat.com/articles/3078)

CentOS

* [Release Notes for supported CentOS distributions](https://wiki.centos.org/Manuals/ReleaseNotes)

Debian

* [Debian Long Term Support](https://wiki.debian.org/LTS)
* [DebianReleases](https://wiki.debian.org/DebianReleases)

Ubuntu

* [Ubuntu release end of life][ubuntureleaseeol]
* [Releases](https://wiki.ubuntu.com/Releases)
* [The ubuntu-announce Archives](https://lists.ubuntu.com/archives/ubuntu-announce/)

SUSE

* [Product Support Lifecycle](https://www.suse.com/lifecycle/)

OpenSUSE

* [Lifetime](https://en.opensuse.org/Lifetime)
* [openSUSE Release Notes](https://doc.opensuse.org/release-notes/)
* [Mailinglist Archive](https://lists.opensuse.org/opensuse/)


## Shell Script
Shell script is hosted on [GitLab][lifecyclescript], but it just supports to extract info of [RHEL][rhel]/[CentOS][centos]、 [Debian][debian]/[Ubuntu][ubuntu].

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxLifeCycleInfo.sh | bash -s --
```

<script src="https://asciinema.org/a/189194.js" id="asciicast-189194" async></script>


**Note**：[CentOS][centos] doesn't provide release version page, I looped the page [The CentOS-announce Archives](https://lists.centos.org/pipermail/centos-announce/) to extract needed info. In order to increase the operation speed, I use command [parallel](https://www.gnu.org/software/parallel/ "GNU Parallel") to operate parallelly.


## Terminology
Terminology & Acronym

acronym|terminology|explanation
---|---|---
`EOL`|End Of Life|生命結束
`ELS`|Extended Life-cycle Support|延長生命週期支持
`EUS`|Extended Update Support|延長更新支持
`LTS`|Long Term Support|長期支持
`LTSS`|Long Term Service Pack Support|長期服務支持
`FCS`|First Customer Shipment|原始釋出版本
`GAD`|General Availability Date|釋出時間


## Life Cycle
### RedHat

Version|Release Date|EUS Date|Kernel Version
---|---|---|---
7.5|2018-04-10|2020-04-30|3.10.0-862
7.4|2017-07-31|2019-08-31|3.10.0-693
7.3|2016-11-03|2018-11-30|3.10.0-514
7.2|2015-11-19|2017-11-30|3.10.0-327
7.1|2015-03-05|2017-03-31|3.10.0-229
7.0|2014-06-09||3.10.0-123
6.9|2017-03-21||2.6.32-696
6.8|2016-05-10||2.6.32-642
6.7|2015-07-22|2018-12-31|2.6.32-573
6.6|2014-10-14|2016-10-31|2.6.32-504
6.5|2013-11-21|2015-11-30|2.6.32-431
6.4|2013-02-21|2015-03-03|2.6.32-358
6.3|2012-06-20|2014-06-30|2.6.32-279
6.2|2011-12-06|2014-01-07|2.6.32-220
6.10|2018-06-19||2.6.32-754
6.1|2011-05-19|2013-05-31|2.6.32-131.0.15
6.0|2010-11-09|2012-11-30|2.6.32-71
5.9|2013-01-07|2015-03-31|2.6.18-348
5.8|2012-02-20||2.6.18-308
5.7|2011-07-21||2.6.18-274
5.6|2011-01-13|2013-07-31|2.6.18-238
5.5|2010-03-30||2.6.18-194
5.4|2009-09-02|2011-07-31|2.6.18-164
5.3|2009-01-20|2010-11-30|2.6.18-128
5.2|2008-05-21|2010-03-31|2.6.18-92
5.11|2014-09-16||2.6.18-398
5.10|2013-10-01||2.6.18-371
5.1|2007-11-07||2.6.18-53
5.0|2007-03-15||2.6.18-8
4.7||2011-08-31
4.5||2009-01-31
4|2011-02-16||2.6.9-100
3|2007-06-20||
2.1|2005-04-28||


### CentOS

Version|Release Date|EUS Date|Release Note
---|---|---|---
[7.1804](https://lists.centos.org/pipermail/centos-announce/2018-May/022829.html)|2018-05-10 UTC|2020-04-30|[CentOS7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7)
[7.1708](https://lists.centos.org/pipermail/centos-announce/2017-September/022532.html)|2017-09-13 UTC|2019-08-31|[CentOS7.1708](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1708)
[7.1611](https://lists.centos.org/pipermail/centos-announce/2016-December/022172.html)|2016-12-12 UTC|2018-11-30|[CentOS7.1611](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1611)
[7.1511](https://lists.centos.org/pipermail/centos-announce/2015-December/021518.html)|2015-12-14 UTC|2017-11-30|[CentOS7.1511](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1511)
[7.1503](https://lists.centos.org/pipermail/centos-announce/2015-March/021006.html)|2015-03-31 UTC|2017-03-31|[CentOS7.1503](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1503)
[7.1406](https://lists.centos.org/pipermail/centos-announce/2014-July/020393.html)|2014-07-07 UTC||[CentOS7.1406](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1406)
[6.10](https://lists.centos.org/pipermail/centos-announce/2018-July/022925.html)|2018-07-03 UTC||[CentOS6.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.10)
[6.9](https://lists.centos.org/pipermail/centos-announce/2017-April/022351.html)|2017-04-05 UTC||[CentOS6.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.9)
[6.8](https://lists.centos.org/pipermail/centos-announce/2016-May/021895.html)|2016-05-25 UTC||[CentOS6.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.8)
[6.7](https://lists.centos.org/pipermail/centos-announce/2015-August/021298.html)|2015-08-07 UTC|2018-12-31|[CentOS6.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.7)
[6.6](https://lists.centos.org/pipermail/centos-announce/2014-October/020709.html)|2014-10-28 UTC|2016-10-31|[CentOS6.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.6)
[6.5](https://lists.centos.org/pipermail/centos-announce/2013-December/020032.html)|2013-12-01 UTC|2015-11-30|[CentOS6.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.5)
[6.4](https://lists.centos.org/pipermail/centos-announce/2013-March/019276.html)|2013-03-09 UTC|2015-03-03|[CentOS6.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.4)
[6.3](https://lists.centos.org/pipermail/centos-announce/2012-July/018706.html)|2012-07-09 UTC|2014-06-30|[CentOS6.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.3)
[6.2](https://lists.centos.org/pipermail/centos-announce/2011-December/018335.html)|2011-12-20 UTC|2014-01-07|[CentOS6.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.2)
[6.1](https://lists.centos.org/pipermail/centos-announce/2011-December/018312.html)|2011-12-10 UTC|2013-05-31|[CentOS6.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.1)
[6.0](https://lists.centos.org/pipermail/centos-announce/2011-July/017645.html)|2011-07-10 UTC|2012-11-30|[CentOS6.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS6.0)
[5.11](https://lists.centos.org/pipermail/centos-announce/2014-September/020601.html)|2014-09-30 UTC||[CentOS5.11](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.11)
[5.10](https://lists.centos.org/pipermail/centos-announce/2013-October/019978.html)|2013-10-19 UTC||[CentOS5.10](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.10)
[5.9](https://lists.centos.org/pipermail/centos-announce/2013-January/019205.html)|2013-01-17 UTC|2015-03-31|[CentOS5.9](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.9)
[5.8](https://lists.centos.org/pipermail/centos-announce/2012-March/018478.html)|2012-03-08 UTC||[CentOS5.8](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.8)
[5.7](https://lists.centos.org/pipermail/centos-announce/2011-September/017727.html)|2011-09-13 UTC||[CentOS5.7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.7)
[5.6](https://lists.centos.org/pipermail/centos-announce/2011-April/017282.html)|2011-04-08 UTC|2013-07-31|[CentOS5.6](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.6)
[5.5](https://lists.centos.org/pipermail/centos-announce/2010-May/016638.html)|2010-05-14 UTC||[CentOS5.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.5)
[5.4](https://lists.centos.org/pipermail/centos-announce/2009-October/016195.html)|2009-10-21 UTC|2011-07-31|[CentOS5.4](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.4)
[5.3](https://lists.centos.org/pipermail/centos-announce/2009-April/015711.html)|2009-04-01 UTC|2010-11-30|[CentOS5.3](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.3)
[5.2](https://lists.centos.org/pipermail/centos-announce/2008-June/014999.html)|2008-06-24 UTC|2010-03-31|[CentOS5.2](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.2)
[5.1](https://lists.centos.org/pipermail/centos-announce/2007-December/014476.html)|2007-12-02 UTC||[CentOS5.1](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.1)
[5.0](https://lists.centos.org/pipermail/centos-announce/2007-April/013660.html)|2007-04-12 UTC||[CentOS5.0](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS5.0)


### Debian

Version|CodeName|Release Date|EOL Date|LTS Date
---|---|---|---|---
12|[?](https://wiki.debian.org/DebianBookwormBookworm)|||
11|[Bullseye](https://wiki.debian.org/DebianBullseye)|||
10|[Buster](https://wiki.debian.org/DebianBuster)|||
9.4|[Stretch](https://wiki.debian.org/DebianStretch)|[2018-03-10](https://www.debian.org/News/2017/20170617)|approx. 2020|approx. 2022
9.0|[Stretch](https://wiki.debian.org/DebianStretch)|[2017-06-17](https://www.debian.org/News/2017/20170617)|approx. 2020|approx. 2022
8.11|[Jessie](https://wiki.debian.org/DebianJessie)|[2018-06-23](https://www.debian.org/News/2015/20150426)|[2018-06-06](https://www.debian.org/security/faq#lifespan)|[2020-06-06](https://wiki.debian.org/LTS)
8.0|[Jessie](https://wiki.debian.org/DebianJessie)|[2015-04-25](https://www.debian.org/News/2015/20150426)|[2018-06-06](https://www.debian.org/security/faq#lifespan)|[2020-06-06](https://wiki.debian.org/LTS)
7.11|[Wheezy](https://wiki.debian.org/DebianWheezy)|[2016-06-04](https://www.debian.org/News/2013/20130504)|2016-04-26|[May 2018](https://www.debian.org/News/2016/20160212)
7.0|[Wheezy](https://wiki.debian.org/DebianWheezy)|[2013-05-04](https://www.debian.org/News/2013/20130504)|2016-04-26|[May 2018](https://www.debian.org/News/2016/20160212)
6.0|[Squeeze](https://wiki.debian.org/DebianSqueeze)|[2011-02-06](https://www.debian.org/News/2011/20110205a)|[2014-05-31](https://www.debian.org/security/2014/dsa-2907)|[2016-02-29](https://www.debian.org/News/2014/20140424.html)
5.0|[Lenny](https://wiki.debian.org/DebianLenny)|[2009-02-14](https://www.debian.org/News/2009/20090214)|[2012-02-06](https://lists.debian.org/debian-security-announce/2011/msg00238.html)|
4.0|[Etch](https://wiki.debian.org/DebianEtch)|[2007-04-08](https://www.debian.org/News/2007/20070408)|[2010-02-15](https://www.debian.org/News/2010/20100121)|
3.1|[Sarge](https://wiki.debian.org/DebianSarge)|[2005-06-06](https://www.debian.org/News/2005/20050606)|[2008-03-31](https://www.debian.org/News/2008/20080229)|
3.0|[Woody](https://wiki.debian.org/DebianWoody)|[2002-07-19](https://www.debian.org/News/2002/20020719)|[2006-06-30](https://www.debian.org/News/2006/20060601)|
2.2|[Potato](https://wiki.debian.org/DebianPotato)|[2000-08-15](https://www.debian.org/News/2000/20000815)|2003-06-30|
2.1|[Slink](https://wiki.debian.org/DebianSlink)|[1999-03-09](https://www.debian.org/News/1999/19990309)|2000-09-30|[2000-10-30](https://lists.debian.org/debian-security-announce/2000/msg00043.html)
2.0|[Hamm](https://wiki.debian.org/DebianHamm)|[1998-07-24](https://www.debian.org/News/1998/19980724)||
1.3|[Bo](https://wiki.debian.org/DebianBo)|[1997-07-02](https://www.debian.org/News/1997/19970602)||
1.2|[Rex](https://wiki.debian.org/DebianRex)|[1996-12-12](https://lists.debian.org/debian-announce/1996/msg00026.html)||
1.1|[Buzz](https://wiki.debian.org/DebianBuzz)|[1996-06-17](https://lists.debian.org/debian-announce/1996/msg00021.html)||
0.93R6||[1995-10-26](https://lists.debian.org/debian-announce/1995/msg00007.html)||
0.93R5||[March 1995](https://lists.debian.org/debian-announce/1995/msg00004.html)||
0.91||January 1994||


### Ubuntu

Version|CodeName|Release Date|EOL Date|Doc
---|---|---|---|---
18.04 LTS|[Bionic](https://wiki.ubuntu.com/BionicBeaver)|[2018-04-26](https://lists.ubuntu.com/archives/ubuntu-announce/2018-April/000231.html)|April 2023|[Release Notes](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes)
17.10|[Artful](https://wiki.ubuntu.com/ArtfulAardvark)|[2017-10-19](https://lists.ubuntu.com/archives/ubuntu-announce/2017-October/000226.html)|July 2018|[Release Notes](https://wiki.ubuntu.com/ArtfulAardvark/ReleaseNotes)
16.04.4 LTS|[Xenial](https://wiki.ubuntu.com/XenialXerus)|[2018-03-01](https://lists.ubuntu.com/archives/ubuntu-announce/2017-August/000224.html)|[April 2021](https://lists.ubuntu.com/archives/ubuntu-announce/2018-March/000229.html)|[Changes](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes/ChangeSummary/16.04.4)
16.04.3 LTS|[Xenial](https://wiki.ubuntu.com/XenialXerus)|[2017-08-03](https://lists.ubuntu.com/archives/ubuntu-announce/2017-August/000224.html)|[April 2021](https://lists.ubuntu.com/archives/ubuntu-announce/2016-April/000207.html)|[Changes](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes/ChangeSummary/16.04.3)
16.04.2 LTS|[Xenial](https://wiki.ubuntu.com/XenialXerus)|[2017-02-16](https://lists.ubuntu.com/archives/ubuntu-release/2017-February/004036.html)|[April 2021](https://lists.ubuntu.com/archives/ubuntu-announce/2016-April/000207.html)|[Changes](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes/ChangeSummary/16.04.2)
16.04.1 LTS|[Xenial](https://wiki.ubuntu.com/XenialXerus)|[2016-07-21](https://lists.ubuntu.com/archives/ubuntu-announce/2016-July/000209.html)|[April 2021](https://lists.ubuntu.com/archives/ubuntu-announce/2016-April/000207.html)|[Changes](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes/ChangeSummary/16.04.1)
16.04 LTS|[Xenial](https://wiki.ubuntu.com/XenialXerus)|[2016-04-21](https://lists.ubuntu.com/archives/ubuntu-announce/2016-April/000207.html)|[April 2021](https://lists.ubuntu.com/archives/ubuntu-announce/2016-April/000207.html)|[Release Notes](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes)
14.04.5 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2016-08-04](https://lists.ubuntu.com/archives/ubuntu-announce/2016-August/000211.html)|[April 2019](https://lists.ubuntu.com/archives/ubuntu-announce/2014-April/000182.html)|[Changes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes/ChangeSummary/14.04.5)
14.04.4 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2016-02-18](https://lists.ubuntu.com/archives/ubuntu-announce/2016-February/000205.html)|[HWE August 2016](https://wiki.ubuntu.com/Kernel/LTSEnablementStack#Kernel.2BAC8-Support.A14.04.x_Ubuntu_Kernel_Support)|[Changes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes/ChangeSummary/14.04.4)
14.04.3 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2015-08-06](https://lists.ubuntu.com/archives/ubuntu-announce/2015-August/000200.html)|[HWE August 2016](https://wiki.ubuntu.com/Kernel/LTSEnablementStack#Kernel.2BAC8-Support.A14.04.x_Ubuntu_Kernel_Support)|[Changes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes/ChangeSummary/14.04.3)
14.04.2 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2015-02-20](https://lists.ubuntu.com/archives/ubuntu-announce/2015-February/000192.html)|[HWE August 2016](https://wiki.ubuntu.com/Kernel/LTSEnablementStack#Kernel.2BAC8-Support.A14.04.x_Ubuntu_Kernel_Support)|[Changes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes/ChangeSummary/14.04.2)
14.04.1 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2014-07-24](https://lists.ubuntu.com/archives/ubuntu-announce/2014-July/000188.html)|[April 2019](https://lists.ubuntu.com/archives/ubuntu-announce/2014-April/000182.html)|[Changes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes/ChangeSummary/14.04.1)
14.04 LTS|[Trusty](https://wiki.ubuntu.com/TrustyTahr)|[2014-04-17](https://lists.ubuntu.com/archives/ubuntu-announce/2014-April/000182.html)|[April 2019](https://lists.ubuntu.com/archives/ubuntu-announce/2014-April/000182.html)|[Release Notes](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes)
17.04|[Zesty](https://wiki.ubuntu.com/ZestyZapus)|[2017-04-13](https://lists.ubuntu.com/archives/ubuntu-announce/2017-April/000220.html)|[2018-01-13](https://lists.ubuntu.com/archives/ubuntu-announce/2018-January/000227.html)<br>|[Rel](https://wiki.ubuntu.com/ZestyZapus/ReleaseNotes)
16.10|[Yakkety](https://wiki.ubuntu.com/YakketyYak)|[2016-10-13](https://lists.ubuntu.com/archives/ubuntu-announce/2016-October/000213.html)|[2017-07-20](https://lists.ubuntu.com/archives/ubuntu-announce/2017-July/000223.html)<br>|[Rel](https://wiki.ubuntu.com/YakketyYak/ReleaseNotes)
15.10|[Wily](https://wiki.ubuntu.com/WilyWerewolf)|[2015-10-22](https://lists.ubuntu.com/archives/ubuntu-announce/2015-October/000202.html)|[2016-07-28](https://lists.ubuntu.com/archives/ubuntu-announce/2016-July/000210.html)<br>|[Rel](https://wiki.ubuntu.com/WilyWerewolf/ReleaseNotes)
15.04|[Vivid](https://wiki.ubuntu.com/VividVervet)|[2015-04-23](https://lists.ubuntu.com/archives/ubuntu-announce/2015-April/000195.html)|[2016-02-04](https://lists.ubuntu.com/archives/ubuntu-announce/2016-January/000203.html)<br>|[Rel](https://wiki.ubuntu.com/VividVervet/ReleaseNotes)
14.10|[Utopic](https://wiki.ubuntu.com/UtopicUnicorn)|[2014-10-23](https://lists.ubuntu.com/archives/ubuntu-announce/2014-October/000191.html)|[2015-07-23](https://lists.ubuntu.com/archives/ubuntu-announce/2015-July/000197.html)<br>|[Rel](https://wiki.ubuntu.com/UtopicUnicorn/ReleaseNotes)
13.10|[Saucy](https://wiki.ubuntu.com/SaucySalamander)|[2013-10-17](https://lists.ubuntu.com/archives/ubuntu-announce/2013-October/000177.html)|[2014-07-17](https://lists.ubuntu.com/archives/ubuntu-announce/2014-June/000185.html)<br>|[Rel](https://wiki.ubuntu.com/SaucySalamander/ReleaseNotes)
13.04|[Raring](https://wiki.ubuntu.com/RaringRingtail)|[2013-04-25](https://lists.ubuntu.com/archives/ubuntu-announce/2013-April/000171.html)|[2014-01-27](https://lists.ubuntu.com/archives/ubuntu-announce/2014-January/000178.html)<br>|[Rel](https://wiki.ubuntu.com/RaringRingtail/ReleaseNotes)
12.10|[Quantal](https://wiki.ubuntu.com/QuantalQuetzal)|[2012-10-18](https://lists.ubuntu.com/archives/ubuntu-announce/2010-October/000164.html)|[2014-05-16](https://lists.ubuntu.com/archives/ubuntu-security-announce/2014-April/002488.html)<br>|[Tech](https://wiki.ubuntu.com/QuantalQuetzal/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/QuantalQuetzal/ReleaseNotes)
12.04.5 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2014-08-07](https://lists.ubuntu.com/archives/ubuntu-announce/2014-August/000189.html)|[2017-04-28](https://lists.ubuntu.com/archives/ubuntu-announce/2017-March/000218.html)<br>|[Rel](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/ChangeSummary/12.04.5)
12.04.4 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2014-02-06](https://lists.ubuntu.com/archives/ubuntu-announce/2014-February/000180.html)|[HWE 2014-08-08](https://wiki.ubuntu.com/1204_HWE_EOL)<br>|[Changes](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/ChangeSummary/12.04.4)
12.04.3 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2013-08-23](https://lists.ubuntu.com/archives/ubuntu-announce/2013-August/000175.html)|[HWE 2014-08-08](https://wiki.ubuntu.com/1204_HWE_EOL)<br>|[Changes](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/ChangeSummary/12.04.3)
12.04.2 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2013-02-14](https://lists.ubuntu.com/archives/ubuntu-announce/2013-February/000166.html)|[HWE 2014-08-08](https://wiki.ubuntu.com/1204_HWE_EOL)<br>|[Changes](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/ChangeSummary/12.04.2)
12.04.1 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2012-08-24](https://lists.ubuntu.com/archives/ubuntu-announce/2012-August/000160.html)|[2017-04-28](https://lists.ubuntu.com/archives/ubuntu-announce/2017-March/000218.html)<br>|[Changes](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/ChangeSummary/12.04.1)
12.04 LTS|[Precise](https://wiki.ubuntu.com/PrecisePangolin)|[2012-04-26](https://lists.ubuntu.com/archives/ubuntu-announce/2012-April/000159.html)|[2017-04-28](https://lists.ubuntu.com/archives/ubuntu-announce/2017-March/000218.html)<br>|[Tech](https://wiki.ubuntu.com/PrecisePangolin/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes)
11.10|[Oneiric](https://wiki.ubuntu.com/OneiricOcelot)|[2011-10-13](https://lists.ubuntu.com/archives/ubuntu-announce/2011-October/000153.html)|[2013-05-09](https://lists.ubuntu.com/archives/ubuntu-announce/2013-March/000167.html)<br>|[Tech](https://wiki.ubuntu.com/OneiricOcelot/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/OneiricOcelot/ReleaseNotes)
11.04|[Natty](https://wiki.ubuntu.com/NattyNarwhal)|[2011-04-28](https://lists.ubuntu.com/archives/ubuntu-announce/2011-April/000147.html)|[2012-10-28](https://lists.ubuntu.com/archives/ubuntu-announce/2012-October/000165.html)<br>|[Tech](https://wiki.ubuntu.com/NattyNarwhal/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/NattyNarwhal/ReleaseNotes)
10.10|[Maverick](https://wiki.ubuntu.com/MaverickMeerkat)|[2010-10-10](https://lists.ubuntu.com/archives/ubuntu-announce/2010-October/000139.html)|[2012-04-10](https://lists.ubuntu.com/archives/ubuntu-announce/2012-April/000158.html)<br>|[Tech](https://wiki.ubuntu.com/MaverickMeerkat/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/MaverickMeerkat/ReleaseNotes)
10.04.4 LTS|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2012-02-16](https://lists.ubuntu.com/archives/ubuntu-announce/2012-February/000155.html)|[2013-05-09](https://lists.ubuntu.com/archives/ubuntu-announce/2013-March/000169.html)<br>[2015-04-30](https://lists.ubuntu.com/archives/ubuntu-announce/2015-April/000196.html)|[Changes](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes/ChangeSummary/10.04.4)
10.04.3 LTS|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2011-07-21](https://lists.ubuntu.com/archives/ubuntu-announce/2011-July/000150.html)|<br>|[Changes](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes/ChangeSummary/10.04.3)
10.04.2 LTS|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2011-02-18](https://lists.ubuntu.com/archives/ubuntu-announce/2011-February/000141.html)|<br>|[Changes](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes/ChangeSummary/10.04.2)
10.04.1 LTS|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2010-08-17](https://lists.ubuntu.com/archives/ubuntu-announce/2010-August/000134.html)|<br>|[Changes](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes/ChangeSummary/10.04.1)
10.04 LTS|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2010-04-29](https://lists.ubuntu.com/archives/ubuntu-announce/2010-April/000133.html)|<br>|[Tech](https://wiki.ubuntu.com/LucidLynx/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes)
10.04|[Lucid](https://wiki.ubuntu.com/LucidLynx)|[2012-02-16](https://lists.ubuntu.com/archives/ubuntu-announce/2012-February/000155.html)|[2013-05-09](https://lists.ubuntu.com/archives/ubuntu-announce/2013-March/000169.html)<br>|[Changes](https://wiki.ubuntu.com/LucidLynx/ReleaseNotes/ChangeSummary/10.04.4)
9.10|[Karmic](https://wiki.ubuntu.com/KarmicKoala)|[2009-10-29](https://lists.ubuntu.com/archives/ubuntu-announce/2009-October/000127.html)|[2011-04-30](https://lists.ubuntu.com/archives/ubuntu-announce/2011-March/000142.html)<br>|[Tech](https://wiki.ubuntu.com/KarmicKoala/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/KarmicKoala/ReleaseNotes)
9.04|[Jaunty](https://wiki.ubuntu.com/JauntyJackalope)|[2009-04-23](https://lists.ubuntu.com/archives/ubuntu-announce/2009-April/000122.html)|[2010-10-23](https://lists.ubuntu.com/archives/ubuntu-announce/2010-September/000137.html)<br>|[Tech](https://wiki.ubuntu.com/JauntyJackalope/TechnicalOverview) / [Rel](https://wiki.ubuntu.com/JauntyJackalope/ReleaseNotes)
8.10|[Intrepid](https://wiki.ubuntu.com/IntrepidIbex)|[2008-10-30](https://lists.ubuntu.com/archives/ubuntu-announce/2008-October/000116.html)|[2010-04-30](https://lists.ubuntu.com/archives/ubuntu-announce/2010-March/000130.html)<br>|[Rel](https://wiki.ubuntu.com/IntrepidReleaseNotes)
8.04.4 LTS|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2010-01-28](https://lists.ubuntu.com/archives/ubuntu-announce/2010-January/000128.html)|[2013-05-09](https://lists.ubuntu.com/archives/ubuntu-announce/2013-March/000168.html)<br>|[Changes](https://wiki.ubuntu.com/HardyReleaseNotes/ChangeSummary/8.04.4)
8.04.3 LTS|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2009-07-16](https://lists.ubuntu.com/archives/ubuntu-announce/2009-July/000124.html)|<br>|[Changes](https://wiki.ubuntu.com/HardyReleaseNotes/ChangeSummary/8.04.3)
8.04.2 LTS|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2009-01-22](https://lists.ubuntu.com/archives/ubuntu-announce/2009-January/000117.html)|<br>|[Changes](https://wiki.ubuntu.com/HardyReleaseNotes/ChangeSummary/8.04.2)
8.04.1 LTS|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2008-07-03](https://lists.ubuntu.com/archives/ubuntu-announce/2008-July/000112.html)|<br>|[Hardy Heron](https://wiki.ubuntu.com/HardyHeron)
8.04 LTS|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2008-04-24](https://lists.ubuntu.com/archives/ubuntu-announce/2008-April/000111.html)|<br>|[Rel](https://wiki.ubuntu.com/HardyHeron)
8.04|[Hardy](https://wiki.ubuntu.com/HardyHeron)|[2008-04-24](https://lists.ubuntu.com/archives/ubuntu-announce/2008-April/000111.html)|[2011-05-12](https://lists.ubuntu.com/archives/ubuntu-announce/2011-April/000144.html)<br>|[Rel](https://wiki.ubuntu.com/HardyReleaseNotes)
7.10|[Gutsy](https://wiki.ubuntu.com/GutsyGibbon)|[2007-10-18](https://lists.ubuntu.com/archives/ubuntu-announce/2007-October/000105.html)|[2009-04-18](http://www.ubuntu.com/news/ubuntu-7.10-eol)<br>|[Rel](https://wiki.ubuntu.com/GutsyReleaseNotes)
7.04|[Feisty](https://wiki.ubuntu.com/FeistyFawn)|[2007-04-19](https://lists.ubuntu.com/archives/ubuntu-announce/2007-April/000102.html)|[2008-10-19](https://lists.ubuntu.com/archives/ubuntu-announce/2008-September/000113.html)<br>|[Rel](https://wiki.ubuntu.comhttp://www.ubuntu.com/getubuntu/releasenotes/704)
6.10|[Edgy](https://wiki.ubuntu.com/EdgyEft)|[2006-10-26](https://lists.ubuntu.com/archives/ubuntu-announce/2006-October/000093.html)|[2008-04-26](https://lists.ubuntu.com/archives/ubuntu-security-announce/2008-March/000680.html)<br>|
6.06.2 LTS|[Dapper](https://wiki.ubuntu.com/DapperDrake)|[2008-01-21](https://lists.ubuntu.com/archives/ubuntu-announce/2008-January/000107.html)|[2011-06-01](https://lists.ubuntu.com/archives/ubuntu-announce/2011-June/000149.html)<br>|
6.06.1 LTS|[Dapper](https://wiki.ubuntu.com/DapperDrake)|[2006-08-10](https://lists.ubuntu.com/archives/ubuntu-announce/2006-August/000088.html)|<br>|
6.06 LTS|[Dapper](https://wiki.ubuntu.com/DapperDrake)|[2006-06-01](https://lists.ubuntu.com/archives/ubuntu-announce/2006-June/000083.html)|<br>|[Rel](https://wiki.ubuntu.com/DapperReleaseNotes)
6.06|[Dapper](https://wiki.ubuntu.com/DapperDrake)|[2006-06-01](https://lists.ubuntu.com/archives/ubuntu-announce/2006-June/000083.html)|[2009-07-14](https://lists.ubuntu.com/archives/ubuntu-announce/2009-July/000123.html)<br>|[Rel](https://wiki.ubuntu.com/DapperReleaseNotes)
5.10|[Breezy](https://wiki.ubuntu.com/BreezyBadger)|[2005-10-12](https://lists.ubuntu.com/archives/ubuntu-announce/2005-October/000038.html)|[2007-04-13](https://lists.ubuntu.com/archives/ubuntu-security-announce/2007-March/000504.html)<br>|[Rel](https://wiki.ubuntu.comhttp://www.ubuntu.com/getubuntu/releasenotes/510)
5.04|[Hoary](https://wiki.ubuntu.com/HoaryHedgehog)|[2005-04-08](https://lists.ubuntu.com/archives/ubuntu-announce/2005-April/000023.html)|[2006-10-31](https://lists.ubuntu.com/archives/ubuntu-security-announce/2006-October/000418.html)<br>|
4.10|[Warty](https://wiki.ubuntu.com/WartyWarthog)|[2004-10-26](https://lists.ubuntu.com/archives/ubuntu-announce/2004-October/000003.html)|[2006-04-30](https://lists.ubuntu.com/archives/ubuntu-announce/2006-March/000061.html)<br>|


### SUSE
The release info about SUSE is dynamicly generated by javascript. I have not ability to extract them. Official release page is [Product Support Lifecycle](https://www.suse.com/lifecycle/), you may also reference [Wikipedia](https://en.wikipedia.org/wiki/SUSE_Linux#Versions).


### OpenSUSE
Shell script doesn't add this distribution as its chaotic naming rule.  Official release page is [Lifetime
](https://en.opensuse.org/Lifetime), you may also reference [Wikipedia](https://en.wikipedia.org/wiki/OpenSUSE#Releases).


## EOL Status
If you wanna know if a specific release version is end of life (EOL), just specifing flag `-a` in the script. `IsEOL`, the last column of table stands for eol status. (`0` means in maintenance, `1` means out of maintenance).

My system info check script [gnuLinuxMachineInfoDetection.sh](https://github.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxMachineInfoDetection.sh) uses these data to determine if the distro release version is out of maintenance.

The following is the data about EOL until **Jul 11, 2018**.

Distro|Release|CodeName|Release Date|EOL Date|EOL Timestamp|IsEOL
---|---|---|---|---|---
rhel|7.5||2018-04-10|2020-04-30|1588204800|0
rhel|7.4||2017-07-31|2019-08-31|1567209600|0
rhel|7.3||2016-11-03|2018-11-30|1543536000|0
rhel|7.2||2015-11-19|2017-11-30|1512000000|1
rhel|7.1||2015-03-05|2017-03-31|1490918400|1
rhel|7.0||2014-06-09|||1
rhel|6.9||2017-03-21|||0
rhel|6.8||2016-05-10|||0
rhel|6.7||2015-07-22|2018-12-31|1546214400|0
rhel|6.6||2014-10-14|2016-10-31|1477872000|1
rhel|6.5||2013-11-21|2015-11-30|1448841600|1
rhel|6.4||2013-02-21|2015-03-03|1425340800|1
rhel|6.3||2012-06-20|2014-06-30|1404086400|1
rhel|6.2||2011-12-06|2014-01-07|1389052800|1
rhel|6.10||2018-06-19|||0
rhel|6.1||2011-05-19|2013-05-31|1369958400|1
rhel|6.0||2010-11-09|2012-11-30|1354233600|1
rhel|5.9||2013-01-07|2015-03-31|1427760000|1
rhel|5.8||2012-02-20|||1
rhel|5.7||2011-07-21|||1
rhel|5.6||2011-01-13|2013-07-31|1375228800|1
rhel|5.5||2010-03-30|||1
rhel|5.4||2009-09-02|2011-07-31|1312070400|1
rhel|5.3||2009-01-20|2010-11-30|1291075200|1
rhel|5.2||2008-05-21|2010-03-31|1269993600|1
rhel|5.11||2014-09-16|||1
rhel|5.10||2013-10-01|||1
rhel|5.1||2007-11-07|||1
rhel|5.0||2007-03-15|||1
rhel|4.7|||2011-08-31|1
rhel|4.5|||2009-01-31|1
rhel|4||2011-02-16|||1
rhel|3||2007-06-20|||1
rhel|2.1||2005-04-28|||1
centos|7.1804||2018-05-10|2020-04-30|1588204800|0
centos|7.1708||2017-09-13|2019-08-31|1567209600|0
centos|7.1611||2016-12-12|2018-11-30|1543536000|0
centos|7.1511||2015-12-14|2017-11-30|1512000000|1
centos|7.1503||2015-03-31|2017-03-31|1490918400|1
centos|7.1406||2014-07-07|||1
centos|6.9||2017-04-05|||0
centos|6.8||2016-05-25|||0
centos|6.7||2015-08-07|2018-12-31|1546214400|0
centos|6.6||2014-10-28|2016-10-31|1477872000|1
centos|6.5||2013-12-01|2015-11-30|1448841600|1
centos|6.4||2013-03-09|2015-03-03|1425340800|1
centos|6.3||2012-07-09|2014-06-30|1404086400|1
centos|6.2||2011-12-20|2014-01-07|1389052800|1
centos|6.10||2018-07-03|||0
centos|6.1||2011-12-10|2013-05-31|1369958400|1
centos|6.0||2011-07-10|2012-11-30|1354233600|1
centos|5.9||2013-01-17|2015-03-31|1427760000|1
centos|5.8||2012-03-08|||1
centos|5.7||2011-09-13|||1
centos|5.6||2011-04-08|2013-07-31|1375228800|1
centos|5.5||2010-05-14|||1
centos|5.4||2009-10-21|2011-07-31|1312070400|1
centos|5.3||2009-04-01|2010-11-30|1291075200|1
centos|5.2||2008-06-24|2010-03-31|1269993600|1
centos|5.11||2014-09-30|||1
centos|5.10||2013-10-19|||1
centos|5.1||2007-12-02|||1
centos|5.0||2007-04-12|||1
debian|9.4|stretch|2018-03-10|approx. 2022|1640995200|0
debian|9.0|stretch|2017-06-17|approx. 2022|1640995200|0
debian|8.11|jessie|2018-06-23|2020-06-06|1591401600|0
debian|8.0|jessie|2015-04-25|2020-06-06|1591401600|0
debian|7.11|wheezy|2016-06-04|May 2018|1527724800|1
debian|7.0|wheezy|2013-05-04|May 2018|1527724800|1
debian|6.0|squeeze|2011-02-06|2016-02-29|1456704000|1
debian|5.0|lenny|2009-02-14|2012-02-06|1328486400|1
debian|4.0|etch|2007-04-08|2010-02-15|1266192000|1
debian|3.1|sarge|2005-06-06|2008-03-31|1206921600|1
debian|3.0|woody|2002-07-19|2006-06-30|1151625600|1
debian|2.2|potato|2000-08-15|2003-06-30|1056931200|1
debian|2.1|slink|1999-03-09|2000-10-30|972864000|1
debian|2.0|hamm|1998-07-24|||1
debian|1.3|bo|1997-07-02|||1
debian|1.2|rex|1996-12-12|||1
debian|1.1|buzz|1996-06-17|||1
debian|0.93R6||1995-10-26|||1
debian|0.93R5||March 1995|||1
debian|0.91||January 1994|||1
ubuntu|18.04|bionic|2018-04-26|April 2023|1682812800|0
ubuntu|17.10|artful|2017-10-19|July 2018|1532995200|0
ubuntu|16.04.4|xenial|2018-03-01|April 2021|1619740800|0
ubuntu|16.04.3|xenial|2017-08-03|April 2021|1619740800|0
ubuntu|16.04.2|xenial|2017-02-16|April 2021|1619740800|0
ubuntu|16.04.1|xenial|2016-07-21|April 2021|1619740800|0
ubuntu|16.04|xenial|2016-04-21|April 2021|1619740800|0
ubuntu|14.04.5|trusty|2016-08-04|April 2019|1556582400|0
ubuntu|14.04.4|trusty|2016-02-18|August 2016|1472601600|1
ubuntu|14.04.3|trusty|2015-08-06|August 2016|1472601600|1
ubuntu|14.04.2|trusty|2015-02-20|August 2016|1472601600|1
ubuntu|14.04.1|trusty|2014-07-24|April 2019|1556582400|0
ubuntu|14.04|trusty|2014-04-17|April 2019|1556582400|0
ubuntu|17.04|zesty|2017-04-13|2018-01-13|1515801600|1
ubuntu|16.10|yakkety|2016-10-13|2017-07-20|1500508800|1
ubuntu|15.10|wily|2015-10-22|2016-07-28|1469664000|1
ubuntu|15.04|vivid|2015-04-23|2016-02-04|1454544000|1
ubuntu|14.10|utopic|2014-10-23|2015-07-23|1437609600|1
ubuntu|13.10|saucy|2013-10-17|2014-07-17|1405555200|1
ubuntu|13.04|raring|2013-04-25|2014-01-27|1390780800|1
ubuntu|12.10|quantal|2012-10-18|2014-05-16|1400198400|1
ubuntu|12.04.5|precise|2014-08-07|2017-04-28|1493337600|1
ubuntu|12.04.4|precise|2014-02-06|HWE 2014-08-08|1407456000|1
ubuntu|12.04.3|precise|2013-08-23|HWE 2014-08-08|1407456000|1
ubuntu|12.04.2|precise|2013-02-14|HWE 2014-08-08|1407456000|1
ubuntu|12.04.1|precise|2012-08-24|2017-04-28|1493337600|1
ubuntu|12.04|precise|2012-04-26|2017-04-28|1493337600|1
ubuntu|11.10|oneiric|2011-10-13|2013-05-09|1368057600|1
ubuntu|11.04|natty|2011-04-28|2012-10-28|1351382400|1
ubuntu|10.10|maverick|2010-10-10|2012-04-10|1334016000|1
ubuntu|10.04.4|lucid|2012-02-16|2013-05-09|1368057600|1
ubuntu|10.04.3|lucid|2011-07-21|||1
ubuntu|10.04.2|lucid|2011-02-18|||1
ubuntu|10.04.1|lucid|2010-08-17|||1
ubuntu|10.04|lucid|2010-04-29|||1
ubuntu|10.04|lucid|2012-02-16|2013-05-09|1368057600|1
ubuntu|9.10|karmic|2009-10-29|2011-04-30|1304121600|1
ubuntu|9.04|jaunty|2009-04-23|2010-10-23|1287792000|1
ubuntu|8.10|intrepid|2008-10-30|2010-04-30|1272585600|1
ubuntu|8.04.4|hardy|2010-01-28|2013-05-09|1368057600|1
ubuntu|8.04.3|hardy|2009-07-16|||1
ubuntu|8.04.2|hardy|2009-01-22|||1
ubuntu|8.04.1|hardy|2008-07-03|||1
ubuntu|8.04|hardy|2008-04-24|||1
ubuntu|8.04|hardy|2008-04-24|2011-05-12|1305158400|1
ubuntu|7.10|gutsy|2007-10-18|2009-04-18|1240012800|1
ubuntu|7.04|feisty|2007-04-19|2008-10-19|1224374400|1
ubuntu|6.10|edgy|2006-10-26|2008-04-26|1209168000|1
ubuntu|6.06.2|dapper|2008-01-21|2011-06-01|1306886400|1
ubuntu|6.06.1|dapper|2006-08-10|||1
ubuntu|6.06|dapper|2006-06-01|||1
ubuntu|6.06|dapper|2006-06-01|2009-07-14|1247529600|1
ubuntu|5.10|breezy|2005-10-12|2007-04-13|1176422400|1
ubuntu|5.04|hoary|2005-04-08|2006-10-31|1162252800|1
ubuntu|4.10|warty|2004-10-26|2006-04-30|1146355200|1


## Ubuntu Release EOL
[Ubuntu][ubuntu] has listed release date and EOL date of its release version (include unreleased) in official page [Ubuntu release end of life][ubuntureleaseeol]. But the data has been hidden using HTML class `class="u-hide--medium u-hide--large"`, it just be shown on mobile device.

### Ubuntu Server and desktop release end of life
Extracting command

```bash
curl -fsL https://www.ubuntu.com/info/release-end-of-life | sed -r -n '/Ubuntu Server and desktop release end of life/,/<\/tbody>/{/<tbody>/,${s@^[[:space:]]*@@g;s@<\/tr>@---@g;p}}' | sed ':a;N;$!ba;s@\n@@g;s@---@\n@g;' | sed -r -n 's@<\/td>@|@g;s@<[^>]*>@@g;s@\|$@@g;s@&nbsp;@@g;/^$/d;p' | awk -F\| 'BEGIN{OFS="|"; print "Release|Release Data|End of Life\n---|---|---"}{print}'
```

Results

Release|Release Data|End of Life
---|---|---
Ubuntu 22.04 LTS|April 2022|April 2027
Ubuntu 21.10|October 2021|July 2022
Ubuntu 21.04|April 2021|January 2022
Ubuntu 20.10|October 2020|July 2021
Ubuntu 20.04 LTS|April 2020|April 2025
Ubuntu 19.10|October 2019|July 2020
Ubuntu 19.04|April 2019|January 2020
Ubuntu 18.10|October 2018|July 2019
Ubuntu 18.04 LTS|April 2018|April 2023
Ubuntu 17.10|October 2017|July 2018
Ubuntu 17.04|April 2017|January 2018
Ubuntu 16.10|October 2016|June 2017
Ubuntu 16.04 LTS|April 2016|April 2021
Ubuntu 14.04 LTS|April 2014|April 2019
Ubuntu 12.04 LTS|April 2012|April 2017
Ubuntu 10.04 LTS|April 2010|April 2015


### Kernel release end of life
Extracting command

```bash
curl -fsL https://www.ubuntu.com/info/release-end-of-life | sed -r -n '/Kernel release end of life/,/<\/tbody>/{/<tbody>/,${s@^[[:space:]]*@@g;s@<\/tr>@---@g;p}}' | sed ':a;N;$!ba;s@\n@@g;s@---@\n@g;' | sed -r -n 's@<\/td>@|@g;s@<[^>]*>@@g;s@\|$@@g;/^$/d;s@&nbsp;@@g;p' | awk -F\| 'BEGIN{OFS="|"; print "Release|Release Data|End of Life|Extended customer support\n---|---|---|---"}{print}'
```

Results

Release|Release Data|End of Life|Extended customer support
---|---|---|---
Ubuntu 18.04.5 LTS|August 2020|April 2023|
Ubuntu 20.04 LTS|April 2020|April 2025|
Ubuntu 18.04.4 LTS|February 2020|August 2020|
Ubuntu 19.10|October 2019|July 2020|
Ubuntu 18.04.3 LTS|August 2019|February 2020|
Ubuntu 19.04|April 2019|January 2020|
Ubuntu 18.04.2 LTS|February 2019|August 2019|
Ubuntu 18.10|October 2018|July 2019|
Ubuntu 18.04.1 LTS (v.4.15)|July 2018|April 2023|
Ubuntu 16.04.5 LTS (v.4.15)|August 2018|April 2021|
Ubuntu 18.04.0 LTS (v.4.15)|April 2018|April 2023|
Ubuntu 16.04.4 LTS (v.4.13)|February 2018|August 2018|
Ubuntu 17.10 (v4.13)|October 2017|July 2018|
Ubuntu 16.04.1 LTS (v4.4)|August 2016|April 2021|
Ubuntu 14.04.5 LTS LTS (v3.13)|August 2016|April 2019|
Ubuntu 16.04.0 LTS (v4.4)|April 2016|April 2021|
Ubuntu 14.04.1 LTS (v3.13)|August 2014|April 2019|
Ubuntu 12.04.5 LTS (v3.13)|August 2014|April 2017|April 2019
Ubuntu 14.04.0 LTS (v3.13)|April 2014|April 2019|
Ubuntu 12.04.1 LTS (v3.2)|August 2012|April 2017|April 2019
Ubuntu 12.04.0 LTS (v3.2)|April 2012|April 2017|April 2019


### Ubuntu OpenStack release end of life
Extracting command

```bash
curl -fsL https://www.ubuntu.com/info/release-end-of-life | sed -r -n '/Ubuntu OpenStack release end of life/,/<\/tbody>/{/<tbody>/,${s@^[[:space:]]*@@g;s@<\/tr>@---@g;p}}' | sed ':a;N;$!ba;s@\n@@g;s@---@\n@g;' | sed -r -n 's@<\/td>@|@g;s@<[^>]*>@@g;s@\|$@@g;/^$/d;s@&nbsp;@@g;p' | awk -F\| 'BEGIN{OFS="|"; print "Release|Release Data|End of Life|Extended customer support\n---|---|---|---"}{print}'
```

Results

Release|Release Data|End of Life|Extended customer support
---|---|---|---
OpenStack Rocky|August 2018|February 2020|
OpenStack Queens|April 2018|April 2023|
Ubuntu 18.04 LTS|April 2018|April 2023|
OpenStack Queens|February 2018|April 2021|
OpenStack Pike|August 2017|February 2019|
OpenStack Ocata|February 2017|August 2018|February 2020
OpenStack Newton|October 2016|April 2018|
OpenStack Mitaka|April 2016|April 2021|
Ubuntu 16.04 LTS|April 2016|April 2021|
OpenStack Mitaka|April 2016|April 2019|
OpenStack Liberty|October 2015|April 2017|
OpenStack Kilo|April 2015|October 2016|April 2018
OpenStack Juno|October 2014|April 2016|
OpenStack Icehouse|April 2014|April 2019|
Ubuntu 14.10 LTS|April 2014|April 2019|
OpenStack Icehouse|April 2014|April 2017|
OpenStack Havana|October 2013|July 2014|
OpenStack Grizzly|May 2013|August 2014|
OpenStack Folsom|September 2012|June 2014|
OpenStack Essex|April 2012|April 2017|
Ubuntu 12.04 LTS|April 2012|April 2017|


## References
* [linuxlifecycle.com](https://linuxlifecycle.com)


## Change Log
* 2017.02.13 13:40 Mon Asia/Shanghai
    * first draft
* 2017.04.26 00:45 Wed America/Boston
    * add `RHEL 6.9`、`CentOS 6.9`
* 2018.04.17 14:48 Tue America/Boston
    * review, updaet, migrate to new blog
* 2018.04.18 10:14 Wed America/Boston
    * add [Ubuntu release end of life][ubuntureleaseeol]
* 2018.05.10 07:08 Thu America/Boston
    * add [CentOS 7.5.1804](https://lists.centos.org/pipermail/centos-announce/2018-May/022829.html)
* 2018.07.11 11:11 Wed America/Boston
    * update release info


[gnulinux]:https://www.gnu.org "GNU Operating System"
[linuxkernel]:https://www.kernel.org "The Linux Kernel"
[rhel]:https://www.redhat.com/en "RedHat"
[centos]:https://www.centos.org "CentOS"
[amzn]:https://aws.amazon.com/amazon-linux-ami/ "Amazon Linux AMI"
[debian]:https://www.debian.org "Debian"
[ubuntu]:https://www.ubuntu.com "Ubuntu"
[suse]:https://www.suse.com "SUSE"
[opensuse]:https://www.opensuse.org "OpenSUSE"
[lifecyclescript]:https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxLifeCycleInfo.sh
[ubuntureleaseeol]: https://www.ubuntu.com/info/release-end-of-life "Ubuntu release end of life"

<!-- end -->
