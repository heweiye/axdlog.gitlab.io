---
title: Using Google Authenticator To Set Up Multi-Factor Authentication On GNU/Linux
date: 2016-01-25T21:01:52+08:00
lastmod: 2018-04-11T23:00:52-04:00
draft: false
keywords: ["Google Authenticator", "PAM", "SSH", "Multi-Factor Authentication"]
description: "Using Google Authenticator to set up multi-factor authentication on GNU/Linux"

categories:
- Security
tags:
- PAM
- SSH

comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
contentCopyright: ""
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
---

[PAM](https://en.wikipedia.org/wiki/Linux_PAM 'Wikipedia')是一種動態認證機制，通過多重認證提升系統安全係數。[Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator)是基於[TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm 'WikiPedia')和[HOTP](https://en.wikipedia.org/wiki/HMAC-based_One-time_Password_Algorithm 'WikiPedia')的2步認證應用，通過移動端的`Google Authenticator`應用生成token(存活期30秒)。

本文記錄利用其為GNU/Linux桌面系統、遠程SSH連接配置2步認證，並通過Shell Script實現相關功能。

<!--more-->

## Introduction
Google官方關於`2-Step Verification`的介紹頁見 <https://www.google.com/landing/2step/>。

安裝說明見 [2-Step Verification](https://support.google.com/accounts/topic/7189195)。

Google認證PAM模塊 **google-authenticator-libpam** 的代碼託管在[GitHub](https://github.com/google/google-authenticator-libpam)。


## Preparation
本文操作須滿足以下要求：

* 運行GNU/Linux系統的主機；
* 正常訪問[Google](https://www.google.com/) (中國大陸地區可能無法正常訪問)；
* 移動設備，用於安裝安裝 `Google Authenticator` APP，具體支持設備見[Install Google Authenticator](https://support.google.com/accounts/answer/1066447?hl=en)；
* `Google Authenticator`的源碼包，GitHub[地址](https://github.com/google/google-authenticator-libpam),採用源碼編譯安裝；

### Conventions
相關操作將分別在`CentOS 6.9`、`CentOS 7.4`、`Debian Stretch`、`Ubuntu Xenial`和`OpenSUSE Leap`中進行，通過`chrony`同步網路時間

OS|Kernel|SSHD
---|---|---
SUSE Linux Enterprise Server 12 SP3|4.4.114-94.14|7.2p2
Red Hat Enterprise Linux Server release 7.4 (Maipo)|3.10.0-693.21.1|7.4p1
CentOS Linux release 7.4.1708 (Core)|3.10.0-693.21.1|7.4p1
CentOS release 6.9 (Final)|2.6.32-696.23.1|5.3p1
Debian GNU/Linux 9 (stretch)|4.9.0-5|7.4p1
Ubuntu 16.04.4 LTS|4.13.0-37|7.2p2
openSUSE Leap 42.2|4.4.74-18.20-default|7.2p2

執行如下命令安裝SSH的`client`和`server`

```bash
# CentOS 7
sudo yum install -y -q openssh-server openssh-clients

# Debian / Ubuntu
sudo apt-get update
sudo apt-get install -yq openssh-client openssh-server openssh-sftp-server

# OpenSUSE / SLES
sudo zypper in -yl openssh
```

定義源碼下載地址`/tmp`
定義源碼安裝路徑`/opt/googleAuthenticator`

安裝編譯所需的包

```bash
#CentOS
sudo yum install -y gcc make autoconf automake libtool pam-devel

#Debian / Ubuntu
sudo apt-get install -y gcc make autoconf automake libtool libpam0g-dev libqrencode3

#OpenSUSE
sudo zypper in -yl gcc make autoconf automake libtool pam-devel
```

### Downloading Source Code
本文採用`git`下載代碼，git地址為
```http
https://github.com/google/google-authenticator-libpam.git
```

如果不想用`git`下載，可先下載`.zip`格式的壓縮包，再通過命令`unzip`解壓縮。需要系統安裝有`zip`、`unzip`命令。

執行如下命令下載代碼

```bash
cd /tmp
git clone https://github.com/google/google-authenticator-libpam.git
```
下載完成後目錄為`/tmp/google-authenticator-libpam`。

**註**：如果不想使用git下載，可在項目頁點擊`Clone or download`-->`Download ZIP`下載壓縮包。關於如何安裝`git`，可參考本人的「[Compile Install And Configure Git On CentOS7](http://lempstacker.com/tw/Compile-Install-And-Configure-Git-On-CentOS7/)」。

### Compiling & Installing
安裝目錄為`/opt/googleAuthenticator`

執行如下命令進行編譯安裝

**注意**：如果壓縮包解壓目錄(如`/tmp`)是一個單獨的分區，且文件系統設置了`noexec`屬性(`/etc/fstab`)，會造成`./bootstrap.sh`、`./configure`無法執行，通過`sudo`也會提示無執行權限。此時可以考慮將解壓縮目錄放置到其它目錄中，如用戶家目錄。

```bash
#臨時更改umask
umask 022

#創建安裝目錄
[[ -d '/opt/googleAuthenticator' ]] && sudo rm -rf /opt/googleAuthenticator
sudo mkdir -pv /opt/googleAuthenticator

#切換到libpam目錄下
cd /tmp/google-authenticator-libpam

#執行腳本
./bootstrap.sh

#配置安裝路徑
./configure --prefix=/opt/googleAuthenticator

#並行運行4個任務，構建應用程序
make -j 4

#安裝
sudo make install
```

>If you don't have access to "sudo", you have to manually become "root" prior to calling "make install". -- https://github.com/google/google-authenticator-libpam

安裝完成後，出現如下信息
```
Libraries have been installed in:
   /opt/googleAuthenticator/lib/security

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[1]: Leaving directory `/root/google-authenticator-libpam'
```

庫文件路徑`/opt/googleAuthenticator/lib/security`有可能顯示爲`/opt/googleAuthenticator/lib64/security`。

稍後配置需要用到的PAM文件`pam_google_authenticator.so`就存儲在目錄
```bash
/opt/googleAuthenticator/lib/security
# or
/opt/googleAuthenticator/lib64/security
```

具體對應關係如下

path|dirto list
---|---
/opt/googleAuthenticator/lib/security|Debian/Ubuntu/CentOS/RHEL/SLES
/opt/googleAuthenticator/lib64/security|OpenSUSE

目錄`/opt/googleAuthenticator`下有`bin`、`lib`或`lib64`、`share`3個子目錄。

接下來為可執行程序`google-authenticator`配置PATH路徑、庫文件，執行如下命令

```bash
lib_dir=${lib_dir:-'/opt/googleAuthenticator/lib'}
[[ -d '/opt/googleAuthenticator/lib64' ]] && lib_dir='/opt/googleAuthenticator/lib64'
bin_dir=${bin_dir:-'/opt/googleAuthenticator/bin'}

#導出庫文件
sudo bash -c 'echo '"${lib_dir}"' > /etc/ld.so.conf.d/googleAuthenticator.conf'
#讓系統重新生成緩存 Debian中路徑為 /sbin/ldconfig
sudo ldconfig -v

#爲可執行程序添加PATH路徑
sudo bash -c 'echo "export PATH=\$PATH:'"${bin_dir}"'" > /etc/profile.d/googleAuthenticator.sh'
source /etc/profile.d/googleAuthenticator.sh
```

## Running & Configuring Google Authenticator
如想要無交互模式生成(輸出內容存儲在文件`~/google_authenticator`中)，可執行如下命令

```bash
echo 'y' | google-authenticator -t -d -f -Q UTF8 -r 3 -R 30 2>&1 > ~/google_authenticator
```

以下操作是交互模式過程

```bash
google-authenticator
```

操作過程如下

```bash
#是否使用基於時間的認證令牌
Do you want authentication tokens to be time-based (y/n) y
Warning: pasting the following URL into your browser exposes the OTP secret to Google:

#生成的二維碼地址
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@centos%3Fsecret%3DQYCY5YLGZJLCXWIUSPJQGVSQSU%26issuer%3Dcentos

#此處是一個二維碼，可直接使用移動端Google Authenticator APP中的`Scan a barcode`掃描識別

#安全碼，在手機端Google Authenticator APP中的`Enter provided key`中用到
Your new secret key is: QYCY5YLGZJLCXWIUSPJQGVSQSU

#認證碼
Your verification code is 795124

#5組緊急備用驗證碼，主要用於當手機遺失後找回正確的認證碼
Your emergency scratch codes are:
  64621153
  42898735
  96729389
  90142620
  66573383

#是否更新文件 ~/.google_authenticator，該文件默認不存在
Do you want me to update your "/home/flying/.google_authenticator" file? (y/n) y

#是否允許同一認證令牌用於多種用途
Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

#基於時間登錄，每個令牌默認的有效時間是30秒，足夠抵消客戶端到服務器之間的時間延遲
By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) y

#單位時間內顯示登錄嘗試的次數，以防暴力破解，默認每30秒內不能超過3次
If the computer that you are logging into isn\'t hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

查看文件 `~/.google_authenticator`，內容如下

```
QYCY5YLGZJLCXWIUSPJQGVSQSU
" RATE_LIMIT 3 30
" WINDOW_SIZE 17
" DISALLOW_REUSE
" TOTP_AUTH
64621153
42898735
96729389
90142620
66573383
```


## Installing & Configuring Google Authenticator APP
本人手機運行的是Android系統，參考[Install Google Authenticator](https://support.google.com/accounts/answer/1066447?hl=en)中`Android devices`部分，在[Google play](https://play.google.com/store)中搜索安裝[Google Authenticator APP](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)。

安裝成功後，打開該APP，點擊右下十字紅心圓，有兩種添加方式
* Scan a barcode: 直接掃描上一步操作生成的二維碼即可
* Enter provided key
    * `Enter account name`: 可自定義
    * `Enter your key`: 即上一步操作生成的secret key
    * 默認是`Time based`，另一選項是`Counter based`

認證令牌默認每30s更新一次


## Configuring PAM
Google官方稱在PAM配置文件中添加`auth required pam_google_authenticator.so`。——「[PAM Module Instructions](https://github.com/google/google-authenticator-libpam/blob/master/README.md)」

關於PAM，可參閱
* [Linux PAM](https://en.wikipedia.org/wiki/Linux_PAM 'Wikipedia')
* [Linux-PAM](http://www.linux-pam.org/)
* [CHAPTER 5. USING PLUGGABLE AUTHENTICATION MODULES (PAM) - RedHat](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/Pluggable_Authentication_Modules.html 'Red Hat')
* [LDAP/PAM - Debian](https://wiki.debian.org/LDAP/PAM 'Debian')
* [PAM configuration guide for Debian](http://www.rjsystems.nl/en/2100-pam-debian.php)

**重要**： 以下操作非常重要，請務必要執行。

### CentOS & RHEL & SLES
在CentOS 6、CentOS 7、RHEL 7、SLES 12中，`PAM`的配置文件如下

path|explain|format
---|---|---
`/lib64/security/`|模塊路徑
`/etc/pam.conf`|通用配置文件|`application type control module-path module-arguments`
`/etc/pam.d/`|專用配置文件|`type control module-path module-arguments`

`auth required pam_google_authenticator.so`解釋說明
* `auth`: 屬於`type`中一種{auth|account|passwd|session}，表示帳號的認證與授權
* `required`: 屬於`control`中一種{required|requisite|sufficient|optional|include}，表示必須檢查通過，否則即爲失敗，且不論成功與否，都須繼續由後續同種功能的其它模塊進行檢查。
* `pam_google_authenticator.so`: (**重要**)其實是相對於目錄`/lib64/security/`而言的，該模塊當前的絕對路徑是`/opt/googleAuthenticator/lib/security/pam_google_authenticator.so`；如果使用要相對路徑，則需要爲其創建符號鏈接至目錄`/lib64/security/`，也可直接使用絕對路徑。

此處通過創建符號鏈接載入該模塊，執行如下命令

```bash
sudo ln -fs /opt/googleAuthenticator/lib/security/pam_google_authenticator.so /lib64/security/
```

### Debian & Ubuntu
在Debian Stretch和Ubuntu Xenial中，`PAM`的配置文件如下

path|explain|format
---|---|---
`/lib/x86_64-linux-gnu/security/`|模塊路徑
`/etc/pam.conf`|通用配置文件|`application type control module-path module-arguments`
`/etc/pam.d/`|專用配置文件|`type control module-path module-arguments`

`auth required pam_google_authenticator.so`解釋說明
* `auth`: 屬於`type`中一種{auth|account|passwd|session}，表示帳號的認證與授權
* `required`: 屬於`control`中一種{required|requisite|sufficient|optional|include}，表示必須檢查通過，否則即爲失敗，且不論成功與否，都須繼續由後續同種功能的其它模塊進行檢查。
* `pam_google_authenticator.so`: (**重要**)其實是相對於目錄`/lib/x86_64-linux-gnu/security/`而言的，該模塊當前的絕對路徑是`/opt/googleAuthenticator/lib/security/pam_google_authenticator.so`；如果使用要相對路徑，則需要爲其創建符號鏈接至目錄`/lib/x86_64-linux-gnu/security/`，也可直接使用絕對路徑。

此處通過創建符號鏈接載入該模塊，執行如下命令

```bash
sudo ln -fs /opt/googleAuthenticator/lib/security/pam_google_authenticator.so /lib/x86_64-linux-gnu/security/
```

### OpenSUSE Leap
在OpenSUSE Leap中，`PAM`的配置文件如下

path|explain|format
---|---|---
`/lib64/security/`|模塊路徑
`/etc/pam.d/`|專用配置文件|`type control module-path module-arguments`

`auth required pam_google_authenticator.so`解釋說明
* `auth`: 屬於`type`中一種{auth|account|passwd|session}，表示帳號的認證與授權
* `required`: 屬於`control`中一種{required|requisite|sufficient|optional|include}，表示必須檢查通過，否則即爲失敗，且不論成功與否，都須繼續由後續同種功能的其它模塊進行檢查。
* `pam_google_authenticator.so`: (**重要**)其實是相對於目錄`/lib64/security/`而言的，該模塊當前的絕對路徑是`/opt/googleAuthenticator/lib64/security/pam_google_authenticator.so`；如果使用要相對路徑，則需要爲其創建符號鏈接至目錄`/lib64/security/`，也可直接使用絕對路徑。

此處通過創建符號鏈接載入該模塊，執行如下命令

```bash
sudo ln -fs /opt/googleAuthenticator/lib64/security/pam_google_authenticator.so /lib64/security/
```


## GNU/Linux Desktop Authentication
```bash
# GNOME Desktop
/etc/pam.d/gdm-password

# Unity Desktop for Ubuntu
/etc/pam.d/lightdm
```
中進行配置。

### CentOS 7
/etc/pam.d/gdm-password

CentOS 7中文件內容格式如下

```bash
auth     [success=done ignore=ignore default=bad] pam_selinux_permit.so
#此行爲添加的規則
auth        required      pam_google_authenticator.so
auth        substack      password-auth
auth        optional      pam_gnome_keyring.so
auth        include       postlogin
```

### Debian Stretch
/etc/pam.d/gdm-password

Debian Stretch中文件內容格式如下

```bash
#%PAM-1.0
#此行爲添加的規則
auth    required        pam_google_authenticator.so
auth    requisite       pam_nologin.so
auth    required    pam_succeed_if.so user != root quiet_success
@include common-auth
auth    optional        pam_gnome_keyring.so
@include common-account
```

### Ubuntu Xenial
/etc/pam.d/lightdm

Ubuntu Xenial中文件內容格式如下

```bash
#%PAM-1.0
#此行爲添加的規則
auth    required        pam_google_authenticator.so
auth    requisite       pam_nologin.so
auth    sufficient      pam_succeed_if.so user ingroup nopasswdlogin
@include common-auth
auth    optional        pam_gnome_keyring.so
auth    optional        pam_kwallet.so
auth    optional        pam_kwallet5.so
@include common-account
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so close
#session required        pam_loginuid.so
session required        pam_limits.so
@include common-session
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so open
session optional        pam_gnome_keyring.so auto_start
session optional        pam_kwallet.so auto_start
session optional        pam_kwallet5.so auto_start
session required        pam_env.so readenv=1
session required        pam_env.so readenv=1 user_readenv=1 envfile=/etc/default/locale
@include common-password
```

### OpenSUSE
/etc/pam.d/gdm-password

```bash
#%PAM-1.0
# GDM PAM standard configuration (with passwords)
auth     required       pam_google_authenticator.so
auth     include        common-auth
account  include        common-account
password include        common-password
session  required       pam_loginuid.so
session  include        common-session
```

保存修改後，退出系統重新登入，會提示輸入`Verification code`，在Google Authenticator APP中查看當前的認證令牌號碼輸入。令牌驗證有效後，提示照常輸入用戶名、密碼。如果令牌驗證不通過，則會報錯，無法登入。


## SSH Authentication
`ssh`是[Open SSH](https://en.wikipedia.org/wiki/OpenSSH)的Client(客戶端)，`sshd`是Server(服務器端)，常用於連接遠程主機。

配置文件如下

| type | name | configurations |
| :--- | :--- | :--- |
| Server | `sshd` | `/etc/ssh/sshd_config` |
| Client | `ssh` | `/etc/ssh/ssh_config` |

`sshd`在PAM中也有配置文件，路徑`/etc/pam.d/sshd`

爲SSH配置`Google Authenticator`，需要使用如下配置文件
* `/etc/pam.d/sshd`
* `/etc/ssh/sshd_config`

**注意**：此處默認通過key進行SSH通信，如何生成SSH-Kegen Key，可參考本人Blog [SSH Configurations And Usages](https://lempstacker.com/tw/SSH-Configurations-And-Usages/#Generate-Authentication-SSH-Kegen-Keys 'LempStacker')。

操作完成後，須執行如下命令重啟`sshd`服務使修改生效

```bash
sudo systemctl restart sshd
# sudo systemctl restart sshd.service
```

**注意**：如果系統已經啓用了`SELinux`，SSH遠程登錄時會反覆出現要求輸入認證碼的情況，系統日誌輸出類似內容
>sshd(pam_google_authenticator)[13445]: Failed to update secret file "/root/.google_authenticator": Permission denied
>error: PAM: Authentication failure for root from $IP

可通過在`auth required pam_google_authenticator.so`後添加`secret`指令解決。

默認情況下，命令`google-authenticator`生成的文件路徑爲`~/.google_authenticator`，爲了能夠讓PAM讀取含有所需key的文件`~/.google_authenticator`，可先將其複製到`~/.ssh/`目錄中(同時更改文件owner)，再通過設置指令`secret=${HOME}/.ssh/.google_authenticator`。這樣即便已經啓用了SELinux，PAM仍可以讀取所需的文件內容，而不會被SELinux攔截。完整指令如下：

```bash
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator
# or
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator [authtok_prompt=Verification code: ]
```

RHEL、CentOS的配置方式相同，Debian、Ubuntu的配置方式相同，OpenSUSE、SLES的配置方式相同。以下分情況討論

### CentOS7 With SSH Key
此前使用key進行SSH連接

* /etc/pam.d/sshd

```bash
#%PAM-1.0
auth       required     pam_sepermit.so
#auth       substack     password-auth
auth       required     pam_google_authenticator.so
# auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator    # If SELinux Enabled
auth       include      postlogin
```

將`auth substack password-auth`行用符號`#`註釋掉，在其後添加新行

```
auth required pam_google_authenticator.so
```
因為是用key登錄，註釋`password-auth`可跳過密碼認證這一步驟。


* /etc/ssh/sshd_config

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**重要**： 因為是用key登錄,請務必添加指令`AuthenticationMethods`，否則配置不起作用，仍舊是驗證key後直接登錄。

具體操作命令

```bash
#UsePAM
sudo sed -i '/^UsePAM /d;' /etc/ssh/sshd_config
sudo bash -c 'echo "UsePAM yes" >> /etc/ssh/sshd_config'

#AuthenticationMethods
sudo sed -i '/^AuthenticationMethods /d' /etc/ssh/sshd_config
sudo bash -c 'echo "AuthenticationMethods publickey,password publickey,keyboard-interactive" >> /etc/ssh/sshd_config'

#ChallengeResponseAuthentication
sudo sed -i -r 's@(ChallengeResponseAuthentication) no@\1 yes@g' /etc/ssh/sshd_config
```

操作完成後，執行如下命令重啟`sshd`服務

```bash
sudo systemctl restart sshd
```

演示示例

```bash
flying@lempstacker:~$ ssh -C -c blowfish flying@192.168.0.140
Authenticated with partial success.
Verification code:
Last login: Thu Jan 12 18:34:41 2017 from 192.168.0.179
[flying@lempstacker ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@lempstacker ~]$ exit
logout
Connection to 192.168.0.140 closed.
flying@lempstacker:~$
```

### CentOS7 With Password
此前使用密碼進行SSH連接(未執行`ssh-copy-id`)

* /etc/pam.d/sshd

```bash
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       required     pam_google_authenticator.so
auth       include      postlogin
```

在`auth substack password-auth`後添加新行
```bash
auth required pam_google_authenticator.so
```

**重要**： 因是通過密碼進行遠程連接，請務必保證行`auth substack password-auth`不被註釋，否則只驗證`Google Authenticator`的token即可登錄，不建議這麼操作。


* /etc/ssh/sshd_config
務必保證如下配置為`yes`
```
UsePAM yes
ChallengeResponseAuthentication yes
```

具體操作命令
```bash
#UsePAM
sudo sed -i '/^UsePAM /d;' /etc/ssh/sshd_config
sudo bash -c 'echo "UsePAM yes" >> /etc/ssh/sshd_config'

#ChallengeResponseAuthentication
sudo sed -i -r 's@(ChallengeResponseAuthentication) no@\1 yes@g' /etc/ssh/sshd_config
```

操作完成後，執行如下命令重啟`sshd`服務
```bash
sudo systemctl restart sshd
```

演示示例

```bash
flying@lempstacker:~$ ssh -C -c blowfish flying@192.168.0.140
Password:
Verification code:
Last login: Thu Jan 12 18:38:01 2017 from 192.168.0.179
[flying@lempstacker ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@lempstacker ~]$ cat .ssh/authorized_keys |wc -l
0
[flying@lempstacker ~]$ exit
logout
Connection to 192.168.0.140 closed.
flying@lempstacker:~$
```

### Debian With SSH Key
此前使用key進行SSH連接

* /etc/pam.d/sshd

默認文件末尾是

```
# Standard Un*x password updating.
@include common-password
```

需要進行的操作：
1. 在文件末尾添加`auth required pam_google_authenticator.so`；
2. 註釋行`@include common-auth`；

最終內容如下

```bash
# Standard Un*x authentication.
#@include common-auth

# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so
```

* /etc/ssh/sshd_config

確保設置了如下指令

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**重要**： 因為是用key登錄,請務必添加指令`AuthenticationMethods`，否則配置不起作用，仍舊是驗證key後直接登錄。

具體操作命令
```bash
#UsePAM
sudo sed -i '/^UsePAM /d;' /etc/ssh/sshd_config
sudo bash -c 'echo "UsePAM yes" >> /etc/ssh/sshd_config'

#AuthenticationMethods
sudo sed -i '/^AuthenticationMethods /d' /etc/ssh/sshd_config
sudo bash -c 'echo "AuthenticationMethods publickey,password publickey,keyboard-interactive" >> /etc/ssh/sshd_config'

#ChallengeResponseAuthentication
sudo sed -i -r 's@(ChallengeResponseAuthentication) no@\1 yes@g' /etc/ssh/sshd_config
```

操作完成後，執行如下命令重啟`sshd`服務

```bash
sudo systemctl restart sshd
```

演示示例

```bash
[flying@lempstacker ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@lempstacker ~]$ ssh -C -c aes256-ctr flying@192.168.0.179
Authenticated with partial success.
Verification code:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms \for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Fri Jan 13 10:07:01 2017 from 192.168.0.140
flying@lempstacker:~$ sed -n -r '1s@.*"(.*)"$@\1@p' /etc/os-release
Debian GNU/Linux 8 (jessie)
flying@lempstacker:~$ cat .ssh/authorized_keys | wc -l
1
flying@lempstacker:~$ exit
logout
Connection to 192.168.0.179 closed.
[flying@lempstacker ~]$
```


### Debian With Password
此前使用密碼進行SSH連接(未執行`ssh-copy-id`)

* /etc/pam.d/sshd

默認文件末尾是

```
# Standard Un*x password updating.
@include common-password
```

在文件末尾添加`auth required pam_google_authenticator.so`；

**注意**：請勿註釋行`@include common-auth`、`@include common-password`；如果註釋行`@include common-auth`，會造成只驗證`Google Authenticator`的token即可登錄，不建議這麼操作。

最終內容如下

```bash
# Standard Un*x authentication.
@include common-auth

# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so
```


* /etc/ssh/sshd_config
確保設置了如下指令
```bash
UsePAM yes
ChallengeResponseAuthentication yes
```

**注意**： 不要設置指令`AuthenticationMethods`，否則會報錯
>Permission denied (publickey).


演示示例

```bash
[flying@lempstacker ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@lempstacker ~]$ ssh -C -c aes256-ctr flying@192.168.0.179
Password:
Verification code:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms \for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Fri Jan 13 10:29:11 2017 from 192.168.0.140
flying@lempstacker:~$ sed -n -r '1s@.*"(.*)"$@\1@p' /etc/os-release
Debian GNU/Linux 8 (jessie)
flying@lempstacker:~$ cat .ssh/authorized_keys | wc -l
0
flying@lempstacker:~$ exit
logout
Connection to 192.168.0.179 closed.
[flying@lempstacker ~]$
```

### OpenSUSE With SSH Key
此前使用key進行SSH連接

* /etc/pam.d/sshd

默認文件內容

```bash
#%PAM-1.0
auth        requisite   pam_nologin.so
auth        include     common-auth
account     requisite   pam_nologin.so
account     include     common-account
password    include     common-password
session     required    pam_loginuid.so
session     include     common-session
session  optional       pam_lastlog.so   silent noupdate showfailed
```

需要進行的操作：
1. 註釋行`auth        include     common-auth`；
2. 在該行之後添加`auth    required pam_google_authenticator.so nullok`；

最終內容如下

```bash
#%PAM-1.0
auth        requisite   pam_nologin.so
#auth        include     common-auth
auth       required     pam_google_authenticator.so
# auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator    # If SELinux Enabled
account     requisite   pam_nologin.so
account     include     common-account
password    include     common-password
session     required    pam_loginuid.so
session     include     common-session
session  optional       pam_lastlog.so   silent noupdate showfailed
```

* /etc/ssh/sshd_config

確保設置了如下指令

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**重要**： 因為是用key登錄,請務必添加指令`AuthenticationMethods`，否則配置不起作用，仍舊是驗證key後直接登錄。


### OpenSUSE With Password
此前使用密碼進行SSH連接(未執行`ssh-copy-id`)

* /etc/pam.d/sshd

默認文件內容

```bash
#%PAM-1.0
auth        requisite   pam_nologin.so
auth        include     common-auth
account     requisite   pam_nologin.so
account     include     common-account
password    include     common-password
session     required    pam_loginuid.so
session     include     common-session
session  optional       pam_lastlog.so   silent noupdate showfailed
```

在行`auth include common-auth`之後添加`auth required pam_google_authenticator.so`；

**注意**：請勿註釋行`common-auth`、`common-password`；如果註釋行`common-auth`，會造成只驗證`Google Authenticator`的token即可登錄，不建議這麼操作。

最終內容如下

```bash
#%PAM-1.0
auth        requisite   pam_nologin.so
auth        include     common-auth
auth 		required pam_google_authenticator.so nullok
account     requisite   pam_nologin.so
account     include     common-account
password    include     common-password
session     required    pam_loginuid.so
session     include     common-session
session  optional       pam_lastlog.so   silent noupdate showfailed
```


* /etc/ssh/sshd_config
確保設置了如下指令
```bash
UsePAM yes
ChallengeResponseAuthentication yes
```

**注意**： 不要設置指令`AuthenticationMethods`，否則會報錯
>Permission denied (publickey).

## Shell Script
Shell腳本託管在[GitHub](https://github.com/MaxdSre/axd-ShellScript/blob/master/assets/software/GoogleAuthenticator.sh)。

使用方式如下

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
curl -fsL https://raw.githubusercontent.com/MaxdSre/axd-ShellScript/master/assets/software/GoogleAuthenticator.sh | bash -s --
```

<!-- [![asciicast](https://asciinema.org/a/175735.png)](https://asciinema.org/a/175735?autoplay=1) -->

<script src="https://asciinema.org/a/175735.js" id="asciicast-175735" async></script>


## Error Occuring
編譯安裝過程中出現的報錯
```bash
#./bootstrap.sh 需要安裝 autoconf
./bootstrap.sh: line 15: exec: autoreconf: not found


#./bootstrap.sh http://ask.xmodulo.com/fix-failed-to-run-aclocal.html 需要安裝 automake
Can\'t exec "aclocal": No such file or directory at /usr/share/autoconf/Autom4te/FileUtils.pm line 326.
autoreconf: failed to run aclocal: No such file or directory


#./bootstrap.sh 需要安裝 libtool
configure.ac:8: installing 'build/install-sh'
configure.ac:8: installing 'build/missing'
Makefile.am:33: error: Libtool library used but 'LIBTOOL' is undefined
Makefile.am:33:   The usual way to define 'LIBTOOL' is to add 'LT_INIT'
Makefile.am:33:   to 'configure.ac' and run 'aclocal' and 'autoconf' again.
Makefile.am:33:   If 'LT_INIT' is in 'configure.ac', make sure
Makefile.am:33:   its definition is in aclocal\'s search path.
Makefile.am: installing 'build/depcomp'
parallel-tests: installing 'build/test-driver'
autoreconf: automake failed with exit status: 1


#./configure 需要安裝 pam-devel
configure: error: Unable to find the PAM library or the PAM header files
```


## References
* [2-Step Verification](https://support.google.com/accounts/topic/28786?hl=en 'Google')
* [Trying to get SSH with public key (no password) + google authenticator working on Ubuntu 14.04.1](https://serverfault.com/questions/629883/trying-to-get-ssh-with-public-key-no-password-google-authenticator-working-o)
* [How To Set Up Multi-Factor Authentication for SSH on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04 'Digital Ocean')
* [Secure Your Linux Desktop and SSH Login Using Two Factor Google Authenticator](http://www.cyberciti.biz/open-source/howto-protect-linux-ssh-login-with-google-authenticator/)
* [Secure SSH with Google Authenticator Two-Factor Authentication on CentOS 7](https://www.howtoforge.com/tutorial/secure-ssh-with-google-authenticator-on-centos-7/)
* [How To Set Up Multi-Factor Authentication for SSH on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-centos-7 'Digital Ocean')
* [Dual factor SSH: Google Authenticator, SElinux, and CentOS](http://www.untrustedconnection.com/2013/06/dual-factor-ssh-google-authenticator.html)

## Bibliography
* [Gnome Keyring](http://www.nurdletech.com/linux-notes/agents/keyring.html)


## Change Logs
* 2016.01.15 21:00 Mon Asia/Beijing
    * 初稿完成
* 2017.01.12 18:41 Thu Asia/Shanghai
    * 內容重構，增加各種應用場景的具體配置
* 2017.01.13 10:41 Fri Asis/Shanghai
    * 增加SSH在Debian中的認證配置
* 2017.07.17 16:45 Mon Asia/Shanghai
    * 添加對`OpenSUSE Leap`的認證配置
* 2018.03.21 11:24 Wed America/Boston
    * 添加SUSE，添加SELinux啓用情況下的處理
* 2018-04-11 23:00 Wed America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
