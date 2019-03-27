---
title: Using OpenSSL To Create A Private CA On GNU/Linux
slug: Using OpenSSL To Create A Private CA On GNU Linux
date: 2017-01-27T04:20:09+08:00
lastmod: 2019-01-25T22:30:09-0500
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

[Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority)是通信雙方都信任的第三方機構，是[Public Key Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure)的重要組成，主要用於簽發數字證書。數字證書在網路通信中扮演了很重要的角色，通過驗證公鑰的所有者實現通信安全。CA可分為`root ca`和`intermediate ca`，`intermediate ca`由`root ca`簽發。出於安全因素考慮，由intermediate ca代表root ca簽發數字證書，遵循鏈式信任。

本文記錄使用[OpenSSL][openssl]創建私有CA，並通過私有CA創建[CRL](https://en.wikipedia.org/wiki/Certificate_revocation_list 'WikiPedia')、[Online Certificate Status Protocol](https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol 'WikiPedia')，簽發、吊銷數字證書的過程。本文中的相關操作參考自[OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/)、[OpenSS\L Cookbook](https://www.feistyduck.com/library/openssl-cookbook/)。

<!--more-->

[OpenSSL][openssl]的命令介紹可參考本人Blog [A Brief Introduction Of OpenSSL]({{< relref "2017-01-08-Simple-Introduction-Of-OpenSSL.md" >}})。

本文將首先創建`root ca`，再由`root ca`簽發`intermediate ca`。數字證書由`intermediate ca`代表`root ca`簽發。


## Introduction
WikiPedia

>In cryptography, a certificate authority or certification authority (CA) is an entity that **issues digital certificates**. A digital certificate certifies the ownership of a public key by the named subject of the certificate. This allows others (relying parties) to rely upon signatures or on assertions made about the private key that corresponds to the certified public key. In this model of trust relationships, a CA is a trusted third party—trusted both by the subject (owner) of the certificate and by the party relying upon the certificate. The most commonly encountered public-key infrastructure (PKI) schemes are those used to implement https on the world-wide web. All these are based upon the X.509 standard and feature CAs. – https://en.wikipedia.org/wiki/Certificate_authority

GlobalSign

>Certificate Authorities, or Certificate Authorities / CAs, **issue Digital Certificates**. Digital Certificates are verifiable small data files that contain identity credentials to help websites, people, and devices represent their authentic online identity (authentic because the CA has verified the identity). CAs play a critical role in how the Internet operates and how transparent, trusted transactions can take place online. CAs issue millions of Digital Certificates each year, and these certificates are used to protect information, encrypt billions of transactions, and enable secure communication. – https://www.globalsign.com/en/ssl-information-center/what-are-certification-authorities-trust-hierarchies/


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
# 操作目錄
work_dir=$(mktemp -d -t XXXXXX)
# work_dir='/tmp/ca'
intermediate_dir="${work_dir}/intermediate"
common_name='axdlog'
pass_phrase="AxdLog@$(date +'%Y')"

# Key
key_len=4096

# Days
key_days=3650

# Country Name
cert_C='CN'
# State or Province Name
cert_ST='Shanghai'
# Locality Name
cert_L='Shanghai'
# Organization Name
cert_O='AxdLog'
# Organizational Unit Name
cert_OU='AxdLog Certificate Authority'
# Common Name
cert_CN='AxdLog Root CA'
# Email Address
cert_email='admin@axdlog.com'
```


## Creating A Root CA
創建新的CA須進行如下操作

1. 構建OpenSSL配置文件;
2. 創建相關目錄結構；
3. 初始化相關密鑰文件；
4. 生成root ca的私鑰和證書；

OpenSSL默認的配置文件路徑為

```bash
ls -lha $(openssl version -a | sed -r -n '/^OPENSSLDIR/s@.*"(.*)"@\1@p')/openssl.cnf

# lrwxrwxrwx 1 root root 20 Mar 29 06:51 /usr/lib/ssl/openssl.cnf -> /etc/ssl/openssl.cnf
```

其格式、指令說明見

```bash
# man ca
man ca | sed -r -n '/^[[:space:]]*a sample configuration/I,/^[[:upper:]]+/{s@^[[:space:]]*@@g;p}' | sed '$d'
```

部分指令來自 `man x509v3_config`, `man req`, `man ocsp`。

默認的OpenSSL配置文件中只有`certs`(存放數字證書)、`private`(存放私鑰)兩個目錄。出於安全、優化目錄結構等因素，設置如下目錄

dir|explanation
---|---
certs | 存放數字證書，如root ca的證書、intermediate ca的證書
newcerts | 存放新簽發的數字證書
private | 存放私鑰
db | 存放index.txt、serial、crlnumber等文件
crl | 存放生成的crl文件

其中目錄`private`設置讀寫權限為`700`。

執行如下命令創建目錄並初始化相關文件

```bash
#create root ca dir
[[ -d "${work_dir}" ]] || mkdir -pv "${work_dir}"
cd "${work_dir}"

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

此處root ca的目錄是`${work_dir}`，在該目錄下創建文件`openssl.cnf`，內容如下

```bash
tee "${work_dir}/openssl.cnf" 1> /dev/null << EOF
[ ca ]
# man ca
name            = root_ca
name_opt        = utf8,esc_ctrl,multiline,lname,align  # man x509 --> NAME OPTIONS
default_ca      = CA_default


[ CA_default ]
# Directory and file locations.
dir               = ${work_dir}                 # main CA directory
certs             = \$dir/certs                 # certificate output file
new_certs_dir     = \$dir/newcerts              # new certs dir
crl_dir           = \$dir/crl

database          = \$dir/db/index.txt          # CA text database file
serial            = \$dir/db/serial             # CA serial number file
RANDFILE          = \$dir/private/.rand         # CA random seed information

private_key       = \$dir/private/ca.key        # CA private key
certificate       = \$dir/certs/ca.crt          # CA certificate

# For certificate revocation lists.
crlnumber         = \$dir/db/crlnumber
crl               = \$dir/crl/ca.crl
crl_extensions    = crl_ext
default_crl_days  = 365         # how long before next CRL

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
# countryName, organizationName 須匹配
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
countryName_default             = ${cert_C}
stateOrProvinceName_default     = ${cert_ST}
localityName_default            = ${cert_L}
0.organizationName_default      = ${cert_O}
organizationalUnitName_default  = ${cert_OU}
commonName_default              = ${cert_CN}
emailAddress_default            = ${cert_email}


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
EOF
```


### Signing Root Certificate
為避免出現`Passphrase`提示，使用選項`-passout`、`-passin`顯式指定`pass phrase`，此處設置為`AxdLog@$(date +'%Y')`，實際操作時可將該選項去除，以確保操作安全。

為避免出現`Subject`(distinguished name)提示，使用選項`-subj`顯式指定相關參數，可根據個人情況選擇使用。

執行如下命令創建私鑰、證書，私鑰存儲在目錄private中，證書存儲在certs中。

```bash
cd "${work_dir}"

#Create the root key, set file attributes 400 via umask 266
(umask 266; openssl genrsa -passout pass:"${pass_phrase}" -out ./private/ca.key -aes256 "${key_len}")

#remove pass phrase in private key
# openssl rsa -passin pass:"${pass_phrase}" -in ./private/ca.key -out ./private/ca_out.key

# Create the root certificate， set file attributes 444 via umask 222
# expire days 3650 (10 years), use extensions v3_ca in configuration file
(umask 222; openssl req -new -x509 -days "${key_days}" -extensions v3_ca -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}" -key ./private/ca.key -out ./certs/ca.crt)


#Verify the root certificate
# openssl x509 -noout -text -in ./certs/ca.crt
```

校驗證書，輸出如下

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            a8:ad:24:9c:d8:f7:bc:e8
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Validity
            Not Before: Jan 26 02:55:00 2019 GMT
            Not After : Jan 23 02:55:00 2029 GMT
        Subject: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b9:c7:d4:ff:ea:f8:eb:de:a0:75:46:2a:b1:b2:
                    ...
                    ...
                    a3:78:f4:5b:b3:0b:79:73:63:ef:5c:dd:4d:9d:36:
                    57:0a:a9
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                60:36:A6:DF:22:6E:C4:AA:99:1E:75:AC:54:D8:D6:B1:B8:25:47:F2
            X509v3 Authority Key Identifier:
                keyid:60:36:A6:DF:22:6E:C4:AA:99:1E:75:AC:54:D8:D6:B1:B8:25:47:F2

            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha512WithRSAEncryption
         18:ea:f5:34:28:cf:46:4b:6e:6b:8d:13:e7:a5:64:4c:35:d8:
         ...
         ...
         b7:d7:9e:f9:af:c6:31:85:36:7a:1f:e4:55:ca:45:13:6d:39:
         87:dd:8d:c5:61:91:d4:e7

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
mkdir -pv "${intermediate_dir}"
cd "${intermediate_dir}"

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

此處intermediate ca的目錄是`${intermediate_dir}`，在該目錄下創建文件`openssl.cnf`，內容如下

```bash
tee "${intermediate_dir}/openssl.cnf" 1> /dev/null << EOF
[ ca ]
# man ca
name            = sub_ca
name_opt        = utf8,esc_ctrl,multiline,lname,align  # man x509 --> NAME OPTIONS
default_ca      = CA_default


[ CA_default ]
# Directory and file locations.
dir               = ${intermediate_dir}         # main CA directory
certs             = \$dir/certs                 # certificate output file
new_certs_dir     = \$dir/newcerts              # new certs dir
crl_dir           = \$dir/crl

database          = \$dir/db/index.txt          # CA text database file
serial            = \$dir/db/serial             # CA serial number file
RANDFILE          = \$dir/private/.rand         # CA random seed information

private_key       = \$dir/private/intermediate.key     # CA private key
certificate       = \$dir/certs/intermediate.crt      # CA certificate

# For certificate revocation lists.
crlnumber         = \$dir/db/crlnumber
crl               = \$dir/crl/ca.crl
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
# x509_extensions     = v3_ca
x509_extensions     = v3_intermediate_ca

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
countryName_default             = ${cert_C}
stateOrProvinceName_default     = ${cert_ST}
localityName_default            = ${cert_L}
0.organizationName_default      = ${cert_O}
organizationalUnitName_default  = ${cert_OU}
commonName_default              = ${cert_CN}
emailAddress_default            = ${cert_email}

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
# 以下URL根據實際情況設置
# authorityInfoAccess = OCSP;URI:http://ocsp.example.com
# crlDistributionPoints = URI:http://example.com/intermediate.crl


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
EOF
```

### Signing Intermediate Certificate
為避免出現Passphrase提示，使用選項`-passout`、`-passin`顯式指定pass phrase，此處設置為`${pass_phrase}`，實際操作時可將該選項去除，以確保操作安全。

為避免出現`Subject`(distinguished name)提示，使用選項`-subj`顯式指定相關參數，可根據個人情況選擇使用。

執行如下命令創建私鑰、CSR文件、證書，私鑰存儲在目錄private中，CSR文件存儲在`csr`中，證書存儲在`certs`中。

```bash
cd "${intermediate_dir}"

# - Create the intermediate key
(umask 266; openssl genrsa -passout pass:"${pass_phrase}" -out ./private/intermediate.key -aes256 "${key_len}")

# - Remove pass phrase in private key
openssl rsa -passin pass:"${pass_phrase}" -in ./private/intermediate.key -out ./private/intermediate_out.key

# - Generate CSR
openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}" -key ./private/intermediate.key -out ./csr/intermediate.csr

# - Signing Intermediate Self-Signed Certificate Via Root Cert Conf
# expire days 365 (1 years), use extensions v3_intermediate_ca in configuration file

# cd "${work_dir}"  # change to root ca directory
# umask 222  set file attributes 444 via umask

# mothod 1 - interactive mode
# (umask 222; openssl ca -days "${key_days}" -notext -md sha512 -config "${work_dir}"/openssl.cnf -extensions v3_intermediate_ca -passin pass:"${pass_phrase}" -in "${intermediate_dir}"/csr/intermediate.csr -out "${intermediate_dir}"/certs/intermediate.crt)
# mothod 2 - quiet mode
(umask 222; openssl x509 -req -days "${key_days}" -sha512 -text -in "${intermediate_dir}"/csr/intermediate.csr -out "${intermediate_dir}"/certs/intermediate.crt -passin pass:"${pass_phrase}" -CA "${work_dir}"/certs/ca.crt -CAkey "${work_dir}"/private/ca.key -CAcreateserial -extfile "${work_dir}"/openssl.cnf -extensions v3_intermediate_ca)

# Verify the intermediate certificate
# openssl x509 -noout -text -in "${intermediate_dir}"/certs/intermediate.crt

# Verify the intermediate certificate against the root certificate. An `OK` indicates that the chain of trust is intact.
# openssl verify -CAfile "${work_dir}"/certs/ca.crt "${intermediate_dir}"/certs/intermediate.crt
```

校驗證書，輸出如下

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            27:eb:af:ac:ae:a7:cf:37:69:22:e8:96:75:8d:4c:1d:47:ef:fc:8a:f5:c0:56:03:33:6b:bd:c7:05:09:cd:0e
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Validity
            Not Before: Jan 26 03:08:18 2019 GMT
            Not After : Jan 26 03:08:18 2020 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:c1:26:51:61:2f:ec:95:57:91:45:2e:22:c8:e5:
                    ...
                    ...
                    2d:98:44:99:33:87:85:e7:37:2e:dd:02:3e:71:3e:
                    c2:4b:3d
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                7A:11:F1:E6:7D:29:4C:1D:A3:87:0F:D1:DC:D2:18:DC:3B:C7:51:A6
            X509v3 Authority Key Identifier:
                keyid:60:36:A6:DF:22:6E:C4:AA:99:1E:75:AC:54:D8:D6:B1:B8:25:47:F2

            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha512WithRSAEncryption
         24:3b:71:f2:c7:76:11:d4:d3:53:ea:fc:1f:8c:6f:69:d2:13:
         ...
         ...
         76:16:35:f3:5e:c0:04:f3:94:29:91:b4:5b:1b:29:0c:df:9e:
         8a:67:af:70:44:9a:ad:1e
```

#### Create Certificate Chain File
Web瀏覽器有時並不信任某些intermediate ca簽發的證書，故而需將root ca的證書加入證書文件，以確保瀏覽器信任該intermediate ca。

在root ca所在路徑中執行如下操作，創建證書信任鏈文件，此處命名為`ca-chain`

```bash
# cd "${work_dir}"
# (umask 222; cat ./intermediate/certs/intermediate.crt ./certs/ca.crt > ./intermediate/certs/ca-chain.crt)    # set file attributes 444 via umask 222

(umask 222; cat "${intermediate_dir}"/certs/intermediate.crt "${work_dir}"/certs/ca.crt > "${intermediate_dir}"/certs/ca-chain.crt)
```

在[Nginx](https://www.nginx.com/)中使用指令[ssl_trusted_certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_trusted_certificate)進行設置，指令如下：

```bash
ssl_trusted_certificate "${intermediate_dir}"/certs/ca-chain.crt;
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
wget -qO- https://letsencrypt.org/certs/isrgrootx1.pem https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem >> /tmp/ca/intermediate/certs/letsencrypt-ca.crt
```


## Sign Server & Client Certificates
使用intermediate ca簽發證書

* 對於server certificate(Web服務器)，**Common Name** 必須是[FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)形式，如www.axdlog.com；
* 對於client certificate(郵件)，**Common Name** 可以是任意唯一標誌符，如郵件地址等；

usage|extension|Common Name
---|---|---
server cert|server_cert|FQDN形式，如www.axdlog.com
client cert|usr_cert|任意唯一標誌符，如郵件地址等

**註**： 表格中的extension在intermediate ca的配置文件 `${intermediate_dir}/openssl.cnf`。


### For Single Hostname
執行如下命令簽發證書

```bash
cd "${intermediate_dir}"

#Generate private key
(umask 266; openssl genrsa -aes256 -passout pass:"${pass_phrase}" -out ./private/"${common_name}".key "${key_len}")

#remove pass phrase in private key
openssl rsa -passin pass:"${pass_phrase}" -in ./private/"${common_name}".key -out ./private/"${common_name}"_out.key

#Generate CSR
# "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}"
openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=CN/ST=Shanghai/O=AxdLog/OU=AxdLog Web Service/CN=axdlog.com/emailAddress=admin@axdlog.com" -key ./private/"${common_name}".key -out ./csr/"${common_name}".csr

# Disable prompt 'Sign the certificate? [y/n]:' via parameter `-batch`

#Scene1: sign server cert via intermediate CA, use extension 'server_cert'
# (umask 222; openssl ca -batch -days "${key_days}" -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:"${pass_phrase}" -in ./csr/"${common_name}".csr -out ./newcerts/"${common_name}".crt)

#Scene2: sign client cert via intermediate CA, use extension 'usr_cert'
# (umask 222; openssl ca -batch -days "${key_days}" -notext -md sha512 -config ./openssl.cnf -extensions usr_cert -passin pass:"${pass_phrase}" -in ./csr/"${common_name}".csr -out ./newcerts/"${common_name}".crt)

#Verify the intermediate certificate
# openssl x509 -noout -text -in ./newcerts/"${common_name}".crt

#Verify the intermediate certificate against the root certificate. An `OK` indicates that the chain of trust is intact.
# openssl verify -CAfile ./certs/ca-chain.crt ./newcerts/"${common_name}".crt
```

證書校驗過程

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c9:ba:6f:0c:0d:84:58:87:29:dd:63:e9:83:44:68:a0:88:77:c3:80:37:a2:a8:65:64:4d:3a:96:80:ea:a4:d2
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Validity
            Not Before: Jan 26 03:14:08 2019 GMT
            Not After : Jan 26 03:14:08 2020 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Web Service, CN = www.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b4:dd:ac:c0:b7:5c:c6:e4:4a:86:66:03:a4:9a:
                    ...
                    ...
                    46:34:7d:3b:fc:66:9c:9c:76:b6:06:88:e1:1d:5e:
                    50:83:1b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            Netscape Comment:
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier:
                2D:0D:31:DB:A1:BA:BE:B3:83:E8:7D:AF:06:C7:D6:6B:7C:A6:08:D5
            X509v3 Authority Key Identifier:
                keyid:7A:11:F1:E6:7D:29:4C:1D:A3:87:0F:D1:DC:D2:18:DC:3B:C7:51:A6
                DirName:/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=AxdLog Certificate Authority/CN=AxdLog Root CA
                serial:27:EB:AF:AC:AE:A7:CF:37:69:22:E8:96:75:8D:4C:1D:47:EF:FC:8A:F5:C0:56:03:33:6B:BD:C7:05:09:CD:0E

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
    Signature Algorithm: sha512WithRSAEncryption
         0b:62:88:36:06:cf:ea:0e:92:19:3c:6b:ba:59:57:e9:56:d5:
         ...
         ...
         2b:b0:35:77:09:bf:b4:46:05:03:a1:37:18:ac:12:c1:6e:f2:
         b1:99:0b:5e:ee:08:4a:ff

```

在Nginx中配置SSL證書，指令如下：

```bash
server {
    ...
    ssl_certificate /tmp/ca/intermediate/newcerts/axdlog.com.crt;
    ssl_certificate_key /tmp/ca/intermediate/private/axdlog.com.key;
    ssl_trusted_certificate /tmp/ca/intermediate/certs/ca-chain.crt;
    ...
}
```


### For Multiple Hostnames
執行如下命令簽發證書

```bash
# - Reference
# https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/
# https://github.com/kingkool68/generate-ssl-certs-for-local-development/blob/master/generate-ssl.sh
# https://github.com/kingkool68/generate-ssl-certs-for-local-development

# https://security.stackexchange.com/questions/74345/provide-subjectaltname-to-openssl-directly-on-the-command-line#answer-198409
# https://unix.stackexchange.com/questions/63209/how-do-i-add-multiple-email-addresses-to-an-ssl-certificate-via-the-command-line/333325#333325
# https://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl/41366949#41366949


cd "${intermediate_dir}"

# - Generate private key
# -- For RSA
(umask 266; openssl genrsa -aes256 -passout pass:"${pass_phrase}" -out ./private/"${common_name}".key "${key_len}")
openssl rsa -passin pass:"${pass_phrase}" -in ./private/"${common_name}".key -out ./private/"${common_name}"_out.key
# -- For EC
curve_choose='secp521r1'
openssl ecparam -genkey -name "${curve_choose}" | openssl ec -out ./private/"${common_name}".key -passout pass:"${pass_phrase}" -aes256  #生成私鑰
openssl ec -in ./private/"${common_name}".key -passin pass:"${pass_phrase}" -out ./private/"${common_name}"_out.key

# - Generate CSR
openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=CN/ST=Shanghai/O=AxdLog/OU=AxdLog Web Service/CN=axdlog.com/emailAddress=admin@axdlog.com" -key ./private/"${common_name}".key -out ./csr/"${common_name}".csr

tee "${intermediate_dir}/csr_override.cnf" 1>/dev/null << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
IP.1 = 127.0.0.1
IP.2 = ::1
DNS.1 = localhost
DNS.2 = axdlog.com
DNS.3 = *.axdlog.com
EOF

# - Sign server cert
openssl x509 -req -days "${key_days}" -sha512 -in ./csr/"${common_name}".csr -out ./newcerts/"${common_name}".crt -CA "${intermediate_dir}"/certs/intermediate.crt -CAkey ./private/intermediate_out.key -CAcreateserial -extfile "${intermediate_dir}/csr_override.cnf"

# - View csr info
openssl req -in ./csr/"${common_name}".csr -noout -text

# - Verify the intermediate certificate
# X509v3 Subject Alternative Name
openssl x509 -noout -text -in ./newcerts/"${common_name}".crt
```

## Importing CA Root Certificates
爲使通過私有CA簽署的證書受信，需將導入到系統或Web瀏覽器的數據庫中。

* [Certificates for localhost](https://letsencrypt.org/docs/certificates-for-localhost/)
* [chromium - Linux Cert Management](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_cert_management.md)
* [How to import CA root certificates on Linux and Windows](https://thomas-leister.de/en/how-to-import-ca-root-certificate/)
* [gist - How to install CA certificates and PKCS12 key bundles on different platforms](https://gist.github.com/marians/b6ce3f2307a1a1ece69355a26c0a688a)
* [SDB:Share certificates between applications or whole system](https://en.opensuse.org/SDB:Share_certificates_between_applications_or_whole_system)
* [stackoverflow - Programmatically Install Certificate into Mozilla](https://stackoverflow.com/questions/1435000/programmatically-install-certificate-into-mozilla)
* [mkcert](https://github.com/FiloSottile/mkcert#linux)


### For System
```bash
os_ca_cert_dir='/usr/local/share/ca-certificates/custom'
[[ -d "${os_ca_cert_dir}" ]] || mkdir -p "${os_ca_cert_dir}"
sudo cp "${intermediate_dir}"/certs/intermediate.crt "${os_ca_cert_dir}/root_ca.crt"
```

Update ca database

```bash
# Fedora, RHEL, CentOS
update-ca-trust

# Ubuntu, Debian
update-ca-certificates

# Arch
trust
```


### For Web Browser
>certutil - Manage keys and certificate in both NSS databases and other NSS tokens

Install `certutil`

```bash
# Debian/Ubuntu

sudo apt-get -y install libnss3-tools

# CentOS
sudo yum -y install nss-tools

# Fedora
sudo dnf -y install nss-tools

# SUSE/OpenSUES
sudo zypper in -y mozilla-nss

# ArchLinux
sudo pacman -S nss
```


>Use the -L option to see a list of the current certificates and trust attributes in a certificate database.

```bash
# certutil -L
certutil: function failed: SEC_ERROR_LEGACY_DATABASE: The certificate/key database is in an old, unsupported format.
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
crlDistributionPoints = URI:http://example.com/intermediate.crl
```
此處通過如下命令啟用crlDistributionPoints

```bash
cd "${intermediate_dir}"
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(authorityInfoAccess)@#\1@' ./openssl.cnf
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(crlDistributionPoints)@\1@' ./openssl.cnf
```

執行如下命令創建crl文件

```bash
cd "${intermediate_dir}"

openssl ca -gencrl -config ./openssl.cnf -passin pass:"${pass_phrase}" -out ./crl/intermediate.crl

#Verify the crl file
openssl crl -noout -text -in ./crl/intermediate.crl
```

驗證信息如下

```bash
Certificate Revocation List (CRL):
        Version 2 (0x1)
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: /C=CN/ST=Shanghai/O=AxdLog/OU=AxdLog Certificate Authority/CN=AxdLog Root CA
        Last Update: Jan 26 03:16:10 2019 GMT
        Next Update: Feb 25 03:16:10 2019 GMT
        CRL extensions:
            X509v3 Authority Key Identifier:
                keyid:7A:11:F1:E6:7D:29:4C:1D:A3:87:0F:D1:DC:D2:18:DC:3B:C7:51:A6

            X509v3 CRL Number:
                4096
No Revoked Certificates.
    Signature Algorithm: sha512WithRSAEncryption
         4a:0f:07:cf:fb:33:f6:a3:94:c9:66:c1:8f:12:a1:7d:0c:cb:
         ...
         ...
         5d:a0:41:28:07:bc:d0:e6:1b:4f:e0:01:f3:76:37:3b:59:ad:
         79:3d:71:8f:4e:cb:b6:e9

```

在Nginx中配置CRL，使用指令[ssl_crl](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_crl)

```bash
server {
    ...
    ssl_crl /tmp/ca/intermediate/crl/intermediate.crl;
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

創建測試證書`crl.${common_name}.crt`

創建測試證書

```bash
cd "${intermediate_dir}"

(umask 266; openssl genrsa -aes256 -passout pass:"${pass_phrase}" -out ./private/crl."${common_name}".key "${key_len}")

openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}" -key ./private/crl."${common_name}".key -out ./csr/crl."${common_name}".csr

(umask 222; openssl ca -days "${key_days}" -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:"${pass_phrase}" -in ./csr/crl."${common_name}".csr -out ./newcerts/crl."${common_name}".crt)
```

執行如下命令，可查看到在配置文件中設置的`crlDistributionPoints`的URL

```bash
cd "${intermediate_dir}"
# find Full Name
openssl x509 -noout -text -in ./newcerts/crl."${common_name}".crt

# openssl x509 -noout -text -in ./newcerts/crl."${common_name}".crt | awk '$0~/Full Name/{getline;print gensub(/^[[:space:]]+(.*)/,"\\1","g",$0)}'
# URI:http://example.com/intermediate.crl
```

在文件`${intermediate_dir}/db/index.txt`中有如下信息

```bash
#吊銷前
V	200126031408Z		C9BA6F0C0D84588729DD63E9834468A08877C38037A2A865644D3A9680EAA4D2	unknown	/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com
```

行首字母`V`表示該證書驗證有效、受信任。吊銷該證書後，符號會由`V`變成`R`，表示已吊銷。

執行如下命令進行證書吊銷操作

```bash
cd "${intermediate_dir}"
openssl ca -config ./openssl.cnf -passin pass:"${pass_phrase}" -crl_reason keyCompromise -revoke ./newcerts/crl."${common_name}".crt
```

執行證書吊銷命令後，信息改變為

```bash
#吊銷後
R	200126031738Z	190126032137Z,keyCompromise	C9BA6F0C0D84588729DD63E9834468A08877C38037A2A865644D3A9680EAA4D3	unknown	/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Web Service/CN=www.axdlog.com
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
cd "${intermediate_dir}"
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(authorityInfoAccess)@\1@' ./openssl.cnf
sed -i -r '/^\[ server_cert \]/,/^\[ crl_ext \]/s@^#?(crlDistributionPoints)@#\1@' ./openssl.cnf
```

執行如下命令創建ocsp證書`ocsp.${common_name}.crt`

```bash
cd "${intermediate_dir}"

(umask 266; openssl genrsa -aes256 -passout pass:"${pass_phrase}" -out ./private/ocsp."${common_name}".key "${key_len}")

#generate csr
openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=CN/ST=Shanghai/O=AxdLog Ltd/OU=AxdLog Ltd Certificate Authority/CN=ocsp.axdlog.com" -key ./private/ocsp."${common_name}".key -out ./csr/ocsp."${common_name}".csr

#sign server cert via intermediate CA, use extension oscp
(umask 222; openssl ca -days 30 -notext -md sha512 -config ./openssl.cnf -extensions ocsp -passin pass:"${pass_phrase}" -in ./csr/ocsp."${common_name}".csr -out ./certs/ocsp."${common_name}".crt)

#Verify the ocsp certificate
openssl x509 -noout -text -in ./certs/ocsp."${common_name}".crt
```

證書校驗信息如下

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c9:ba:6f:0c:0d:84:58:87:29:dd:63:e9:83:44:68:a0:88:77:c3:80:37:a2:a8:65:64:4d:3a:96:80:ea:a4:d4
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CN, ST = Shanghai, O = AxdLog, OU = AxdLog Certificate Authority, CN = AxdLog Root CA
        Validity
            Not Before: Jan 26 03:23:59 2019 GMT
            Not After : Feb 25 03:23:59 2019 GMT
        Subject: C = CN, ST = Shanghai, O = AxdLog Ltd, OU = AxdLog Ltd Certificate Authority, CN = ocsp.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:ca:25:89:59:0b:bf:05:62:15:26:1c:d6:94:6b:
                    ...
                    ...
                    0f:ee:8a:95:f7:b4:a1:b6:0d:7d:c5:5c:79:0f:13:
                    ba:08:a9
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:
                AE:BF:DE:13:E7:9E:94:ED:F4:04:54:69:1A:DE:8C:CA:2C:A2:4E:44
            X509v3 Authority Key Identifier:
                keyid:7A:11:F1:E6:7D:29:4C:1D:A3:87:0F:D1:DC:D2:18:DC:3B:C7:51:A6

            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage: critical
                OCSP Signing
    Signature Algorithm: sha512WithRSAEncryption
         30:67:a4:7c:59:cd:02:8f:8f:36:d0:cc:65:b7:e6:d0:d0:1c:
         ...
         ...
         ba:d9:44:d0:71:bc:89:c6:46:e9:15:09:ff:37:3f:90:63:da:
         27:e2:bf:ee:a1:db:74:c5

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
    ssl_trusted_certificate /tmp/ca/intermediate/certs/ca-chain.crt;
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

創建測試證書`ocsp_test1.${common_name}".crt`

```bash
cd "${intermediate_dir}"

(umask 266; openssl genrsa -aes256 -passout pass:"${pass_phrase}" -out ./private/ocsp_test1."${common_name}".key "${key_len}")

openssl req -new -sha512 -config ./openssl.cnf -passin pass:"${pass_phrase}" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=AxdLog Web Service/CN=ocsp_test1.axdlog.com/emailAddress=${cert_email}" -key ./private/ocsp_test1."${common_name}".key -out ./csr/ocsp_test1."${common_name}".csr


(umask 222; openssl ca -days "${key_days}" -notext -md sha512 -config ./openssl.cnf -extensions server_cert -passin pass:"${pass_phrase}" -in ./csr/ocsp_test1."${common_name}".csr -out ./newcerts/ocsp_test1."${common_name}".crt)
```

同時開啟2個Terminal(Shell終端)，在GNome Desktop中是`gnome-terminal`。

在Terminal 1 中執行

```bash
cd "${intermediate_dir}"

openssl ocsp -text \
      -index ./db/index.txt \
      -CA ./certs/ca-chain.crt \
      -rkey ./private/ocsp_test1."${common_name}".key \
      -rsigner ./newcerts/ocsp_test1."${common_name}".crt \
      -port 9999 \
      -nrequest +1

# -sha512
# ocsp: Digest must be before -cert or -serial
```

按要求輸入pass phrase的值後，出現信息

>Waiting for OCSP client connections…

在Terminal2中執行

```bash
cd "${intermediate_dir}"

openssl ocsp -resp_text \
      -CAfile ./certs/ca-chain.crt \
      -issuer ./certs/intermediate.crt \
      -cert ./newcerts/ocsp_test1."${common_name}".crt \
      -url http://127.0.0.1:9999
```

返回信息如下

```bash
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: C = CN, ST = Shanghai, L = Shanghai, O = AxdLog, OU = AxdLog Web Service, CN = ocsp_test1.axdlog.com
    Produced At: Jan 26 03:38:56 2019 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: B013098CC5D1C97F0DFF8A219012992BC62FAE07
      Issuer Key Hash: 7A11F1E67D294C1DA3870FD1DCD218DC3BC751A6
      Serial Number: C9BA6F0C0D84588729DD63E9834468A08877C38037A2A865644D3A9680EAA4D5
    Cert Status: good
    This Update: Jan 26 03:38:56 2019 GMT

    Response Extensions:
        OCSP Nonce:
            04108EC3D841B9B6692D278802B84C81E3EF
    Signature Algorithm: sha256WithRSAEncryption
         45:58:b2:e4:b0:7b:84:d2:86:b7:5b:e3:93:24:f4:3a:0b:07:
         ...
         d1:ac:8f:d6:0b:61:5b:ab:7a:db:88:78:b5:7a:89:84:e3:a2:
         15:3b:7d:42:bb:45:ae:12
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            c9:ba:6f:0c:0d:84:58:87:29:dd:63:e9:83:44:68:a0:88:77:c3:80:37:a2:a8:65:64:4d:3a:96:80:ea:a4:d5
    Signature Algorithm: sha512WithRSAEncryption
        Issuer: C=CN, ST=Shanghai, O=AxdLog, OU=AxdLog Certificate Authority, CN=AxdLog Root CA
        Validity
            Not Before: Jan 26 03:26:33 2019 GMT
            Not After : Jan 26 03:26:33 2020 GMT
        Subject: C=CN, ST=Shanghai, L=Shanghai, O=AxdLog, OU=AxdLog Web Service, CN=ocsp_test1.axdlog.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:d0:e3:5d:82:be:f0:1d:58:62:c3:a6:e6:5b:6a:
                    ...
                    fc:0f:29:a9:77:68:ce:d1:7b:be:90:7d:92:98:2c:
                    d0:7b:b9
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            Netscape Comment:
                OpenSSL Generated Server Certificate
            X509v3 Subject Key Identifier:
                59:45:46:3D:F9:F3:CC:EB:25:44:77:E6:A4:B5:DB:B3:BB:48:5D:06
            X509v3 Authority Key Identifier:
                keyid:7A:11:F1:E6:7D:29:4C:1D:A3:87:0F:D1:DC:D2:18:DC:3B:C7:51:A6
                DirName:/C=CN/ST=Shanghai/L=Shanghai/O=AxdLog/OU=AxdLog Certificate Authority/CN=AxdLog Root CA
                serial:27:EB:AF:AC:AE:A7:CF:37:69:22:E8:96:75:8D:4C:1D:47:EF:FC:8A:F5:C0:56:03:33:6B:BD:C7:05:09:CD:0E

            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            Authority Information Access:
                OCSP - URI:http://ocsp.example.com

    Signature Algorithm: sha512WithRSAEncryption
         97:ef:c3:fa:fe:b6:0f:bb:31:59:c3:51:27:ae:1b:85:eb:28:
         ...
         e0:f4:6b:f5:eb:f5:bc:8e:5c:5a:6e:37:35:5e:f1:a6:4a:1a:
         67:6a:93:4d:e2:8c:ab:18
-----BEGIN CERTIFICATE-----
MIIHMzCCBRugAwIBAgIhAMm6bwwNhFiHKd1j6YNEaKCId8OAN6KoZWRNOpaA6qTV
...
kyXpDtpm2zP/sPmahMhdGu/b+50ubLAQaVzF+svDzoaX7DRrBbup+Iox735p4PRr
9ev1vI5cWm43NV7xpkoaZ2qTTeKMqxg=
-----END CERTIFICATE-----
Response Verify Ok
./newcerts/ocsp_test1.axdlog.com.crt: good

```

其中有

```bash
#未被吊銷
Cert Status: good
```

吊銷後，狀態值會變成`revoked`

### Revoking A Certificate
執行如下命令吊銷證書`ocsp_test1.${common_name}.crt`

```bash
cd "${intermediate_dir}"

openssl ca -config ./openssl.cnf -passin pass:"${pass_phrase}" -revoke ./newcerts/ocsp_test1."${common_name}".crt
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
./newcerts/ocsp_test1.axdlog.com.crt: revoked
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
* 2018.07.27 18:24 Fri America/Boston
    * 勘誤，更新，遷移到新Blog
* 2019.01.25 22:30 Fri America/Boston
    * 重新編寫


[openssl]:https://www.openssl.org


<!-- End -->
