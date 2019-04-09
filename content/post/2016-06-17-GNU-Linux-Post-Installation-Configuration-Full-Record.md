---
title: GNU/Linux Post Installation Configuration Full Record
slug: GNU Linux Post Installation Configuration Full Record
date: 2016-06-17T11:30:54+08:00
lastmod: 2019-03-08T12:06:54-05:00
draft: false
keywords: ["SELinux", "Firewall", "Optimization", "Shell script"]
description: ""
categories:
- GNU/Linux
tags:
- Shell Script

comment: true
toc: true

---

[GNU/Linux][gnulinux]安裝完成後，需要進行一系列的初始化配置才能滿足使用需求。本文詳細記錄各初始化操作，並用Shell腳本實現相關過程（在`root`模式中操作）。

<!--more-->

[GNU/Linux][gnulinux]發行版衆多，目前有3大主流分支：[RHEL][rhel]、[Debian][debian]、[SUSE][suse]。本文中的操作支持如下主流發行版：[RHEL][rhel]/[CentOS][centos]/[Fedora][fedora]/[Amazon Linux][amzn]、 [Debian][debian]/[Ubuntu][ubuntu]、[SUSE][suse]/[OpenSUSE][opensuse]，暫不支持[Arch Linux][archlinux]、[Gentoo][gentoo]、[Slackware][slackware]等發行版。


## Official Documents
各發行版官方文檔頁

distribution|doc
---|---
RedHat|https://access.redhat.com/documentation/en/red-hat-enterprise-linux/
Debian|https://www.debian.org/doc/
Ubuntu|https://help.ubuntu.com
Suse|https://www.suse.com/documentation/
OpenSUSE|https://doc.opensuse.org
AWS|https://aws.amazon.com/documentation/


本人通過Shell腳本實現下載操作，目前支持[RHEL][rhel]/[AWS][amzn]/[SUSE][suse]/[OpenSUSE][opensuse]的文檔下載。代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxOfficialDocumentationDownload.sh | bash -s --
```

<script src="https://asciinema.org/a/189219.js" id="asciicast-189219" async></script>

## System Info Detection
爲快速偵測GNU/Linux系統信息，本人通過Shell腳本實現該操作過程(暫不支持對RAID磁盤的信息偵測)。代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxMachineInfoDetection.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxMachineInfoDetection.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189175.js" id="asciicast-189175" async></script>


## Shell Script
系統初始化操作通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189224.js" id="asciicast-189224" async></script>


## Tips And Tricks

### Bootable USB Drive
官方通常會在提供鏡像文件下載頁提供鏡像文件的Hash校驗值(通常爲sha256)，出於安全考慮，建議對下載到本地的系統鏡像進行Hash校驗。通過校驗後，將系統鏡像燒錄到U盤中，通過U盤安裝系統。此處以鏡像文件`~/Downloads/debian-9.4.0-amd64-DVD-1.iso`爲例。

```bash
# List the partition tables for the specified devices
fdisk -l
# Disk /dev/sdb: 16.1 GB, 16131293184 bytes, 31506432 sectors

# https://www.debian.org/CD/faq/#write-usb
dd if=~/Downloads/debian-9.4.0-amd64-DVD-1.iso of=/dev/sdb bs=4M; sync
```

### Screen Lock Hotkey
快速鎖屏幕

Desktop|Command
---|---
`Cinnamon`|Ctrl + Alt + L
`GNome`|Command(Win) + L


### X11 Forwarding Vis SSH
如果需要通過`ssh`調用遠程主機(Remote)中的GUI圖形化界面

在遠程主機(Remote)中進行如下配置

1. 修改`/etc/ssh/sshd_config`；
    * `AllowTcpForwarding yes`
    * `X11Forwarding yes`
2. 執行`startx`命令；

在本地主機(Local)中進行如下配置

1. 在文件`~/.bashrc`中添加`export DISPLAY=:0.0`；

**注意**：如果目標主機(Remote)的`~/.bashrc`中存在該指令，須將其註釋掉，否則無法正常啓用GUI。

使用方法

```bash
ssh -Y remoteUser@remoteHost -p remotePort
```

初次連接時會提示

>/usr/bin/xauth:  file /home/remoteUser/.Xauthority does not exist

該文件會自動創建，無須擔心。

### Rsync Synchronizing
通過`rsync`命令進行文件同步、備份，分別在本機和遠程主機中進行。命令使用參考[Rsync(Remote Sync): 10 Practical Examples of Rsync Command in Linux](https://www.tecmint.com/rsync-local-remote-file-synchronization-commands/ "Tecmint")

通過計劃任務(cron)進行定時操作，備份用戶家目錄`/home/maxdsre`。

注意目標路徑的owner，使用ssh進行文件傳輸時須指定ssh key。

相關命令

* `crontab -e`：修改、更新計劃任務
* `crontab -l`：查看已存在的計劃任務

執行`crontab -e`，寫入如下信息

```bash
# Back up to local directory /opt/Backup/
*/20 * * * * rsync --bwlimit=2048 -avz --delete /home/maxdsre /opt/Backup/

# Back up to remote host DataBack /opt/Backup/, 'DataBack' is defined in file ~/.ssh/config
*/30 */2 * * * rsync --bwlimit=800 --delete -avze "ssh -i /home/maxdsre/.ssh/id_ed25519" /home/maxdsre DataBack:/opt/Backup/
```

### GNOME Desktop Setting

#### Recent Files
在[GNome 3](https://www.gnome.org/gnome-3/)桌面環境中，默認會記錄當前用戶打開的文件，在 `Recent` 窗口中列出。

信息存放路徑爲`~/.local/share/recently-used.xbel`

```bash
[[ -f ~/.local/share/recently-used.xbel ]] && rm -f ~/.local/share/recently-used.xbel
```
但刪除後仍會自動創建。

方案1: 禁止`Recent`按鈕出現，參考[鏈接](http://forums.fedoraforum.org/showthread.php?t=308731)，通過`gsettings`設置

```bash
gsettings get org.gnome.desktop.privacy remember-recent-files
gsettings set org.gnome.desktop.privacy remember-recent-files false
```

方案2：通過GNome Extension [Recent Items](https://extensions.gnome.org/extension/72/recent-items/)手動清理，參考[鏈接](https://bbs.archlinux.org/viewtopic.php?id=116870)

可將二者結合起來使用。

#### CustomCorner
[CustomCorner](https://extensions.gnome.org/extension/1037/customcorner/)是GNome的一個擴展，用於在屏幕的四個角落創建快捷功能，代碼託管在[Gitlab](https://gitlab.com/eccheng/customcorner/tree/master)。

**注意**： 該擴展不支持GNome `3.14`及之前的版本。

根據[說明](https://gitlab.com/eccheng/customcorner/blob/master/README.md)編寫操作命令，通過如下命令安裝擴展

```bash
fileSavaPath='/tmp/archive.tar.gz'
curl -s https://gitlab.com/eccheng/customcorner/repository/archive.tar.gz?ref=master -o $fileSavaPath

targetPath="~/.local/share/gnome-shell/extensions/customcorner@eccheng.gitlab.com"
[[ -d "$targetPath" ]] && rm -rf "$targetPath"/* || mkdir -p "$targetPath"

tar xf $fileSavaPath -C "$targetPath" --strip-components=1

gnomeShellVersion=$(gnome-shell --version | sed -n -r 's@GNOME Shell (.*)@\1@p')

sed -i -r '/shell-version/s@("shell-version": \[\").*(\"\],)@\1'"$gnomeShellVersion"'\2@' "$targetPath"/metadata.json

[[ -f $fileSavaPath ]] && rm -f $fileSavaPath
unset fileSavaPath
unset targetPath
unset gnomeShellVersion
```

通過組合命令`Alt + F2`重啓GNome Shell，在桌面窗口左上側依次點擊 `Applications` --> `System Tools` --> `Tweak Tool` --> `Extensions`中找到 *Customcorner* ，根據個人需求進行設置。


## SELinux
SELinux配置文件路徑`/etc/selinux/config`，其模式分爲3種：`disabled`, `enforcing`, `permissive`，默認爲`permissive`。

RHEL/CentOS默認安裝有SELinux，修改配置文件即可。

對於SLES/OpenSUSE，只能先設置爲`permissive`重啓系統後才能修正爲`enforcing`，否則會出現無法問題。

不建議在Debian/Ubuntu中使用SELinux，配置過於繁瑣。

對於如何配置，此處不做討論。具體見本人Shell腳本`funcSELinuxConfiguration`、`funcSELinuxSemanageOperation`函數。

**重要**：如果對SELinux不瞭解，建議將模式調爲`permissive`，否則會出現因配置不當而導致應用無法正常使用的情況。


## User Management
### Config File
1. `/etc/security/pwquality.conf` 用於設置密碼生成策略
2. `/etc/login.defs` 用於設置有效期，加密方式等

### sudo privilege
爲普通用戶添加`sudo`權限

默認配置文件爲`/etc/sudoers`，若不存在，則須先安裝`sudo`安裝包。

**注意**：Debian系(Debian/Ubuntu)使用的sudo用戶組名爲`sudo`，RHEL/SUSE使用的則是`wheel`。

```bash
# disable sudo su - / sudo su root
sudo_config_path='/etc/sudoers'

# - For using group 'sudo' (Debian/Ubuntu)
sed -r -i 's@#*[[:space:]]*(%sudo[[:space:]]+ALL=\(ALL:ALL\)[[:space:]]+ALL)@# \1@;/%sudo ALL=NOPASSWD:ALL/d;/group sudo/a %sudo ALL=NOPASSWD:ALL,!/bin/su' "${sudo_config_path}"

# - For using group 'wheel'
sed -r -i 's@#*[[:space:]]*(%wheel[[:space:]]+ALL=\(ALL\)[[:space:]]+ALL)@# \1@;s@#*[[:space:]]*(%wheel[[:space:]]+ALL=\(ALL\)[[:space:]]+NOPASSWD: ALL).*@\1,!/bin/su@' "${sudo_config_path}"
```

爲用戶賦權

```bash
sudo_group_name='sudo'    # for group 'sudo'
sudo_group_name='wheel'    # for group 'wheel'

# - For existed user
usermod -a -G "${sudo_group_name}" "${username}"

# - For new user
usedadd -mN -G "${sudo_group_name}" "${new_username}"
```

### New User
創建新用戶

```bash
useradd -mN "${new_username}"

```

設置新密碼

```bash
# Debian/SUSE not support --stdin

# - For apt-get
# https://debian-administration.org/article/668/Changing_a_users_password_inside_a_script
echo "${new_username}:${new_password}" | chpasswd

# - For dnf/yum
echo "${new_password}" | passwd --stdin "${new_username}"

# - For Zypper
# https://stackoverflow.com/questions/27837674/changing-a-linux-password-via-script#answer-27837785
echo -e "${new_password}\n${new_password}" | passwd "${new_username}"
```

設置密碼失效時間

```bash
# setting user password expired date
passwd -n "${pass_change_minday}" -x "${pass_change_maxday}" -w "${pass_change_warnningday}" "${new_username}"
```

設置第一次登錄時強制重置密碼

```bash
# new created user have to change passwd when first login
chage -d0 "${new_username}"
```

## TimeZone
時區如`Asia/Singapore`, `America/New_York`, `Europe/Berlin`等，文件存放在目錄`/usr/share/zoneinfo/`中。

若存在`timedatectl`命令

```bash
timedatectl set-timezone "${new_timezone}"
timedatectl set-local-rtc false
timedatectl set-ntp true
```

若不存在`timedatectl`命令，分兩種情況：

```bash
# - Condition 1: For Debian/Ubuntu using apt-get
echo "${new_timezone}" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata

# - Condition 2:
# For RHEL/OpenSUSE
ln -fs "/usr/share/zoneinfo/${new_timezone}" /etc/localtime

# For CentOS6 or older
sed -r -i '/^ZONE=/{s@^([^=]+=).*@\1"'"${new_timezone}"'"@g;}' /etc/sysconfig/clock
```

## HostName
設置主機名

如果命令`hostnamectl`存在，執行

```bash
# The static hostname is stored in /etc/hostname, see hostname(5) for more information. The pretty hostname, chassis type, and icon name are stored in /etc/machine-info, see machine-info(5). -- man hostnamectl

# --static, --transient, --pretty
hostnamectl set-hostname "${new_hostname}"
hostnamectl --pretty set-hostname "${new_hostname}"
hostnamectl --static set-hostname "${new_hostname}"

# - set-deployment
# Debian Jessie has no set-deployment
hostnamectl set-deployment [development|integration|staging|production]

# hostnamectl set-location "${current_location}"
```

如果命令`hostnamectl`不存在

```bash
# temporarily change, when reboot, it will recover
hostname "${new_hostname}"

# For /etc/sysconfig/network exists (RHEL)
sed -r -i '/^HOSTNAME=/s@^(HOSTNAME=).*@\1'"${new_hostname}"'@g' /etc/sysconfig/network

# For /etc/hostname exists (Debian/OpenSUSE)
echo "${new_hostname}" > /etc/hostname
```

**注意**：修改主機名後，須在文件`/etc/hosts`中添加

```bash
127.0.0.1 "${new_hostname}"
::1 "${new_hostname}"
```
否則執行`sudo`時會出現如下報錯

>sudo: unable to resolve host AWS-Xenial

## Software Management
配置文件路徑

* yum: `/etc/yum.conf`
* dnf: `/etc/dnf/dnf.conf`
* apt-get: `/etc/apt/apt.conf.d`
* zypper: `/etc/zypp/zypp.conf`, `/etc/zypp/zypper.conf`

軟件源配置路徑

* yum/dnf: `/etc/yum.repos.d/`
* apt-get: `/etc/apt/sources.list`, `/etc/apt/sources.list.d/`
* zypper: `etc/zypp/repos.d/`

其中[CentOS][centos]需要單獨安裝`epel-release`源

各包管理器支持週期性更新配置，但此處不作敘述，詳細配置見本人Shell腳本`funcAutomaticPackageUpdate`函數。

### Essential Packages
一些必要的安裝包

* `bash-completion`： Tab自動補全
* `chrony`：時間同步


## Kernel
通常系統中的軟件源提供的[Kernel][linuxkernel]並不是最新版本，如果需要安裝最新版本的kernel，須進行相關設置。

For Debian/Ubuntu

```bash
# For Ubuntu
apt-get -yq -f install $(apt-cache search linux-generic-hwe 2> /dev/null | sed -r -n '1{s@^([^[:space:]]+).*$@\1@g;p}')


# For Debian
apt-get -t $(lsb_release -cs)-backports -y -q upgrade
```

For OpenSUSE
```bash
# http://pvdm.xs4all.nl/wiki/index.php/How_to_have_the_latest_kernel_in_openSUSE
# multiversion = provides:multiversion(kernel)
# multiversion.kernels = latest,latest-1,running
zypper ar -fcg http://download.opensuse.org/repositories/Kernel:/HEAD/standard/ kernel-repo
zypper dup -r kernel-repo
```

For CentOS
```bash
# ELRepo /etc/yum.repos./elrepo.repo

# version_id=7    # CentOS 7
# version_id=6    # CentOS 6
elrepo_info=$(curl -fsL https://elrepo.org/tiki/tiki-index.php | sed -r -n '/rpm (--import|-Uvh)/{s@<[^>]*>@@g;s@.*(http.*)$@\1@g;p}' | sed ':a;N;$!ba;s@\n@|@g;')

rpm --import "${elrepo_info%%|*}" 2> /dev/null
elrepo_info="${elrepo_info#*|}"
case "${version_id}" in
    7 ) elrepo_info="${elrepo_info%%|*}" ;;
    6 ) elrepo_info="${elrepo_info##*|}" ;;
esac

yum -y -q install "${elrepo_info}"

# Long term support kernel package name is kernel-lt version
# Mainline stable kernel package name is kernel-ml version

# yum --disablerepo='*' --enablerepo='elrepo-kernel' list available
# yum --disablerepo='*' --enablerepo='elrepo-kernel' -y -q install kernel-lt
yum --disablerepo='*' --enablerepo='elrepo-kernel' -y -q install kernel-ml

case "${version_id}" in
    7 )
        # egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
        grub2-set-default 0 2> /dev/null
    ;;
    6 )
        [[ -s '/etc/grub.conf' ]] && sed -r -i '/default=/{s@^([^+]+=).*$@\10@g;}' /etc/grub.conf
    ;;
esac
```

## Unicode Fonts
>A Unicode font is a computer font that maps glyphs to Unicode characters (i.e. the glyphs in the font can be accessed using code points defined in the Unicode Standard).  -- https://en.wikipedia.org/wiki/Unicode_font

對於正體中文，日韓文字，可通過 CJK Unified Ideographs ([中日韓統一表意文字](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97)) 實現，Wiki百科頁面 [Unicode擴充漢字](https://zh.wikipedia.org/zh-tw/Wikipedia:Unicode%E6%89%A9%E5%B1%95%E6%B1%89%E5%AD%97) 提供可用字體信息。

可通過 [字體試驗頁](https://ctext.org/font-test-page/zh) 測試系統是否支持擴展漢字。

可通過 [Unicode® Character Table](https://unicode-table.com/en/) 查找文字、符號的 unicode 編碼。

中日韓統一表意文字擴展區信息

擴展區 | 版本 | 日期 | 字數 | 位置
---|---|---|---|---
[擴展區A](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80A) | 3.0 | 2000 | [6,582](http://www.unicode.org/charts/PDF/U3400.pdf) | U+3400 - U+4DB5
[擴展區B](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80B) | 3.1 | 2001 | [42,711](http://www.unicode.org/charts/PDF/U20000.pdf) | U+20000 - U+2A6D6
[擴展區C](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80C) | 5.2 | 2003第五修訂版 | [4,149](http://www.unicode.org/charts/PDF/U2A700.pdf) | U+2A700 - U+2B734
[擴展區D](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80D) | 6.0 | 2010 | [222](http://www.unicode.org/charts/PDF/U2B740.pdf) | U+2B740 - U+2B81F
[擴展區E](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80E) | 8.0 | 2015 | [5,762](http://www.unicode.org/charts/PDF/U2B820.pdf) | U+2B820 - U+2CEAF
[擴展區F](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%97%A5%E9%9F%93%E7%B5%B1%E4%B8%80%E8%A1%A8%E6%84%8F%E6%96%87%E5%AD%97%E6%93%B4%E5%B1%95%E5%8D%80F) | 10.0 | 2017 | [7,473](http://www.unicode.org/charts/PDF/U2CEB0.pdf) | U+2CEB0 - U+2EBEF


可用字體信息

Font | Name |Introduction | Download Page
---|---|---|---
[Hanazono Mincho (花園明朝)](https://www.freejapanesefont.com/hanazono-mincho-%E8%8A%B1%E5%9C%92%E6%98%8E%E6%9C%9D/) | HanaMinA</br>HanaMinB | [花園フォントについて](http://fonts.jp/hanazono/) | [2017年09月04日版](https://osdn.net/projects/hanazono-font/releases/)
華秀月明 | H-SiuNiu | | [3.2](https://code.google.com/archive/p/ifont/downloads)
[Noto Sans CJK](https://www.google.com/get/noto/help/cjk/) | Noto Sans CJK {TC,SC,KR,JP}</br>Noto Sans Mono CJK {TC,SC,KR,JP} | [思源黑體](https://zh.wikipedia.org/wiki/%E6%80%9D%E6%BA%90%E9%BB%91%E9%AB%94)</br>[Noto Serif CJK is here!](https://opensource.googleblog.com/2017/04/noto-serif-cjk-is-here.html) | https://www.google.com/get/noto/help/cjk/
[BabelStone Han](http://www.babelstone.co.uk/Fonts/Han.html) | BabelStone Han | | [v. 12.0.0 (2019-03-01)](http://www.babelstone.co.uk/Fonts/Download/BabelStoneHan.zip)

>花園明朝字型分成 HanaMinA（花園明朝A）、HanaMinB（花園明朝B）兩部分，其中HanaMinA僅對中日韓統一表意文字區及其擴充A區提供全面支援，HanaMinB提供了對B區、C區、D區、E區、F區的完整支援。

字體

* fonts-hanazono - Japanese TrueType mincho font by KAGE system and FontForge
* fonts-noto-cjk - "No Tofu" font families with large Unicode coverage (CJK)
* fonts-noto-cjk-extra - "No Tofu" font families with large Unicode coverage (CJK all weight)

如何安裝見

* [Debian wiki | Fonts](https://wiki.debian.org/Fonts#Manually)
* [How to install fonts](https://www.google.com/get/noto/help/install/)

>Install a font manually by downloading the appropriate .ttf or otf files and placing them into `/usr/local/share/fonts` (system-wide), `~/.local/share/fonts` (user-specific) or  `~/.fonts` (user-specific). These files should have the permission 644 (-rw-r--r--), otherwise they may not be usable.
>
>Run `fc-cache` to update the font cache (add -v for verbose output). -- https://wiki.debian.org/Fonts


```bash
# list install fonts
fc-list

# rebuilds cached list of fonts (in ~/.config/fontconfig, older caches may also be in ~/.fontconfig)
fc-cache -fv
```

## Input Methods
如果需要安裝中文輸入法，可通過[ibus](https://github.com/ibus/ibus)或[fcitx](https://www.fcitx-im.org)框架實現。

推薦使用 [RIME | 中州韻輸入法引擎](http://rime.im)，具體安裝說明見[RimeWithIBus](https://github.com/rime/home/wiki/RimeWithIBus)。

### Via ibus
使用[ibus](https://github.com/ibus/ibus)(Intelligent Input Bus)作為輸入法框架。可通過如下命令查看相關軟件包的名稱

```bash
apt-cache search -n ^ibus | awk '{print $1}'
```

```bash
sudo apt-get install -y ibus-rime ibus-pinyin ibus-cangjie ibus-sunpinyin
```

安裝完成後執行如下命令配置ibus

```bash

sudo ibus-setup
```

出現提示信息

>The IBus daemon is not running. Do you wish to start it?

選擇`Yes`後出現如下提示信息

>IBus has been started! If you cannot use IBus, add the following lines to your $HOME/.bashrc; then relog into your desktop.
  export GTK_IM_MODULE=ibus
  export XMODIFIERS=@im=ibus
  export QT_IM_MODULE=ibus

在`IBus Preferences`窗口中選`Input Method`選項卡，點擊`Add`按鈕，選擇`Chinese`，可看到已經安裝的輸入法，選擇需要的輸入法，點擊`Add`添加。

在窗口左上角的`Applications`中找到`Settings`(通常在`System Tools`中)，打開`All Settings`窗口後，找到`Region&Language`圖標點擊進去，在`Input Reources`下方點`+`(加號)，點擊`More`(三個豎點那欄)，在其中搜索`chinese`，可列出已安裝的中文輸入法。

此處選擇`Chinese(Rime)`，點擊窗口右上角的`Add`按鈕進行添加。操作完成後即可在桌面右上角看到輸入法圖標，可點擊進行切換。也可通過按鍵`Super`+`Space`(Win鍵+空格鍵)進行輸入法的切換。

`Rime`輸入法支持多種輸入方案，可通過如下方式進行切換
1. 功能鍵`F4`
2. 組合按鍵<code>Ctrl+\`</code> (鍵盤左上角的反引號)

**注**：可能需要退出重新登入系統才能使用。

### Via Fcitx
fcitx官方文檔 [Install and Configure
](https://fcitx-im.org/wiki/Install_and_Configure)。


```bash
# - For Debian/Ubuntu
sudo apt-get install fcitx fcitx-rime -y

# - For OpenSUSE
# search
sudo zypper ref
zypper se fcitx*
# install fcitx and rime
sudo zypper in -y fcitx fcitx-rime
# This overwrites the system default set in /etc/sysconfig/language
sed -i '/^INPUT_METHOD=/d' ~/.profile
echo "INPUT_METHOD=\"fcitx\"" >> ~/.profile
```


```bash
# 選擇input method
im-config

# 點符號 + 添加rime，取消勾選 Only Show Current Language後查詢
fcitx-config-gtk3

# 重啓fcitx
# https://fcitx-im.org/wiki/Install_input_method
fcitx -r
```

退出當前登錄狀態，重新登錄。通過組合按鍵`Ctrl+Space`啓用Rime輸入法，通過按鍵`L Shift`臨時切換輸入法。


## Web Browser
### Google Chrome
Google Chrome官網爲 <https://www.google.com/chrome/>，點擊頁面的 **Download now** 按鈕，會跳出彈框提示需要下載何種安裝包，主要兩種

* [64 bit .deb (For Debian/Ubuntu)](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)
* [64 bit .rpm (For Fedora/openSUSE)](https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm)

在GNome Desktop環境中，`/usr/bin/x-www-browser`用於設置系統默認瀏覽器，以下是Google Chrome在安裝過程中的配置記錄

```bash
update-alternatives: using /usr/bin/google-chrome-stable to provide /usr/bin/x-www-browser (x-www-browser) in auto mode
update-alternatives: using /usr/bin/google-chrome-stable to provide /usr/bin/gnome-www-browser (gnome-www-browser) in auto mode
update-alternatives: using /usr/bin/google-chrome-stable to provide /usr/bin/google-chrome (google-chrome) in auto mode
```

Google Chrome的配置文件(如登錄信息、設置、插件等)目錄爲
```bash
~/.config/google-chrome
```

安裝成功後，即可在`Menu`菜單中找到Google Chrome應用程序(通常是`Applications`->`Internet`)。

在瀏覽器地址欄中輸入如下地址，可顯示對應功能、信息

```bash
#瀏覽器版本信息
chrome://version

#Experiments功能控制
chrome://flags

#清除瀏覽歷史
chrome://settings/clearBrowserData
```

#### Plugins
在瀏覽器地址欄中輸入

```bash
chrome://plugins/
```
禁用不需要的插件，如`Chrome PDF Viewer`。

啟用`Do Not Track` (Setting-->Privacy中)

#### Extensions
出於某些需要，安裝如下擴展

* [uBlock Origin](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm/related?hl=en-US)
* [WOT: Web of Trust, Website Reputation Ratings](https://chrome.google.com/webstore/detail/wot-web-of-trust-website/bhmmomiinigofkjcapegjjndpbikblnp?hl=en-US)
* [crxMouse Chrome Gestures](https://chrome.google.com/webstore/detail/crxmouse-chrome-gestures/jlgkpaicikihijadgifklkbpdajbkhjo?hl=en-US)
* [Disconnect](https://chrome.google.com/webstore/detail/disconnect/jeoacafpbcihiomhlakheieifhpjdfeo?hl=en-US)
* [HTTPS Everywhere](https://chrome.google.com/webstore/detail/https-everywhere/gcbommkclmclpchllfjekcdonpmejbdp?hl=en-US)
* [Decentraleyes](https://chrome.google.com/webstore/detail/decentraleyes/ldpochfccmkkmhdbclfhpagapcfdljkj?hl=en-US)
* [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en-US)
* [Random User-Agent](https://chrome.google.com/webstore/detail/random-user-agent/einpaelgookohagofgnnkcfjbkkgepnp?hl=en-US)
* [Privacy Badger](https://chrome.google.com/webstore/detail/privacy-badger/pkehgijcmpdhfbdbbnkijodmdjhbjlgp?hl=en-US)
* [Click&Clean](https://chrome.google.com/webstore/detail/clickclean/ghgabhipcejejjmhhchfonmamedcbeod?hl=en-US)
* [LastPass](https://chrome.google.com/webstore/detail/lastpass-free-password-ma/hdokiejnpimakedhajhdlcegeplioahd?hl=en-US)
* [1Password X](https://chrome.google.com/webstore/detail/1password-x-%E2%80%93-password-ma/aeblfdkhhhdcdjpifhhbdiojplfjncoa?hl=en-US)
* [KeePassXC-Browser](https://chrome.google.com/webstore/detail/keepassxc-browser/oboonakemofpalcgghocfoadofidjkkk?hl=en-US)
* [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo?hl=en-US)
* [Wappalyzer](https://chrome.google.com/webstore/detail/wappalyzer/gppongmhjkpfnbhagpmjfkannfbllamg?hl=en-US)

Apps
* [Marxico](https://chrome.google.com/webstore/detail/marxico/kidnkfckhbdkfgbicccmdggmpgogehop?hl=en-US)

### Mozilla Firefox
系統自帶的Mozilla Firefox版本較低，若要使用最新版本，需從[Mozilla Firefox](https://www.mozilla.org/en-US/firefox/)官網下載，FTP[路徑](https://download-installer.cdn.mozilla.net/pub/firefox/releases/)。Mozilla Firefox默認未安裝`Adobe Flash Player`無法播放視頻，需從Adobe官網下載[Adobe Flash Player](https://get.adobe.com/flashplayer/)後手動安裝，按需選擇下載格式，此處選擇`.tar.gz`格式。


#### Extensions
出於某些需要，安裝如下擴展

* [Firefox Multi-Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
* [uBlock Origin](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)
* [WOT](https://addons.mozilla.org/en-US/firefox/addon/wot-safe-browsing-tool/)
* [Gesturefy](https://addons.mozilla.org/en-US/firefox/addon/gesturefy/)
* [HTTPS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/https-everywhere/)
* [Disconnect](https://addons.mozilla.org/en-US/firefox/addon/disconnect/)
* [Privacy Badger](https://addons.mozilla.org/en-US/firefox/addon/privacy-badger17/)
* [Clear Cache](https://addons.mozilla.org/en-US/firefox/addon/clearcache/)
* [Random User-Agent](https://addons.mozilla.org/en-US/firefox/addon/random_user_agent/)
* [Country Flags & IP Whois](https://addons.mozilla.org/en-US/firefox/addon/country-flags-ip-whois/)
* [1Password X](https://addons.mozilla.org/en-US/firefox/addon/1password-x-password-manager/)
* [LastPass Password Manager](https://addons.mozilla.org/en-US/firefox/addon/lastpass-password-manager/)
* [Tampermonkey](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)


### SRWare Iron
SRWare Iron官方網站 <https://www.srware.net/en/>，[介紹頁面](https://www.srware.net/en/software_srware_iron.php)，[下載頁面](https://www.srware.net/en/software_srware_iron_download.php)。

其中Linux版本下載頁的鏈接可通過如下命令獲取

```bash
curl -fsL https://www.srware.net/en/software_srware_iron_download.php | sed -n '/Iron for Linux/p' | sed -r -n 's@.*href="(.*)" .*@\1@p;' | sed -r 's@\&amp\;@\&@g'
# http://www.srware.net/forum/viewtopic.php?f=18&t=29465

# package download url
# http://www.srware.net/downloads/iron-linux-64.tar.gz
```


## Change Logs
* 2016.06.17 11:30 Fri Asia/Shanghai
    * 初稿完成
* 2018.04.24 17:03 Tue America/Boston
    * 勘誤，更新，遷移到新blog
* 2019.01.18 11:54 Fri America/Boston
    * 增加CJK擴充漢字信息


[gnulinux]:https://www.gnu.org "GNU Operating System"
[linuxkernel]:https://www.kernel.org "The Linux Kernel"
[rhel]:https://www.redhat.com/en "RedHat"
[centos]:https://www.centos.org "CentOS"
[fedora]:https://getfedora.org "Fedora"
[amzn]:https://aws.amazon.com/amazon-linux-ami/ "Amazon Linux AMI"
[debian]:https://www.debian.org "Debian"
[ubuntu]:https://www.ubuntu.com "Ubuntu"
[suse]:https://www.suse.com "SUSE"
[opensuse]:https://www.opensuse.org "OpenSUSE"
[archlinux]:https://www.archlinux.org "Arch Linux"
[gentoo]:https://www.gentoo.org "Gentoo Linux"
[slackware]:http://www.slackware.com "Slackware Linux"
[lifecyclescript]:https://github.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxLifeCycleInfo.sh

<!-- End -->
