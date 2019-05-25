---
title: Using OpenSSL To Create X.509 Self-Signed Certificate On GNU/Linux
slug: Using OpenSSL To Create X509 Self-Signed Certificate On GNU Linux
date: 2017-01-09T17:16:09+08:00
lastmod: 2019-01-25T20:45:09-0500
draft: false
keywords: ["openssl"]
description: "Try to create X.509 self-signed certificate via OpenSSL On GNU/Linux"
categories:
- Security
tags:
- OpenSSL
- X.509
comment: true
toc: true

---

[OpenSSL][openssl]是一個提供`SSL/TLS`工具集和加密庫的開源項目，其中一項功能是創建基於 [X.509](https://en.wikipedia.org/wiki/X.509 'WikiPedia') 標準的數字證書(Digital Certificate)。數字證書可由[CA](https://en.wikipedia.org/wiki/Certificate_authority 'WikiPedia')機構(Certificate Authoriy)簽發，也可由自己簽發。自己簽發的證書稱為自簽證書([Self-Signed Certificate](https://en.wikipedia.org/wiki/Self-signed_certificate))，本文記錄使用OpenSSL創建自簽證書的過程。

<!--more-->

本文大部分內容梳理自 [Ivan Ristic](https://twitter.com/ivanristic 'Twitter') 的 [OpenSSL Cookbook](https://www.feistyduck.com/books/openssl-cookbook/) (Last update: March 2016)，推薦閱讀。

[OpenSSL][openssl]的命令介紹可參考本人Blog [A Brief Introduction Of OpenSSL]({{< relref "2017-01-08-Simple-Introduction-Of-OpenSSL.md" >}})。


## Introduction
數字證書又稱為`Digital Certificate`(或`Identity Certificate`或`Public Key Certificate`)，採用非對稱加密(又名公鑰加密 [Public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography 'WikiPedia'))，用於證明公鑰(public key)的所有者。證書中含有公鑰信息、所有者身份信息、簽發者的數字簽名等信息。證書是`SSL/TLS`的重要組成部分，用於保證網絡通信安全。通常所說的SSL證書即數字證書。

數字證書可由[OpenSSL](https://www.openssl.org/)創建，基於[X.509](https://en.wikipedia.org/wiki/X.509 'WikiPedia')標準，具體見[ITU-T X.509](http://www.itu.int/ITU-T/recommendations/rec.aspx?rec=13031 'ITU')。

需注意，與CA簽發的證書相比，自簽證書有安全隱患，建議根據使用場景選擇。具體參見 [The Dangers of Self-Signed SSL Certificates](https://www.globalsign.com/en/ssl-information-center/dangers-self-signed-certificates/ 'GlobalSign')。


## Conventions
操作平臺信息

item|version details
---|---
os | Debian GNU/Linux 9.8 (stretch)
kernel | 4.9.0-8-amd64
openssl | OpenSSL 1.1.0j  20 Nov 2018


## Variables Definition
文本中生成的文件定義如下

type | suffix | explanation
---|---|---
Private Key | .key | private key
Public Key | .pubkey | public key (generated from private key)
CSR | .csr | Certificate Signing Request
Cert | .crt | certificate


操作中使用到的變量

```bash
# Working directory
work_dir=$(mktemp -d -t XXXXXX)
common_name='axdlog'
pass_phrase="AxdLog@$(date +'%Y')"

# Key
key_len=4096

# Days
key_days=365

# Country Name
cert_C='CN'
# State or Province Name
cert_ST='Shanghai'
# Locality Name
cert_L='Shanghai'
# Organization Name
cert_O='AxdLog'
# Organizational Unit Name
cert_OU='DevSecOps'
# Common Name
cert_CN='axdlog.com'
# Email Address
cert_email='admin@axdlog.com'
```

## Private Key Generation
對於數字證書而言，私鑰起著非常重要的作用。**私鑰生成算法**(Key Algorithm)和 **私鑰長度**(Key size)決定了數字證書的安全程度。目前主要有3中私鑰生成算法：[RSA](https://simple.wikipedia.org/wiki/RSA_algorithm 'WikiPedia')、[DSA](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm 'WikiPedia')、[ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm 'WikiPedia')。

創建自簽證書需經歷3個主要步驟：

1. 生成私鑰(Private Key)；
2. 生成證書簽署請求文件CSR(Certificate Signing Request)；
3. 生成自簽證書(Self-Signed Certificate)；

但不一定要嚴格遵守，在實際操作過程中，可使用`req`命令，直接通過私鑰生成自簽證書。

私鑰長度的選擇，取決於算法。對於`RSA`、`DSA`，建議使用`2048`bits及以上，推薦`4096`bits；對於`ECDSA`，建議使用`256`bits及以上的長度。

生成過程中會出現`Passphrase`提示，建議設置該選項，如果直接按`Enter`鍵跳過，在某些OpenSSL版本中會出現如下報錯信息

>140203959170704:error:28069065:lib(40):UI_set_result:result too small:ui_lib.c:823:You must type in 4 to 1023 characters

如果需要設置，但不想出現此類報錯提示，可通過選項`-passout`、`-passin`實現。如果確實不需要`pass phrase`，可在私鑰生成後，通過命令將其移除。

以下分別列出3種算法的使用命令，相關命令的具體使用可通過`man command`查看，如`man genrsa`。

### RSA Key
對於`RSA`

* 使用命令`genrsa`生成私鑰；
* 使用命令`rsa`輸出公鑰、私鑰組件或提取公鑰；

#### RSA Private Key
生成私鑰

```bash
# interactive mode (Enter pass phrase)
openssl genrsa --out "${work_dir}/${common_name}.key" -aes256 "${key_len}"

# Or quiet mode via '-passout'
# openssl genrsa -passout pass:"${pass_phrase}" --out "${work_dir}/${common_name}.key" -aes256 "${key_len}"
```

命令選項說明

>openssl-genrsa, genrsa - generate an RSA private key
>
>* `-passout arg` - the output file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-out filename` - the output filename.
>* `-aes256` - encrypt the private key with specified cipher before outputting it.
>* `numbits` - the size of the private key to generate in bits. This must be the last option specified. The default is 512. 該選項必須在最後位置

**注意**：不建議使用`DES`、`3DES`、`SEED`等算法。

#### Remove Pass Phrase
移除RSA私鑰中的`pass phrase`(可選)

```bash
# interactive mode
openssl rsa -inform PEM -outform PEM -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"

# Or quiet mode
# openssl rsa -inform PEM -outform PEM -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}_out.key"
```

比較二者，含有pass phrase的私鑰多了`Proc-Type`、`DEK-Info`兩行

```bash
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,42A202431D6BEB6B219ECE33E25333CE
```

#### Extract Public Key
從私鑰中提取公鑰

```bash
# interactive mode
openssl rsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout

# Or quiet mode
# openssl rsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout
# openssl rsa -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout
```

命令選項說明

>openssl-rsa, rsa - RSA key processing tool
>
>* `-pubout` - by default a private key is output: with this option a public key will be output instead. This option is automatically set if the input is a public key.
>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` -  This specifies the output filename to write a key to or standard output if this option is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.

#### Keys Info
輸出公鑰、私鑰組件

```bash
# interactive mode
openssl rsa -in "${work_dir}/${common_name}.key" -text -noout

# Or quiet mode
# openssl rsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout
# openssl rsa -in "${work_dir}/${common_name}_out.key" -text -noout
```

命令選項說明

>* `-text` - prints out the various public or private key components in plain text in addition to the encoded version.
>* `-noout` - this option prevents output of the encoded version of the key.


### DSA Key
對於`DSA`，須首先生成DSA參數(DSA parameters)

* 使用命令`dsaparam`生成DSA參數；
* 使用命令`dsa`或`genpkey`生成私鑰；

#### DSA Private Key
生成私鑰 (方式較多)

```bash   
# - method1: directly
openssl dsaparam -genkey "${key_len}" | openssl dsa -out "${work_dir}/${common_name}.key" -aes256
# openssl dsaparam -genkey "${key_len}" | openssl dsa -passout pass:"${pass_phrase}" -out "${work_dir}/${common_name}.key" -aes256

# - method2: via dsaparam + dsa
openssl dsaparam -out "${work_dir}/${common_name}_param.key" -genkey "${key_len}"
openssl dsa -in "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -aes256
# openssl dsa -in "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -passout pass:"${pass_phrase}" -aes256

# - method3: via dsaparam + genpkey
openssl dsaparam -out "${work_dir}/${common_name}_param.key" -genkey "${key_len}"
openssl genpkey -paramfile "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -aes256
# openssl genpkey -paramfile "${work_dir}/${common_name}_param.key" -pass pass:"${pass_phrase}" -out "${work_dir}/${common_name}.key" -aes256
```

命令選項說明

>openssl-dsaparam, dsaparam - DSA parameter manipulation and generation
>
>* `-genkey` - this option will generate a DSA either using the specified or generated parameters.
>* `-out filename` -  This specifies the output filename parameters to. Standard output is used if this option is not present. The output filename should not be the same as the input filename.
>* `numbits` - this option specifies that a parameter set should be generated of size numbits. It must be the last option. If this option is included then the input file (if any) is ignored.

>openssl-dsa, dsa - DSA key processing
>
>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` -  This specifies the output filename to write a key to or standard output by is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.
>* `-aes256` - encrypt the private key with specified cipher before outputting it.

>openssl-genpkey, genpkey - generate a private key
>
>* `-out filename` - the output filename. If this argument is not specified then standard output is used.
>* `-paramfile filename` -  Some public key algorithms generate a private key based on a set of parameters.  They can be supplied using this option. If this option is used the public key algorithm used is determined by the parameters. If used this option must precede and `-pkeyopt` options. The options `-paramfile` and `-algorithm` are mutually exclusive.
>* `-pass arg` -  the output file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-cipher` - This option encrypts the private key with the supplied cipher.

#### Remove Pass Phrase
移除DSA私鑰中的`pass phrase`(可選)

```bash
# interactive mode
openssl dsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"

# Or quiet mode
# openssl dsa -passin pass:"${pass_phrase}" -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"
```

比較二者，含有pass phrase的私鑰多了`Proc-Type`、`DEK-Info`兩行

```bash
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,19A902D1681D5079DCF0EB070C8A6263
```

#### Extract Public Key
提取公鑰

```bash
# interactive mode
openssl dsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout

# Or quiet mode
# openssl dsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout
# openssl dsa -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout
```

命令選項說明

>* `-pubout` - by default a private key is output. With this option a public key will be output instead. This option is automatically set if the input is a public key.


#### Keys Info
提取私鑰組件

```bash
# interactive mode
openssl dsa -in "${work_dir}/${common_name}.key" -text noout

# Or quiet mode
# openssl dsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout
# openssl dsa -in "${work_dir}/${common_name}_out.key" -text -noout
```

命令選項說明

>* `-text` - prints out the public, private key components and parameters.
>* `-noout` - this option prevents output of the encoded version of the key.


### ECDSA Key
`ECDSA`是 *Elliptic Curve Digital Signature Algorithm* 的縮寫，即 **橢圓曲線數字簽名算法**。要生成私鑰，須首先生成EC參數(EC parameters)。生成EC參數時需指定 *命名曲線*(named curves)，OpenSSL中有多種 *命名曲線* 可供選擇。執行如下命令查看可供選擇的 *命名曲線*:

```bash
# list_curves - get a list of all currently implemented EC parameters
openssl ecparam -list_curves

# get short name
openssl ecparam -list_curves | sed -r -n '/:/{s@^[[:space:]]*([^:]+):.*$@\1@g;p}'
```

對於 Web Server 而言，建議使用 `prime256v1`(secp256r1), `secp384r1`, `secp521r1` 這三種EC參數，具體見[Security/Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS)。

相關操作命令

* 使用命令`ecparam`生成EC參數；
* 使用命令`ec`或`genpkey`生成私鑰；


#### ECDSA Private Key
生成私鑰 (方式較多)

```bash
# prime256v1 (secp256r1), secp384r1, secp521r1
curve_choose='secp521r1'

# - method1: directly
openssl ecparam -genkey -name "${curve_choose}" | openssl ec -out "${work_dir}/${common_name}.key" -aes256
# openssl ecparam -genkey -name "${curve_choose}" | openssl ec -out "${work_dir}/${common_name}.key" -passout pass:"${pass_phrase}" -aes256

# - method2: via ecparam + ec
openssl ecparam -out "${work_dir}/${common_name}_param.key" -name "${curve_choose}" -genkey
openssl ec -in "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -aes256
# openssl ec -in "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -passout pass:"${pass_phrase}" -aes256

# - method3: via ecparam + genpkey
openssl ecparam -out "${work_dir}/${common_name}_param.key" -name "${curve_choose}"
openssl genpkey -paramfile "${work_dir}/${common_name}_param.key" -out "${work_dir}/${common_name}.key" -aes256
# openssl genpkey -paramfile "${work_dir}/${common_name}_param.key" -pass pass:"${pass_phrase}" -out "${work_dir}/${common_name}.key" -aes256
```

命令選項說明

>openssl-ecparam, ecparam - EC parameter manipulation and generation
>
>* `-genkey` - This option will generate a EC private key using the specified parameters.
>* `-name arg` - Use the EC parameters with the specified 'short' name. Use `-list_curves` to get a list of all currently implemented EC parameters.

> openssl-ec, ec - EC key processing
>
>* `-in filename` - This specifies the input filename to read a key from or standard input if this option is not specified. If the key is encrypted a pass phrase will be prompted for.
>* `-out filename` - This specifies the output filename to write a key to or standard output by is not specified. If any encryption options are set then a pass phrase will be prompted for. The output filename should not be the same as the input filename.

#### Remove Pass Phrase
移除EC私鑰中的`pass phrase`(可選)

```bash
# interactive mode
openssl ec -in "${work_dir}/${common_name}.key" "${work_dir}/${common_name}_out.key"

# Or quiet mode
# openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}_out.key"
```

比較二者，含有pass phrase的私鑰多了`Proc-Type`、`DEK-Info`兩行

```bash
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,07D6A97D5C869B941970681DF3507F5E
```

#### Extract Public Key
提取公鑰

```bash
# interactive mode
openssl ec -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout

# Or quiet mode
# openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout
# openssl ec -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout
```

#### Keys Info
提取私鑰組件

```bash
# interactive mode
openssl ec -in "${work_dir}/${common_name}.key" -text noout

# Or quiet mode
# openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout
# openssl ec -in "${work_dir}/${common_name}_out.key" -text -noout
```

命令選項說明

>* `-text` - prints out the public, private key components and parameters.
>* `-noout` - this option prevents output of the encoded version of the key.



## Review Private Key Generation
生成私鑰命令彙總

如果需要設置 `passphrases`，但不想出現輸入提示，可通過選項`-passout`、`-pass`實現。

#### Interactive Mode
```bash
# - RSA
openssl genrsa --out "${work_dir}/${common_name}.key" -aes256 "${key_len}"  #生成私鑰
openssl rsa -inform PEM -outform PEM -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase
openssl rsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
openssl rsa -in "${work_dir}/${common_name}.key" -text -noout  #查看私鑰組件

# - DSA
openssl dsaparam -genkey "${key_len}" | openssl dsa -out "${work_dir}/${common_name}.key" -aes256  #生成私鑰
openssl dsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase
openssl dsa -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
openssl dsa -in "${work_dir}/${common_name}.key" -text noout  #查看私鑰組件

# - ECDSA
curve_choose='secp521r1'
openssl ecparam -genkey -name "${curve_choose}" | openssl ec -out "${work_dir}/${common_name}.key" -aes256  #生成私鑰
openssl ec -in "${work_dir}/${common_name}.key" "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase
openssl ec -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
openssl ec -in "${work_dir}/${common_name}.key" -text noout  #查看私鑰組件
```


#### Quiet Mode
```bash
# - RSA
openssl genrsa -passout pass:"${pass_phrase}" --out "${work_dir}/${common_name}.key" -aes256 "${key_len}"  #生成私鑰
openssl rsa -inform PEM -outform PEM -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase

openssl rsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
# openssl rsa -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout

openssl rsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout  #查看私鑰組件
# openssl rsa -in "${work_dir}/${common_name}_out.key" -text -noout

# - DSA
openssl dsaparam -genkey "${key_len}" | openssl dsa -passout pass:"${pass_phrase}" -out "${work_dir}/${common_name}.key" -aes256  #生成私鑰
openssl dsa -passin pass:"${pass_phrase}" -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase

openssl dsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
# openssl dsa -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout

openssl dsa -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout  #查看私鑰組件
# openssl dsa -in "${work_dir}/${common_name}_out.key" -text -noout

# - ECDSA
curve_choose='secp521r1'
openssl ecparam -genkey -name "${curve_choose}" | openssl ec -out "${work_dir}/${common_name}.key" -passout pass:"${pass_phrase}" -aes256  #生成私鑰
openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}_out.key"  #移除私鑰中pass phrase

openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -out "${work_dir}/${common_name}.pubkey" -pubout  #生成公鑰
# openssl ec -in "${work_dir}/${common_name}_out.key" -out "${work_dir}/${common_name}.pubkey" -pubout

openssl ec -in "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -text -noout  #查看私鑰組件
# openssl ec -in "${work_dir}/${common_name}_out.key" -text -noout
```


## Certificate Signing Requests(CSR)
基於已生成的私鑰創建CSR文件，該過程通過命令`req`或`x509`實現。該過程是一個交互式樣的過程，需根據提示輸入證書申請著的相關信息。

需要填寫的信息如下

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


### Createing New CSR
#### Generation
生成新的CSR

```bash
# - method1 (interactive mode)
openssl req -new -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr"
# openssl req -new -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr"

# - method2 using -subj
openssl req -new -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"
# openssl req -new -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"

# - method3 Unattended CSR Generation
#此處 input_password 即生成私鑰時設置的 pass pharse，為 ${pass_phrase}；如果使用的是移除pass pharse後的私鑰，可將指令 input_password 移除
tee "${work_dir}/csr_override.cnf" 1>/dev/null << EOF
[ req ]
prompt = no
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

input_password = ${pass_phrase}

[ req_distinguished_name ]
C                    = ${cert_C}
ST                   = ${cert_ST}
L                    = ${cert_L}
O                    = ${cert_O}
OU                   = ${cert_OU}
CN                   = ${cert_CN}
emailAddress         = ${cert_email}

[ req_attributes ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = CA:true
EOF

openssl req -new -config "${work_dir}/csr_override.cnf" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr"
# openssl req -new -config "${work_dir}/csr_override.cnf" -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.csr"
```

**注意**： `[ req ]`中設置的`distinguished_name`、`attributes`、`x509_extensions`的值即為之後的區塊(section)的名稱，須注意。具體設置見`man req`。

命令選項說明

>openssl-req, req - PKCS#10 certificate request and certificate generating utility
>
>* `-new` - this option generates a new certificate request. If the `-key` option is not used it will generate a new RSA private key using information specified in the configuration file.
>* `-passin arg` -  the input file password source. For more information about the format of arg see the *PASS PHRASE ARGUMENTS* section in openssl(1).
>* `-key filename` - This specifies the file to read the private key from. It also accepts PKCS#8 format private keys for PEM format files.
>* `-subj arg` - Replaces subject field of input request with specified data and outputs modified request.
>* `-config filename` - this allows an alternative configuration file to be specified, this overrides the compile time filename or any specified in the `OPENSSL_CONF` environment variable.


#### View CSR Info
查看證書請求信息

```bash
openssl req -in "${work_dir}/${common_name}.csr" -noout -text
```

命令選項說明

>* `-text` - prints out the certificate request in text form.
>* `-noout` - this option prevents output of the encoded version of the request.


#### Example
演示過程(使用ECDSA生成的私鑰)

```bash
# openssl req -in "${work_dir}/${common_name}.csr" -noout -text
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = DevSecOps, CN = axdlog.com, emailAddress = admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (521 bit)
                pub:
                    04:01:15:64:e9:0a:d4:be:da:5c:19:ca:ef:36:cc:
                    b3:02:69:e5:d1:2f:be:29:4b:3f:d9:fd:b5:96:ba:
                    bf:59:0a:89:19:84:fb:5b:0c:30:e4:55:91:45:1c:
                    45:c3:f1:6a:77:d7:48:3b:1e:b4:4b:3f:2e:e3:eb:
                    ed:af:3e:24:3f:07:95:00:4c:88:11:1c:42:24:43:
                    f7:36:51:53:48:80:00:08:d4:44:ee:e3:b7:fe:8a:
                    3b:28:46:cc:0e:3e:06:4c:fa:00:8b:eb:98:e4:bf:
                    71:b6:b9:f1:d9:29:2f:a8:ec:17:dd:4a:36:46:3d:
                    15:21:5d:d7:39:d0:54:fb:f2:82:c0:0a:5f
                ASN1 OID: secp521r1
                NIST CURVE: P-521
        Attributes:
            a0:00
    Signature Algorithm: ecdsa-with-SHA256
         30:81:88:02:42:01:10:0f:4d:22:c7:69:83:a5:31:89:09:b4:
         8b:2b:dc:62:c4:91:9a:44:72:ef:f9:57:c3:9b:7e:85:c6:64:
         a0:66:68:d9:54:fa:ce:cc:08:d6:52:00:e6:24:82:bc:61:69:
         5b:33:7a:7c:ba:3e:9e:24:e2:dc:3d:ad:84:73:ab:6b:58:02:
         42:01:50:85:46:a0:45:93:f7:12:88:d3:c6:c2:b0:b9:8d:cb:
         96:63:b1:51:a8:6e:3e:25:e4:46:0a:1f:29:bd:ea:ed:dc:23:
         c0:43:72:97:45:19:c5:56:db:10:b7:b4:ab:0d:50:f7:1c:0b:
         11:64:f3:61:a5:56:28:32:aa:dd:d8:a2:b2
```

### Creating CSR from Existing Certificates
對於已經簽發的證書，可通過命令`x509`生成新的CSR文件。

操作命令

```bash
openssl x509 -x509toreq -signkey "${work_dir}/${common_name}.key" -in "${work_dir}/${common_name}.crt" -out "${work_dir}/${common_name}.csr"
# openssl x509 -x509toreq -passin pass:"${pass_phrase}" -signkey "${work_dir}/${common_name}.key" -in "${work_dir}/${common_name}.crt" -out "${work_dir}/${common_name}.csr"
```

命令選項說明

>openssl-x509, x509 - Certificate display and signing utility
>
>* `-x509toreq` - converts a certificate into a certificate request. The `-signkey` option is used to pass the required private key.
>* `-signkey filename` - this option causes the input file to be self signed using the supplied private key.


**注意**： 在申請新的證書時，通常建議生成新的私鑰以降低私鑰洩漏風險。除非使用了HPKP等機制，需要用到已經存在的私鑰。


## Signing Self-Signed Certificate
可通過命令`x509`或`req`簽署自簽證書，方式較多。

### For Single Hostname
生成單域名證書

```bash
# method1 需生成csr文件
openssl x509 -req -days "${key_days}" -signkey "${work_dir}/${common_name}.key" -in "${work_dir}/${common_name}.csr" -out "${work_dir}/${common_name}.crt"
# openssl x509 -req -days "${key_days}" -passin pass:"${pass_phrase}" -signkey "${work_dir}/${common_name}.key" -in "${work_dir}/${common_name}.csr" -out "${work_dir}/${common_name}.crt"

# method2 無需生成csr文件，有交互過程
openssl req -new -x509 -days "${key_days}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt"
# openssl req -new -x509 -days "${key_days}" -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt"

# method3 無需生成csr文件，使用-subj指令
openssl req -new -x509 -days "${key_days}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"
# openssl req -new -x509 -days "${key_days}" -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"

# method4 使用指令 -config，無交互過程
openssl req -new -x509 -days "${key_days}" -config "${work_dir}/csr_override.cnf" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt"
# openssl req -new -x509 -days "${key_days}" -config "${work_dir}/csr_override.cnf" -passin pass:"${pass_phrase}" -key "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt"

#-x509: this option outputs a self signed certificate instead of a certificate request. This is typically used to generate a test certificate or a self signed root CA. The extensions added to the certificate (if any) are specified in the configuration file. Unless specified using the set_serial option 0 will be used for the serial number. 用於生成自簽證書
```

查看已簽署證書的詳細信息
```bash
openssl x509 -in "${work_dir}/${common_name}.crt" -noout -text
```

演示示例

```bash
# openssl x509 -in "${work_dir}/${common_name}.crt" -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ba:17:8b:2a:18:ec:82:f1
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = DevSecOps, CN = axdlog.com, emailAddress = admin@axdlog.com
        Validity
            Not Before: Jan 26 01:21:59 2019 GMT
            Not After : Jan 26 01:21:59 2020 GMT
        Subject: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = DevSecOps, CN = axdlog.com, emailAddress = admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (521 bit)
                pub:
                    04:01:15:64:e9:0a:d4:be:da:5c:19:ca:ef:36:cc:
                    b3:02:69:e5:d1:2f:be:29:4b:3f:d9:fd:b5:96:ba:
                    bf:59:0a:89:19:84:fb:5b:0c:30:e4:55:91:45:1c:
                    45:c3:f1:6a:77:d7:48:3b:1e:b4:4b:3f:2e:e3:eb:
                    ed:af:3e:24:3f:07:95:00:4c:88:11:1c:42:24:43:
                    f7:36:51:53:48:80:00:08:d4:44:ee:e3:b7:fe:8a:
                    3b:28:46:cc:0e:3e:06:4c:fa:00:8b:eb:98:e4:bf:
                    71:b6:b9:f1:d9:29:2f:a8:ec:17:dd:4a:36:46:3d:
                    15:21:5d:d7:39:d0:54:fb:f2:82:c0:0a:5f
                ASN1 OID: secp521r1
                NIST CURVE: P-521
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                F9:6C:6A:D1:A5:4A:04:93:33:6E:45:25:F0:58:DF:61:DB:67:50:31
            X509v3 Authority Key Identifier:
                keyid:F9:6C:6A:D1:A5:4A:04:93:33:6E:45:25:F0:58:DF:61:DB:67:50:31
                DirName:/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=DevSecOps/CN=axdlog.com/emailAddress=admin@axdlog.com
                serial:BA:17:8B:2A:18:EC:82:F1

            X509v3 Basic Constraints:
                CA:TRUE
    Signature Algorithm: ecdsa-with-SHA256
         30:81:88:02:42:00:dd:d8:ca:fd:9f:e7:90:d3:76:8b:f1:73:
         2b:77:bc:60:74:e2:fc:b5:b0:17:0a:84:90:e0:ee:e2:41:a2:
         cf:ec:77:e0:bf:d2:e4:65:af:bc:43:6e:e0:0f:27:a3:10:34:
         cf:22:66:6b:e3:70:55:81:ca:e7:d3:88:57:28:76:be:96:02:
         42:00:9b:03:e5:58:29:37:8f:e7:ff:38:5e:04:ce:fd:ca:d6:
         a5:ca:ed:b8:7d:e1:9e:5f:0b:90:c0:45:c3:55:53:a2:c0:91:
         d0:3f:52:7e:23:df:62:69:07:99:92:14:fb:92:3a:4d:dd:f4:
         70:42:c3:2c:11:61:09:0f:d6:95:4d:58:10
```

### For Multiple Hostnames
生成通配符證書，如`*.axdlog.com`。

要使證書支持多域名，可通過2種機制實現

1. 使用`X.509`擴展中的`Subject Alternative Name` ([SAN](https://en.wikipedia.org/wiki/Subject_Alternative_Name))，具體說明見`man x509v3_config`；
    * The subject alternative name extension allows various literal values to be included in the configuration file. These include `email` (an email address) URI a uniform resource indicator, `DNS` (a DNS domain name), `RID` (a registered ID: OBJECT IDENTIFIER), `IP` (an IP address), `dirName` (a distinguished name) and otherName.

2. 使用`wildcards`(通配符)；

通常是2種混合使用，在實際操作中，指定一個二級域名和一個通配符域名，如`axdlog.com`和`*.axdlog.com`。

**注意**: `X509 V3 certificate extension`有很多指令，具體說明可通過`man x509v3_config`查看。

>x509v3_config - X509 V3 certificate extension configuration format

操作命令

```bash
# - Method 1 -- openssl x509
#-extfile filename: file containing certificate extensions to use. If not specified then no extensions are added to the certificate.
# -extfile is only used by x509

tee "${work_dir}/SAN.cnf" 1>/dev/null << EOF
subjectAltName = DNS:*.axdlog.com, DNS:axdlog.com, IP:127.0.0.1
EOF

openssl x509 -req -days "${key_days}" -signkey "${work_dir}/${common_name}.key" -in "${work_dir}/${common_name}.csr" -out "${work_dir}/${common_name}.crt" -extfile "${work_dir}/SAN.cnf"
# openssl x509 -req -days "${key_days}" -signkey "${work_dir}/${common_name}.key" -passin pass:"${pass_phrase}" -in "${work_dir}/${common_name}.csr" -out "${work_dir}/${common_name}.crt" -extfile "${work_dir}/SAN.cnf"


# - Method 2 -- openssl req
# sefl-signed certificate (generate .key and .crt simulat)
openssl req -x509 -newkey rsa:"${key_len}" -sha512 -days "${key_days}" -nodes -passin pass:"${pass_phrase}" -keyout "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}.crt" -extensions san -config <(printf "[req]\ndistinguished_name=req\n[san]\nsubjectAltName=DNS:*.axdlog.com, DNS:axdlog.com, IP:127.0.0.1")  -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"

# remove pass phrase in private key
# openssl rsa -passin pass:"${pass_phrase}" -in "${work_dir}/${common_name}.key" -out "${work_dir}/${common_name}_out.key"
```


查看已簽署證書的詳細信息
```bash
openssl x509 -in "${work_dir}/${common_name}.crt" -noout -text
```

演示示例，注意查看`X509v3 Subject Alternative Name`行信息。

```bash
# openssl x509 -in "${work_dir}/${common_name}.crt" -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            f6:8f:22:dd:c8:97:86:84
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = DevSecOps, CN = axdlog.com, emailAddress = admin@axdlog.com
        Validity
            Not Before: Jan 26 01:37:26 2019 GMT
            Not After : Jan 26 01:37:26 2029 GMT
        Subject: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = DevSecOps, CN = axdlog.com, emailAddress = admin@axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (521 bit)
                pub:
                    04:01:9b:e1:31:5c:1c:a8:3f:17:e3:ac:7e:47:af:
                    3b:64:02:92:d9:77:f2:95:10:e4:f5:47:8c:b0:82:
                    c5:1a:55:33:e5:24:5f:98:3a:91:c1:04:cc:ab:68:
                    44:8a:d9:91:c0:f2:69:6c:aa:03:20:e2:7f:ba:4f:
                    92:e1:cc:14:60:b8:dd:01:c8:0a:7c:df:95:95:92:
                    2f:90:34:42:c8:2d:74:1b:2d:9e:45:36:fa:4f:c2:
                    e7:5c:97:06:d4:26:c5:b7:4a:79:ac:b5:8c:58:42:
                    75:48:5e:ae:fa:10:a7:ef:c7:21:8c:2d:e6:5e:2e:
                    2d:27:04:72:43:60:1e:d6:da:88:15:f8:b4
                ASN1 OID: secp521r1
                NIST CURVE: P-521
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:*.axdlog.com, DNS:axdlog.com, IP Address:127.0.0.1
    Signature Algorithm: ecdsa-with-SHA256
         30:81:87:02:41:3f:3b:ac:1b:91:a5:b2:24:d1:ff:2a:91:fe:
         c6:ae:ce:1d:7d:0c:4f:75:c7:23:09:5c:35:e0:9a:0b:c6:38:
         5e:34:ae:5a:92:04:43:e3:ee:9c:16:53:69:d8:f0:e3:37:3d:
         e6:80:cd:96:10:0f:fd:59:3d:2b:9a:6b:d3:3e:d7:c0:02:42:
         01:1d:89:90:57:6f:33:67:b0:8a:bf:ac:d6:6a:7c:76:b1:25:
         57:ad:fd:0c:43:ea:65:33:f0:f9:ce:34:ab:fe:a2:23:d5:d6:
         c9:a5:3a:7d:ea:2f:9c:79:49:50:3d:7b:1b:3b:77:fa:6a:19:
         55:62:8a:e0:2d:b8:bc:e1:92:39:3b:4f
```


## Display Certificate Info
通過如下命令查看證書內容

```bash
#Display the contents of a certificate
openssl x509 -in "${work_dir}/${common_name}.crt" -noout -text

#Display the certificate serial number
openssl x509 -in "${work_dir}/${common_name}.crt" -noout -serial

#Display the certificate subject name
openssl x509 -in "${work_dir}/${common_name}.crt" -noout -subject

#Display the certificate SHA1 fingerprint
openssl x509 -sha1 -in "${work_dir}/${common_name}.crt" -noout -fingerprint
```


## Testing In Browsers
在本地LEMP開發環境中配置生成的SSL證書，通過瀏覽器查看

### Preparation
生成自簽證書

```bash
maxdsre@jessie:~$ cat /tmp/passphrases
AxdLog2019
maxdsre@jessie:~$ cat /tmp/override.cnf
[ req ]
prompt = no
distinguished_name = req_distinguished_name
attributes = req_attributes
x509_extensions = v3_ca

input_password = AxdLog2019

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
![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:38.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:45.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:50.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:01:55.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:13.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:19.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/firefox-2017-01-09_17:02:50.png)


### Via Google Chrome

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:21.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:26.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:31.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:03:45.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2017-01-09-self-signed-cert/chrome-2017-01-09_17:04:35.png)


## Bibliography
* [Survival guides - TLS/SSL and SSL (X.509) Certificates](http://www.zytrax.com/tech/survival/ssl.html)
* [OpenSSL Command-Line HOWTO](https://www.madboa.com/geek/openssl/)
* [Why I don't Use 2048 or 4096 RSA Key Sizes](https://blog.josefsson.org/2016/11/03/why-i-dont-use-2048-or-4096-rsa-key-sizes/)


## Reference
* [Creating a Self-Signed SSL Certificate](https://devcenter.heroku.com/articles/ssl-certificate-self 'Heroku')
* [How to create a self-signed SSL Certificate](http://www.akadia.com/services/ssh_test_certificate.html)
* [Configuring ssl requests with SubjectAltName with openssl](http://apetec.com/support/generatesan-csr.htm)
* [OpenSSL Command Cheatsheet](https://medium.freecodecamp.org/openssl-command-cheatsheet-b441be1e8c4a)


## Change Logs
* 2017.01.09 17:06 Mon Asia/Shanghai
    * 初稿完成
* 2017.01.09 21:25 Mon Asia/Shanghai
    * 添加選項`-passout`、`-passin`、`-pass`用於設置passphrases值。
* 2017.01.11 11:14 Wed Asia/Shanghai
    * 添加Bibliography `Why I don't Use 2048 or 4096 RSA Key Sizes`
* 2018.07.27 18:26:30 Fri America/Boston
    * 勘誤，更新，遷移到新Blog
* 2019.01.25 20:45 Fri America/Boston
    * 重新撰寫


[openssl]:https://www.openssl.org

<!-- End -->
