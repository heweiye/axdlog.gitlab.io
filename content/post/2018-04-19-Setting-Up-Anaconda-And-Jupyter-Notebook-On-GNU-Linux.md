---
title: Setting Up Anaconda And Jupyter Notebook On GNU/Linux
slug: Setting Up Anaconda And Jupyter Notebook On GNU Linux
date: 2018-04-19T10:42:53-04:00
lastmod: 2018-07-11T11:38:53-04:00
draft: false
keywords: ["AxdLog", "Anaconda", "Jupyter", "Jupyter notebook", "SSL", "Shell script"]
description: "How to set up Anaconda and Jupyter notebook on GNU/Linux via Shell script"
categories:
- Python
tags:
- Python
- Anaconda
- Jupyter
- SSL
- Shell Script

comment: true
toc: true

---

The [Jupyter Notebook][jupyter] is an open-source interactive web application developed by [Python][python] language. The official recommends installing [Python][python] and [Jupyter Notebook][jupyter] using the [Anaconda Distribution][anaconda]. This article documents how to set up [Anaconda][anaconda] and [Jupyter Notebook][jupyter], and implement the entire process through a shell script.

<!--more-->

## Introduction
>We strongly recommend installing Python and Jupyter using the [Anaconda Distribution][anaconda], which includes Python, the Jupyter Notebook, and other commonly used packages for scientific computing and data science. -- [Installing Jupyter](https://jupyter.org/install.html)

Anaconda

>Anaconda Distribution is the easiest way to do Python data science and machine learning. It includes 250+ popular data science packages and the conda package and virtual environment manager for Windows, Linux, and MacOS. Conda makes it quick and easy to install, run, and upgrade complex data science and machine learning environments like Scikit-learn, TensorFlow, and SciPy. Anaconda Distribution is the foundation of millions of data science projects as well as Amazon Web Services' Machine Learning AMIs and Anaconda for Microsoft on Azure and Windows. -- <https://www.anaconda.com/what-is-anaconda/>

Jupyter

>The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more.  -- <https://jupyter.org/>


## Official Document

* Anaconda https://docs.anaconda.com/anaconda/
* Jupyter
    * https://jupyter.org/documentation
    * https://jupyter-notebook.readthedocs.io/en/stable/
    * http://jupyter-notebook.readthedocs.io/en/latest/changelog.html


## Shell Script
The entire installation and configuration process has been implemented through a shell script, the code is hosted on [GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/gnulinux/gnuLinuxPostInstallationConfiguration.sh), usage info

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Anaconda.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189217.js" id="asciicast-189217" async></script>

To facilitate managing [Jupyter Notebook][jupyter], I set up some command aliases in `~/.bashrc`, as follows

```bash
# jupyter notebook Start
alias jnl="jupyter notebook list | sed '/running servers/d'"
alias jnb="(nohup jupyter notebook &> /dev/null &); sleep 2; jnl"
alias jne="ps -ef | awk 'match(\$0,/(jupyter-noteboo|Anaconda\/bin)/)&&!match(\$0,/awk/){print \$2}' | xargs kill -9 2> /dev/null"
alias jni="curl -fsL https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Anaconda.sh | sudo bash -s -- -U"
alias jnr="sudo /opt/Anaconda/bin/conda remove"
# jupyter notebook End
```

* `jnl`：list running server；
* `jnb`：start new server；
* `jne`：stop all started server；
* `jni`：search, install, update package via `conda`， use command `jni a` to update entire conda environment;
* `jnr`: remove package via `conda`

## Anaconda
Official download page is <https://www.anaconda.com/download>, it supports both  Python **3.6** and **2.7**. Choosing the corresponding version according to your needs.

### Release Version
The latest release version of [Anaconda][anaconda] is `5.2`.

You can use the following command to extract the latest version information

```bash
curl -fsL https://www.anaconda.com/download/ | sed -r -n 's@<\/[^>]+>@\n@g;p' | sed -r -n '/>Release Date:/{s@[[:space:]]*<[^>]*>[[:space:]]*@@g;s@^[^:]*:[[:space:]]*(.*)@\1@g;p}; /Anaconda.*Linux-x86_64/{/Installer/{s@.*href="([^"]*)".*@\1@g;s@.*Anaconda[^-]*-([^-]*).*$@\1@g;p;q}}' | sed ':a;N;$!ba;s@\n@|@g'
```

Output results

~~February 15, 2018|5.1.0~~

```
May 30, 2018|5.2.0
```

### Verification
[Anaconda][anaconda] doesn't provide hash verification info for the package directly on its download page. The relevant information is stored on the page [Anaconda installer file hashes](https://docs.anaconda.com/anaconda/install/hashes/). The page [Hashes for all files](https://docs.anaconda.com/anaconda/install/hashes/all) lists the sha256 hash values of the historical versions of [Anaconda][anaconda].

Here use `Anaconda3-5.2.0-Linux-x86_64.sh` as an example, the page [Hashes for Anaconda3-5.2.0-Linux-x86_64.sh](https://docs.anaconda.com/anaconda/install/hashes/Anaconda3-5.2.0-Linux-x86_64.sh-hash) lists the installation package information.

item|details
---|---
Last Modified | `2018-05-30 13:05:43`
size(byte) | `651745206`
md5 | `3e58f494ab9fbe12db4460dc152377b5`
sha256 | `09f53738b0cd3bb96f5b1bac488e5528df9906be2480fe61df40e0e0d19e3d48`

Hash check can be performed by the following command

```bash
file_path='~/Downloads/Anaconda3-5.2.0-Linux-x86_64.sh'

# via sha256sum
sha256sum "${file_path}"

# via openssl
openssl dgst -sha256 "${file_path}"
```

Demonstration example

```bash
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $sha256sum Anaconda3-5.2.0-Linux-x86_64.sh
09f53738b0cd3bb96f5b1bac488e5528df9906be2480fe61df40e0e0d19e3d48  Anaconda3-5.2.0-Linux-x86_64.sh
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $openssl dgst -sha256 Anaconda3-5.2.0-Linux-x86_64.sh
SHA256(Anaconda3-5.2.0-Linux-x86_64.sh)= 09f53738b0cd3bb96f5b1bac488e5528df9906be2480fe61df40e0e0d19e3d48
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $
```

### Installation
After the sha256 check is passed, refer to the official document <https://docs.anaconda.com/anaconda/install/> for installation.

Run the following command to install

```bash
bash ~/Downloads/Anaconda3-5.2.0-Linux-x86_64.sh
```

By default, the installing process of [Anaconda][anaconda] is **interactive** which requires user interaction , details in [Installing on Linux](https://docs.anaconda.com/anaconda/install/linux).

It also supports **silent mode** installation, according to official documents

* https://conda.io/docs/user-guide/install/index.html#installing-in-silent-mode
* https://conda.io/docs/user-guide/install/linux.html#install-linux-silent
* https://conda.io/docs/user-guide/install/macos.html#install-macos-silent

Specifing flag `-b` to make it into silent mode, you can also specify flag `-p` to custom installation path, as explained below:

1. **-b**： Batch mode with no PATH modifications to `~/.bashrc`. Assumes that you agree to the license agreement. Does not edit the `.bashrc` or `.bash_profile` files.
2. **-p**： Installation prefix/path.
3. **-f**： Force installation even if prefix -p already exists.

Here use installation path `/opt/Anaconda` as an example, the installation command is

```bash
installation_dir='/opt/Anaconda'
bash ~/Downloads/Anaconda3-5.2.0-Linux-x86_64.sh -b -f -p ${installation_dir}
```

### $PATH
After [Anaconda][anaconda] is installed, it still can't directly execute the command `conda`. The reason is that the executable path `${installation_dir}/bin/` is not in the environment variable `$PATH.` You need to use `${installation_dir}/bin/activateadd` to add into `$PATH`.

[Anaconda][anaconda] just adds the following directive into file `~/.bashrc`.

```bash
export PATH="${installation_dir}/bin:$PATH"
```

But personal advice is place it under directory `/etc/profile.d/`, so that other users can also use `conda`.

Run the following command to update

```bash
# 1 - in $PATH
conda update conda
# 2 - absolute path
/opt/Anaconda/conda update conda
```

## Jupyter
As [Anaconda][anaconda] includes [Jupyter Notebook][jupyter], what you need to do is change its default configuration.

[Jupyter Notebook][jupyter] listens on the default port `8888`, logging in via a password or token.

Generate the configuration file through the following command

```bash
jupyter notebook --generate-config
```

The generated configuration file path

```
~/.jupyter/jupyter_notebook_config.py
```

Details in [Configuration Overview](https://jupyter-notebook.readthedocs.io/en/stable/config_overview.html)

The important directive

* `NotebookApp.allow_root`
* `NotebookApp.base_url`
* `NotebookApp.ip`
* `NotebookApp.port`
* `c.NotebookApp.password`
* `NotebookApp.allow_password_change`
* `NotebookApp.notebook_dir`
* `NotebookApp.certfile`
* `NotebookApp.disable_check_xsrf`


For remote access, you need to open the Jupyter port (default is `8888`) in firewall rule.

### Password Generation
[Jupyter Notebook][jupyter] uses an interactive way to generate hashed password.

#### Interactive mode
Official document [Alternatives to token authentication](https://jupyter-notebook.readthedocs.io/en/stable/security.html#alternatives-to-token-authentication) mentioned

>New in version 5.0: **jupyter notebook password** command is added.

The generated hashed password stores in file

```
~/.jupyter/jupyter_notebook_config.json
```

The demo process is as follows (raw password `Axdlog@2018_Python`)

```bash
┌─[maxdsre@Stretch]─[~]
└──╼ $jupyter notebook password
Enter password:
Verify password:
[NotebookPasswordApp] Wrote hashed password to /home/maxdsre/.jupyter/jupyter_notebook_config.json
┌─[maxdsre@Stretch]─[~]
└──╼ $cat ~/.jupyter/jupyter_notebook_config.json
{
  "NotebookApp": {
    "password": "sha1:4b55a390f103:1e3bf1e48ca12fda8a0fad32799fc0ad82e0301c"
  }
}┌─[maxdsre@Stretch]─[~]
└──╼ $
```

#### Silent Mode
But the interactive mode is not conducive to the automatic operation of the shell script. If there is a possibility to generate hashed password in silent mode?

Here use [Python 3][python] as an example, the functions used by [Jupyter Notebook][jupyter] to generated hashed password are `passwd`、`passwd_check`、`set_password`、`persist_config`, they lists in file

```bash
${installation_dir}/lib/python3.6/site-packages/notebook/auth/security.py
```

Extracting the core codes into file `/tmp/passwd.py`, here still use the raw password `Axdlog@2018_Python` as an example.

```bash
# For Python 3.6
import hashlib
import random
from ipython_genutils.py3compat import cast_bytes, str_to_bytes
from notebook.auth.security import passwd_check

salt_len = 12

def passwd(password,algorithm='sha1'):
    h = hashlib.new(algorithm)
    salt = ('%0' + str(salt_len) + 'x') % random.getrandbits(4 * salt_len)
    h.update(cast_bytes(password, 'utf-8') + str_to_bytes(salt, 'ascii'))
    return ':'.join((algorithm, salt, h.hexdigest()))

pass_str='Axdlog@2018_Python'
result_str=passwd(pass_str)

# print hashed passwd
print(result_str)

# check if is legal
print(passwd_check(result_str,pass_str))
```

executing the following command to generated hashed password

```bsah
jupyter run /tmp/test.py
```

operating process

```bash
┌─[maxdsre@Stretch]─[/tmp]
└──╼ $jupyter run /tmp/test.py
sha1:492e18c9b198:5c00c6ea49e8426766ffb698ced3827e579369c6
True
┌─[maxdsre@Stretch]─[/tmp]
└──╼ $
```

The verification result is `True`.

However, there is a problem with this: [Anaconda][anaconda] supports both [Python][python] 3 and 2, there are 2 `security.py` copies need to be processed separately. If [Jupyter Notebook][jupyter] changes the code, the shell script must also be changed accordingly. I'm not ensure that the script can be updated in time. Based on this consideration, this function (custom password setting) was not added to the shell script.

### SSL
To improve the security of data transmission, SSL certificates can be configured. More details in [Using SSL for encrypted communication](http://jupyter-notebook.readthedocs.io/en/stable/public_server.html#using-ssl-for-encrypted-communication).

Here I use command `openssl` to create a self-signed SSL certificate. This process is also in interactive mode. However, you can disable it by flag `-subj`.

Shell code sample

```bash
jupyter_name='Jupyter'
jupyter_conf_dir="/tmp/${jupyter_name}"

self_cert_path=${self_cert_path:-"${jupyter_conf_dir}/${jupyter_name}.pem"}
# openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
[[ -d "${jupyter_conf_dir}" ]] || mkdir -p "${jupyter_conf_dir}"
if [[ ! -f "${self_cert_path}" ]]; then
    cert_C=${cert_C:-'CN'}    # Country Name
    cert_ST=${cert_ST:-'Shanghai'}    # State or Province Name
    cert_L=${cert_L:-'Shanghai'}    # Locality Name
    cert_O=${cert_O:-'MaxdSre'}    # Organization Name
    cert_OU=${cert_OU:-'Python'}    # Organizational Unit Name
    cert_CN=${cert_CN:-'jupyter.org'}    # Common Name
    # https://jupyter.org/community
    cert_email=${cert_email:-'jupyter@googlegroups.com'}    # Email Address

    cert_C="${ip_public_country_code}"
    cert_ST="${ip_public_locate%%.*}"
    cert_L="${ip_public_locate##*.}"

    # expire date 3650 days, crypt type RSA, key length 4096
    openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout "${self_cert_path}" -out "${self_cert_path}" -subj "/C=${cert_C}/ST=${cert_ST}/L=${cert_L}/O=${cert_O}/OU=${cert_OU}/CN=${cert_CN}/emailAddress=${cert_email}" 2> /dev/null
    # openssl rsa -in "${self_cert_path}" -text -noout 2> /dev/null
    [[ -f "${self_cert_path}" ]] && chmod 644 "${self_cert_path}"
fi
```

The testing process is as follows

```bash
# bash -x ssl.sh
+ jupyter_name=Jupyter
+ jupyter_conf_dir=/tmp/Jupyter
+ self_cert_path=/tmp/Jupyter/Jupyter.pem
+ [[ -d /tmp/Jupyter ]]
+ mkdir -p /tmp/Jupyter
+ [[ ! -f /tmp/Jupyter/Jupyter.pem ]]
+ cert_C=CN
+ cert_ST=Shanghai
+ cert_L=Shanghai
+ cert_O=MaxdSre
+ cert_OU=Python
+ cert_CN=jupyter.org
+ cert_email=jupyter@googlegroups.com
+ cert_C=
+ cert_ST=
+ cert_L=
+ openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout /tmp/Jupyter/Jupyter.pem -out /tmp/Jupyter/Jupyter.pem -subj /C=/ST=/L=/O=MaxdSre/OU=Python/CN=jupyter.org/emailAddress=jupyter@googlegroups.com
+ [[ -f /tmp/Jupyter/Jupyter.pem ]]
+ chmod 644 /tmp/Jupyter/Jupyter.pem
```

## Jupyter Extension
In order to enhance the function of [Jupyter Notebook][jupyter], you could consider installing extension [jupyter_contrib_nbextensions](https://github.com/ipython-contrib/jupyter_contrib_nbextensions). More details in [Unofficial Jupyter Notebook Extensions](http://jupyter-contrib-nbextensions.readthedocs.io/en/latest/install.html).

```bash
# Install the python package
conda install -c conda-forge jupyter_contrib_nbextensions

# Install javascript and css files
jupyter contrib nbextension install --user
```

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-05-30_16-36-52.png)


## Testing
### Command Line
```bash
┌─[maxdsre@Stretch]─[~]
└──╼ $jni a
Solving environment: done

# All requested packages already installed.

Finishing executing conda update --all for Anaconda!

┌─[maxdsre@Stretch]─[~]
└──╼ $jnl
┌─[maxdsre@Stretch]─[~]
└──╼ $jnb
https://127.0.0.1:33525/Jupyter/?token=2709e9966fe2772e00a76ebfddfc12ac3d544eef518c530c :: /home/maxdsre/Jupyter
┌─[maxdsre@Stretch]─[~]
└──╼ $jnl
https://127.0.0.1:33525/Jupyter/?token=2709e9966fe2772e00a76ebfddfc12ac3d544eef518c530c :: /home/maxdsre/Jupyter
┌─[maxdsre@Stretch]─[~]
└──╼ $jne
┌─[maxdsre@Stretch]─[~]
└──╼ $jnl
┌─[maxdsre@Stretch]─[~]
└──╼ $
```

### Web Browser
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-24.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-52.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-57.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-23-33.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-23-47.png)

### Private SSL
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-22.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-33.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-49.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-25-03.png)


## Reference
* [Configuring the Notebook and Server](https://testnb.readthedocs.io/en/stable/examples/Notebook/Configuring%20the%20Notebook%20and%20Server.html)
* [Examples](http://jupyter-notebook.readthedocs.io/en/latest/examples/Notebook/examples_index.html)


## Change Logs
* 2018.04.19 10:42 Wed America/Boston
	* first draft
* 2018.07.11 11:38 Wed America/Boston
    * update version to 5.2


[anaconda]:https://www.anaconda.com "The Most Popular Python Data Science Platform"
[jupyter]:https://jupyter.org
[python]:https://www.python.org

<!-- End -->
