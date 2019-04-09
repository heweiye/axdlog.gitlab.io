---
title: A Brief Introduction Of OpenSSL
slug: A Brief Introduction Of OpenSSL
date: 2017-01-08T18:13:23+08:00
lastmod: 2019-01-25T21:28:23-0500
draft: false
keywords: ["openssl"]
description: "A Brief Introduction Of OpenSSL"
categories:
- Security
tags:
- OpenSSL
comment: true
toc: true

---

[OpenSSL][openssl]是一個開源項目，由`SSL/TLS`工具集和加密庫構成，使用[Apache stype](https://wiki.openssl.org/index.php/License 'OpenSSL')許可，可免費獲取、使用。本文簡單介紹`OpenSSL`的版本信息、提供的命令。

<!--more-->

## Conventions
操作平臺信息

item|version details
---|---
os | Debian GNU/Linux 9.8 (stretch)
kernel | 4.9.0-8-amd64
openssl | OpenSSL 1.1.0j  20 Nov 2018


## OpenSSL Info

### Introduction Reference
以下介紹來自`OpenSSL`官網

>OpenSSL is an open source project that provides a robust, commercial-grade, and full-featured toolkit for the Transport Layer Security (TLS) and Secure Sockets Layer (SSL) protocols. It is als*a general-purpose cryptography library.

>The OpenSSL toolkit is licensed under an Apache-style license, which basically means that you are free t*get and use it for commercial and non-commercial purposes subject t*some simple license conditions. -- <https://www.openssl.org/>

以下介紹來自`WikiPedia`

>OpenSSL is a software library t*be used in applications that need t*secure communications over computer networks against eavesdropping or need t*ascertain the identity of the party at the other end. It has found wide use in internet web servers, serving a majority of all web sites.

>OpenSSL contains an open-source implementation of the SSL and TLS protocols. The core library, written in the C programming language, implements basic cryptographic functions and provides various utility functions. Wrappers allowing the use of the OpenSSL library in a variety of computer languages are available. -- <https://en.wikipedia.org/wiki/OpenSSL>

### Functions Provided
`OpenSSL`提供如下功能(`man openssl`)

* Creation and management of private keys, public keys and parameters
* Public key cryptographic operations
* Creation of X.509 certificates, CSRs and CRLs
* Calculation of Message Digests
* Encryption and Decryption with Ciphers
* SSL/TLS Client and Server Tests
* Handling of S/MIME signed or encrypted mail
* Time Stamp requests, generation and verification


### OpenSSL Version
OpenSSL當前(Jan 25, 2019)最新穩定版本是`1.1.1`，同時也是長期支持版本(Long Term Suppout)。

>The latest stable version is the 1.1.1 series. This is also our Long Term Support (LTS) version, supported until 11th September 2023.

`OpenSSL`的生命週期如下

Version | Type | EOL
---|---|---
1.1.1 | Latest stable, LTS | Sep 11, 2023
1.0.2 | Previous LTS version | Dec 31, 2019
1.1.0 | | Sep 11, 2019
1.0.1 | | out of support
1.0.1 | | out of support
0.9.8 | | out of support

>Our previous LTS version (1.0.2 series) will continue to be supported until 31st December 2019 (security fixes only during the last year of support). The 1.1.0 series is currently only receiving security fixes and will go out of support on 11th September 2019. All users of 1.0.2 and 1.1.0 are encouraged to upgrade to 1.1.1 as soon as possible. The 0.9.8, 1.0.0 and 1.0.1 versions are now out of support and should not be used.  -- https://www.openssl.org/source/


如需要使用最新穩定版，需手動進行編譯安裝，具體可參閱

* [Compilation and Installation](https://wiki.openssl.org/index.php/Compilation_and_Installation 'OpenSSL')
* [OpenSSL - Beyond Linux® From Scratch](http://www.linuxfromscratch.org/lfs/view/stable/chapter06/openssl.html 'BLFS')

雖然`v1.0.1`已經於`Dec 31, 2016`被官方停止支持，但該版本是OpenSSL很重要的一個版本。`OpenSSL`從`v1.0.1`開始支持 *TLS v1.1* 和 *TLS v1.2* 協議。而`v1.01`之前的版本不支持該協議。詳見

* [OpenSSL 1.0.1 Series Release Notes](https://www.openssl.org/news/openssl-1.0.1-notes.html 'OpenSSL')的 *Major changes between OpenSSL 1.0.0h and OpenSSL 1.0.1 [14 Mar 2012]* 部分；
* [Changelog](https://www.openssl.org/news/changelog.html#x26 'Changes between 1.0.0h and 1.0.1  [14 Mar 2012]')

### Release Notice & Vulnerability
有關`OpenSSL`的官方聲明、版本釋出、漏洞修補等信息，可通過如下鏈接訪問

item|link
---|---
Newslog|<https://www.openssl.org/news/newslog.html>
Changelog|<https://www.openssl.org/news/changelog.html>
**Vulnerabilities** |<https://www.openssl.org/news/vulnerabilities.html>

尤其是漏洞修補，對於保護系統安全、網路通信安全至關重要，建議不定期瀏覽該頁面，以及時更新`OpenSSL`版本。


### OpenSSL Check
1. 通過如下命令判斷系統中是否安裝了`OpenSSl`
```bash
command -v openssl &> /dev/null && echo 'installed' || echo 'not install'
```

2. 通過如下命令查看版本信息

可通過如下命令查看`OpenSSl`版本信息
```bash
# simple version info
openssl version
# openssl version | cut -d' ' -f2

# complete version info
openssl version -a
```

操作過程如下

```bash
$ command -v openssl &> /dev/null && echo 'installed' || echo 'not install'
installed
$ openssl version
OpenSSL 1.1.0j  20 Nov 2018
$ openssl version | cut -d' ' -f2
1.1.0j
$ openssl version -a
OpenSSL 1.1.0j  20 Nov 2018
built on: reproducible build, date unspecified
platform: debian-amd64
options:  bn(64,64) rc4(16x,int) des(int) blowfish(ptr)
compiler: gcc -DDSO_DLFCN -DHAVE_DLFCN_H -DNDEBUG -DOPENSSL_THREADS -DOPENSSL_NO_STATIC_ENGINE -DOPENSSL_PIC -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DRC4_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPADLOCK_ASM -DPOLY1305_ASM -DOPENSSLDIR="\"/usr/lib/ssl\"" -DENGINESDIR="\"/usr/lib/x86_64-linux-gnu/engines-1.1\""
OPENSSLDIR: "/usr/lib/ssl"
ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-1.1"

```

此處的`/usr/lib/ssl`是`OpenSSL`的配置文件和證書所在的路徑，具體見如下命令結果

```bash
$ cd /usr/lib/ssl/
$ pwd
/usr/lib/ssl
$ ls -lhF
total 0
lrwxrwxrwx 1 root root 14 Mar 29  2018 certs -> /etc/ssl/certs/
drwxr-xr-x 2 root root 32 Dec  2 18:45 misc/
lrwxrwxrwx 1 root root 20 Nov 28 17:43 openssl.cnf -> /etc/ssl/openssl.cnf
lrwxrwxrwx 1 root root 16 Mar 29  2018 private -> /etc/ssl/private/
$ ls -lhF misc/
total 16K
-rwxr-xr-x 1 root root 6.6K Nov 28 17:43 CA.pl*
-rwxr-xr-x 1 root root 6.5K Nov 28 17:43 tsget*
```

該目錄的子目錄`./misc`中含有實現私有CA(cerfication authority)的腳本，腳本中的路徑及命令的說明可通過命令`man ca`查看。


## OpenSSL Commands
`OpenSSL`是一個工具集，提供了很多命令。可通過如下命令查看
```bash
openssl help

#詳細信息查看
man openssl
```

### List Available Commands
操作結果如下

```
$ openssl help

Standard commands
asn1parse         ca                ciphers           cms               
crl               crl2pkcs7         dgst              dhparam           
dsa               dsaparam          ec                ecparam           
enc               engine            errstr            exit              
gendsa            genpkey           genrsa            help              
list              nseq              ocsp              passwd            
pkcs12            pkcs7             pkcs8             pkey              
pkeyparam         pkeyutl           prime             rand              
rehash            req               rsa               rsautl            
s_client          s_server          s_time            sess_id           
smime             speed             spkac             srp               
ts                verify            version           x509              

Message Digest commands (see the `dgst' command for more details)
blake2b512        blake2s256        gost              md4               
md5               rmd160            sha1              sha224            
sha256            sha384            sha512            

Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb       
aes-256-cbc       aes-256-ecb       base64            bf                
bf-cbc            bf-cfb            bf-ecb            bf-ofb            
camellia-128-cbc  camellia-128-ecb  camellia-192-cbc  camellia-192-ecb  
camellia-256-cbc  camellia-256-ecb  cast              cast-cbc          
cast5-cbc         cast5-cfb         cast5-ecb         cast5-ofb         
des               des-cbc           des-cfb           des-ecb           
des-ede           des-ede-cbc       des-ede-cfb       des-ede-ofb       
des-ede3          des-ede3-cbc      des-ede3-cfb      des-ede3-ofb      
des-ofb           des3              desx              rc2               
rc2-40-cbc        rc2-64-cbc        rc2-cbc           rc2-cfb           
rc2-ecb           rc2-ofb           rc4               rc4-40            
seed              seed-cbc          seed-cfb          seed-ecb          
seed-ofb          

```

輸出信息分為三部分

* Standard commands
* Message Digest commands   (`man dgst`)
* Cipher commands   (`man enc`)

但如果要分別顯示，可通過如下命令


```bash
# Standard commands (共48個)
openssl list -commands

#Message Digest commands (共12個)
openssl list -digest-commands

#Cipher commands (共53個)
openssl list -cipher-commands
```

以上三個皆為 *pseudo-command* ，同樣還有

```bash
#list all cipher algorithms (共134個)
openssl list -cipher-algorithms

#list all supported public key algorithms (共13個)
openssl list -public-key-algorithms
```

### Standard Commands
標準命令目前有`48`個，具體使用可通過命令`man command`查看，如`man x509`。

command|explain
---|---
`asn1parse` |Parse an ASN.1 sequence.
`ca`        |Certificate Authority (CA) Management.
`ciphers`   |Cipher Suite Description Determination.
`cms`       |CMS (Cryptographic Message Syntax) utility
`crl`       |Certificate Revocation List (CRL) Management.
`crl2pkcs7` |CRL to PKCS#7 Conversion.
`dgst`      |Message Digest Calculation.
`dh`        |Diffie-Hellman Parameter Management.<br/>Obsoleted by `dhparam`.
`dhparam`   |Generation and Management of Diffie-Hellman Parameters.<br/>Superseded by `genpkey` and `pkeyparam`
`dsa`       |DSA Data Management.
`dsaparam`  |DSA Parameter Generation and Management.<br/>Superseded by `genpkey` and `pkeyparam`
`ec`        |EC (Elliptic curve) key processing
`ecparam`   |EC parameter manipulation and generation
`enc`       |Encoding with Ciphers.
`engine`    |Engine (loadble module) information and manipulation.
`errstr`    |Error Number to Error String Conversion.
`gendh`     |Generation of Diffie-Hellman Parameters.<br/>Obsoleted by dhparam.
`gendsa`    |Generation of DSA Private Key from Parameters.<br/>Superseded by `genpkey` and `pkey`
`genpkey`   |Generation of Private Key or Parameters.
`genrsa`    |Generation of RSA Private Key.<br/>Superceded by `genpkey`.
`nseq`      |Create or examine a netscape certificate sequence
`ocsp`      |Online Certificate Status Protocol utility.
`passwd`    |Generation of hashed passwords.
`pkcs12`    |PKCS#12 Data Management.
`pkcs7`     |PKCS#7 Data Management.
`pkcs8`     |PKCS#8 format private key conversion tool.
`pkey`      |Public and private key management.
`pkeyparam` |Public key algorithm parameter management.
`pkeyutl`   |Public key algorithm cryptographic operation utility.
`rand`      |Generate pseudo-random bytes.
`rehash`    |Create symbolic links to certificate and CRL files named by the hash values.
`req`       |PKCS#10 X.509 Certificate Signing Request (CSR) Management.
`rsa`       |RSA key management.
`rsautl`    |RSA utility for signing, verification, encryption, and decryption. Superseded by  pkeyutl
`s_client`  |This implements a generic SSL/TLS client which can establish a transparent connection to a remote server speaking SSL/TLS.
`s_server`  |This implements a generic SSL/TLS server which accepts connections from remote clients speaking SSL/TLS.
`s_time`    |SSL Connection Timer.
`sess_id`   |SSL Session Data Management.
`smime`     |S/MIME mail processing.
`speed`     |Algorithm Speed Measurement.
`spkac`     |SPKAC printing and generating utility
`ts`        |Time Stamping Authority tool (client/server)
`verify`    |X.509 Certificate Verification.
`version`   |OpenSSL Version Information.
`x509`      |X.509 Certificate Data Management.


### Message Digest Commands
消息摘要命令

command|explain
---|---
`md2`       |MD2 Digest
`md5`       |MD5 Digest
`mdc2`      |MDC2 Digest
`rmd160`    |RMD-160 Digest
`sha`       |SHA Digest
`sha1`      |SHA-1 Digest
`sha224`    |SHA-224 Digest
`sha256`    |SHA-256 Digest
`sha384`    |SHA-384 Digest
`sha512`    |SHA-512 Digest

可通過如下命令查看可用的摘要命令

```bash
# list all available ciphers
openssl ciphers -v

# This lists ciphers compatible with any of TLSv1, TLSv1.1 or TLSv1.2.
openssl ciphers -v -tls1

# list only high encryption ciphers (key lengths larger than 128 bits)
# 'MEDIUM' represents 128 bit encryption
openssl ciphers -v 'HIGH'

# list only cipher suites using RSA key exchange
openssl ciphers -v 'RSA+HIGH'

# list only high encryption ciphers using the RSA algorithm
openssl ciphers -v 'RSA+HIGH'
```


### Encoding And Ciph  Commands

Base64 Encoding|Base64
---|---
Blowfish Cipher|bf、bf-cbc、bf-cfb、bf-ecb、bf-ofb
CAST Cipher|cast、cast-cbc
CAST5 Cipher|cast5-cb、cast5-cfb、cast5-ecb、cast5-ofb
DES Cipher|des、des-cbc、des-cfb、des-ecb、des-ede、des-ede-cbc、des-ede-cfb、des-ede-ofb、des-ofb
Triple-DES Cipher|des3、desx、des-ede3、des-ede3-cbc、des-ede3-cfb、des-ede3-ofb
IDEA Cipher|idea、idea-cbc、idea-cfb、idea-ecb、idea-ofb
RC2 Cipher|rc2、rc2-cbc、rc2-cfb、rc2-ecb、rc2-ofb
RC4 Cipher|rc4       
RC5 Cipher|rc5、rc5-cbc、rc5-cfb、rc5-ecb、rc5-ofb


## Bibliography
>Ivan Ristić, the creator of  <https://ssllabs.com>, has a free download of his OpenSSL Cookbook that covers the most frequently used OpenSSL features and commands. It is updated often, and is available at  <https://www.feistyduck.com/books/openssl-cookbook/>. It is highly recommended. -- <https://www.openssl.org/docs/>


## Change Logs
* 2017.01.08 18:12 Sun Asia/Shanghai
    * 初稿完成
* 2018.07.27 16:12 Fri America/Boston
    * 勘誤，遷移到新Blog
* 2019.01.25 21:28 Fri America/Boston
    * 重新撰寫


[openssl]:https://www.openssl.org

<!-- End -->
