---
title: Using Enigmail To Secure Mozilla Thunderbird On GNU/Linux
slug: Using Enigmail To Secure Mozilla Thunderbird On GNU Linux
date: 2016-11-22T02:42:00+08:00
lastmod: 2019-04-29T16:45:20-04:00
draft: false
keywords: ["Thunderbird", "OpenPGP", "Enigmail"]
description: "Using Enigmail To Secure Mozilla Thunderbird On GNU/Linux"
categories:
- Security
tags:
- Thunderbird
- OpenPGP
- Enigmail

comment: true
toc: true

---

[Thunderbird][thunderbird]是一款免費、開源、跨平臺的郵件客戶端，由Mozilla基金會開發。[Enigmail][enigmail]是與[Thunderbird][thunderbird]無縫集成的安全性擴展，允許用戶使用[OpenPGP][openpgp]對郵件進行加密、解密、添加數字簽名等操作。

本文記錄在[GNOME][gnome]桌面下安裝、配置[Thunderbird][thunderbird]、[Enigmail][enigmail]的過程。

<!--more-->

**注意**： 郵件收發雙方必須同時使用`Thunderbird`並配置`Enigmail`，否則對方接收到的郵件只是2個附件文件，無法正常顯示郵件內容。

本文截圖較少，如果要查看詳細的圖文教程，可參閱 [THUNDERBIRD, ENIGMAIL AND OPENPGP FOR LINUX - SECURE EMAIL](https://securityinabox.org/en/guide/thunderbird/linux)。

## Preparation
### OS Info

item|details
---|---
OS Version|Debian GNU/Linux 8 (jessie)
Kernel Version|3.16.0-4-amd64
Gnome Version|GNOME Shell 3.14.4
GnuPG Version|gpg (GnuPG) 1.4.18
GnuPG2 Version|gpg (GnuPG) 2.0.26

Gnome版本信息通過命令`gnome-shell --version`獲取。

**重要** 在某些GNOME版本中，Enigmail會出現生成密鑰對失敗的情況，報錯信息如下

>gpg: WARNING: The GNOME keyring manager hijacked the GnuPG agent.
gpg: WARNING: GnuPG will not work properly - please configure that tool to not interfere with the GnuPG system!

提示`GnuPG agent`被`GNOME keyring manager`劫持。

解決方案見GnuPG官方的 [Problem](https://wiki.gnupg.org/GnomeKeyring 'GnuPG')。

在Debian中，執行如下命令

```bash
#此為一行命令
sudo dpkg-divert --local --rename --divert \ /etc/xdg/autostart/gnome-keyring-gpg.desktop-disable --add \ /etc/xdg/autostart/gnome-keyring-gpg.desktop
```

若要復原，執行如下命令

```bash
sudo dpkg-divert --rename --remove /etc/xdg/autostart/gnome-keyring-gpg.desktop
```

### Download Package
當前最新釋出版本是`45.6.0`，釋出時間`Dec 28, 2016`，下載[頁面](https://www.mozilla.org/en-US/thunderbird/all/)。

根據實際情況選擇對應語言版本，此處選擇 **English(US)**。

```bash
#鏈接
https://download.mozilla.org/?product=thunderbird-45.6.0-SSL&os=linux64&lang=en-US

#重定向後的下載地址
https://download-installer.cdn.mozilla.net/pub/thunderbird/releases/45.6.0/linux-x86_64/en-US/thunderbird-45.6.0.tar.bz2
```

當前文件名稱爲 `thunderbird-45.6.0.tar.bz2`。

文件下載路徑爲`~/Downloads/`

```bash
$ namei -o ~/Downloads/thunderbird-45.6.0.tar.bz2
f: /home/flying/Downloads/thunderbird-45.6.0.tar.bz2
 d root   root   /
 d root   root   home
 d flying flying flying
 d flying flying Downloads
 - flying flying thunderbird-45.6.0.tar.bz2
$
```

## Uncompression & Extraction
將軟件包解壓至目錄`/opt`，子目錄爲`/opt/thunderbird`

執行如下命令

```bash
# 創建目錄
[[ -d '/opt/thunderbird' ]] && sudo rm -rf /opt/thunderbird/* || sudo mkdir -pv /opt/thunderbird

# 解壓縮
sudo tar xf ~/Downloads/thunderbird-45.6.0.tar.bz2 -C /opt/thunderbird --strip-components=1
```

### thunderbird.desktop
爲Thunderbird在GNOME中創建圖標

執行如下命令
```bash
# 複製圖片Logo
[[ -f '/usr/share/pixmaps/thunderbird.png' ]] && sudo rm -f /usr/share/pixmaps/thunderbird.png
sudo cp /opt/thunderbird/chrome/icons/default/default256.png /usr/share/pixmaps/thunderbird.png

# 創建Desktop
sudo tee /usr/share/applications/thunderbird.desktop <<-'EOF'
[Desktop Entry]
Encoding=UTF-8
Name=Thunderbird
GenericName=Email
Comment=Send and Receive Email
#TryExec=thunderbird
Exec=/opt/thunderbird/thunderbird %u
Icon=thunderbird.png
Terminal=false
Type=Application
MimeType=message/rfc822;x-scheme-handler/mailto;
StartupNotify=true
Categories=Network;Email;Application;
EOF
```

操作完成後，在GNOME桌面左上角依次點擊`Application`-->`Internet`-->`Thunderbird`，即可打開`Mozilla Thunderbird`客戶端。

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/1.overview-2017-01-11_20:12:49.png)


## Add Email Accounts
此處以本人Hotmail、Gmail郵箱為例

Email|Master Password|Enigmail Passphrase
---|---|---
lempstacker@hotmail.com|`hotmail@Master`|`hotmail@Enigmail`
lempstacker@gmail.com|`gmail@Master`|`gmail@Enigmail`

鼠標在頁面標籤欄(Tab)中右擊點選`Menu Bar`調出菜單欄。

依次點擊`Edit`-->`Account Settings`，在彈框左下方的`Account Actions`中選擇`Add Mail Account`，依次添加這兩個郵箱賬號。

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/2.emailaccount-2017-01-11_20:21:24.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/2.emailaccount-2017-01-11_20:21:42.png)


## Enigmail Configuration
依次點擊`Tools`-->`Add-ons`，在搜索框中輸入`Enigmail`搜索，安裝完成後點擊藍色的`Restart Now`重啟Thunderbird客戶端。

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/3.enigmailInstation-2017-01-11_20:23:01.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/3.enigmailInstation-2017-01-11_20:23:11.png)

重啟後，自動跳出 *Enigmail Setup Wizard* 彈框。也可手動點擊`Enigmail`-->`Setup Wizard`進行設置。

此處需要大量隨機數資源，可通過安裝`rng-tools`和`haveged`解決。
<!-- ，具體參見本人Blog [Use Haveged & rng-tools To Speed Up Entropy For Random Number Generation On GNU/Linux](https://lempstacker.com/tw/Use-Haveged-rng-tools-To-Speed-Up-Entropy-For-Random-Number-Generation-On-GNU-Linux/)。 -->

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/4.revocateError-2017-01-11_20:34:51.png)

出現報錯

>The revocation certificate could not be created.
>GnuPG reported an error in the communication with gpg-agent (a component of GnuPG).
>This is a system setup or configuration error that prevents Enigmail from working properly and cannot be fixed automatically.
>We strongly recommend that you consult our support web site at https://enigmail.net/faq.


解決方案上文已經說明，Debian中執行

```bash
sudo dpkg-divert --local --rename --divert /etc/xdg/autostart/gnome-keyring-gpg.desktop-disable --add /etc/xdg/autostart/gnome-keyring-gpg.desktop
```

執行後須重啟操作系統。

操作過程

```bash
$ sudo dpkg-divert --local --rename --divert /etc/xdg/autostart/gnome-keyring-gpg.desktop-disable --add /etc/xdg/autostart/gnome-keyring-gpg.desktop
Adding 'local diversion of /etc/xdg/autostart/gnome-keyring-gpg.desktop to /etc/xdg/autostart/gnome-keyring-gpg.desktop-disable'
$
```

依次點擊`Enigmail`-->`Key Managerment`，將其中的key刪除，鼠標右擊。點擊`Enigmail`-->`Setup Wizard`進行設置。如果操作成功，彈框頁面如下

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/6.keygenerate-2017-01-11_20:47:15.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/5.revocateSuccess-2017-01-11_20:47:05.png)


## Testing
點擊`Write`寫新郵件，有紅色標紅文字
>This message will be unsigned and unencrypted

點擊`Attach My Public Key`，郵件發送後。接收方看到的內容

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/7.enigmailencrypt-2017-01-11_20:59:21.png)

直接在瀏覽器中打開，只有2個附件文件，無法顯示內容

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-11-thunderbird-enigmail/8.gmail-2017-01-11_21:23:37.png)


## References
* [THUNDERBIRD, ENIGMAIL AND OPENPGP FOR LINUX - SECURE EMAIL](https://securityinabox.org/en/guide/thunderbird/linux)
* [Configuring Certificates](https://support.mozilla.org/en-US/kb/configuring-certificates 'Mozilla')


## Bibliography
* [Problem - GnuPG](https://wiki.gnupg.org/GnomeKeyring 'GnuPG')
* [Gnome Keyring](http://www.nurdletech.com/linux-notes/agents/keyring.html)
* [GPG-agent communication error](https://sourceforge.net/p/enigmail/forum/support/thread/f99ae0fa/)
* [Encrypting and/or signing my emails fails since upgrade to 1.9.0](https://admin.hostpoint.ch/pipermail/enigmail-users_enigmail.net/2016-February/003674.html)
* [Troubleshooting - Enigmail](https://www.enigmail.net/index.php/en/faq?view=topic&id=14)


## Change Logs
* 2016.11.22 10:13 Tue Asia/Shanghai
    * 初稿完成
* 2017.01.11 21:23 Wed Asia/Shanghai
    * 內容重構，添加`Enigmail`配置
* 2019.04.29 16:41 Mon America/Boston
    * 勘誤，遷移到新Blog


[thunderbird]:https://www.thunderbird.net
[enigmail]:https://www.enigmail.net/index.php/en/
[gnome]:https://www.gnome.org
[openpgp]:https://www.openpgp.org


<!-- End -->
