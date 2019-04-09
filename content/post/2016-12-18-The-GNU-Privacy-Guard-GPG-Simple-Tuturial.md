---
title: The GNU Privacy Guard(GPG) Simple Usage Tuturial
slug: The GPG Simple Usage Tuturial
date: 2016-12-18T19:55:23+08:00
lastmod: 2018-08-01T22:25:08+08:00
draft: false
keywords: ["GPG"]
description: "The GNU Privacy Guard(GPG) Simple Tuturial"
categories:
- Security
tags:
- GnuPG
- GPG
comment: true
toc: true

---


`GnuPG`是由GNU組織開發的一款免費的加密軟件，採用`OpenPGP`標準([RFC4880](https://tools.ietf.org/html/rfc4880))，可用於數據、通信的加密和簽名。本文主要介紹的是[GnuPG](https://www.gnupg.org/ 'The GNU Privacy Guard')的簡單使用，如生成公鑰/私鑰對、列出公鑰、顯示公鑰指紋、生成Revoke Key、與keyserver交互、導出公鑰/私鑰、導入公鑰、文件加密/解密、文件簽名及校驗、刪除公鑰/私鑰、吊銷keyserver中的公鑰等。

<!--more-->

具體使用實例見

* MySQL - [2.1.3.2 Signature Checking Using GnuPG](http://dev.mysql.com/doc/refman/5.7/en/checking-gpg-signature.html 'MySQL 5.7')
* Nginx - [Updating the GPG Key for NGINX Products](https://www.nginx.com/blog/updating-gpg-key-nginx-products/ 'Nginx')


## Introduction

以下是`WikiPedia`對其的介紹

* **PGP**: [Pretty Good Privacy](https://en.wikipedia.org/wiki/Pretty_Good_Privacy 'WikiPedia')
* **GPG**/**GnuPG**: [GNU Privacy Guard](https://en.wikipedia.org/wiki/GNU_Privacy_Guard 'WikiPedia')

>`GNU Privacy Guard` (GnuPG or GPG) is a *free software* replacement for Symantec's PGP cryptographic software suite.

>`GnuPG` is a complete and free implementation of the OpenPGP standard as defined by RFC4880 (also known as PGP). `GnuPG` allows to encrypt and sign your data and communication, features a versatile key management system as well as access modules for all kinds of public key directories. `GnuPG`, also known as GPG, is a command line tool with features for easy integration with other applications. A wealth of frontend applications and libraries are available. *Version 2 of GnuPG also provides support for S/MIME and Secure Shell (ssh)*.

>`Pretty Good Privacy` (PGP) is an *encryption program* that provides cryptographic privacy and authentication for data communication.


## Preparation

```bash
maxdsre@lemp:~$ gpg --version
gpg (GnuPG) 1.4.18
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: ~/.gnupg
Supported algorithms:
Pubkey: RSA, RSA-E, RSA-S, ELG-E, DSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: MD5, SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
maxdsre@lemp:~$
```

其中的`Cipher`可在參數`--cipher-algo`中指定，NASA的建議是使用
`AES256`，具體見[鏈接](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html)。

也可在文件`~/.gnupg/gpg.conf`中添加`cipher-algo AES256`，可直接執行如下命令

```bash
[[ `sed -r -n '/^cipher-algo/p' ~/.gnupg/gpg.conf` == '' ]] && sudo sed -i -r '$a\cipher-algo AES256' ~/.gnupg/gpg.conf || sudo sed -i -r '/^cipher-algo/ s@cipher-algo.*@cipher-algo AES256@g' ~/.gnupg/gpg.conf
```

關於`--cipher-algo`

>`--cipher-algo name`
Use name as cipher algorithm. Running the program with the  command --version yields a list of supported algorithms. If this is not used the cipher algorithm is selected from  the  preferences
stored  with  the  key.  In general, you do not want to use this option as it allows you to violate the OpenPGP standard.  `--per‐sonal-cipher-preferences` is the safe way to accomplish the same
thing. -- `man gpg`


## Generate A New KeyPair
執行如下命令生成密鑰對(KeyPair)

```bash
gpg --gen-key
```

此處的配置信息如下

item|value
---|---
keypair type|`1`
keypair size|`4096`
keypair exipre|`1y`
Real name|`sunday`
Email|`sunday@test.com`
Comment|`sunday test`
passphrase|`ahappyday`

具體操作過程

```txt
maxdsre@lemp:~$ gpg --gen-key
gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Mon 18 Dec 2017 04:51:26 PM CST
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: sunday
Email address: sunday@test.com
Comment: sunday test
You selected this USER-ID:
    "sunday (sunday test) <sunday@test.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
You need a Passphrase to protect your secret key.

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 187 more bytes)
.............+++++

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 190 more bytes)
....+++++
gpg: key 26B15903 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2017-12-18
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
      Key fingerprint = 23A6 D526 005F 1A87 94B0  51A1 2411 FE8C 26B1 5903
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

maxdsre@lemp:~$

```

此處`key_ID`是`26B15903`。

### Attentions
注意事項

#### Entropy Generation
該過程需要大量隨機數資源，如果系統無法提供足夠的隨機數資源，會導致執行過程耗時過長，且會對生成的 *密鑰對* 的安全性產生影響。可通過安裝`rng-tools`和`haveged`解決，具體參見本人Blog [Use Haveged & rng-tools To Speed Up Entropy For Random Number Generation On GNU/Linux](https://lempstacker.com/tw/Use-Haveged-rng-tools-To-Speed-Up-Entropy-For-Random-Number-Generation-On-GNU-Linux/)。

#### The GNOME keyring manager hijacked the GnuPG agent
在某些GNOME版本中，執行`gpg --gen-key`時會出現如下報錯信息

>gpg: WARNING: The GNOME keyring manager hijacked the GnuPG agent.
gpg: WARNING: GnuPG will not work properly - please configure that tool to not interfere with the GnuPG system!

提示`GnuPG agent`被`GNOME keyring manager`劫持。

解決方案見GnuPG官方的 [Problem](https://wiki.gnupg.org/GnomeKeyring 'GnuPG')。

在Debian中，執行如下命令

```bash
#此為一行命令
sudo dpkg-divert --local --rename --divert /etc/xdg/autostart/gnome-keyring-gpg.desktop-disable --add /etc/xdg/autostart/gnome-keyring-gpg.desktop
```

若要復原，執行如下命令

```bash
sudo dpkg-divert --rename --remove /etc/xdg/autostart/gnome-keyring-gpg.desktop
```


## List Keys On Local Host
執行如下命令查看系統中存在的公鑰

```bash
gpg --list-key [key_ID]
gpg --list-keys [key_ID]

#列出系統中存在的公鑰
gpg --list-public-keys

#列出系統中存在的私鑰
gpg --list-secret-keys
```

操作過程如下
```bash
maxdsre@lemp:~$ gpg --list-keys
/home/flying/.gnupg/pubring.gpg
-------------------------------
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

maxdsre@lemp:~$ gpg --list-keys 26B15903
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

maxdsre@lemp:~$ gpg --list-keys unday@test.com
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

maxdsre@lemp:~$
```

## List FingerPrints
執行如下命令列出所有或指定key的指紋
```bash
# List all keys (or the specified ones) along with  their  fingerprints
gpg --fingerprint key_ID

#--with-colons Print key listings delimited by colons.
gpg --fingerprint --with-colon key_ID
```

操作過程如下
```bash
# 列出所有key的指紋
maxdsre@lemp:~$ gpg --fingerprint
/home/flying/.gnupg/pubring.gpg
-------------------------------
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
      Key fingerprint = 23A6 D526 005F 1A87 94B0  51A1 2411 FE8C 26B1 5903
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

# 通過key_ID查詢指定key的指紋
maxdsre@lemp:~$ gpg --fingerprint 26B15903
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
      Key fingerprint = 23A6 D526 005F 1A87 94B0  51A1 2411 FE8C 26B1 5903
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

# 通過個人email查詢指定key的指紋
maxdsre@lemp:~$ gpg --fingerprint sunday@test.com
pub   4096R/26B15903 2016-12-18 [expires: 2017-12-18]
      Key fingerprint = 23A6 D526 005F 1A87 94B0  51A1 2411 FE8C 26B1 5903
uid                  sunday (sunday test) <sunday@test.com>
sub   4096R/868F064F 2016-12-18 [expires: 2017-12-18]

maxdsre@lemp:~$ gpg --fingerprint --with-colon 26B15903
tru::1:1482051689:1513587169:3:1:5
pub:u:4096:1:2411FE8C26B15903:2016-12-18:2017-12-18::u:sunday (sunday test) <sunday@test.com>::scESC:
fpr:::::::::23A6D526005F1A8794B051A12411FE8C26B15903:
sub:u:4096:1:9750F92A868F064F:2016-12-18:2017-12-18:::::e:
maxdsre@lemp:~$
```


## Generate Revoke Key
執行如下命令生成revoke key，可用於吊銷存儲在遠程服務器上的公鑰
```bash
#Generate  a  revocation  certificate  for  the  complete key.
gpg --gen-revoke key_ID

#生成文件
gpg --gen-revoke --armor --output=RevocationCertificate.asc key_ID
gpg --output RevocationCertificate.asc --gen-revoke key_ID
```

操作過程如下

```txt
maxdsre@lemp:~$ gpg --gen-revoke --armor --output=RevocationCertificate.asc 26B15903

sec  4096R/26B15903 2016-12-18 sunday (sunday test) <sunday@test.com>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 0
Enter an optional description; end it with an empty line:
> nothing description
>
Reason for revocation: No reason specified
nothing description
Is this okay? (y/N) y

You need a passphrase to unlock the secret key for
user: "sunday (sunday test) <sunday@test.com>"
4096-bit RSA key, ID 26B15903, created 2016-12-18

Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!

#查看生成的文件內容
maxdsre@lemp:~$ cat RevocationCertificate.asc
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1
Comment: A revocation certificate should follow

iQIyBCABAgAcBQJYVlSnFR0Abm90aGluZyBkZXNjcmlwdGlvbgAKCRAkEf6MJrFZ
A8PCD/sHTCsV2qJr3/bulQjp7Uoyi9IrKjOSKfJs3gOkDT/OExb1/pMcNN+0k20H
q+aPdfHK2MehJxYxbvkOiUccedaIBMR1pwuWb89oru52nDaqGq2xSrnFg5KaoFjx
zM7s9eATkusRR86YL4r6lmtp4+jzO+t+Rxg/bHbmyzBG+xd66t+BqeX1YsbJvAzj
zlPE/PH9IO+JqQiy7Fpp/5KHDbgyKQJV6DjEo4Iyzr/uory8GG0xMVLtNPwAMqAY
L0v922/qq+RyeRkx/ZtWUzL6wsBrxjx+KZvVQ5/XWz/lFFSY2Pi+ARcVueoUR3dV
XplIviCWJDz5261QF2BLPIPlJaAMJfs8Zwrbf+q6l04wuqH5MB2+wNA4empCmuQm
HW8g+9x6yMNjEBI+NkGHQNRZ4sd6H671Aps7ZYwmrxhYBdzIkvk/W1P85cE46/k9
LwKRl9QdkVGmuaOatHCVcEPbKHf61D410ZtUDaO+E+baDtfP7Tnaognewhkpo/Qg
LpB6f4gkwDSOyflxTWwTngAxXe5C5KXiSSXjosEia0EiVEYNVgZL2hfgiGHiGQ8k
RtpGRp+ASqYk8PFRpmfuoBt26tkxf/1+D0KASs0a157EDRxKK5VHNcJjIrG0XQci
q1+9JqzBHPWALr1nI7bseu+Va+UMPe/dEaTdGj/NV6Fh7Ax+9Q==
=+N+n
-----END PGP PUBLIC KEY BLOCK-----
maxdsre@lemp:~$
```


## Communiction With Remote KeyServer
可選擇

```http
keys.gnupg.net
```

此處以MIT的keyserver為例

```http
pgp.mit.edu
```

執行如下命令

```bash
# Search the keyserver for the given names.
gpg --keyserver pgp.mit.edu --search-keys key_ID

# Send key to keyserver
gpg --keyserver pgp.mit.edu --send-keys key_ID
#fingerprint eg: 23A6 D526 005F 1A87 94B0  51A1 2411 FE8C 26B1 5903
gpg --keyserver pgp.mit.edu --send-keys fingerprint

# Receive key from keyserver
gpg --keyserver pgp.mit.edu --recv-keys key_ID
```


## Exporting Public&Private Key

```bash
# export a public key
# 二進制格式存儲
gpg --output gpg-public.key --export key_ID
# http://dev.mysql.com/doc/refman/5.7/en/checking-rpm-signature.html
gpg -a --export key_ID > fileName.asc

# 添加指令-a / --armor，文本形式存儲
gpg -a --output gpg-public.key --export key_ID
#gpg --armor --output gpg-public.key --export key_ID

# export a private key
gpg --output gpg-private.key --export-secret-keys key_ID
#文本形式存儲
gpg -a --output gpg-private.key --export-secret-keys key_ID
#gpg --armor --output gpg-private.key --export-secret-keys key_ID
```


## Importing A Public Key

```bash
# import from file (gpg-public.key)
gpg --import fileName

# Receive key from keyserver
gpg --keyserver pgp.mit.edu --recv-keys key_ID
```


## Encryption & Decryption File
加密時可添加壓縮參數`--compress-algo`，具體使用，參見
NASA的[文檔](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html)

執行如下命令進行加密

```bash
#加密
gpg --encrypt --recipient key_ID fileName
#gpg -e -r key_ID fileName
#gpg --encrypt --recipient realname fileName
#gpg -e -r key_ID --compress-algo zlib fileName
```
加密完成後會在文件所在目錄生成名為`fileName.gpg`的文件，此文件即為加密後的文件。

```bash
#解密
gpg --output newFileName --decrypt fileName.gpg
gpg -o newFileName -d fileName.gpg
```
如果生成keypair時設置了passphrase，在解密時會要求輸入該passphrase。操作完成後會在文件所在目錄生成指定名稱的文件，此處指定的名稱是`newFileName`，此即解密後的文件。

如果是要將文件加密後給其他人解密，或是解密其他人給自己的加密文件，首先需要有對應的公鑰，然後在`--recipient`中指定對應的key_ID或email或realname即可。


## File Signature & Verify

```bash
#生成帶簽名的文件fileName.gpg(二進制形式)
gpg --sign fileName
#驗證文件簽名
gpg --verify fileName.gpg
```

也可以使用參數`--clearsign`生成文本形式的帶簽名文件
```bash
#生成帶簽名的文件fileName.asc(文本形式)
gpg --clearsign fileName
```

但無法直接驗證，會包如下錯誤

```bash
flying@lempstacker:/tmp$ gpg --verify test.txt.asc
gpg: Signature made Sun 18 Dec 2016 06:47:45 PM CST using RSA key ID 26B15903
gpg: Good signature from "sunday (sunday test) <sunday@test.com>"
gpg: WARNING: not a detached signature; file 'test.txt' was NOT verified!
flying@lempstacker:/tmp$ gpg --verify test.txt.asc test.txt
gpg: not a detached signature
flying@lempstacker:/tmp$
```

可通過參數`--detach-sign`生成獨立的簽名文件`fileName.sig`

```bash
#二進制形式fileName.sig
gpg --detach-sign fileName
#驗證
gpg --verify fileName.sig fileName


#文本形式fileName.asc
gpg -a --detach-sign fileName
#驗證
gpg --verify fileName.asc fileName
```

操作過程如下

```bash
#二進制形式
lying@lempstacker:~$ gpg --detach-sign /tmp/test.txt

You need a passphrase to unlock the secret key for
user: "sunday (sunday test) <sunday@test.com>"
4096-bit RSA key, ID 26B15903, created 2016-12-18

maxdsre@lemp:~$ gpg --verify /tmp/test.txt.sig /tmp/test.txt
gpg: Signature made Sun 18 Dec 2016 07:02:23 PM CST using RSA key ID 26B15903
gpg: Good signature from "sunday (sunday test) <sunday@test.com>"
maxdsre@lemp:~$ rm -f /tmp/test.txt.sig
maxdsre@lemp:~$

#文本形式
maxdsre@lemp:~$ gpg -a --detach-sign /tmp/test.txt

You need a passphrase to unlock the secret key for
user: "sunday (sunday test) <sunday@test.com>"
4096-bit RSA key, ID 26B15903, created 2016-12-18

maxdsre@lemp:~$ gpg --verify /tmp/test.txt.asc /tmp/test.txt
gpg: Signature made Sun 18 Dec 2016 07:03:04 PM CST using RSA key ID 26B15903
gpg: Good signature from "sunday (sunday test) <sunday@test.com>"
maxdsre@lemp:~$
```


## Deleting Key From Host
刪除key，注意，需先刪除私鑰，再刪除公鑰，否則會報錯

```bash
#刪除私鑰
gpg --delete-secret-keys key_ID

#刪除公鑰
gpg --delete-keys key_ID

#同時刪除公鑰、私鑰
gpg --delete-secret-and-public-keys key_ID
```

出現報錯的操作過程
```bash
maxdsre@lemp:~$ gpg --delete-keys 26B15903
gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: there is a secret key for public key "26B15903"!
gpg: use option "--delete-secret-keys" to delete it first.
maxdsre@lemp:~$
```


## Revoke Public Key From Remote Server
如果要revoke遠程keyserver上的公鑰，執行如下操作

1. 下載公鑰
2. 導入revoke key
3. 將導入revoke key的公鑰再次上傳

操作命令

```bash
#導入公鑰
gpg --import fileName
#gpg --keyserver pgp.mit.edu --recv-keys key_ID

#導入revoke key
gpg --import RevocationCertificate.asc

#再次提交到kerserver
gpg --keyserver pgp.mit.edu --send-keys key_ID
```

操作過程如下

```txt
maxdsre@lemp:~$ gpg --import RevocationCertificate.asc
gpg: key 26B15903: no public key - can't apply revocation certificate
gpg: Total number processed: 1
maxdsre@lemp:~$ gpg --keyserver pgp.mit.edu --recv-keys 26B15903
gpg: requesting key 26B15903 from hkp server pgp.mit.edu
gpg: key 26B15903: public key "sunday (sunday test) <sunday@test.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
maxdsre@lemp:~$ gpg --import RevocationCertificate.asc
gpg: key 26B15903: "sunday (sunday test) <sunday@test.com>" revocation certificate imported
gpg: Total number processed: 1
gpg:    new key revocations: 1
gpg: no ultimately trusted keys found
maxdsre@lemp:~$ gpg --keyserver pgp.mit.edu --send-keys 26B15903
gpg: sending key 26B15903 to hkp server pgp.mit.edu
maxdsre@lemp:~$ gpg --keyserver pgp.mit.edu --search-keys 26B15903
gpg: searching for "26B15903" from hkp server pgp.mit.edu
(1)	sunday (sunday test) <sunday@test.com>
	  4096 bit RSA key 26B15903, created: 2016-12-18, expires: 2017-12-18 (revoked)
Keys 1-1 of 1 for "26B15903".  Enter number(s), N)ext, or Q)uit > 1
gpg: requesting key 26B15903 from hkp server pgp.mit.edu
gpg: key 26B15903: "sunday (sunday test) <sunday@test.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
maxdsre@lemp:~$
```


## References
* [The GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual.html 'GnuPG')
* [GPG Quick Start](https://www.madboa.com/geek/gpg-quickstart/ 'Madboa')
* [GnuPG (GNU Privacy Guard)](https://www.suse.com/communities/blog/gnupg-gnu-privacy-guard/ 'Suse')
* [Using GPG to Encrypt Your Data](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html 'NASA')
* [GPG Tutorial](https://www.futureboy.us/pgp.html 'futureboy')
* [Revoking a GPG key](https://www.hackdiary.com/2004/01/18/revoking-a-gpg-key/ 'hackdiary')


## Change Logs
* 2016.12.18 19:54 Sun Asia/Shanghai
    * 初稿完成
* 2017.01.11 15:51 Wed Asia/Shanghai
    * 添加隨機數資源生成方法、`GNOME keyring manager`劫持`GnuPG agent`的解決方案
* 2018.08.01 22:31:26 Wed Asia/Shanghai
    * 勘誤，排版，遷移到新Blog


<!-- End -->
