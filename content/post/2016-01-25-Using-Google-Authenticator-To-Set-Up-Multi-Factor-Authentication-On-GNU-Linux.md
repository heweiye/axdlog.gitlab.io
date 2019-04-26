---
title: Using Google Authenticator To Set Up Multi-Factor Authentication On GNU/Linux
slug: Using Google Authenticator To Set Up Multi-Factor Authentication On GNU Linux
date: 2016-01-25T21:01:52+08:00
lastmod: 2018-07-12T12:07:52-04:00
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
---

PAM is a dynamic authentication mechanism that enhances system security by multiple authentication

[PAM](https://en.wikipedia.org/wiki/Linux_PAM 'Wikipedia') is a dynamic authentication mechanism that enhances system security by multiple authentication. [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator) is a two-factor authentication application which is based on  [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm 'WikiPedia') and [HOTP](https://en.wikipedia.org/wiki/HMAC-based_One-time_Password_Algorithm 'WikiPedia'). It generates token (survival 30 seconds) via mobile device.

This article documents how to set up 2-step verification for GNU/Linux desktop, remote ssh conection, and implement the entire process through a shell script.

<!--more-->

## Introduction
Google's official document about `2-Step Verification` can be found at <https://www.google.com/landing/2step/>.

Installing documents [2-Step Verification](https://support.google.com/accounts/topic/7189195).

The code of module **google-authenticator-libpam** is hosted on [GitHub](https://github.com/google/google-authenticator-libpam).

## Shell Script
Shell script is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/GoogleAuthenticator.sh), usage info

```bash
# curl -fsL / wget -qO-

# if need help, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/GoogleAuthenticator.sh | bash -s --
```

<!-- [![asciicast](https://asciinema.org/a/189215.png)](https://asciinema.org/a/189215?autoplay=1) -->

<script src="https://asciinema.org/a/189215.js" id="asciicast-189215" async></script>

## Preparation
Operations in this article must meet the following requirements:

* Target hosts is running GNU/Linux system;
* Can visit [Google](https://www.google.com/) site;
* Mobile advice, more details can be found in [Install Google Authenticator](https://support.google.com/accounts/answer/1066447?hl=en)；
* Source code of `Google Authenticator` is hosted on [GitHub](https://github.com/google/google-authenticator-libpam), using comile and install.

### Conventions
Operations will be performed in `CentOS 6.9`、`CentOS 7.4`、`Debian Stretch`、`Ubuntu Xenial` and `OpenSUSE Leap`, and the network time is synchronized by `chrony`.

OS|Kernel|SSHD
---|---|---
SUSE Linux Enterprise Server 12 SP3|4.4.114-94.14|7.2p2
Red Hat Enterprise Linux Server release 7.4 (Maipo)|3.10.0-693.21.1|7.4p1
CentOS Linux release 7.4.1708 (Core)|3.10.0-693.21.1|7.4p1
CentOS release 6.9 (Final)|2.6.32-696.23.1|5.3p1
Debian GNU/Linux 9 (stretch)|4.9.0-5|7.4p1
Ubuntu 16.04.4 LTS|4.13.0-37|7.2p2
openSUSE Leap 42.2|4.4.74-18.20-default|7.2p2

Run the following command to install ssh `client` and `server`.

```bash
# CentOS 7
sudo yum install -y -q openssh-server openssh-clients

# Debian / Ubuntu
sudo apt-get update
sudo apt-get install -yq openssh-client openssh-server openssh-sftp-server

# OpenSUSE / SLES
sudo zypper in -yl openssh
```

Defining source code download path `/tmp`
Defining installing path `/opt/googleAuthenticator`

Installing the packages required for compilation.

```bash
#CentOS
sudo yum install -y gcc make autoconf automake libtool pam-devel

#Debian / Ubuntu
sudo apt-get install -y gcc make autoconf automake libtool libpam0g-dev libqrencode3

#OpenSUSE
sudo zypper in -yl gcc make autoconf automake libtool pam-devel
```

### Downloading Source Code
This article use `git` to download code, git link

```http
https://github.com/google/google-authenticator-libpam.git
```

If you don't wanna use `git`, you may could download compressed package in `.zip` format, then decompressing it with the command `unzip`. But it needs utility `zip`、`unzip`

Run the following command to download code

```bash
cd /tmp
git clone https://github.com/google/google-authenticator-libpam.git
```

After the downloading process is finished, the directory is `/tmp/google-authenticator-libpam`.

**Note**：You can also download compressed package via clicking `Clone or download`-->`Download ZIP` in project page.

### Compiling & Installing
Installation directory is `/opt/googleAuthenticator`.

Run the following command to compile and install

**Note**：If the target decompressing directory is in a seperated partition (e.g: `/tmp`), and the file system has attribution `noexec` in `/etc/fstab`. It will make file `./bootstrap.sh`、`./configure` fail to execute, even if you use `sudo`. At this time, you may consider decompress it into another directory (e.g: user home directory).

```bash
#chang umask temporary
umask 022

#create installation directory
[[ -d '/opt/googleAuthenticator' ]] && sudo rm -rf /opt/googleAuthenticator
sudo mkdir -pv /opt/googleAuthenticator

#change directory
cd /tmp/google-authenticator-libpam

#execute script
./bootstrap.sh

#configure install path
./configure --prefix=/opt/googleAuthenticator

#execute 4 tasks in parallel, build application
make -j 4

#install
sudo make install
```

>If you don't have access to "sudo", you have to manually become "root" prior to calling "make install". -- https://github.com/google/google-authenticator-libpam

After the installation is finished, it will prompt

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

Libary file path `/opt/googleAuthenticator/lib/security` may list as `/opt/googleAuthenticator/lib64/security`。

PAM file `pam_google_authenticator.so` is list in

```bash
/opt/googleAuthenticator/lib/security
# or
/opt/googleAuthenticator/lib64/security
```

The relationship of libary path and distribution system

path|dirto list
---|---
/opt/googleAuthenticator/lib/security|Debian/Ubuntu/CentOS/RHEL/SLES
/opt/googleAuthenticator/lib64/security|OpenSUSE

There are 3 sub directories (`bin`, `lib` or `lib64`, `share`) in `/opt/googleAuthenticator/`.

Configuring PATH path, libary path for executable utility `google-authenticator`

```bash
lib_dir=${lib_dir:-'/opt/googleAuthenticator/lib'}
[[ -d '/opt/googleAuthenticator/lib64' ]] && lib_dir='/opt/googleAuthenticator/lib64'
bin_dir=${bin_dir:-'/opt/googleAuthenticator/bin'}

# export libary files
sudo bash -c 'echo '"${lib_dir}"' > /etc/ld.so.conf.d/googleAuthenticator.conf'
#regenerate libary cache, path in Debian is /sbin/ldconfig
sudo ldconfig -v

# add PATH path
sudo bash -c 'echo "export PATH=\$PATH:'"${bin_dir}"'" > /etc/profile.d/googleAuthenticator.sh'
source /etc/profile.d/googleAuthenticator.sh
```

## Running & Configuring Google Authenticator
If you wanna generate the output into file `~/google_authenticator` in non-interactive mode, running the following commands

```bash
echo 'y' | google-authenticator -t -d -f -Q UTF8 -r 3 -R 30 2>&1 > ~/google_authenticator
```

The following is the interactive process

```bash
google-authenticator

Do you want authentication tokens to be time-based (y/n) y
Warning: pasting the following URL into your browser exposes the OTP secret to Google:

#barcode url link
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@centos%3Fsecret%3DQYCY5YLGZJLCXWIUSPJQGVSQSU%26issuer%3Dcentos

#here is a barcode, you can scan it via `Scan a barcode` in Google Authenticator APP

#secret key, it is used in `Enter provided key` in Google Authenticator APP
Your new secret key is: QYCY5YLGZJLCXWIUSPJQGVSQSU

#verification code
Your verification code is 795124

#5 emergency scratch codes, if you lost you phone, use them to retrieve the correct authentication code
Your emergency scratch codes are:
  64621153
  42898735
  96729389
  90142620
  66573383

# file ~/.google_authenticator is not exist by default
Do you want me to update your "/home/flying/.google_authenticator" file? (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y


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

If the computer that you are logging into isn\'t hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

Contents in file `~/.google_authenticator`

```txt
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
If your mobile phone is running Android system, referring to the section `Android devices` of [Install Google Authenticator](https://support.google.com/accounts/answer/1066447?hl=en)中`Android devices`. Searching [Google Authenticator APP](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) in [Google play](https://play.google.com/store).

After installation, open the app
* Scan a barcode:
* Enter provided key
    * `Enter account name`
    * `Enter your key` (secret key)
    * `Time based` (or `Counter based`)

Authentication token is refreshed every 30s.


## Configuring PAM
Adding `auth required pam_google_authenticator.so` in PAM config file. ——「[PAM Module Instructions](https://github.com/google/google-authenticator-libpam/blob/master/README.md)」

References about PAM

* [Linux PAM](https://en.wikipedia.org/wiki/Linux_PAM 'Wikipedia')
* [Linux-PAM](http://www.linux-pam.org/)
* [CHAPTER 5. USING PLUGGABLE AUTHENTICATION MODULES (PAM) - RedHat](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/Pluggable_Authentication_Modules.html 'Red Hat')
* [LDAP/PAM - Debian](https://wiki.debian.org/LDAP/PAM 'Debian')
* [PAM configuration guide for Debian](http://www.rjsystems.nl/en/2100-pam-debian.php)

**Important**： The following operation is very important, must execute.

### CentOS & RHEL & SLES
CentOS 6、CentOS 7、RHEL 7、SLES 12

path|explain|format
---|---|---
`/lib64/security/`|module path
`/etc/pam.conf`|common configuration file|`application type control module-path module-arguments`
`/etc/pam.d/`|dedicated configuration file|`type control module-path module-arguments`

**auth required pam_google_authenticator.so**

* `auth`: belong to `type` {auth|account|passwd|session}，account authentication and authorization
* `required`: belong to `control` {required|requisite|sufficient|optional|include}，check must approve, or it fails. No matter success or fail, the check process is continue.
* `pam_google_authenticator.so`: (**Important**) relative to directory `/lib64/security/`, the absolute path of the module is `/opt/googleAuthenticator/lib/security/pam_google_authenticator.so`. if you need relative path, just create a soft link to `/lib64/security/`.

Create soft link

```bash
sudo ln -fs /opt/googleAuthenticator/lib/security/pam_google_authenticator.so /lib64/security/
```

### Debian & Ubuntu
Debian Stretch, Ubuntu Xenial中

path|explain|format
---|---|---
`/lib/x86_64-linux-gnu/security/`|module path
`/etc/pam.conf`|common configuration file|`application type control module-path module-arguments`
`/etc/pam.d/`|dedicated configuration file|`type control module-path module-arguments`


**auth required pam_google_authenticator.so**

* `auth`: belong to `type` {auth|account|passwd|session}，account authentication and authorization
* `required`: belong to `control` {required|requisite|sufficient|optional|include}，check must approve, or it fails. No matter success or fail, the check process is continue.
* `pam_google_authenticator.so`: (**Important**) relative to directory `/lib/x86_64-linux-gnu/security/`, the absolute path of the module is `/opt/googleAuthenticator/lib/security/pam_google_authenticator.so`. if you need relative path, just create a soft link to `/lib/x86_64-linux-gnu/security/`.

Create soft link

```bash
sudo ln -fs /opt/googleAuthenticator/lib/security/pam_google_authenticator.so /lib/x86_64-linux-gnu/security/
```

### OpenSUSE Leap
OpenSUSE Leap

path|explain|format
---|---|---
`/lib64/security/`|module path
`/etc/pam.d/`|common configuration file|`type control module-path module-arguments`

**auth required pam_google_authenticator.so**

* `auth`: belong to `type` {auth|account|passwd|session}，account authentication and authorization
* `required`: belong to `control` {required|requisite|sufficient|optional|include}，check must approve, or it fails. No matter success or fail, the check process is continue.
* `pam_google_authenticator.so`: (**Important**) relative to directory `/lib64/security/`, the absolute path of the module is `/opt/googleAuthenticator/lib/security/pam_google_authenticator.so`. if you need relative path, just create a soft link to `/lib64/security/`.

Create soft link

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

### CentOS 7
/etc/pam.d/gdm-password

```bash
auth     [success=done ignore=ignore default=bad] pam_selinux_permit.so
# add new custom rule
auth        required      pam_google_authenticator.so
auth        substack      password-auth
auth        optional      pam_gnome_keyring.so
auth        include       postlogin
```

### Debian Stretch
/etc/pam.d/gdm-password

```bash
#%PAM-1.0
# add new custom rule
auth    required        pam_google_authenticator.so
auth    requisite       pam_nologin.so
auth    required    pam_succeed_if.so user != root quiet_success
@include common-auth
auth    optional        pam_gnome_keyring.so
@include common-account
```

### Ubuntu Xenial
/etc/pam.d/lightdm

```bash
#%PAM-1.0
# add new custom rule
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

To make chage effect, logging out system. While re-login, it will prompt `Verification code`. Input the authentication toke listed in Google Authenticator APP. If token is valid, it will prompt input username, password as normal. Or it will prompt error info.


## SSH Authentication
`ssh` is the client side of [Open SSH](https://en.wikipedia.org/wiki/OpenSSH), and `sshd` is its server side which used to connect remote host.

Configuration file

| type | name | configurations |
| :--- | :--- | :--- |
| Server | `sshd` | `/etc/ssh/sshd_config` |
| Client | `ssh` | `/etc/ssh/ssh_config` |

The path of PAM configuration file about `sshd` is `/etc/pam.d/sshd`.

The following files are used to configure `Google Authenticator` for SSH.

* `/etc/pam.d/sshd`
* `/etc/ssh/sshd_config`

Run the following command to make change effect.

```bash
sudo systemctl restart sshd
# sudo systemctl restart sshd.service
```

**Note**： If system has enabled `SELinux`，SSH will repeatedly ask authorization token even if the code is valid, log output like

>sshd(pam_google_authenticator)[13445]: Failed to update secret file "/root/.google_authenticator": Permission denied
>error: PAM: Authentication failure for root from $IP

To solve it, just add directive `secret` after `auth required pam_google_authenticator.so`.

By default, the path of file generated by `google-authenticator` is `~/.google_authenticator`. In order to enable PAM to read the file `~/.google_authenticator` with the required key, copy it to the `~/.ssh/` directory (change the file owner at the same time), and then set the command `secret=${ HOME}/.ssh/.google_authenticator`. In the condition, even if SELinux is enabled, PAM can still read the required file content without being intercepted by SELinux.

```bash
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator
# or
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator [authtok_prompt=Verification code: ]
```

Configuration method, RHEL and CentOS is same, Debian and Ubuntu is same, OpenSUSE and SLES is same.

### CentOS7 With SSH Key
SSH connection via key

* /etc/pam.d/sshd

```bash
#%PAM-1.0
auth       required     pam_sepermit.so
#auth       substack     password-auth
auth       required     pam_google_authenticator.so
# auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator    # If SELinux Enabled
auth       include      postlogin
```

uncomment the symbol `#` in line `auth substack password-auth`, add new line after it

```
auth required pam_google_authenticator.so
```

Because it is logged in with a key, comment `password-auth` skips the step of password authentication.

* /etc/ssh/sshd_config

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**Important**： Must add directive `AuthenticationMethods` if you're login via ssh key, otherwise it will still just verify key then login directly.

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

Run the following command to restart sshd service
```bash
sudo systemctl restart sshd
```

Demonstration example

```bash
flying@stretch:~$ ssh -C -c blowfish flying@192.168.0.140
Authenticated with partial success.
Verification code:
Last login: Thu Jan 12 18:34:41 2017 from 192.168.0.179
[flying@stretch ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@stretch ~]$ exit
logout
Connection to 192.168.0.140 closed.
flying@stretch:~$
```

### CentOS7 With Password

* /etc/pam.d/sshd

```bash
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       required     pam_google_authenticator.so
auth       include      postlogin
```

add new line after `auth substack password-auth`
```bash
auth required pam_google_authenticator.so
```

**Important**: Please make sure that the `auth substack password-auth` is not commented. Otherwise, it will just verify the token generated by `Google Authenticator`. This is not recommended.

* /etc/ssh/sshd_config
```
UsePAM yes
ChallengeResponseAuthentication yes
```

```bash
#UsePAM
sudo sed -i '/^UsePAM /d;' /etc/ssh/sshd_config
sudo bash -c 'echo "UsePAM yes" >> /etc/ssh/sshd_config'

#ChallengeResponseAuthentication
sudo sed -i -r 's@(ChallengeResponseAuthentication) no@\1 yes@g' /etc/ssh/sshd_config
```

Run the following command to restart sshd service
```bash
sudo systemctl restart sshd
```

Demonstration example

```bash
flying@stretch:~$ ssh -C -c blowfish flying@192.168.0.140
Password:
Verification code:
Last login: Thu Jan 12 18:38:01 2017 from 192.168.0.179
[flying@stretch ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@stretch ~]$ cat .ssh/authorized_keys |wc -l
0
[flying@stretch ~]$ exit
logout
Connection to 192.168.0.140 closed.
flying@stretch:~$
```

### Debian With SSH Key
* /etc/pam.d/sshd

By default, the end of file is

```
# Standard Un*x password updating.
@include common-password
```

Operation:

1. add `auth required pam_google_authenticator.so` at the end of file；
2. comment line `@include common-auth`；

final content

```bash
# Standard Un*x authentication.
#@include common-auth

# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so
```

* /etc/ssh/sshd_config

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**Important**： Must add directive `AuthenticationMethods` if you're login via ssh key, otherwise it will still just verify key then login directly.

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

Run the following command to restart sshd service
```bash
sudo systemctl restart sshd
```

Demonstration example

```bash
[flying@stretch ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@stretch ~]$ ssh -C -c aes256-ctr flying@192.168.0.179
Authenticated with partial success.
Verification code:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms \for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Fri Jan 13 10:07:01 2017 from 192.168.0.140
flying@stretch:~$ sed -n -r '1s@.*"(.*)"$@\1@p' /etc/os-release
Debian GNU/Linux 8 (jessie)
flying@stretch:~$ cat .ssh/authorized_keys | wc -l
1
flying@stretch:~$ exit
logout
Connection to 192.168.0.179 closed.
[flying@stretch ~]$
```


### Debian With Password
* /etc/pam.d/sshd

The end of file is
```
# Standard Un*x password updating.
@include common-password
```

add line `auth required pam_google_authenticator.so` at the end of file.

**Note**： Please don't comment line `@include common-auth`, `@include common-password`. If line `@include common-auth` is commented, it will just verify key then login directly. This is not recommend.

Final content

```bash
# Standard Un*x authentication.
@include common-auth

# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so
```


* /etc/ssh/sshd_config

```bash
UsePAM yes
ChallengeResponseAuthentication yes
```

**Note**： Don't set directive `AuthenticationMethods`, or it will prompt error info

>Permission denied (publickey).


Demonstration example

```bash
[flying@stretch ~]$ cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
[flying@stretch ~]$ ssh -C -c aes256-ctr flying@192.168.0.179
Password:
Verification code:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms \for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Fri Jan 13 10:29:11 2017 from 192.168.0.140
flying@stretch:~$ sed -n -r '1s@.*"(.*)"$@\1@p' /etc/os-release
Debian GNU/Linux 8 (jessie)
flying@stretch:~$ cat .ssh/authorized_keys | wc -l
0
flying@stretch:~$ exit
logout
Connection to 192.168.0.179 closed.
[flying@stretch ~]$
```

### OpenSUSE With SSH Key

* /etc/pam.d/sshd

Default file content

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

Operation:

1. comment line `auth        include     common-auth`；
2. add line `auth    required pam_google_authenticator.so nullok` after it；

final content

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

```
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

**Important**: Must add directive `AuthenticationMethods` if you're login via ssh key, otherwise it will still just verify key then login directly.


### OpenSUSE With Password

* /etc/pam.d/sshd
default file content

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

add line `auth required pam_google_authenticator.so` after line `auth include common-auth`.

**Note**： Please don't comment line `common-auth`, `common-password`. If line `common-auth` is commented, it will just verify key then login directly. This is not recommend.

Final content

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

```bash
UsePAM yes
ChallengeResponseAuthentication yes
```

**Note**： Don't set directive `AuthenticationMethods`, or it will prompt error info

>Permission denied (publickey).


## Error Occuring
Error occuring in compile and install process
```bash
#./bootstrap.sh need install autoconf
./bootstrap.sh: line 15: exec: autoreconf: not found


#./bootstrap.sh http://ask.xmodulo.com/fix-failed-to-run-aclocal.html need install automake
Can\'t exec "aclocal": No such file or directory at /usr/share/autoconf/Autom4te/FileUtils.pm line 326.
autoreconf: failed to run aclocal: No such file or directory


#./bootstrap.sh need install libtool
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


#./configure need install pam-devel
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
    * first draft
* 2017.01.12 18:41 Thu Asia/Shanghai
    * reconfiguration, add specific configuration for different scenes
* 2017.01.13 10:41 Fri Asis/Shanghai
    * add authentication configuration in Debian
* 2017.07.17 16:45 Mon Asia/Shanghai
    * add authentication configuration in OpenSUSE Leap
* 2018.03.21 11:24 Wed America/Boston
    * add authentication configuration in SUSE and while SELinux is enabled
* 2018.04.11 23:00 Wed America/Boston
    * Corrigendum, migrate to new blog
* 2018.07.12 12:07 Thu America/Boston
    * format optimization, add English version

<!-- End -->
