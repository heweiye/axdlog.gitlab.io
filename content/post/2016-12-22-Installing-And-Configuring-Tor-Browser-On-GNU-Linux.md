---
title: Installing And Configuring Tor Browser On GNU/Linux
slug: Installing And Configuring Tor Browser On GNU Linux
date: 2016-12-22T18:45:24+08:00
lastmod: 2018-04-11T11:55:08-04:00
draft: false
keywords: ["AxdLog", "Tor", "Tor browser", "GPG"]
description: "How to use shell script to install and configure Tor Browser on GNU/Linux"
categories:
- GNU/Linux
tags:
- Tor
- GPG
comment: true
toc: true
---

[Tor Browser][torbrowser]是由[Tor Project](https://www.torproject.org)推出的一款集成了[Tor](https://www.torproject.org/)，[Firefox ESR](https://www.mozilla.org/en-US/firefox/organizations/), [Torbutton](https://www.torproject.org/docs/torbutton/), [TorLauncher](https://github.com/micahflee/torbrowser-launcher), [NoScript](https://noscript.net),和 [HTTPS-Everywhere](https://www.eff.org/https-everywhere)的瀏覽器。[Tor Browser][torbrowser]默認監聽`9150`、`9151`端口，通過[Tor (anonymity network)](https://en.wikipedia.org/wiki/Tor_(anonymity_network) 'WikiPedia')實現匿名訪問，保障用戶隱私。

本文記錄如何在GNU/Linux中下載、校驗、安裝[Tor Browser][torbrowser]及在[GNOME][gnome]桌面中創建快捷圖標。

<!--more-->

>The Tor software protects you by bouncing your communications around a distributed network of relays run by volunteers all around the world: it prevents somebody watching your Internet connection from learning what sites you visit, it prevents the sites you visit from learning your physical location, and it lets you access sites which are blocked. -- <https://www.torproject.org/projects/torbrowser.html.en>


## OS Info
主機操作系統信息如下

item|detail
---|---
OS Version|`Debian GNU/Linux 9.4 (stretch)`
Kernel Version|`4.9.0-5-amd64`


## Downloading && Verification
`Tor Browser`的下載頁面在[Tor Browser Downloads](https://www.torproject.org/projects/torbrowser.html#downloads 'Tor')，GPG簽名的校驗方法在[Verify package signatures](https://www.torproject.org/docs/verifying-signatures.html.en 'How to verify signatures for packages')。Release信息可在其官方blog中查看[鏈接](https://blog.torproject.org/category/tags/tor-browser)。

### Download
官網當前提供的下載版本是`7.5.3`(Apr 11, 2018)，release信息見[ Tor Browser 7.5.3 is released](https://blog.torproject.org/tor-browser-753-released)。

下載鏈接

```bash
https://www.torproject.org/dist/torbrowser/7.5.3/tor-browser-linux64-7.5.3_en-US.tar.xz

# GPG Signature File
https://www.torproject.org/dist/torbrowser/7.5.3/tor-browser-linux64-7.5.3_en-US.tar.xz.asc
```

此處定義下載路徑`~/Downloads`

### Importing GPG Public Key
下載完成後進行校驗操作，參考官方文檔 [Verify package signatures](https://www.torproject.org/docs/verifying-signatures.html.en 'How to verify signatures for packages')。

**注意**：必須在操作系統中導入指定的公鑰，否則無法進行校驗工作

KeyId是`0x4E2C6E8793298290`

執行如下命令安裝公鑰
```bash
#列出本機中的公鑰
gpg --list-keys
gpg --list-key 0x4E2C6E8793298290

#在keyserver中查詢指定的公鑰
gpg --keyserver keys.gnupg.net --search-keys 0x4E2C6E8793298290

#從keyserver下載指定的公鑰
gpg --keyserver keys.gnupg.net --recv-keys 0x4E2C6E8793298290

#查看公鑰及其指紋
gpg --fingerprint 0x4E2C6E8793298290
```

具體操作過程
```bash
#列出本機中的公鑰
[maxdsre@Stretch ~]$ gpg --list-key 0x4E2C6E8793298290
gpg: error reading key: No public key

#在keyserver中查詢公鑰
[maxdsre@Stretch ~]$ gpg --keyserver keys.gnupg.net --search-keys 0x4E2C6E8793298290
gpg: searching for "0x4E2C6E8793298290" from hkp server keys.gnupg.net
(1)	Tor Browser Developers (signing key) <torbrowser@torproject.org>
	  4096 bit RSA key 93298290, created: 2014-12-15, expires: 2020-08-24
Keys 1-1 of 1 for "0x4E2C6E8793298290".  Enter number(s), N)ext, or Q)uit > 1
gpg: requesting key 93298290 from hkp server keys.gnupg.net
gpg: key 93298290: public key "Tor Browser Developers (signing key) <torbrowser@torproject.org>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)

#從keyserver下載公鑰
[maxdsre@Stretch ~]$ gpg --keyserver keys.gnupg.net --recv-keys 0x4E2C6E8793298290
gpg: requesting key 93298290 from hkp server keys.gnupg.net
gpg: key 93298290: "Tor Browser Developers (signing key) <torbrowser@torproject.org>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
[maxdsre@Stretch ~]$ gpg --list-key 0x4E2C6E8793298290
pub   4096R/93298290 2014-12-15 [expires: 2020-08-24]
uid                  Tor Browser Developers (signing key) <torbrowser@torproject.org>
sub   4096R/F65C2036 2014-12-15 [expires: 2017-08-25]
sub   4096R/D40814E0 2014-12-15 [expires: 2017-08-25]
sub   4096R/C3C07136 2016-08-24 [expires: 2018-08-24]

#查看公鑰及其指紋
[maxdsre@Stretch ~]$ gpg --fingerprint 0x4E2C6E8793298290
pub   4096R/93298290 2014-12-15 [expires: 2020-08-24]
      Key fingerprint = EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
uid                  Tor Browser Developers (signing key) <torbrowser@torproject.org>
sub   4096R/F65C2036 2014-12-15 [expires: 2017-08-25]
sub   4096R/D40814E0 2014-12-15 [expires: 2017-08-25]
sub   4096R/C3C07136 2016-08-24 [expires: 2018-08-24]

[maxdsre@Stretch ~]$
```

### Verifying GPG Signature
安裝公鑰後進行校驗，使用命令`gpg --verify`進行校驗

操作過程如下

```bash
#列出文件
[maxdsre@Stretch ~]$ ls -lhF ~/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz*
-rw-r--r-- 1 maxdsre maxdsre 68M Feb  3 14:54 /home/maxdsre/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz
-rw-r--r-- 1 maxdsre maxdsre 801 Feb  3 14:50 /home/maxdsre/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz.asc

#校驗 校驗文件在前，源文件在後
[maxdsre@Stretch ~]$ gpg --verify ~/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz{.asc,}
gpg: Signature made Tue 24 Jan 2017 09:42:49 AM EST using RSA key ID C3C07136
gpg: Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: EF6E 286D DA85 EA2A 4BA7  DE68 4E2C 6E87 9329 8290
     Subkey fingerprint: A430 0A6B C93C 0877 A445  1486 D148 3FA6 C3C0 7136
[maxdsre@Stretch ~]$
```

校驗結果中出現
>gpg: Good signature from "Tor Browser Developers (signing key) <torbrowser@torproject.org>"

說明校驗成功，證明下載的文件是由Tor官方釋出，未經過第三方篡改，可以信任。


## Installation
`Tor Browser`的安裝可參考官方文檔 [Tor Installation guides](https://www.torproject.org/docs/installguide.html.en 'Tor')。

此處定義安裝路徑`/opt/torBrowser`，執行如下命令進行解壓
```bash
#創建目標路徑
[[ ! -d /opt/torBrowser ]] && sudo mkdir -m 755 -pv /opt/torBrowser || sudo rm -rf /opt/torBrowser/*

#解壓壓縮包到目標路徑
sudo tar xf ~/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz -C /opt/torBrowser --strip-components=1
```

解壓完成後，進入目標路徑，執行
```bash
./start-tor-browser.desktop
```
出現
>Launching './Browser/start-tor-browser --detach'...

即可啓動`Tor Browser`。

操作過程如下
```bash
[maxdsre@Stretch ~]$ [[ ! -d /opt/torBrowser ]] && sudo mkdir -pv /opt/torBrowser
mkdir: created directory ‘/opt/torBrowser’
[maxdsre@Stretch ~]$ sudo tar xf ~/Downloads/tor-browser-linux64-7.5.3_en-US.tar.xz -C /opt/torBrowser --strip-components=1
[maxdsre@Stretch ~]$ cd /opt/torBrowser/
[maxdsre@Stretch torBrowser]$ ls -lhF
total 8.0K
drwx------ 11 maxdsre root   4.0K Apr 11 12:22 Browser/
-rwx------  1 maxdsre maxdsre 1.7K Apr 11 12:21 start-tor-browser.desktop*

[maxdsre@Stretch torBrowser]$ ls -lhF Browser/
total 95M
-rw------- 1 maxdsre root    440 Dec 31  1999 application.ini
drwx------ 6 maxdsre root    126 Dec 31  1999 browser/
-rw------- 1 maxdsre root      0 Dec 31  1999 chrome.manifest
drwx------ 3 maxdsre root     18 Dec 31  1999 defaults/
-rw------- 1 maxdsre root    144 Dec 31  1999 dependentlibs.list
drwxr-xr-x 2 maxdsre maxdsre    6 Apr 11 12:21 Desktop/
drwx------ 2 maxdsre root     40 Dec 31  1999 dictionaries/
-rwx------ 1 maxdsre root    284 Dec 31  1999 execdesktop*
-rwx------ 1 maxdsre root   147K Dec 31  1999 firefox*
drwx------ 2 maxdsre root   4.0K Dec 31  1999 fonts/
drwx------ 2 maxdsre root     25 Dec 31  1999 icons/
-rw------- 1 maxdsre root    11M Dec 31  1999 icudt58l.dat
-rwx------ 1 maxdsre root   507K Dec 31  1999 libfreeblpriv3.so*
-rwx------ 1 maxdsre root   121K Dec 31  1999 liblgpllibs.so*
-rwx------ 1 maxdsre root   1.6M Dec 31  1999 libmozavcodec.so*
-rwx------ 1 maxdsre root   207K Dec 31  1999 libmozavutil.so*
-rwx------ 1 maxdsre root   163K Dec 31  1999 libmozsandbox.so*
-rwx------ 1 maxdsre root   674K Dec 31  1999 libmozsqlite3.so*
-rwx------ 1 maxdsre root   213K Dec 31  1999 libnspr4.so*
-rwx------ 1 maxdsre root   978K Dec 31  1999 libnss3.so*
-rwx------ 1 maxdsre root   625K Dec 31  1999 libnssckbi.so*
-rwx------ 1 maxdsre root   119K Dec 31  1999 libnssdbm3.so*
-rwx------ 1 maxdsre root   167K Dec 31  1999 libnssutil3.so*
-rwx------ 1 maxdsre root    19K Dec 31  1999 libplc4.so*
-rwx------ 1 maxdsre root    15K Dec 31  1999 libplds4.so*
-rwx------ 1 maxdsre root   139K Dec 31  1999 libsmime3.so*
-rwx------ 1 maxdsre root   220K Dec 31  1999 libsoftokn3.so*
-rwx------ 1 maxdsre root   259K Dec 31  1999 libssl3.so*
-rwx------ 1 maxdsre root    73M Dec 31  1999 libxul.so*
-rw------- 1 maxdsre root   5.6M Dec 31  1999 omni.ja
-rw------- 1 maxdsre root     48 Dec 31  1999 platform.ini
-rwx------ 1 maxdsre root   131K Dec 31  1999 plugin-container*
-rw------- 1 maxdsre root    99K Dec 31  1999 precomplete
-rw------- 1 maxdsre root    707 Dec 31  1999 removed-files
-rwx------ 1 maxdsre root   8.8K Dec 31  1999 run-mozilla.sh*
-rwx------ 1 maxdsre root    13K Dec 31  1999 start-tor-browser*
-rwx------ 1 maxdsre root   1.7K Dec 31  1999 start-tor-browser.desktop*
drwx------ 6 maxdsre root     59 Apr 11 12:22 TorBrowser/
-rwx------ 1 maxdsre root   126K Dec 31  1999 updater*
-rw------- 1 maxdsre root    689 Dec 31  1999 updater.ini
-rw------- 1 maxdsre root    138 Dec 31  1999 update-settings.ini

[maxdsre@Stretch torBrowser]$
```

## Create Desktop For GNOME
如果要在GNOME中爲Tor Browser創建快捷圖標，可通過在目錄`/usr/share/applications/`中創建`.desktop`文件實現。

### start-tor-browser.desktop
查看啓動腳本`start-tor-browser.desktop`，內容如下

```bash
#!/usr/bin/env ./Browser/execdesktop
#
# This file is a self-modifying .desktop file that can be run from the shell.
# It preserves arguments and environment for the start-tor-browser script.
#
# Run './start-tor-browser.desktop --help' to display the full set of options.
#
# When invoked from the shell, this file must always be in a Tor Browser root
# directory. When run from the file manager or desktop GUI, it is relocatable.
#
# After first invocation, it will update itself with the absolute path to the
# current TBB location, to support relocation of this .desktop file for GUI
# invocation. You can also add Tor Browser to your desktop's application menu
# by running './start-tor-browser.desktop --register-app'
#
# If you use --register-app, and then relocate your TBB directory, Tor Browser
# will no longer launch from your desktop's app launcher/dock. However, if you
# re-run --register-app from inside that new directory, the script
# will correct the absolute paths and re-register itself.
#
# This file will also still function if the path changes when TBB is used as a
# portable app, so long as it is run directly from that new directory, either
# via the shell or via the file manager.

[Desktop Entry]
Type=Application
Name=Tor Browser
GenericName=Web Browser
Comment=Tor Browser is +1 for privacy and -1 for mass surveillance
Categories=Network;WebBrowser;Security;
Exec=sh -c '"/opt/torBrowser/Browser/start-tor-browser" --detach || ([ !  -x "/opt/torBrowser/Browser/start-tor-browser" ] && "$(dirname "$*")"/Browser/start-tor-browser --detach)' dummy %k
X-TorBrowser-ExecShell=./Browser/start-tor-browser --detach
Icon=/opt/torBrowser/Browser/browser/icons/mozicon128.png
StartupWMClass=Tor Browser
```

根據該文件中的內容創建定製化的`.desktop`文件

### Custom Create torbrowser.desktop
Tor Browser的logo圖片可下如下路徑中找到
```bash
#cd /opt/torBrowser/
./Browser/browser/chrome/icons/default/
./Browser/browser/icons/mozicon128.png
```

須將logo圖片複製或創建符號鏈接至路徑`/usr/share/pixmaps/`中

執行如下命令創建定製化的`.desktop`文件

```bash
[[ -f /usr/share/pixmaps/torbrowser.png ]] && sudo rm -f /usr/share/pixmaps/torbrowser.png
sudo cp -a /opt/torBrowser/Browser/browser/chrome/icons/default/default48.png /usr/share/pixmaps/torbrowser.png

sudo tee /usr/share/applications/torbrowser.desktop <<-'EOF'
[Desktop Entry]
Encoding=UTF-8
Name=Tor Browser
GenericName[en]=Web Browser
Comment=Tor Browser is +1 for privacy and -1 for mass surveillance
Type=Application
Categories=Network;WebBrowser;Security;
Exec=sh -c '/opt/torBrowser/Browser/start-tor-browser --detach' dummy %k
X-TorBrowser-ExecShell=/opt/torBrowser/Browser/start-tor-browser --detach
Icon=TorBrowser.png
Terminal=false
StartupWMClass=Tor Browser
MimeType=text/html;text/xml;application/xhtml+xml;application/vnd.mozilla.xul+xml;text/mml;
EOF
```

在 *Applications* -> *Internet* 中即可看到Tor Browser的圖標，點擊該圖標可正常啓動Tor Browser。

連接到Tor Network需要一段時間，請耐心等待。

連接成功後，可通過如下地址檢測Tor瀏覽器是否正常工作

```http
https://check.torproject.org/
```

## Setting Custom Bridge
在 **Tor Bridges Configuration** 設置過程中，可配置`custom bridges`。具體解釋說明見官方文檔 [Tor: Bridges](https://www.torproject.org/docs/bridges.html.en)。

`custom bridges`可通過如下鏈接獲取

```http
https://bridges.torproject.org

https://bridges.torproject.org/options
```

也可通過給<bridges@bridges.torproject.org>發送題爲 **get bridges** 的郵件獲取，但只支持Gmail，Riseup，Yahoo。

>You can also get bridges by sending mail to *bridges@bridges.torproject.org* with the line "get bridges" by itself in the body of the mail. You'll need to send this request from a Gmail, Riseup!, or Yahoo! account, though — we only accept these providers because otherwise we make it too easy for an attacker to make a lot of email addresses and learn about all the bridges.  -- <https://www.torproject.org/docs/bridges.html.en#FindingMore>

獲取到的`custom bridges`格式如下

```bash
#type 1
23.229.18.162:8584 107316EC0170228DADE8F5FD521B4987FB0C93A8
13.54.46.194:8443 A9B168022DDF6B0AA7D728CD233C6E42617A58C0
91.219.68.181:443 53935564E40B358BADDF9169ABBDEDB237C7EA59

#type2
obfs4 89.177.136.11:45423 9CD55E88DA7B9D861EEFB6156D74E369BF964C70 cert=LjPVCVAsil6PFjaa8/VbfcMQ90I33NN5H1KIgVgSbdp1zICsX6LG8PTD9Q+21/yIYYq8Qg iat-mode=0
obfs4 37.218.246.32:33081 B9B43048D75BF880DCC25606B0E10CFCDEDCB161 cert=ZnlWh5pqbNzc4HOON0rDQ+t+6a42FustCCJx45iKRLTLl/JKytbyK4ccQItjoZ86yV2MMA iat-mode=0
obfs4 185.101.218.42:41758 E3BBFDD00B287113C7A1EB2A2C2EC1D9D6A80218 cert=ePT0ASv45frcpQ9pSzQ3Ae/wNMufga/x5jGdqdZpLGLbHdYKm3s5QQGhPkQKivRwz/Csfg iat-mode=0
```

在如下文本框中填寫即可

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/3.tor-bridge-configuration-2016-12-22_17-54-44.png)


## Snapshots About Anonymous Connection
此處使用本地代理連接成功，以下是操作過程截圖

### Network Setting
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/1.tor-network-setting-2016-12-22_17-54-16.png)

### Tor Bridge Connection
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/2.tor-bridge-configuration-2016-12-22_17-54-35.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/3.tor-bridge-configuration-2016-12-22_17-54-44.png)

### Local Proxy Connection
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/4.local-proxy-configuration-2016-12-22_17-54-56.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/5.local-proxy-configuration-2016-12-22_17-55-08.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/6.local-proxy-configuration-2016-12-22_17-55-23.png)

### Connecting To The Tor Network
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/7.establishing-encrypted-connection-2016-12-22_17-55-29.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/8.retrieving-network-status-2016-12-22_17-55-32.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/9.loading-network-status-2016-12-22_17-55-33.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/10.establishing-a-tor-circuit-2016-12-22_17-55-36.png)

### Welcome Page
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/11.welcome-page-2016-12-22_17-55-56.png)

### IP Status
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/12.ip-check-2016-12-22_17-57-34.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-12-22_Tor_Borwser_Installation/13.ip-check2016-12-22_18-06-16.png)


## References
* [How to add Tor Browser to my menu/launcher for regular use?](http://askubuntu.com/questions/348777/how-to-add-tor-browser-to-my-menu-launcher-for-regular-use 'askubuntu')


## Bibliography
* [TOR BROWSER FOR LINUX - ONLINE ANONYMITY AND CIRCUMVENTION](https://securityinabox.org/en/guide/torbrowser/linux 'security in-a-box')


[gnome]: https://www.gnome.org
[torbrowser]: https://www.torproject.org/projects/torbrowser.html.en

## Change Logs
* 2016.12.22 17:24 Thu Asia/Shanghai
    * 初稿完成
* 2017.02.03 15:02 Fri America/Boston
	* 添加`custom bridges`配置
* 2018.04.11 11:55 Wed America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
