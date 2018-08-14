---
title: Using OpenSSL To Create X.509 Self-Signed Certificate On GNU/Linux
slug: Using OpenSSL To Create X.509 Self-Signed Certificate On GNU Linux
date: 2017-01-09T17:16:09+08:00
lastmod: 2018-07-27T15:53:08-04:00
draft: false
keywords: ["AxdLog", "openssl"]
description: "Try to create X.509 self-signed certificate via OpenSSL On GNU/Linux"
categories:
- Security
tags:
- OpenSSL
- X.509
comment: true
toc: true

---

[OpenSSL](https://www.openssl.org/)是一個提供`SSL/TLS`工具集和加密庫的開源項目，其中一項功能是創建基於[X.509](https://en.wikipedia.org/wiki/X.509 'WikiPedia')標準的數字證書(Digital Certificate)。數字證書可由[CA](https://en.wikipedia.org/wiki/Certificate_authority 'WikiPedia')機構(Certificate Authoriy)簽發，也可由自己簽發。自己簽發的證書稱為自簽證書([Self-Signed Certificate](https://en.wikipedia.org/wiki/Self-signed_certificate))，本文記錄使用OpenSSL創建自簽證書的過程。

<!--more-->

本文大部分內容梳理自[Ivan Ristic](https://twitter.com/ivanristic 'Twitter')的[OpenSSL Cookbook](https://www.feistyduck.com/books/openssl-cookbook/)，推薦閱讀。

## Introduction
數字證書又稱為`Digital Certificate`(或`Identity Certificate`或`Public Key Certificate`)，採用非對稱加密(又名公鑰加密 [Public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography 'WikiPedia'))，用於證明公鑰(public key)的所有者。證書中含有公鑰信息、所有者身份信息、簽發者的數字簽名等信息。證書是`SSL/TLS`的重要組成部分，用於保證網絡通信安全。通常所說的SSL證書即數字證書。

數字證書可由[OpenSSL](https://www.openssl.org/)創建，基於[X.509](https://en.wikipedia.org/wiki/X.509 'WikiPedia')標準，具體見[ITU-T X.509](http://www.itu.int/ITU-T/recommendations/rec.aspx?rec=13031 'ITU')。

需注意，與CA簽發的證書相比，自簽證書有安全隱患，建議根據使用場景選擇。具體參見[The Dangers of Self-Signed SSL Certificates](https://www.globalsign.com/en/ssl-information-center/dangers-self-signed-certificates/ 'GlobalSign')。


## Conventions
文本中生成的文件定義如下

file|format|explanation
---|---|---
私鑰|`key.pem`|類 xxx.key
公鑰|`pubkey.pem`|類 xxx.pubkey
CSR|`req.pem`|類 xxx.csr
證書|`cert.pem`|類 xxx.crt

操作目錄為`/tmp/`，`Passphrase`值設置為`AxdLog2017`。

`.crt`為已簽署的證書，crt是certificate的縮寫。

## Private Key Generation
對於數字證書而言，私鑰起著非常重要的作用。**私鑰生成算法**(Key Algorithm)和 **私鑰長度**(Key size)的選擇決定了數字證書的安全程度。目前主要有3中私鑰生成算法：[RSA](https://simple.wikipedia.org/wiki/RSA_algorithm 'WikiPedia')、[DSA](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm 'WikiPedia')、[ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm 'WikiPedia')。

創建自簽證書需經歷3個主要步驟：

1. 生成私鑰(Private Key)；
2. 生成證書簽署請求文件CSR(Certificate Signing Request)；
3. 生成自簽證書(Self-Signed Certificate)；

但不一定要嚴格遵守，在實際操作過程中，可使用`req`命令，通過私鑰直接生成自簽證書。

私鑰生成長度的選擇，取決於算法。對於`RSA`、`DSA`，建議使用`2048`bits及以上；對於`ECDSA`，建議使用`256`bits及以上的長度。

生成過程中會出現`Passphrase`提示，建議設置，如果直接按`Enter`鍵跳過，在某些OpenSSL版本中會出現如下報錯信息

>140203959170704:error:28069065:lib(40):UI_set_result:result too small:ui_lib.c:823:You must type in 4 to 1023 characters

如果需要設置，但不想出現該提示，可通過選項`-passout`、`-passin`實現。如果確實不需要`pass phrase`，可在私鑰生成後，通過命令將其移除。

以下分別列出3種算法的使用命令，相關命令的具體使用可通過`man command`查看，如`man genrsa`。

### RSA Key Generation
對於`RSA`

* 使用命令`genrsa`生成私鑰；
* 使用命令`rsa`輸出公鑰、私鑰組件或提取公鑰；

1. 生成私鑰

```bash
openssl genrsa -out /tmp/key.pem -aes256 4096
#或 (無需輸入)
openssl genrsa -passout pass:AxdLog2017 -out /tmp/key.pem -aes256 4096
```

命令選項說明

`genrsa` - generate an RSA private key.

>* `-passout arg` - the output file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-out filename` - the output filename.
>* `-aes256` - encrypt the private key with specified cipher before outputting it.
>* `numbits` - the size of the private key to generate in bits. This must be the last option specified. The default is 512. 該選項必須在最後位置

**注意**：不建議使用`DES`、`3DES`、`SEED`等算法。

2. 移除RSA私鑰中的`pass phrase`(可選)

```bash
openssl rsa -in /tmp/key.pem -out /tmp/keyout.pem
# openssl rsa -passin pass:AxdLog2017 -in /tmp/key.pem -out /tmp/keyout.pem
```

3. 提取公鑰

```bash
openssl rsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
```

命令選項說明

`rsa` - RSA key processing tool

>* `-pubout` - by default a private key is output: with this option a public key will be output instead. This option is automatically set if the input is a public key.
>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` -  This specifies the output filename to write a key to or standard output if this option is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.


4. 輸出公鑰、私鑰組件

```bash
openssl rsa -in /tmp/key.pem -text
#或
openssl rsa -in /tmp/key.pem -text -noout
```

命令選項說明

>* `-text` - prints out the various public or private key components in plain text in addition to the encoded version.
>* `-noout` - this option prevents output of the encoded version of the key.


演示過程
```bash
#生成私鑰
maxdsre@jessie:/tmp$ openssl genrsa -out /tmp/key.pem -aes256 4096
Generating RSA private key, 4096 bit long modulus
..........++
...........++
e is 65537 (0x10001)
Enter pass phrase for /tmp/key.pem:
Verifying - Enter pass phrase for /tmp/key.pem:

#提取公鑰
maxdsre@jessie:/tmp$ openssl rsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
Enter pass phrase for /tmp/key.pem:
writing RSA key

#移除私鑰中的pass phrase
maxdsre@jessie:/tmp$ openssl rsa -in /tmp/key.pem -out /tmp/keyout.pem
Enter pass phrase for /tmp/key.pem:
writing RSA key

#使用已移除pass phrase的私鑰生成公鑰，未出現pass phrase輸入提示
maxdsre@jessie:/tmp$ openssl rsa -in /tmp/keyout.pem -pubout -out /tmp/pubkeyout.pem
writing RSA key
maxdsre@jessie:/tmp$

#私鑰比較，含有pass phrase的私鑰多了Proc-Type、DEK-Info兩行
maxdsre@jessie:/tmp$ cat /tmp/key.pem
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,D7C5FEB32AEB672F35C820FB056AE7FE

BxnUmw0ElK8kZWPuMrDV+aQsAR42pLLYi2YcWPU6OJGJDVRnHzQ8JB9efV+i29Qt
r83qAqT5bTNbuSZ3mM2DXWurNM49fuaLQjhd9PhOZ40ZJxmbKpHru4jfFZQ/D3Hi
...
省略
...
qBjNDHCc1++Y+S3U9RHKOQS1UBwSxsND4FSnj3MCRxhZ8VpgftqOc2Llb6PdnRYo
RBdKI17KoCT5nRZRuRT2mExDJGv2Wi5kuob0uKZbN6fMSLTNCZSiPO7WHguOrvQw
-----END RSA PRIVATE KEY-----
maxdsre@jessie:/tmp$ cat /tmp/keyout.pem
-----BEGIN RSA PRIVATE KEY-----
MIIJKQIBAAKCAgEAxnnhKw+X3f9+Yj7Lx4eE2xjFVi3iu1BTqLe8kC4sAGwmJs61
GJu5UhdkhSG/AMAcpbc6ghBbsbQdDhFKnMaF5yhEhxaIivcZ4gkSKZgdO/C3r3Yw
...
省略
...
yIyo9kOfWYcfMXeXKUrcN0rDEmdvq3R+3rMwaMRPqnWZW8v/N3SvisKl1xQy9NfB
PjYdgieVDbEPAy/OHeQdlGZCLIPcxxsoXP71LMFXah2ghQFvOA4yhNF7PXah
-----END RSA PRIVATE KEY-----
maxdsre@jessie:/tmp$

#公鑰比較
maxdsre@jessie:~$ diff /tmp/pubkey{,out}.pem && echo 'same' || echo 'different'
same
maxdsre@jessie:~$ cat /tmp/pubkeyout.pem
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAxnnhKw+X3f9+Yj7Lx4eE
2xjFVi3iu1BTqLe8kC4sAGwmJs61GJu5UhdkhSG/AMAcpbc6ghBbsbQdDhFKnMaF
5yhEhxaIivcZ4gkSKZgdO/C3r3YwwQYMRQ8HEYj/EZOeWGbPTcsb96kc3GS5zdqA
dzdAiB3By44gA93XMANkF1cKRawBS+z8Pu7mAFv93QlJA60Ovu1d9qdi6tQ1RoDm
FOhE2M0K+6+gLcVoGTHk/+xJgPakDxOQ2mm/BWLOYsqZd/Wd0eVT3peyBed8/WE5
P1/MCeI7xG6QPIxpuTHiykdaUSALZeGN89UDmK6Mj/MSKwKmCkdgqlDdpVsZtU6r
u5LVnT+mtlcH9XZMT3iiNjnUklXId/3E3x2GcBfYPTNhWAw689Vactp7GgeqhHaq
sgpGbBuhy8ZJZZ5oaNWnD3RsioEHL1msKdX3a1eLRK6ITAcmijWn7zZIC4E2cT3Y
DG1DjvVsTYTAuDvdEZq70gsiuLQTCf9YOc0+SE4NuBiuRMUkd06bowuIYvf+MLUG
B7i5Q8Y7wn5mIFYZU53eJSVDCfP8oRpTvaSV4v5ZswExuxO8P/yesvSEihCdbtbt
QfmuTssApj1TxTiI0RHQupf7LBwUuVfs0uctnM7qs4nrL9aKi+0SuvwE9yEecH/0
3M8fdEBg6f33Uf8zFdezycMCAwEAAQ==
-----END PUBLIC KEY-----
maxdsre@jessie:~$
```


### DSA Key Generation
對於`DSA`，須首先生成DSA參數(DSA parameters)

* 使用命令`dsaparam`生成DSA參數；
* 使用命令`dsa`或`genpkey`生成私鑰；

1. 生成私鑰 (方式較多)

```bash
# method1: directly
openssl dsaparam -genkey 4096 | openssl dsa -out /tmp/key.pem -aes256
# openssl dsaparam -genkey 4096 | openssl dsa -passout pass:AxdLog2017 -out /tmp/key.pem -aes256

# method2: using dsa
openssl dsaparam -out /tmp/dsa_para.pem -genkey 4096
openssl dsa -in /tmp/dsa_para.pem -out /tmp/key.pem -aes256
# openssl dsa -passout pass:AxdLog2017 -in /tmp/dsa_para.pem -out /tmp/key.pem -aes256

# method3: using genpkey
openssl dsaparam -out /tmp/dsa_para.pem -genkey 4096
openssl genpkey -paramfile /tmp/dsa_para.pem -out /tmp/key.pem -aes256
# openssl genpkey -paramfile /tmp/dsa_para.pem -pass pass:AxdLog2017 -out /tmp/key.pem -aes256
```

命令選項說明

`dsaparam` - DSA parameter manipulation and generation

>* `-genkey` - this option will generate a DSA either using the specified or generated parameters.
>* `-out filename` -  This specifies the output filename parameters to. Standard output is used if this option is not present. The output filename should not be the same as the input filename.
>* `numbits` - this option specifies that a parameter set should be generated of size numbits. It must be the last option. If this option is included then the input file (if any) is ignored.

`dsa` - DSA key processing

>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` -  This specifies the output filename to write a key to or standard output by is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.
>* `-aes256` - encrypt the private key with specified cipher before outputting it.

`genpkey` - generate a private key

>* `-out filename` - the output filename. If this argument is not specified then standard output is used.
>* `-paramfile filename` -  Some public key algorithms generate a private key based on a set of parameters.  They can be supplied using this option. If this option is used the public key algorithm used is determined by the parameters. If used this option must precede and `-pkeyopt` options. The options `-paramfile` and `-algorithm` are mutually exclusive.
>* `-pass arg` -  the output file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-cipher` - This option encrypts the private key with the supplied cipher.

2. 移除DSA私鑰中的`pass phrase`(可選)

```bash
openssl dsa -in /tmp/key.pem -out /tmp/keyout.pem
# openssl dsa -passin pass:AxdLog2017 -in /tmp/key.pem -out /tmp/keyout.pem
```

3. 提取公鑰

```bash
openssl dsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
```

命令選項說明

>* `-pubout` - by default a private key is output. With this option a public key will be output instead. This option is automatically set if the input is a public key.


4. 提取私鑰組件

```bash
openssl dsa -in /tmp/dsa_key.pem -text
#或
openssl dsa -in /tmp/dsa_key.pem -text -noout
```

命令選項說明

>* `-text` - prints out the public, private key components and parameters.
>* `-noout` - this option prevents output of the encoded version of the key.


演示過程

```bash
#生成私鑰
maxdsre@jessie:~$ openssl dsaparam -genkey 4096 | openssl dsa -out /tmp/key.pem -aes256
read DSA key
Generating DSA parameters, 4096 bit long prime
This could take some time
.........+.......+..+++++++++++++++++++++++++++++++++++++++++++++++++++*
.........+...+..+...............+....+...+........+...............+..+.+...+.........+........+..+..+.....+.............+.....+...............+..+..................+.........................+.....+..+.+................+..+..+.+................+.......+.......................+.........+......+.......+....................................+++++++++++++++++++++++++++++++++++++++++++++++++++*
writing DSA key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

#移除私鑰中的pass phrase
maxdsre@jessie:~$ openssl dsa -in /tmp/key.pem -out /tmp/keyout.pem
read DSA key
Enter pass phrase for /tmp/key.pem:
writing DSA key

#生成公鑰
maxdsre@jessie:~$ openssl dsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
read DSA key
Enter pass phrase for /tmp/key.pem:
writing DSA key

#使用已移除pass phrase的私鑰生成公鑰，未出現pass phrase輸入提示
maxdsre@jessie:~$ openssl dsa -in /tmp/keyout.pem -pubout -out /tmp/pubkeyout.pem
read DSA key
writing DSA key

#私鑰比較，含有pass phrase的私鑰多了Proc-Type、DEK-Info兩行
maxdsre@jessie:~$ cat /tmp/key.pem
-----BEGIN DSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,DD0F724FE050BF04DB93F98084C56725

8tFr8UR+N6PAqu2AzKN9CXjgkjXDDD3U8Qhl2EH9MTRkyeT8+2WMtnctNiDhiarX
YRXm9OtaStzl0ycHn3zGe3H9D+9iMauxos6l2GrJyDxc1D5BZXWHD3u4xRpxoBYk
...
省略
...
XKbwixhYyCw1duKLE46qZh16NGkbSDORPARPEjZPA4lHuptPaf0rRXSpQDgR82+B
U2bkZO4nbiZ22mKJG4iEnWwDyBC4/cdFSHUA5wCyX2/kW5JIyvQ/gqSyQKG9W4As
-----END DSA PRIVATE KEY-----
maxdsre@jessie:~$ cat /tmp/keyout.pem
-----BEGIN DSA PRIVATE KEY-----
MIIGVQIBAAKCAgEApddIILGPXOD5O800lIjh1UVpiinRLBhDtRA4dleZBIfZD2tj
g2lE0Wh0lNPgPDSROwcVAOzmVPcn+OvFS4b3GRJtbEvrxGDzbwuWhBuQnBKbK8ar
...
省略
...
+/ZBnFBS6lUJp6kGgPWQvw+MTrnXPrrDrmctFZZPE26nQXjREpFjeXYOh1sE70V8
C69fzJCt8X/NChT7yWM8r+vbcD4MB/AJOmYziI3BhclNaPVTJ7ohRMgU91lhgBb2
7Dri+KcvDAIgA6fy19W9CBPqckgl3aTlmZc5jczt7zaJpwMTwq2wg20=
-----END DSA PRIVATE KEY-----

#公鑰比較
maxdsre@jessie:~$ diff /tmp/pubkey{,out}.pem && echo 'same' || echo 'different'
same
maxdsre@jessie:~$ cat /tmp/pubkeyout.pem
-----BEGIN PUBLIC KEY-----
MIIGRjCCBDkGByqGSM44BAEwggQsAoICAQCl10ggsY9c4Pk7zTSUiOHVRWmKKdEs
GEO1EDh2V5kEh9kPa2ODaUTRaHSU0+A8NJE7BxUA7OZU9yf468VLhvcZEm1sS+vE
...
...
...
l8Tvx3CNMqCDyE8Dr7g/Di5mJfv2QZxQUupVCaepBoD1kL8PjE651z66w65nLRWW
TxNup0F40RKRY3l2DodbBO9FfAuvX8yQrfF/zQoU+8ljPK/r23A+DAfwCTpmM4iN
wYXJTWj1Uye6IUTIFPdZYYAW9uw64vinLww=
-----END PUBLIC KEY-----
maxdsre@jessie:~$
```


### ECDSA Key Generation
`ECDSA`是 *Elliptic Curve Digital Signature Algorithm* 的縮寫，即 **橢圓曲線數字簽名算法**。要生成私鑰，須首先生成EC參數(EC parameters)。生成EC參數時需指定 *命名曲線*(named curves)，OpenSSL中有多種 *命名曲線* 可供選擇。執行如下命令查看可供選擇的 *命名曲線*:

```bash
#-list_curves - get a list of all currently implemented EC parameters
openssl ecparam -list_curves

# openssl ecparam -list_curves | sed -r -n 's@^[[:space:]]*([^:].*): .*@\1@p'  #get short name
```

對於Web Server而言，建議使用 `prime256v1`(secp256r1), `secp384r1`, `secp521r1` 這三種EC參數，具體見[Security/Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS)。

相關操作命令

* 使用命令`ecparam`生成EC參數；
* 使用命令`ec`或`genpkey`生成私鑰；

此處選擇的曲線參數是`secp384r1`。

1. 生成私鑰 (方式較多)

```bash
#method1: directly
openssl ecparam -genkey -name secp384r1 | openssl ec -out /tmp/key.pem -aes256
# openssl ecparam -genkey -name secp384r1 | openssl ec -passout pass:AxdLog2017 -out /tmp/key.pem -aes256

# method2: using ec
openssl ecparam -out /tmp/ec_param.pem -name secp384r1 -genkey
openssl ec -in /tmp/ec_param.pem -out /tmp/key.pem -aes256
# openssl ec -passout pass:AxdLog2017 -in /tmp/ec_param.pem -out /tmp/key.pem -aes256

# method3: using genpkey
openssl ecparam -out /tmp/ec_param.pem -name secp384r1
openssl genpkey -paramfile /tmp/ec_param.pem -out /tmp/key.pem -aes256
# openssl genpkey -paramfile /tmp/ec_param.pem -pass pass:AxdLog2017 -out /tmp/key.pem -aes256
```

命令選項說明

`ecparam` - EC parameter manipulation and generation

>* `-genkey` - This option will generate a EC private key using the specified parameters.
>* `-name arg` - Use the EC parameters with the specified 'short' name. Use `-list_curves` to get a list of all currently implemented EC parameters.

`ec` - EC key processing

>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` - This specifies the output filename to write a key to or standard output by is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.


2. 移除EC私鑰中的`pass phrase`(可選)

```bash
openssl ec -in /tmp/key.pem -out /tmp/keyout.pem
# openssl ec -passin pass:AxdLog2017 -in /tmp/key.pem -out /tmp/keyout.pem
```

3. 提取公鑰

```bash
openssl ec -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
```

4. 提取私鑰組件

```bash
openssl ec -in /tmp/key.pem -text
#或
openssl ec -in /tmp/key.pem -text -noout
```

命令選項說明

>* `-text` - prints out the public, private key components and parameters.
>* `-noout` - this option prevents output of the encoded version of the key.


演示過程

```bash
#生成私鑰
maxdsre@jessie:~$ openssl ecparam -genkey -name secp521r1 | openssl ec -out /tmp/key.pem -aes256
read EC key
writing EC key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

##移除私鑰中的pass phrase
maxdsre@jessie:~$ openssl ec -in /tmp/key.pem -out /tmp/keyout.pem
read EC key
Enter PEM pass phrase:
writing EC key

##生成公鑰
maxdsre@jessie:~$ openssl ec -in /tmp/key.pem -pubout -out /tmp/pubkey.pem
read EC key
Enter PEM pass phrase:
writing EC key

##使用已移除pass phrase的私鑰生成公鑰，未出現pass phrase輸入提示
maxdsre@jessie:~$ openssl ec -in /tmp/keyout.pem -pubout -out /tmp/pubkeyout.pem
read EC key
writing EC key
maxdsre@jessie:~$

#私鑰比較，含有pass phrase的私鑰多了Proc-Type、DEK-Info兩行
maxdsre@jessie:~$ cat /tmp/key.pem
-----BEGIN EC PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,5B2CD0794F59FF4745858C8AD77CFB2D

/hITI77ZR+anETnKFgBBd3GlzhYcrnGHI+QWE3Bz3vYOihlb0zl7d3P+Oy99H0lo
YFiwhBCNI/Ems+cSCxLD+Wo62Ox9LHuZx91D3SETjtpsRXnmc4l5ZBrMY36ZBUCQ
ng0z1BhuRAQzYszmYTh3FDGqWfjw1Jnr12yHfRDJJcQT2yEI4hyMGrqoBUnaKnyn
O+Xt3zTS+0ZIOrZ4AHS8anOmwjSdeIClXS9dpbtVqTE0Ek4j31OIRltG5IMrR7Zz
RSTJTc73EWhVa9A9Ps4H3/qMrCtoXA75465vv0Rm788=
-----END EC PRIVATE KEY-----
maxdsre@jessie:~$ cat /tmp/keyout.pem
-----BEGIN EC PRIVATE KEY-----
MIHcAgEBBEIBmwlqEGFE3l8ir6qLMNliqlsKAAlRR2KF+vkKaXVIRQH4UKvihjZ+
IQOwpGHE09RfU46/e84s/O2Bl/BS6HRLoG6gBwYFK4EEACOhgYkDgYYABABYUvSa
UqfMNH9HUtCK89byq2AhN0u5JYSccGeqotKvAQrh04n6kPPLjRkvZoufAGZ6fSwn
Bd9arQDs4nHC72AWlwASX8jU9Er6qCwfZdlZADzvK9rDU0Vl45akNMjZd2LxejIz
mtzaVJ8QqP99aDmp7CRCUiFlhbdp7VTg9V27XXDsPw==
-----END EC PRIVATE KEY-----

#公鑰比較
maxdsre@jessie:~$ diff /tmp/pubkey{,out}.pem && echo 'same' || echo 'different'
same
maxdsre@jessie:~$ cat /tmp/pubkeyout.pem
-----BEGIN PUBLIC KEY-----
MIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQAWFL0mlKnzDR/R1LQivPW8qtgITdL
uSWEnHBnqqLSrwEK4dOJ+pDzy40ZL2aLnwBmen0sJwXfWq0A7OJxwu9gFpcAEl/I
1PRK+qgsH2XZWQA87yvaw1NFZeOWpDTI2Xdi8XoyM5rc2lSfEKj/fWg5qewkQlIh
ZYW3ae1U4PVdu11w7D8=
-----END PUBLIC KEY-----
maxdsre@jessie:~$
```


### Review Of Private Key Generation
生成私鑰命令彙總

如果需要設置passphrases，但不想出現輸入提示，可通過選項`-passout`、`-pass`實現。

```bash
#RSA
openssl genrsa -out /tmp/key.pem -aes256 4096   #生成私鑰
#openssl rsa -in /tmp/key.pem -out /tmp/keyout.pem  #移除私鑰中pass phrase
#openssl rsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem  #生成公鑰
#openssl rsa -in /tmp/key.pem -text -noout  #查看公鑰、私鑰組件

#DSA
openssl dsaparam -genkey 4096 | openssl dsa -out /tmp/key.pem -aes256   #生成私鑰
#openssl dsa -in /tmp/key.pem -out /tmp/keyout.pem  #移除私鑰中pass phrase
#openssl dsa -in /tmp/key.pem -pubout -out /tmp/pubkey.pem  #生成公鑰
#openssl dsa -in /tmp/dsa_key.pem -text -noout  #查看公鑰、私鑰組件

#ECDSA
openssl ecparam -genkey -name secp384r1 | openssl ec -out /tmp/key.pem -aes256   #生成私鑰
# openssl ec -in /tmp/key.pem -out /tmp/keyout.pem  #移除私鑰中pass phrase
# openssl ec -in /tmp/key.pem -pubout -out /tmp/pubkey.pem  #生成公鑰
# openssl ec -in /tmp/key.pem -text -noout  #查看公鑰、私鑰組件
```


## Certificate Signing Requests(CSR)
基於已生成的私鑰創建CSR文件，該過程通過命令`req`或`x509`實現。該過程是一個交互式樣的過程，需根據提示輸入證書申請著的相關信息。需要填寫的信息如下

Item|CompleteName|Explanation
---|---|---
`C`|Country Name|所處國家
`ST`|State or Province Name|所處州或省
`L`|Locality Name|所處城市
`O`|Organization Name|組織或公司名稱
`OU`|Organizational Unit Name|部分名稱
`CN`|Common Name|網站域名
`emailAddress`|Email Address|聯繫郵箱

當然也可以通過某些手段規避該過程，如設置選項`-subj`或`-config`。

生成CSR的應用場景有2種

1. 生成新的CSR；
2. 基於已存在的證書生成CSR，如證書續期；

此處的填寫信息設置如下

Item|Value
---|---
`C`|CN
`ST`|Shanghai
`L`|Shanghai
`O`|AxdLog
`OU`|DevOps
`CN`|axdlog.com
`emailAddress`|admin@axdlog.com

### Createing New CSR
1. 生成SCR

```bash
#method1
openssl req -new -key /tmp/key.pem -out /tmp/req.csr
# openssl req -new -passin pass:AxdLog2017 -key /tmp/key.pem -out /tmp/req.csr

#method2 using -subj
openssl req -new -key /tmp/key.pem -out /tmp/req.csr -subj "/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=DevOps/CN=axdlog.com/emailAddress=admin@axdlog.com"

#method3 Unattended CSR Generation
#此處 input_password 即生成私鑰時設置的 pass pharse，為 AxdLog2017；如果使用的是移除pass pharse後的私鑰，可將指令 input_password 移除
tee /tmp/override.cnf <<-'EOF'
[ req ]
prompt = no
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

input_password = AxdLog2017

[ req_distinguished_name ]
C                    = CN
ST                   = Shanghai
L                    = Shanghai
O                    = AxdLog
OU                   = DevOps
CN                   = axdlog.com
emailAddress         = admin@axdlog.com

[ req_attributes ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = CA:true
EOF

openssl req -new -config /tmp/override.cnf -key /tmp/key.pem -out /tmp/req.csr
# openssl req -new -config /tmp/override.cnf -passin pass:AxdLog2017 -key /tmp/key.pem -out /tmp/req.csr

```

**注意**： `[ req ]`中設置的`distinguished_name`、`attributes`、`x509_extensions`的值即為之後的區塊(section)的名稱，請仔細查看。具體設置見`man req`。

命令選項說明

`req` - PKCS#10 certificate request and certificate generating utility.

>* `-new` - this option generates a new certificate request. If the `-key` option is not used it will generate a new RSA private key using information specified in the configuration file.
>* `-passin arg` -  the input file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-key filename` - This specifies the file to read the private key from. It also accepts PKCS#8 format private keys for PEM format files.
>* `-subj arg` - Replaces subject field of input request with specified data and outputs modified request.
>* `-config filename` - this allows an alternative configuration file to be specified, this overrides the compile time filename or any specified in the `OPENSSL_CONF` environment variable.


2. 查看證書請求信息

```bash
openssl req -in /tmp/req.csr -noout -text
```

命令選項說明

>* `-text` - prints out the certificate request in text form.
>* `-noout` - this option prevents output of the encoded version of the request.


演示過程(使用ECDSA生成的私鑰)

```bash
#使用ECDSA生成私鑰
maxdsre@jessie:~$ openssl ecparam -genkey -name secp384r1 | openssl ec -out /tmp/key.pem -aes256
read EC key
writing EC key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

#查看配置信息
maxdsre@jessie:~$ cat /tmp/override.cnf
[ req ]
prompt = no
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

input_password = AxdLog2017

[ req_distinguished_name ]
C                    = CN
ST                   = Shanghai
L                    = Shanghai
O                    = AxdLog
OU                   = DevOps
CN                   = axdlog.com
emailAddress         = admin@axdlog.com

[ req_attributes ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = CA:true

#生成CSR
maxdsre@jessie:~$ openssl req -new -config /tmp/override.cnf -key /tmp/key.pem -out /tmp/req.csr

#查看證書請求信息
maxdsre@jessie:/tmp$ openssl req -in /tmp/req.csr -noout -text
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=DevOps, CN=axdlog.com/emailAddress=admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:f6:e0:82:42:72:45:af:6d:7b:35:22:6d:e2:b4:
                    5c:c7:73:c7:13:89:04:b6:37:73:95:1c:3f:f2:df:
                    09:8c:ea:71:67:76:16:0c:03:00:df:7a:ab:ee:47:
                    b4:da:c9:b1:04:c8:a6:92:0c:5e:21:1a:f8:c5:0e:
                    f4:06:0d:36:a3:d6:1c:66:bb:3e:7b:63:fd:3f:fa:
                    1b:9b:6b:3a:18:ef:93:11:28:53:84:56:22:76:0c:
                    ac:e8:ec:79:ca:77:da
                ASN1 OID: secp384r1
        Attributes:
            a0:00
    Signature Algorithm: ecdsa-with-SHA256
         30:65:02:30:38:2e:8e:31:d7:b0:59:d7:e2:13:29:93:ac:cb:
         9a:1e:b9:b7:03:99:28:44:e9:f5:19:b9:f5:b0:1e:a6:23:0c:
         c3:f6:c0:d2:bd:a7:f6:f4:dc:03:91:33:6a:5b:95:61:02:31:
         00:ba:f3:d4:49:74:0c:43:9e:87:05:e5:95:52:9b:29:a2:98:
         80:77:e5:64:6b:6d:87:b7:05:91:4d:0d:a3:29:c0:02:52:35:
         ce:66:d0:70:f6:9f:74:18:fc:08:a7:48:f7
maxdsre@jessie:/tmp$
```

### Creating CSR from Existing Certificates
對於已經簽發的證書，可通過命令`x509`生成新的CSR文件。

操作命令

```bash
openssl x509 -x509toreq -signkey /tmp/key.pem -in /tmp/cert.pem -out /tmp/req.csr
# openssl x509 -x509toreq -passin pass:AxdLog2017 -signkey /tmp/key.pem -in /tmp/cert.pem -out /tmp/req.csr
```

命令選項說明

`x509` - Certificate display and signing utility

>* `-x509toreq` - converts a certificate into a certificate request. The `-signkey` option is used to pass the required private key.
>* `-signkey filename` - this option causes the input file to be self signed using the supplied private key.


**注意**： 在申請新的證書時，通常建議生成新的私鑰以降低私鑰洩漏風險。除非使用了HPKP等機制，需要用到已經存在的私鑰。


## Signing Self-Signed Certificate
可通過命令`x509`或`req`簽署自簽證書，方式較多。

### Creating Certificates Valid for Single Hostname
生成單域名證書

```bash
#method1 需生成csr文件
openssl x509 -req -days 365 -signkey /tmp/key.pem -in /tmp/req.csr -out /tmp/cert.pem
# openssl x509 -req -days 365 -passin pass:AxdLog2017 -signkey /tmp/key.pem -in /tmp/req.csr -out /tmp/cert.pem

#method2 無需生成csr文件，有交互過程
openssl req -new -x509 -days 365 -key /tmp/key.pem -out /tmp/cert.pem
# openssl req -new -x509 -days 365 -passin pass:AxdLog2017 -key /tmp/key.pem -out /tmp/cert.pem

#method3 無需生成csr文件，使用-subj指令，無交互過程
openssl req -new -x509 -days 365 -key /tmp/key.pem -out /tmp/cert.pem -subj "/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=DevOps/CN=axdlog.com/emailAddress=admin@axdlog.com"

#method4 使用指令-config
openssl req -new -x509 -days 365 -config /tmp/override.cnf -key /tmp/key.pem -out /tmp/cert.pem
# openssl req -new -x509 -days 365 -config /tmp/override.cnf -passin pass:AxdLog2017 -key /tmp/key.pem -out /tmp/cert.pem

#-x509: this option outputs a self signed certificate instead of a certificate request. This is typically used to generate a test certificate or a self signed root CA. The extensions added to the certificate (if any) are specified in the configuration file. Unless specified using the set_serial option 0 will be used for the serial number. 用於生成自簽證書
```

演示示例

```bash
maxdsre@jessie:/tmp$ openssl x509 -in /tmp/cert.pem -noout -textCertificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            a4:ff:ed:4e:5b:58:09:6f
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=DevOps, CN=axdlog.com/emailAddress=admin@axdlog.com
        Validity
            Not Before: Jan  9 08:43:16 2017 GMT
            Not After : Jan  9 08:43:16 2018 GMT
        Subject: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=DevOps, CN=axdlog.com/emailAddress=admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:f6:e0:82:42:72:45:af:6d:7b:35:22:6d:e2:b4:
                    5c:c7:73:c7:13:89:04:b6:37:73:95:1c:3f:f2:df:
                    09:8c:ea:71:67:76:16:0c:03:00:df:7a:ab:ee:47:
                    b4:da:c9:b1:04:c8:a6:92:0c:5e:21:1a:f8:c5:0e:
                    f4:06:0d:36:a3:d6:1c:66:bb:3e:7b:63:fd:3f:fa:
                    1b:9b:6b:3a:18:ef:93:11:28:53:84:56:22:76:0c:
                    ac:e8:ec:79:ca:77:da
                ASN1 OID: secp384r1
    Signature Algorithm: ecdsa-with-SHA256
         30:65:02:31:00:eb:af:d4:be:ce:2d:7b:cc:f1:05:ce:da:cb:
         d9:c1:d5:0f:7f:39:22:ee:6a:c2:4e:e7:ab:aa:4e:78:a5:85:
         91:c0:5a:2f:e8:a9:57:a7:a9:93:ae:12:77:3c:9c:4c:d1:02:
         30:00:bf:48:9e:46:4d:97:41:78:80:90:99:c7:b1:27:e2:17:
         6d:88:88:c9:93:c5:84:44:b3:50:16:21:5f:26:80:c0:6c:88:
         57:ee:76:05:f0:62:d4:11:56:b3:59:ad:66
maxdsre@jessie:/tmp$
```

### Creating Certificates Valid for Multiple Hostnames
生成通配符證書，如`*.axdlog.com`

要使證書支持多域名，可通過2中機制實現

1. 使用`X.509`擴展中的`Subject Alternative Name` (SAN)，具體說明見`man x509v3_config`；
2. 使用`wildcards`(通配符)；

通常是2種混合使用，在實際操作中，指定一個二級域名和一個通配符域名，如`axdlog.com`和`*.axdlog.com`。

**注意**: `X509 V3 certificate extension`有很多指令，具體說明可通過`man x509v3_config`查看。

>x509v3_config - X509 V3 certificate extension configuration format

操作命令

```bash
#-extfile filename: file containing certificate extensions to use. If not specified then no extensions are added to the certificate.

tee /tmp/extension.cnf <<-'EOF'
subjectAltName = DNS:*.axdlog.com, DNS:axdlog.com
EOF

openssl x509 -req -days 365 -signkey /tmp/key.pem -in /tmp/req.csr -out /tmp/cert.pem -extfile /tmp/extension.cnf
# openssl x509 -req -days 365 -signkey /tmp/key.pem -passin pass:AxdLog2017 -in /tmp/req.csr -out /tmp/cert.pem -extfile /tmp/extension.cnf

```

演示示例

```bash
maxdsre@jessie:/tmp$ cat /tmp/extension.cnf
subjectAltName = DNS:*.axdlog.com, DNS:axdlog.com

#生成多域名證書
maxdsre@jessie:/tmp$ openssl x509 -req -days 365 -signkey /tmp/key.pem -in /tmp/req.csr -out /tmp/cert.pem -extfile /tmp/extension.cnf
Signature ok
subject=/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=DevOps/CN=axdlog.com/emailAddress=admin@axdlog.com
Getting Private key
Enter pass phrase for /tmp/key.pem:

#查看證書內容
maxdsre@jessie:/tmp$ openssl x509 -in /tmp/cert.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            d1:72:f7:4c:86:66:1b:a4
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=DevOps, CN=axdlog.com/emailAddress=admin@axdlog.com
        Validity
            Not Before: Jan  9 08:37:04 2017 GMT
            Not After : Jan  9 08:37:04 2018 GMT
        Subject: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=DevOps, CN=axdlog.com/emailAddress=admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:f6:e0:82:42:72:45:af:6d:7b:35:22:6d:e2:b4:
                    5c:c7:73:c7:13:89:04:b6:37:73:95:1c:3f:f2:df:
                    09:8c:ea:71:67:76:16:0c:03:00:df:7a:ab:ee:47:
                    b4:da:c9:b1:04:c8:a6:92:0c:5e:21:1a:f8:c5:0e:
                    f4:06:0d:36:a3:d6:1c:66:bb:3e:7b:63:fd:3f:fa:
                    1b:9b:6b:3a:18:ef:93:11:28:53:84:56:22:76:0c:
                    ac:e8:ec:79:ca:77:da
                ASN1 OID: secp384r1
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:*.axdlog.com, DNS:axdlog.com
    Signature Algorithm: ecdsa-with-SHA256
         30:65:02:30:2d:34:7e:ed:a0:99:b8:e8:00:30:d8:32:79:6f:
         89:0d:2b:ac:41:a3:39:df:99:28:19:06:b4:c3:c2:48:ae:5c:
         5f:7d:5b:e9:87:1a:9e:a5:e4:d3:e5:b4:3e:b9:0d:43:02:31:
         00:f8:2a:fc:11:39:32:27:ee:96:98:af:51:24:ac:27:19:d8:
         51:51:7a:d0:80:81:6a:e1:95:87:6d:a7:97:1d:3e:10:08:18:
         24:e5:6b:95:58:63:23:16:b8:c5:81:63:96
maxdsre@jessie:/tmp$
```


### Display Certificate Info
通過如下命令查看證書內容

```bash
#Display the contents of a certificate
openssl x509 -in /tmp/cert.pem -noout -text

#Display the certificate serial number
openssl x509 -in /tmp/cert.pem -noout -serial

#Display the certificate subject name
openssl x509 -in /tmp/cert.pem -noout -subject

#Display the certificate MD5 fingerprint
openssl x509 -in /tmp/cert.pem -noout -fingerprint

#Display the certificate SHA1 fingerprint
openssl x509 -sha1 -in /tmp/cert.pem -noout -fingerprint
```


## Testing In Browsers
在本地LEMP開發環境中配置生成的SSL證書，通過瀏覽器查看

### Preparation
生成自簽證書

```bash
maxdsre@jessie:~$ cat /tmp/passphrases
AxdLog2017
maxdsre@jessie:~$ cat /tmp/override.cnf
[ req ]
prompt = no
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

input_password = AxdLog2017

[ req_distinguished_name ]
C                    = CN
ST                   = Shanghai
L                    = Shanghai
O                    = AxdLog
OU                   = DevOps
CN                   = axdlog.com
emailAddress         = admin@axdlog.com

[ req_attributes ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = CA:true

#生成私鑰
maxdsre@jessie:~$ openssl ecparam -genkey -name secp384r1 | openssl ec -out /tmp/key.pem -aes256
read EC key
writing EC key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

#創建CSR
maxdsre@jessie:~$ openssl req -new -config /tmp/override.cnf -key /tmp/key.pem -out /tmp/req.csr

#簽署證書
maxdsre@jessie:~$ openssl req -new -x509 -days 365 -config /tmp/override.cnf -key /tmp/key.pem -out /tmp/cert.pem

#文件查看
maxdsre@jessie:~$ ls -lhF /tmp/
total 24K
-rw-r--r-- 1 maxdsre maxdsre 1.3K Jan  9 16:54 cert.pem
-rw-r--r-- 1 maxdsre maxdsre   60 Jan  9 16:36 extension.cnf
-rw-r--r-- 1 maxdsre maxdsre  379 Jan  9 16:54 key.pem
-rw-r--r-- 1 maxdsre maxdsre  551 Jan  9 15:32 override.cnf
-rw-r--r-- 1 maxdsre maxdsre   16 Jan  9 16:04 passphrases
-rw-r--r-- 1 maxdsre maxdsre  623 Jan  9 16:54 req.csr
maxdsre@jessie:~$
```

Nginx配置，文件`/etc/nginx/conf.d/default.conf`，內容如下(簡易配置)
```bash
server {
    listen 80;
    server_name localhost;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
	server_name localhost;
    root   /usr/share/nginx/html;
    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
    }

	location ~ \.php$ {
    	try_files $uri = 404;
    	root   /usr/share/nginx/html;
    	fastcgi_pass unix:/var/run/php/php5-fpm.sock;
    	fastcgi_index index.php;
    	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    	include fastcgi_params;
	}

    #重要參數，此處引用相關文件
    ssl_certificate /tmp/cert.pem;
	ssl_certificate_key /tmp/key.pem;
	ssl_password_file /tmp/passphrases;

    #ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305 ECDHE-RSA-CHACHA20-POLY1305 EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH DHE-RSA-CHACHA20-POLY1305 EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4 !SEED !CAMELLIA";
    ssl_ecdh_curve secp384r1; # Specifies a curve for ECDHE ciphers.
    #ssl_session_cache    shared:SSL:10m;
    #ssl_session_timeout  10m;
    #ssl_session_tickets on;
    #ssl_session_ticket_key /etc/nginx/ssl/ticket.key;
    # Google DNS, Open DNS, Dyn DNS
    resolver 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 216.146.35.35 216.146.36.36 valid=300s;
    resolver_timeout 5s;
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
    add_header Content-Security-Policy 'default-src self';
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY";
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
}
```

Nginx語法測試

```bash
sudo nginx -t
sudo nginx -s reload
```

啟動Nginx、PHP-FPM服務

```bash
sudo systemctl start nginx
sudo systemctl start php5-fpm
```

### Via Mozilla Firefox
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:38.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:45.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:50.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:55.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:13.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:19.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:50.png)


### Via Google Chrome

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:21.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:26.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:31.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:45.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:04:35.png)


## Bibliography
* [Survival guides - TLS/SSL and SSL (X.509) Certificates](http://www.zytrax.com/tech/survival/ssl.html)
[Why I don't Use 2048 or 4096 RSA Key Sizes](https://blog.josefsson.org/2016/11/03/why-i-dont-use-2048-or-4096-rsa-key-sizes/)

## Reference
* [Creating a Self-Signed SSL Certificate](https://devcenter.heroku.com/articles/ssl-certificate-self 'Heroku')
* [How to create a self-signed SSL Certificate](http://www.akadia.com/services/ssh_test_certificate.html)


## Change Logs
* 2017.01.09 17:06 Mon Asia/Shanghai
    * 初稿完成
* 2017.01.09 21:25 Mon Asia/Shanghai
    * 添加選項`-passout`、`-passin`、`-pass`用於設置passphrases值。
* 2017.01.11 11:14 Wed Asia/Shanghai
    * 添加Bibliography `Why I don't Use 2048 or 4096 RSA Key Sizes`
* 2018.07.27 18:26:30 Fri America/Boston
    * 勘誤，更新，遷移到新Blog


<!-- End -->
