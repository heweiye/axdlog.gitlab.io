---
title: Using OpenSSL To Create A Private CA On GNU/Linux
slug: Using OpenSSL To Create A Private CA On GNU Linux
date: 2017-01-27T04:20:09+08:00
lastmod: 2018-07-27T15:53:08-04:00
draft: false
keywords: ["AxdLog", "openssl"]
description: "Try to create a private CA via OpenSSL On GNU/Linux"
categories:
- Security
tags:
- OpenSSL
- OCSP
comment: true
toc: true

---

[Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority)是通信雙方都相信的第三方機構，是[Public Key Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure)的重要組成，主要用於簽發數字證書。數字證書在網路通信中扮演了很重要的角色，通過驗證公鑰的所有者實現通信安全。CA可分為`root ca`和`intermediate ca`，`intermediate ca`由`root ca`簽發。出於安全因素考慮，由intermediate ca代表root ca簽發數字證書，遵循鏈式信任。

本文記錄使用[OpenSSL](https://www.openssl.org/)創建私有CA，並通過私有CA創建[CRL](https://en.wikipedia.org/wiki/Certificate_revocation_list 'WikiPedia')、[Online Certificate Status Protocol](https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol 'WikiPedia')，簽發、吊銷數字證書的過程。本文中的相關操作參考自[OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/)、[OpenSS\L Cookbook](https://www.feistyduck.com/library/openssl-cookbook/)。

<!--more-->

## Introduction
WikiPedia

>In cryptography, a certificate authority or certification authority (CA) is an entity that **issues digital certificates**. A digital certificate certifies the ownership of a public key by the named subject of the certificate. This allows others (relying parties) to rely upon signatures or on assertions made about the private key that corresponds to the certified public key. In this model of trust relationships, a CA is a trusted third party—trusted both by the subject (owner) of the certificate and by the party relying upon the certificate. The most commonly encountered public-key infrastructure (PKI) schemes are those used to implement https on the world-wide web. All these are based upon the X.509 standard and feature CAs. – https://en.wikipedia.org/wiki/Certificate_authority

GlobalSign

>Certificate Authorities, or Certificate Authorities / CAs, **issue Digital Certificates**. Digital Certificates are verifiable small data files that contain identity credentials to help websites, people, and devices represent their authentic online identity (authentic because the CA has verified the identity). CAs play a critical role in how the Internet operates and how transparent, trusted transactions can take place online. CAs issue millions of Digital Certificates each year, and these certificates are used to protect information, encrypt billions of transactions, and enable secure communication. – https://www.globalsign.com/en/ssl-information-center/what-are-certification-authorities-trust-hierarchies/

本文將首先創建`root ca`，再由`root ca`簽發`intermediate ca`。數字證書由`intermediate ca`代表`root ca`簽發。存儲路徑定義為`/tmp/ca`。


## Preparation
操作平臺信息

item|version details
---|---
os|Debian GNU/Linux 9.5 (stretch)
kernel|4.9.0-7-amd64
openssl|OpenSSL 1.1.0f  25 May 2017


## Creating A Root CA
創建新的CA須進行如下操作

1. 構建OpenSSL配置文件;
2. 創建相關目錄結構；
3. 初始化相關密鑰文件；
4. 生成root ca的私鑰和證書；

OpenSSL默認的配置文件路徑為

```bash
# ls -lha $(openssl version -a | sed -r -n '/OPENSSLDIR/s@.*"(.*)"@\1@p')/openssl.cnf
lrwxrwxrwx 1 root root 20 Mar 29 06:51 /usr/lib/ssl/openssl.cnf -> /etc/ssl/openssl.cnf
```

其格式、指令說明見

```bash
man ca
# man ca | sed -r -n '/^[[:space:]]*A sample configuration/,/^ENVIRONMENT VARIABLES/p' | sed '$d'
```

部分指令來自

```bash
man x509v3_config
man req
man ocsp
```

默認的OpenSSL配置文件中只有`certs`(存放數字證書)、`private`(存放私鑰)兩個目錄。出於安全、優化目錄結構等因素，設置如下目錄

dir|explanation
---|---
certs|用於存放數字證書，如root ca的證書、intermediate ca的證書
newcerts|用於存放新簽發的數字證書
private|用於存放私鑰
db|用於存放index.txt、serial、crlnumber等文件
crl|用於存放生成的crl文件

其中目錄`private`設置讀寫權限為`700`。

執行如下命令創建目錄並初始化相關文件

```bash
#create root ca dir
mkdir -pv /tmp/ca
cd /tmp/ca

#create relevant dirs
mkdir -pv {certs,newcerts,db,crl}
mkdir -pv -m=700 private

#create CA text database file
touch ./db/index.txt
echo "unique_subject = no" > ./db/index.txt.attr

#create CA serial number file
openssl rand -hex 32 > ./db/serial
#create text file containing the next CRL number to use in hex
echo 1000 > ./db/crlnumber
```

### Configuration File
出於安全考慮，此處的message digest(md)算法使用SHA-2(SHA-1已經被廢棄)，即`sha512`。可通過命令`openssl list --digest-commands`查看支持的算法。

>The default digest was changed from MD5 to SHA256 in OpenSSL 1.1.0 The FIPS-related options were removed in OpenSSL 1.1.0 -- https://www.openssl.org/docs/manmaster/man1/dgst.html

此處root ca的目錄是`/tmp/ca`，在該目錄下創建文件`openssl.cnf`，內容如下

```bash
[ ca ]
# man ca
name            = root_ca
name_opt        = utf8,esc_ctrl,multiline,lname,align  # man x509 --> NAME OPTIONS
default_ca      = CA_default


[ CA_default ]
# Directory and file locations. 此處路徑可自定義
dir               = /tmp/ca                     # main CA directory
certs             = $dir/certs                  # certificate output file
new_certs_dir     = $dir/newcerts               # new certs dir
crl_dir           = $dir/crl

database          = $dir/db/index.txt           # CA text database file
serial            = $dir/db/serial              # CA serial number file
RANDFILE          = $dir/private/.rand          # CA random seed information

private_key       = $dir/private/ca.key.pem     # CA private key
certificate       = $dir/certs/ca.cert.pem      # CA certificate

# For certificate revocation lists.
crlnumber         = $dir/db/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 365       # how long before next CRL

default_md        = sha512          # message digest to use
name_opt          = ca_default      # Subject name display option
cert_opt          = ca_default      # Certificate display option
default_days      = 3650            # how long to certify for， 10years
preserve          = no
email_in_dn       = no              # Don't add the email into cert DN
unique_subject    = no
copy_extensions   = none            # Don't copy extensions from request
policy            = policy_strict   # default policy man ca --> POLICY FORMAT


# For all root CA signatures, root CA just only sign intermediate certificates that match.
# countryName, organizationName須匹配
[ policy_strict ]
countryName             = match
stateOrProvinceName     = optional
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional


# For all intermediate CA signatures
[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the ca man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional


#  For creating certificates or certificate signing requests
[ req ]
# Options for the req tool (man req).
default_bits        = 4096
default_md          = sha512
encrypt_key         = yes
utf8                = yes
string_mask         = utf8only
distinguished_name  = req_distinguished_name

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

# See https://en.wikipedia.org/wiki/Certificate_signing_request.
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name (full name)
localityName                    = Locality Name (eg, city)
0.organizationName              = Organization Name (eg, company)
organizationalUnitName          = Organizational Unit Name (eg, section)
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.  此處參數值可自定義
countryName_default             = CN
stateOrProvinceName_default     = Shanghai
localityName_default            =
0.organizationName_default      = AxdLog Ltd
organizationalUnitName_default  = AxdLog Ltd Certificate Authority
commonName_default              = AxdLog Ltd Root CA
emailAddress_default            =


[ v3_ca ]
# Extensions for a typical CA (man x509v3_config).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign


[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (man x509v3_config).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign


# Extensions for server certificates (man x509v3_config).
[ server_cert ]
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth


# Extensions for client certificates (man x509v3_config).
[ usr_cert ]
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection


# Extension for CRLs(certificate revocation list) (man x509v3_config).
[ crl_ext ]
authorityKeyIdentifier=keyid:always


# Extension for Online Certificate Status Protocol (OCSP) (man ocsp).
[ ocsp ]
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning

#Configuration End
```

注意： 如果要使用如下命令寫入內容

```bash
sudo tee /tmp/ca/openssl.cnf <<-EOF
...
EOF
```

須在`$dir`之前加斜線`\`用以轉義符號`$`，即將`$dir`改寫為`\$dir`。

### Signing Root Certificate
為避免出現`Passphrase`提示，使用選項`-passout`、`-passin`顯式指定`pass phrase`，此處設置為`AxdLog2018`，實際操作時可將該選項去除，以確保操作安全。

為避免出現`Subject`(distinguished name)提示，使用選項`-subj`顯式指定相關參數，可根據個人情況選擇使用。

執行如下命令創建私鑰、證書，私鑰存儲在目錄private中，證書存儲在certs中。

```bash
cd /tmp/ca

#Create the root key, set file attributes 400 via umask 266
(umask 266; openssl genrsa -passout pass:AxdLog2018 -out ./private/ca.key.pem -aes256 4096)

#remove pass phrase in private key
# openssl rsa -passin pass:AxdLog2018 -in ./private/ca.key.pem -out ./private/ca.keyout.pem

#Create the root certificate， set file attributes 444 via umask 222
# expire days 3650 (10 years), use extensions v3_ca in configuration file
(umask 222; openssl req -new -x509 -days 3650 -extensions v3_ca -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=AxdLog Ltd Root CA" -key ./private/ca.key.pem -out ./certs/ca.cert.pem)

#Verify the root certificate
# openssl x509 -noout -text -in ./certs/ca.cert.pem
```

校驗證書，輸出如下

```bash
#openssl x509 -noout -text -in ./certs/ca.cert.pem
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            fa:1e:9f:6d:b9:e1:90:44
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Root CA
        Validity
            Not Before: Jul 27 21:37:52 2018 GMT
            Not After : Jul 24 21:37:52 2028 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:f7:d3:85:93:8a:54:2b:16:e5:38:e0:2c:da:b5:
                    ...
                    ...
                    eb:d3:34:23:5f:9b:9e:35:26:0b:33:f1:6b:dc:1b:
                    ab:c8:1d
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                2D:64:90:A6:12:5F:2E:09:9B:F2:D3:48:EE:D4:47:37:AC:EA:EE:9C
            X509v3 Authority Key Identifier:
                keyid:2D:64:90:A6:12:5F:2E:09:9B:F2:D3:48:EE:D4:47:37:AC:EA:EE:9C

            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha512WithRSAEncryption
         27:55:c4:27:3c:67:8c:c9:70:82:84:15:4e:06:8f:0d:1f:ba:
         ...
         ...
         18:2d:fe:22:b0:9a:0c:e6:22:f0:ba:24:db:b5:f6:b4:5f:3b:
         af:be:d3:5c:44:56:2c:5d

```


## Creating A Intermediate CA
root ca創建完成後，創建intermediate ca，文件目錄較之root ca多了csr，用於存放生成的[CSR](https://en.wikipedia.org/wiki/Certificate_signing_request)文件。

dir|explanation
---|---
certs|用於存放數字證書，如root ca的證書、intermediate ca的證書
newcerts|用於存放新簽發的數字證書
private|用於存放私鑰
db|用於存放index.txt、serial、crlnumber等文件
csr|用於存放證書簽署請求文件CSR
crl|用於存放生成的crl文件


其中目錄`private`設置讀寫權限為`700`。

執行如下命令創建目錄並初始化相關文件

```bash
#create intermediate ca dir
mkdir -pv /tmp/ca/intermediate
cd /tmp/ca/intermediate

#create relevant dirs
mkdir -pv {certs,newcerts,db,csr,crl}
mkdir -pv -m=700 private

#create CA text database file
touch ./db/index.txt
echo "unique_subject = no" > ./db/index.txt.attr

#create CA serial number file
openssl rand -hex 32 > ./db/serial
#create text file containing the next CRL number to use in hex
echo 1000 > ./db/crlnumber
```

### Configuration File
出於安全考慮，此處的message digest(md)算法使用SHA-2(SHA-1已經被廢棄)，即`sha512`。

此處intermediate ca的目錄是`/tmp/ca/intermediate`，在該目錄下創建文件`openssl.cnf`，內容如下

```bash
[ ca ]
# man ca
name            = sub_ca
name_opt        = utf8,esc_ctrl,multiline,lname,align  # man x509 --> NAME OPTIONS
default_ca      = CA_default


[ CA_default ]
# Directory and file locations. 此處路徑可自定義
dir               = /tmp/ca/intermediate        # main CA directory
certs             = $dir/certs                  # certificate output file
new_certs_dir     = $dir/newcerts               # new certs dir
crl_dir           = $dir/crl

database          = $dir/db/index.txt           # CA text database file
serial            = $dir/db/serial              # CA serial number file
RANDFILE          = $dir/private/.rand          # CA random seed information

private_key       = $dir/private/intermediate.key.pem     # CA private key
certificate       = $dir/certs/intermediate.cert.pem      # CA certificate

# For certificate revocation lists.
crlnumber         = $dir/db/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30       # how long before next CRL

default_md        = sha512          # message digest to use
name_opt          = ca_default      # Subject name display option
cert_opt          = ca_default      # Certificate display option
default_days      = 365             # how long to certify for， 10years
preserve          = no
email_in_dn       = no              # Don't add the email into cert DN
unique_subject    = no
copy_extensions   = none            # Don't copy extensions from request
policy            = policy_loose    # default policy man ca --> POLICY FORMAT


# For all root CA signatures, root CA just only sign intermediate certificates that match.
# countryName, organizationName須匹配
[ policy_strict ]
countryName             = match
stateOrProvinceName     = optional
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional


# For all intermediate CA signatures
[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the ca man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional


#  For creating certificates or certificate signing requests
[ req ]
# Options for the req tool (man req).
default_bits        = 4096
default_md          = sha512
encrypt_key         = yes
utf8                = yes
string_mask         = utf8only
distinguished_name  = req_distinguished_name

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca


# See https://en.wikipedia.org/wiki/Certificate_signing_request.
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name (full name)
localityName                    = Locality Name (eg, city)
0.organizationName              = Organization Name (eg, company)
organizationalUnitName          = Organizational Unit Name (eg, section)
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.  此處參數值可自定義
countryName_default             = CN
stateOrProvinceName_default     = Shanghai
localityName_default            =
0.organizationName_default      = AxdLog Ltd
organizationalUnitName_default  = AxdLog Ltd Certificate Authority
commonName_default              = AxdLog Ltd Intermediate CA
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (man x509v3_config).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign


[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (man x509v3_config).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign


# Extensions for server certificates (man x509v3_config).
[ server_cert ]
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
#以下URL根據實際情況設置
#authorityInfoAccess = OCSP;URI:http://ocsp.example.com
#crlDistributionPoints = URI:http://example.com/intermediate.crl.pem


# Extensions for client certificates (man x509v3_config).
[ usr_cert ]
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection


# Extension for CRLs(certificate revocation list) (man x509v3_config).
[ crl_ext ]
authorityKeyIdentifier=keyid:always


# Extension for Online Certificate Status Protocol (OCSP) (man ocsp).
[ ocsp ]
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning

#Configuration End
```

### Signing Intermediate Certificate
為避免出現Passphrase提示，使用選項`-passout`、`-passin`顯式指定pass phrase，此處設置為`AxdLog2018`，實際操作時可將該選項去除，以確保操作安全。

為避免出現`Subject`(distinguished name)提示，使用選項`-subj`顯式指定相關參數，可根據個人情況選擇使用。

執行如下命令創建私鑰、CSR文件、證書，私鑰存儲在目錄private中，CSR文件存儲在`csr`中，證書存儲在`certs`中。

```bash
cd /tmp/ca/intermediate

#Create the intermediate key
(umask 266; openssl genrsa -passout pass:AxdLog2018 -out ./private/intermediate.key.pem -aes256 4096)

#remove pass phrase in private key
# openssl rsa -passin pass:AxdLog2018 -in ./private/intermediate.key.pem -out ./private/intermediate.keyout.pem

#Generate CSR
openssl req -new -sha512 -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=AxdLog Ltd Intermediate CA" -key ./private/intermediate.key.pem -out ./csr/intermediate.csr.pem

#Signing Intermediate Self-Signed Certificate Via Root Cert Conf
# expire days 365 (1 years), use extensions v3_intermediate_ca in configuration file
#必須切換到root ca所在目錄
cd /tmp/ca

(umask 222; openssl ca -days 365 -notext -md sha512 -config ./openssl.cnf -extensions v3_intermediate_ca -passin pass:AxdLog2018 -in ./intermediate/csr/intermediate.csr.pem -out ./intermediate/certs/intermediate.cert.pem)      # set file attributes 444 via umask

#Verify the intermediate certificate
# openssl x509 -noout -text -in ./intermediate/certs/intermediate.cert.pem

#Verify the intermediate certificate against the root certificate. An `OK` indicates that the chain of trust is intact.
# openssl verify -CAfile ./certs/ca.cert.pem ./intermediate/certs/intermediate.cert.pem
```

校驗證書，輸出如下

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            b7:97:53:4b:e1:40:01:08:f9:cb:6a:98:fc:aa:a6:57:76:8a:9d:bf:d6:95:b3:6b:5c:5f:84:b4:55:95:99:96
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Root CA
        Validity
            Not Before: Jul 27 21:39:51 2018 GMT
            Not After : Jul 27 21:39:51 2019 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Intermediate CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:c2:a4:42:ba:11:2c:ae:ce:69:c7:5d:f0:19:5d:
                    ...
                    ...
                    af:02:5a:78:cb:c2:8d:00:c6:d7:9f:51:41:e9:6e:
                    c3:e5:e7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                C6:5E:93:0D:18:B1:FA:4A:8F:99:5A:F2:29:06:51:35:FA:77:FC:17
            X509v3 Authority Key Identifier:
                keyid:2D:64:90:A6:12:5F:2E:09:9B:F2:D3:48:EE:D4:47:37:AC:EA:EE:9C

            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha512WithRSAEncryption
         17:20:c9:63:7b:37:7a:73:bf:65:50:c2:45:4a:d5:04:67:ce:
         ...
         ...
         17:38:4c:04:90:18:88:96:91:3a:e8:b6:f3:62:92:a3:52:3d:
         67:c3:1c:8b:ea:a8:25:ff

```

#### Create Certificate Chain File
Web瀏覽器有時並不信任某些intermediate ca簽發的證書，故而需將root ca的證書加入證書文件，以確保瀏覽器信任該intermediate ca。

在root ca所在路徑中執行如下操作，創建證書信任鏈文件，此處命名為`ca-chain`

```bash
cd /tmp/ca

(umask 222; cat ./intermediate/certs/intermediate.cert.pem ./certs/ca.cert.pem > ./intermediate/certs/ca-chain.cert.pem)    # set file attributes 444 via umask 222
```

在[Nginx](https://www.nginx.com/)中使用指令[ssl_trusted_certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_trusted_certificate)進行設置，指令如下：

```bash
ssl_trusted_certificate /tmp/ca/intermediate/certs/ca-chain.cert.pem;
```

如果使用[Let’s Encrypt](https://letsencrypt.org/)配置SSL證書，可參考其官網文檔配置證書信任鏈文件

* [Let’s Encrypt Root and Intermediate Certificates](https://letsencrypt.org/2015/06/04/isrg-ca-certs.html)
* [Chain of Trust](https://letsencrypt.org/certificates/)

![](https://letsencrypt.org/images/isrg-keys.png)

![](https://letsencrypt.org/certs/isrg-keys.png)

```bash
# Let's Encrypt Root and Intermediate Certificates

#Active Root Certificates (ISRG Root X1)
https://letsencrypt.org/certs/isrgrootx1.pem

#Active Intermediate Certificates
#Let’s Encrypt Authority X3 (IdenTrust cross-signed)
https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem
```

執行如下命令創建文件

```bash
wget -q -O - https://letsencrypt.org/certs/isrgrootx1.pem https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem | sudo tee -a /tmp/ca/intermediate/certs/letsencrypt-ca-cert.pem  > /dev/null
```


## Sign Server & Client Certificates
使用intermediate ca簽發證書

* 對於server certificate(Web服務器)，**Common Name** 必須是[FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)形式，如www.axdlog.com；
* 對於client certificate(郵件)，**Common Name** 可以是任意唯一標誌符，如郵件地址等；

usage|extension|Common Name
---|---|---
server cert|server_cert|FQDN形式，如www.axdlog.com
client cert|usr_cert|任意唯一標誌符，如郵件地址等

**註**： 表格中的extension在intermediate ca的配置文件

```bash
/tmp/ca/intermediate/openssl.cnf
```

中

執行如下命令簽發證書

```bash
cd /tmp/ca/intermediate

#Generate private key
(umask 266; openssl genrsa -aes256 -passout pass:AxdLog2018 -out ./private/axdlog.com.key.pem 4096)

#Generate CSR
openssl req -new -sha512 -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com/emailAddress=admin@axdlog.com" -key ./private/axdlog.com.key.pem -out ./csr/axdlog.com.csr.pem

#Scene1: sign server cert via intermediate CA, use extension server_cert
(umask 222; openssl ca -days 365 -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:AxdLog2018 -in ./csr/axdlog.com.csr.pem -out ./newcerts/axdlog.com.cert.pem)

#Scene2: sign client cert via intermediate CA, use extension usr_cert
# (umask 222; openssl ca -days 365 -notext -md sha512 -config ./openssl.cnf -extensions usr_cert -passin pass:AxdLog2018 -in ./csr/axdlog.com.csr.pem -out ./newcerts/axdlog.com.cert.pem)

#Verify the intermediate certificate
# openssl x509 -noout -text -in ./newcerts/axdlog.com.cert.pem

#Verify the intermediate certificate against the root certificate. An `OK` indicates that the chain of trust is intact.
# openssl verify -CAfile ./certs/ca-chain.cert.pem ./newcerts/axdlog.com.cert.pem
```

證書校驗過程

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            8b:0c:40:42:2d:a5:27:cd:79:d6:ab:7c:85:a2:76:23:d7:e9:9b:26:1b:0e:78:9f:44:f3:23:6b:c8:47:ac:11
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Intermediate CA
        Validity
            Not Before: Jul 27 21:41:47 2018 GMT
            Not After : Jul 27 21:41:47 2019 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Web Service, CN = www.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:a5:11:e8:1b:e1:13:81:05:4f:0b:00:d4:3c:37:
                    ...
                    ...
                    f9:50:e1:bc:75:49:6d:17:1e:6d:32:6e:36:70:1e:
                    f8:64:d7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            Netscape Comment:
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier:
                4B:43:54:BE:ED:57:28:95:B0:BE:C3:E5:BE:40:9A:3B:59:B3:C9:F1
            X509v3 Authority Key Identifier:
                keyid:C6:5E:93:0D:18:B1:FA:4A:8F:99:5A:F2:29:06:51:35:FA:77:FC:17
                DirName:/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=AxdLog Ltd Root CA
                serial:B7:97:53:4B:E1:40:01:08:F9:CB:6A:98:FC:AA:A6:57:76:8A:9D:BF:D6:95:B3:6B:5C:5F:84:B4:55:95:99:96

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
    Signature Algorithm: sha512WithRSAEncryption
         7e:ed:a9:54:a8:54:81:cd:61:d2:08:6b:96:34:4b:78:ac:ca:
         ...
         ...
         7c:c6:53:34:fd:02:83:6c:8a:25:46:5b:79:33:fa:e5:ee:c0:
         57:9d:75:35:36:e0:4a:2e

```

在Nginx中配置SSL證書，指令如下：

```bash
server {
    ...
    ssl_certificate /tmp/ca/intermediate/newcerts/axdlog.com.cert.pem;
    ssl_certificate_key /tmp/ca/intermediate/private/axdlog.com.key.pem;
    ssl_trusted_certificate /tmp/ca/intermediate/certs/ca-chain.cert.pem;
    ...
}
```


## Certificate Revocation List (CRL)
WikiPedia鏈接

```http
https://en.wikipedia.org/wiki/Certificate_revocation_list
```

RFC鏈接

```http
https://tools.ietf.org/html/rfc5280
```

在PKIs中，CRL文件存儲著已經被吊銷(revocate)的證書清單，用於告知訪問著某證書已不在受信任。證書吊銷理由有多個選項，具體見`man ca`中`-crl_reason reason`部分：

* unspecified
* keyCompromise
* CACompromise
* affiliationChanged
* superseded
* cessationOfOperation
* certificateHold
* removeFromCRL

### Generating CRL File

在intermediate ca所在目錄下進行操作，在配置文件openssl.cnf的`[server_cert]`中添加`crlDistributionPoints`指令，URL根據實際情況設置

```bash
crlDistributionPoints = URI:http://example.com/intermediate.crl.pem
```
此處通過如下命令啟用crlDistributionPoints

```bash
cd /tmp/ca/intermediate
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(authorityInfoAccess)@#\1@' ./openssl.cnf
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(crlDistributionPoints)@\1@' ./openssl.cnf
```

執行如下命令創建crl文件

```bash
cd /tmp/ca/intermediate

openssl ca -gencrl -config ./openssl.cnf -passin pass:AxdLog2018 -out ./crl/intermediate.crl.pem

#Verify the crl file
openssl crl -noout -text -in ./crl/intermediate.crl.pem
```

驗證信息如下

```bash
Certificate Revocation List (CRL):
        Version 2 (0x1)
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: /C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=AxdLog Ltd Intermediate CA
        Last Update: Jul 27 21:46:54 2018 GMT
        Next Update: Aug 26 21:46:54 2018 GMT
        CRL extensions:
            X509v3 Authority Key Identifier:
                keyid:C6:5E:93:0D:18:B1:FA:4A:8F:99:5A:F2:29:06:51:35:FA:77:FC:17

            X509v3 CRL Number:
                4096
No Revoked Certificates.
    Signature Algorithm: sha512WithRSAEncryption
         6b:1d:28:08:1d:67:c2:c9:98:7e:a4:2e:30:de:67:0f:2e:60:
         ...
         ...
         5b:13:c1:8d:48:57:34:6a:ff:d0:91:95:95:b7:f9:71:ce:e5:
         53:8f:79:d9:9f:06:a0:be
```

在Nginx中配置CRL，使用指令[ssl_crl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_crl)

```bash
server {
    ...
    ssl_crl /tmp/ca/intermediate/crl/intermediate.crl.pem;
    ...
}
```

**注意**：每次執行證書吊銷操作後，須重載(reload)Nginx服務，重新讀取crl，使操作生效。

```bash
sudo service nginx reload
#or
sudo systemctl reload nginx
```

### Revoking A Certificate

創建測試證書crl.axdlog.com.cert.pem

創建測試證書

```bash
cd /tmp/ca/intermediate

(umask 266; openssl genrsa -aes256 -passout pass:AxdLog2018 -out ./private/crl.axdlog.com.key.pem 4096)

openssl req -new -sha512 -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com/emailAddress=admin@axdlog.com" -key ./private/crl.axdlog.com.key.pem -out ./csr/crl.axdlog.com.csr.pem

(umask 222; openssl ca -days 365 -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:AxdLog2018 -in ./csr/crl.axdlog.com.csr.pem -out ./newcerts/crl.axdlog.com.cert.pem)
```

執行如下命令，可查看到在配置文件中設置的`crlDistributionPoints`的URL

```bash
cd /tmp/ca/intermediate
# find Full Name
openssl x509 -noout -text -in ./newcerts/crl.axdlog.com.cert.pem

# openssl x509 -noout -text -in ./newcerts/crl.axdlog.com.cert.pem | awk '$0~/Full Name/{getline;print gensub(/^[[:space:]]+(.*)/,"\\1"," ",$0)}'
```

在文件`/tmp/ca/intermediate/db/index.txt`中由如下信息

```bash
#吊銷前
V   190727214852Z       8B0C40422DA527CD79D6AB7C85A27623D7E99B261B0E789F44F3236BC847AC12    unknown /C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com
```

行首字母`V`表示該證書驗證有效、受信任。吊銷該證書後，符號會由`V`變成`R`，表示已吊銷。

執行如下命令進行證書吊銷操作

```bash
cd /tmp/ca/intermediate
openssl ca -config ./openssl.cnf -passin pass:AxdLog2018 -crl_reason keyCompromise -revoke ./newcerts/crl.axdlog.com.cert.pem
```

執行證書吊銷命令後，信息改變為

```bash
#吊銷後
R   190727214852Z   180727215039Z,keyCompromise 8B0C40422DA527CD79D6AB7C85A27623D7E99B261B0E789F44F3236BC847AC12    unknown /C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com
```

## Online Certificate Status Protocol (OCSP)
WikiPedia鏈接

```http
https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol
```

RFC鏈接

```http
https://tools.ietf.org/html/rfc6960
```

### Generating A OCSP File

OCSP需要密鑰對用於簽署發送給請求方的響應信息,**該密鑰對必須由簽署證書的CA簽署**(相同的CA)。由於OCSP證書不包含吊銷信息，無法被吊銷，故可適當縮短該證書的生命週期，如30天。

在intermediate ca所在目錄下進行操作，在配置文件openssl.cnf的`[server_cert]`中添加`authorityInfoAccess`指令，URL根據實際情況設置

```bash
authorityInfoAccess = OCSP;URI:http://ocsp.example.com
```

此處通過如下命令啟用`authorityInfoAccess`

```bash
cd /tmp/ca/intermediate
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(authorityInfoAccess)@\1@' ./openssl.cnf
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(crlDistributionPoints)@#\1@' ./openssl.cnf
```

執行如下命令創建ocsp證書`ocsp.axdlog.com.cert.pem`

```bash
cd /tmp/ca/intermediate

(umask 266; openssl genrsa -aes256 -passout pass:AxdLog2018 -out ./private/ocsp.axdlog.com.key.pem 4096)

#generate csr
openssl req -new -sha512 -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=ocsp.axdlog.com" -key ./private/ocsp.axdlog.com.key.pem -out ./csr/ocsp.axdlog.com.csr.pem

#sign server cert via intermediate CA, use extension oscp
(umask 222; openssl ca -days 30 -notext -md sha512 -config ./openssl.cnf -extensions ocsp -passin pass:AxdLog2018 -in ./csr/ocsp.axdlog.com.csr.pem -out ./certs/ocsp.axdlog.com.cert.pem)

#Verify the ocsp certificate
openssl x509 -noout -text -in ./certs/ocsp.axdlog.com.cert.pem
```

證書校驗信息如下

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            8b:0c:40:42:2d:a5:27:cd:79:d6:ab:7c:85:a2:76:23:d7:e9:9b:26:1b:0e:78:9f:44:f3:23:6b:c8:47:ac:13
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = AxdLog Ltd Intermediate CA
        Validity
            Not Before: Jul 27 21:52:26 2018 GMT
            Not After : Aug 26 21:52:26 2018 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = ocsp.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:a7:d8:f1:51:ab:0e:59:76:93:37:b7:80:dd:88:
                    ...
                    ...
                    d7:90:e3:97:8e:1c:4a:e2:3f:58:90:6b:21:09:f9:
                    a7:f8:1f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:
                2B:CD:70:EF:12:1F:62:DD:99:C3:90:DA:3D:D1:86:F8:A9:23:31:73
            X509v3 Authority Key Identifier:
                keyid:C6:5E:93:0D:18:B1:FA:4A:8F:99:5A:F2:29:06:51:35:FA:77:FC:17

            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage: critical
                OCSP Signing
    Signature Algorithm: sha512WithRSAEncryption
         50:50:1f:0d:18:7d:12:03:78:b3:f8:bc:b1:1f:29:36:53:9b:
         ...
         ...
         4d:4c:06:0c:b9:87:13:18:79:91:f4:1a:d3:35:f3:94:9f:1d:
         07:c0:1c:ff:0e:69:a1:85
```

可看到

```bash
X509v3 Key Usage: critical
    Digital Signature
X509v3 Extended Key Usage: critical
    OCSP Signing
```

在Nginx中啟用`ocsp`，設置如下

```bash
server {
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /tmp/ca/intermediate/certs/ca-chain.cert.pem;
}
```

配置完成，重載Nginx服務後，可通過如下命令查看ocsp配置是否生效

```bash
openssl s_client -connect www.example.com:443 -tls1_2 -tlsextdebug -status
```

可參閱 [Let’ encrypt - nginx - OCSP stapling](https://unix.stackexchange.com/questions/259430/let-encrypt-nginx-ocsp-stapling)

### Testing a certificate
**注意**：在版本`1.1.x`中，執行`openssl ocsp`時如果指定`-sha512`會出現報錯

>ocsp: Digest must be before -cert or -serial

將其移除即可，具體見 [OpenSSL OCSP Responder don't start anymore](https://unix.stackexchange.com/questions/371986/openssl-ocsp-responder-dont-start-anymore#answer-372228)
因為是測試，故在本機進行，端口選擇`9999`。

創建測試證書`ocsp_test1.axdlog.com.cert.pem`

```bash
cd /tmp/ca/intermediate
(umask 266; openssl genrsa -aes256 -passout pass:AxdLog2018 -out ./private/ocsp_test1.axdlog.com.key.pem 4096)

openssl req -new -sha512 -config ./openssl.cnf -passin pass:AxdLog2018 -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=ocsp_test1.axdlog.com/emailAddress=admin@axdlog.com" -key ./private/ocsp_test1.axdlog.com.key.pem -out ./csr/ocsp_test1.axdlog.com.csr.pem

(umask 222; openssl ca -days 365 -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:AxdLog2018 -in ./csr/ocsp_test1.axdlog.com.csr.pem -out ./newcerts/ocsp_test1.axdlog.com.cert.pem)
```

同時開啟2個Terminal(Shell終端)，在GNome Desktop中是`gnome-terminal`。

在Terminal 1 中執行

```bash
cd /tmp/ca/intermediate
openssl ocsp -text \
      -index ./db/index.txt \
      -CA ./certs/ca-chain.cert.pem \
      -rkey ./private/ocsp_test1.axdlog.com.key.pem \
      -rsigner ./newcerts/ocsp_test1.axdlog.com.cert.pem \
      -host 127.0.0.1:9999 \
      -nrequest 1

# -sha512
# ocsp: Digest must be before -cert or -serial
```

按要求輸入pass phrase的值後，出現信息

>Waiting for OCSP client connections…

在Terminal2中執行

```bash
cd /tmp/ca/intermediate
openssl ocsp -resp_text \
      -CAfile ./certs/ca-chain.cert.pem \
      -issuer ./certs/intermediate.cert.pem \
      -cert ./newcerts/ocsp_test1.axdlog.com.cert.pem \
      -url http://127.0.0.1:9999
```

返回信息如下

```bash
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Web Service, CN = ocsp_test1.axdlog.com
    Produced At: Jul 27 22:14:54 2018 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: 0DF30B3DA7865D32F716ED45117800508D352E40
      Issuer Key Hash: C65E930D18B1FA4A8F995AF229065135FA77FC17
      Serial Number: 8B0C40422DA527CD79D6AB7C85A27623D7E99B261B0E789F44F3236BC847AC15
    Cert Status: good
    This Update: Jul 27 22:14:54 2018 GMT

    Response Extensions:
        OCSP Nonce:
            0410344E13C95C212EF9EB0065D0A61D9678
    Signature Algorithm: sha256WithRSAEncryption
         8d:25:18:d4:9c:1d:4b:89:a3:ef:d4:24:46:27:a1:35:7b:7d:
         ...
         ...
         4b:bf:45:79:74:cb:f1:5e:de:f5:55:4d:20:82:99:3a:5c:38:
         79:1c:62:06:69:60:e3:ab
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            8b:0c:40:42:2d:a5:27:cd:79:d6:ab:7c:85:a2:76:23:d7:e9:9b:26:1b:0e:78:9f:44:f3:23:6b:c8:47:ac:15
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C=CN, ST=Shanghai, O=AxdLog Ltd, OU=AxdLog Ltd Certificate Authority, CN=AxdLog Ltd Intermediate CA
        Validity
            Not Before: Jul 27 22:06:15 2018 GMT
            Not After : Jul 27 22:06:15 2019 GMT
        Subject: C=CN, ST=Shanghai, O=AxdLog Ltd, OU=AxdLog Ltd Web Service, CN=ocsp_test1.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:9e:be:15:50:01:99:37:c6:b0:8d:0d:ee:ae:b2:
                    ...
                    ...
                    02:00:8d:b9:94:d3:18:14:03:5e:7e:7e:f7:e1:79:
                    d4:ec:6b:01:7f:63:7a:e4:51:e2:a5:dd:28:83:7f:
                    aa:45:85
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            Netscape Comment:
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier:
                CD:96:0C:82:C6:6F:60:6E:83:02:B3:76:25:28:A5:75:97:F2:65:5C
            X509v3 Authority Key Identifier:
                keyid:C6:5E:93:0D:18:B1:FA:4A:8F:99:5A:F2:29:06:51:35:FA:77:FC:17
                DirName:/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=AxdLog Ltd Root CA
                serial:B7:97:53:4B:E1:40:01:08:F9:CB:6A:98:FC:AA:A6:57:76:8A:9D:BF:D6:95:B3:6B:5C:5F:84:B4:55:95:99:96

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            Authority Information Access:
                OCSP - URI:http://ocsp.example.com

            X509v3 CRL Distribution Points:

                Full Name:
                  URI:http://example.com/intermediate.crl.pem

    Signature Algorithm: sha512WithRSAEncryption
         83:fb:44:e5:85:b3:72:f2:52:d9:a1:35:4e:71:4e:9c:08:6f:
         ...
         ...
         fa:7e:8a:27:d4:fb:39:10:ca:01:57:8b:ae:be:24:bc:e3:e7:
         da:72:3a:18:dd:8b:7a:3e
-----BEGIN CERTIFICATE-----
MIIHbjCCBVagAwIBAgIhAIsMQEItpSfNedarfIWidiPX6ZsmGw54n0TzI2vIR6wV
...
...
F+sAAJ6KKZk1+GgLrVXcvGPf8fO5dwyu8b30RNk9tqUXTYIAVQ98JP+QhHKXXBlW
I714W3mWd9f6foon1Ps5EMoBV4uuviS84+facjoY3Yt6Pg==
-----END CERTIFICATE-----
Response Verify ok
./newcerts/ocsp_test1.axdlog.com.cert.pem: good
	This Update: Jul 27 22:14:54 2018 GMT

```

其中有

```bash
#未被吊銷
Cert Status: good
```

吊銷後，狀態值會變成`revoked`

### Revoking A Certificate
執行如下命令吊銷證書`ocsp_test1.axdlog.com.cert.pem`

```bash
cd /tmp/ca/intermediate

openssl ca -config ./openssl.cnf -passin pass:AxdLog2018 -revoke ./newcerts/ocsp_test1.axdlog.com.cert.pem
```

操作完成後，再次進行上文 **Testing a certificate** 的操作

```bash
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = ocsp.axdlog.com
    Produced At: Jan 26 22:11:08 2017 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: A092DA95CA97A95F723C11CE054D278AA7B3E28A
      Issuer Key Hash: F0846DEBDD48A19A1B33F4748A2B6B21F87C2E06
      Serial Number: 9C625884AE8C03A7095540A53C3A1BAC8305A41D42F199460EA4AB8E4DC152F4
    Cert Status: revoked
    Revocation Time: Jan 26 22:09:30 2017 GMT
    This Update: Jan 26 22:11:08 2017 GMT

    Response Extensions:
        OCSP Nonce:
            04107DD00184815B914D3772DDDFD0C54532
    Signature Algorithm: sha256WithRSAEncryption
         96:cf:66:6d:b5:2a:95:c0:67:e7:0e:4b:ea:5f:de:5d:ff:4a:
         ...
         80:35:ed:5b:fd:7b:41:4a:9c:8d:99:75:a3:69:25:3c:ca:4e:
         cd:2b:69:1a:b2:1a:d4:86
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            9c:62:58:84:ae:8c:03:a7:09:55:40:a5:3c:3a:1b:ac:83:05:a4:1d:42:f1:99:46:0e:a4:ab:8e:4d:c1:52:f3
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=Shanghai, O=AxdLog Ltd, OU=AxdLog Ltd Certificate Authority, CN=AxdLog Ltd Intermediate CA
        Validity
            Not Before: Jan 26 22:00:58 2017 GMT
            Not After : Feb 25 22:00:58 2017 GMT
        Subject: C=CN, ST=Shanghai, O=AxdLog Ltd, OU=AxdLog Ltd Certificate Authority, CN=ocsp.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b4:66:39:ce:e9:2c:40:d2:ed:ff:6e:dc:0f:1e:
                    ...
                    cb:b9:41:1d:c0:a6:c2:4b:7a:55:27:37:0e:bc:cb:
                    37:1f:23
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:
                5D:B4:18:71:30:50:A1:6A:D7:DE:E0:2D:6B:44:48:F2:61:1A:36:6E
            X509v3 Authority Key Identifier:
                keyid:F0:84:6D:EB:DD:48:A1:9A:1B:33:F4:74:8A:2B:6B:21:F8:7C:2E:06

            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage: critical
                OCSP Signing
    Signature Algorithm: sha256WithRSAEncryption
         3c:6f:a7:b8:a0:35:47:a5:7e:f7:df:31:07:a6:8b:3f:8f:7f:
         ...
         80:f3:37:9a:13:a3:2d:7a:81:b0:9a:40:06:40:9a:85:59:da:
         fc:7a:f2:9f:10:39:91:aa
-----BEGIN CERTIFICATE-----
MIIGLzCCBBegAwIBAgIhAJxiWISujAOnCVVApTw6G6yDBaQdQvGZRg6kq45NwVLz
...
001D16/LXHOMH6wLw1i6AqJd84nOqGNRVoDzN5oToy16gbCaQAZAmoVZ2vx68p8Q
OZGq
-----END CERTIFICATE-----
Response verify OK
./newcerts/ocsp_test1.axdlog.com.cert.pem: revoked
	This Update: Jan 26 22:11:08 2017 GMT
	Revocation Time: Jan 26 22:09:30 2017 GMT
```

其中有

```bash
#已被吊銷
Cert Status: revoked
```

## Error Occuring

### Error 1
>problems making Certificate Request
139870547867280:error:0D07A097:asn1 encoding routines:ASN1_mbstring_ncopy:string too long:a_mbstr.c:158:maxsize=2

原因是配置文件openssl.cnf中`[req]`下設置了

```bash
prompt = no
```

註釋或刪除改行即可。

### Error 2
> Can't open /tmp/ca/db/index.txt.attr for reading, No such file or directory

執行如下命令創建文件`index.txt.attr`

```bash
echo "unique_subject = no" > /tmp/ca/db/index.txt.attr
```

## References
* [OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/)
* [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/)
* [OCSP Stapling on nginx](https://raymii.org/s/tutorials/OCSP_Stapling_on_nginx.html)


## Bibliography
* [DEPLOYING TLS THE HARD WAY](https://timtaubert.de/blog/2014/10/deploying-tls-the-hard-way/)


## Change Logs
* 2017.01.26 17:20 Thu America/Boston
    * 初稿完成
* 2018.07.27 18:24:26 Fri America/Boston
    * 勘誤，更新，遷移到新Blog

<!-- End -->
