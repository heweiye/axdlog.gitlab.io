---
title: 在GNU/Linux中安裝配置Anaconda和Jupyter Notebook
slug: Setting Up Anaconda And Jupyter Notebook On GNU Linux
date: 2018-04-19T10:42:53-04:00
lastmod: 2019-04-09T13:32:53-04:00
draft: false
keywords: ["Anaconda", "Jupyter", "Jupyter notebook", "SSL", "Shell script"]
description: "如何在GNU/Linux中安裝配置Anaconda和Jupyter Notebook，並通過Shell腳本實現整個操作過程。"
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

[Jupyter Notebook][jupyter]是一款開源的交互式Web應用，使用[Python][python]語言開發。其官方建議通過[Anaconda][anaconda]安裝[Python][python]和[Jupyter Notebook][jupyter]。本文記錄如何通過[Anaconda][anaconda]安裝、配置[Jupyter Notebook][jupyter]，並通過Shell腳本實現整個過程。

>We are changing versioning in Anaconda Distribution from a `major/minor` version scheme to a `year.month` scheme. We made this change to differentiate between *the open source Anaconda Distribution* and *Anaconda Enterprise*, our managed data science platform. Conda, will continue to use a `major/minor` versioning scheme. -- [Anaconda Distribution 2018.12 Released](https://www.anaconda.com/blog/developer-blog/anaconda-distribution-2018-12-released/)


<!--more-->

## 簡介
>We strongly recommend installing Python and Jupyter using the [Anaconda Distribution][anaconda], which includes Python, the Jupyter Notebook, and other commonly used packages for scientific computing and data science. -- [Installing Jupyter](https://jupyter.org/install.html)

Anaconda

>Anaconda Distribution is the easiest way to do Python data science and machine learning. It includes 250+ popular data science packages and the conda package and virtual environment manager for Windows, Linux, and MacOS. Conda makes it quick and easy to install, run, and upgrade complex data science and machine learning environments like Scikit-learn, TensorFlow, and SciPy. Anaconda Distribution is the foundation of millions of data science projects as well as Amazon Web Services' Machine Learning AMIs and Anaconda for Microsoft on Azure and Windows. -- <https://www.anaconda.com/what-is-anaconda/>

Jupyter

>The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more.  -- <https://jupyter.org/>


## 官方文檔

* Anaconda https://docs.anaconda.com/anaconda/
* Jupyter
    * https://jupyter.org/documentation
    * https://jupyter-notebook.readthedocs.io/en/stable/
    * http://jupyter-notebook.readthedocs.io/en/latest/changelog.html


## Shell 腳本
整個安裝、配置過程已通過Shell腳本實現，代碼託管在[GitLab](https://gitlab.com/MaxdSre/axd-ShellScript/blob/master/assets/software/Anaconda.sh)，通過如下命令執行

```bash
# curl -fsL / wget -qO-

# if need help info, specify '-h'
wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Anaconda.sh | sudo bash -s --
```

<script src="https://asciinema.org/a/189217.js" id="asciicast-189217" async></script>

爲方便管理[Jupyter Notebook][jupyter]，本人在腳本中設置了一些命令別名，存放在文件`~/.bashrc`中，具體如下

```bash
# jupyter notebook Start
alias jnl="jupyter notebook list | sed '/running servers/d'"
alias jnb="(nohup jupyter notebook &> /dev/null &); sleep 2; jnl"
alias jne="ps -ef | awk 'match(\$0,/(jupyter-noteboo|Anaconda\/bin)/)&&!match(\$0,/awk/){print \$2}' | xargs kill -9 2> /dev/null"
alias jni="wget -qO- https://gitlab.com/MaxdSre/axd-ShellScript/raw/master/assets/software/Anaconda.sh | sudo bash -s -- -C"
alias jnr="sudo /opt/Anaconda/bin/conda remove"
# jupyter notebook End
```

* `jnl`：查看系統中已運行的server；
* `jnb`：啓動新的server；
* `jne`：關閉所有已啓動的server；
* `jni`：用於安裝包的搜索、安裝和更新，`jni a`可對所有已安裝的包進行更新。
* `jnr`：移除安裝包

## Anaconda
[Anaconda][anaconda] 下載頁面爲 <https://www.anaconda.com/download>，同時支持Python **3.7** 和 **2.7**，根據需要選擇下載對應版本的[Anaconda][anaconda]。


### 版本信息
[Anaconda][anaconda]當前最新釋出版本爲`2019.03`。

可通過如下命令提取最新版本信息

```bash
download_tool='wget -qO-' # curl -fsL

# deprecated
# $download_tool https://www.anaconda.com/download/ | sed -r -n 's@<\/[^>]+>@\n@g;p' | sed -r -n '/>Release Date:/{s@[[:space:]]*<[^>]*>[[:space:]]*@@g;s@^[^:]*:[[:space:]]*(.*)@\1@g;p}; /Anaconda.*Linux-x86_64/{/Installer/{s@.*href="([^"]*)".*@\1@g;s@.*Anaconda[^-]*-([^-]*).*$@\1@g;p;q}}' | sed ':a;N;$!ba;s@\n@|@g'

package_url=$($download_tool https://www.anaconda.com/download/ | sed -r -n 's@<\/[^>]+>@\n@g;p' | sed -r -n '/href=.*Anaconda.*Linux-x86_64/{/Download/!d;s@.*href="([^"]+)".*@\1@g;p;q}')

release_date=$($download_tool https://repo.anaconda.com/archive/ | sed -r -n '/'"${package_url##*/}"'/,/<\/tr>/{s@[[:space:]]*<[^>]+>[[:space:]]*@@g;/^$/d;p}' | sed ':a;N;$!ba;s@\n@|@g' | cut -d'|' -f3 | date +'%B %d, %Y' -f - 2> /dev/null)

release_version=$(echo "${package_url}" | sed -r -n 's@.*Anaconda[^-]*-([^-]*).*$@\1@g;p')

echo "${release_date}|${release_version}"
```

輸出結果

```
April 04, 2019|2019.03
```

釋出日期 | 版本
---|---
April 04, 2019|2019.03
December 21, 2018|2018.12
November 19, 2018|5.3.1
May 30, 2018|5.2.0
February 15, 2018|5.1.0


### 校驗
[Anaconda][anaconda] 並未直接在下載頁面提供安裝包的hash校驗信息，相關信息存放在文檔頁 [Anaconda installer file hashes](https://docs.anaconda.com/anaconda/install/hashes/)。其中頁面 [Hashes for all files](https://docs.anaconda.com/anaconda/install/hashes/all) 列出了[Anaconda][anaconda]各歷史版本的sha256hash值。

此處以`Anaconda3-2019.03-Linux-x86_64.sh`爲例，頁面 [Hashes for Anaconda3-2019.03-Linux-x86_64.sh](https://docs.anaconda.com/anaconda/install/hashes/Anaconda3-2019.03-Linux-x86_64.sh-hash) 列出了安裝包的相關信息。

item|details
---|---
Last Modified | `2019-04-04 16:00:31`
size(byte) | `685906562`
md5 | `43caea3d726779843f130a7fb2d380a2`
sha256 | `45c851b7497cc14d5ca060064394569f724b67d9b5f98a926ed49b834a6bb73a`

可通過如下命令進行Hash校驗

```bash
file_path='~/Downloads/Anaconda3-2019.03-Linux-x86_64.sh'

# via sha256sum
sha256sum "${file_path}"

# via openssl
openssl dgst -sha256 "${file_path}"
```

演示示例

```bash
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $sha256sum Anaconda3-2019.03-Linux-x86_64.sh
45c851b7497cc14d5ca060064394569f724b67d9b5f98a926ed49b834a6bb73a  Anaconda3-2019.03-Linux-x86_64.sh
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $openssl dgst -sha256 Anaconda3-2019.03-Linux-x86_64.sh
SHA256(Anaconda3-2019.03-Linux-x86_64.sh)= 45c851b7497cc14d5ca060064394569f724b67d9b5f98a926ed49b834a6bb73a
┌─[maxdsre@Stretch]─[~/Downloads]
└──╼ $
```

### 安裝
sha256校驗通過後，參照官方文檔 <https://docs.anaconda.com/anaconda/install/> 進行安裝。

執行如下命令進行安裝

```bash
bash ~/Downloads/Anaconda3-2019.03-Linux-x86_64.sh
```

[Anaconda][anaconda]默認採用的是 **交互式** 安裝，需要用戶參與，詳細說明見官方文檔 [Installing on Linux](https://docs.anaconda.com/anaconda/install/linux)。

[Anaconda][anaconda]同樣支持 **無交互式** 安裝，根據官方文檔

* https://conda.io/docs/user-guide/install/index.html#installing-in-silent-mode
* https://conda.io/docs/user-guide/install/linux.html#install-linux-silent
* https://conda.io/docs/user-guide/install/macos.html#install-macos-silent

安裝時添加參數`-b`即可實現 **靜默** 安裝，如果需要自定義安裝路徑，添加參數`-p`，具體說明如下：

1. **-b**： Batch mode with no PATH modifications to `~/.bashrc`. Assumes that you agree to the license agreement. Does not edit the `.bashrc` or `.bash_profile` files.
2. **-p**： Installation prefix/path.
3. **-f**： Force installation even if prefix -p already exists.

此處以安裝路徑爲`/opt/Anaconda`爲例，安裝命令爲

```bash
installation_dir='/opt/Anaconda'
bash ~/Downloads/Anaconda3-2019.03-Linux-x86_64.sh -b -f -p ${installation_dir}
```

### $PATH
[Anaconda][anaconda]安裝完成後，仍無法直接執行命令`conda`。原因是可執行文件路徑`${installation_dir}/bin/`不在環境變量`$PATH`中。需要通過文件`${installation_dir}/bin/activate`將其添加到`$PATH`中。

[Anaconda][anaconda]給出的方案是在文件`~/.bashrc`中添加如下設置

```bash
export PATH="${installation_dir}/bin:$PATH"
```

但個人建議將其放置在目錄`/etc/profile.d/`中，這樣其他用戶也可以使用`conda`。

執行如下命令進行更新

```bash
# 1 - in $PATH
conda update conda
# 2 - absolute path
/opt/Anaconda/conda update conda
```

## Jupyter
[Anaconda][anaconda]中已集成有[Jupyter Notebook][jupyter]，稍作配置即可使用。

[Jupyter Notebook][jupyter]默認監聽端口`8888`，採用密碼或token方式登入。

通過如下命令生成配置文件

```bash
jupyter notebook --generate-config
```

生成的配置文件路徑爲

```
~/.jupyter/jupyter_notebook_config.py
```

配置文檔 [Configuration Overview](https://jupyter-notebook.readthedocs.io/en/stable/config_overview.html)

需要設置的主要有以下指令

* `NotebookApp.allow_root`
* `NotebookApp.base_url`
* `NotebookApp.ip`
* `NotebookApp.port`
* `c.NotebookApp.password`
* `NotebookApp.allow_password_change`
* `NotebookApp.notebook_dir`
* `NotebookApp.certfile`
* `NotebookApp.disable_check_xsrf`

如果是遠程訪問，則需要在防火牆中開啓Jupyter端口(默認爲`8888`)。

### 生成密碼
[Jupyter Notebook][jupyter]採用的是交互式方式生成Hash密碼。

#### 交互模式
官方文檔 [Alternatives to token authentication](https://jupyter-notebook.readthedocs.io/en/stable/security.html#alternatives-to-token-authentication) 提到

>New in version 5.0: **jupyter notebook password** command is added.

生成的Hash密碼存儲路徑爲

```
~/.jupyter/jupyter_notebook_config.json
```

演示過程如下 (密碼`Axdlog@$(date +'%Y')_Python`)

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

#### 靜默模式
但交互式方式不利於Shell腳本的自動化操作， 能否實現非交互式生成密碼呢？

此處以[Python 3][python]爲例，[Jupyter Notebook][jupyter]生成密碼使用到如下文件，裏面定義了相關函數，如`passwd`、`passwd_check`、`set_password`、`persist_config`。

```bash
${installation_dir}/lib/python3.7/site-packages/notebook/auth/security.py
```

提取其中部分代碼，寫入文件`/tmp/passwd.py`，仍以密碼`Axdlog@$(date +'%Y')_Python`爲例。

```python
# For Python 3.6/3.7
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

pass_str="Axdlog@$(date +'%Y')_Python"
result_str=passwd(pass_str)

# print hashed passwd
print(result_str)

# check if is legal
print(passwd_check(result_str,pass_str))
```

執行如下命令

```bsah
jupyter run /tmp/test.py
```

操作過程

```bash
┌─[maxdsre@Stretch]─[/tmp]
└──╼ $jupyter run /tmp/test.py
sha1:8b4745563177:9773f5dab5d841ba808ef80e953bfa667c84e8d2
True
┌─[maxdsre@Stretch]─[/tmp]
└──╼ $
```
可以看到驗證結果爲`True`。

但這樣做會有一個問題：[Anaconda][anaconda]同時支持[Python][python] 3 和 2，文件`security.py`有2份，需要分別處理；如果[Jupyter Notebook][jupyter]更新了其中的代碼，則Shell腳本也須作出相應更改，本人無法確保能夠及時作出響應。基於此考慮，未將自定義密碼功能加入Shell腳本。


### SSL證書
爲提高數據傳輸安全，可配置SSL證書，官方文檔見 [Using SSL for encrypted communication](http://jupyter-notebook.readthedocs.io/en/stable/public_server.html#using-ssl-for-encrypted-communication)。

通過`openssl`創建自簽SSL證書，該過程是一個交互式過程。但可以通過指令`-subj`實現免交互操作。

Shell代碼示例

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

執行過程如下

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

## Jupyter 插件
爲增強[Jupyter Notebook][jupyter]功能，可選擇安裝插件[jupyter_contrib_nbextensions](https://github.com/ipython-contrib/jupyter_contrib_nbextensions)，官方文檔 [Unofficial Jupyter Notebook Extensions](http://jupyter-contrib-nbextensions.readthedocs.io/en/latest/install.html)。

```bash
# Install the python package
conda install -c conda-forge jupyter_contrib_nbextensions

# Install javascript and css files
jupyter contrib nbextension install --user
```

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-05-30_16-36-52.png)


## 測試
### 命令行
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

### Web 瀏覽器
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-24.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-52.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-22-57.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-23-33.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-23-47.png)

### 私有 SSL
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-22.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-33.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-24-49.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-19_jupyter_notebook_anaconda/2018-04-19_10-25-03.png)


## 參考資料
* [Configuring the Notebook and Server](https://testnb.readthedocs.io/en/stable/examples/Notebook/Configuring%20the%20Notebook%20and%20Server.html)
* [Examples](http://jupyter-notebook.readthedocs.io/en/latest/examples/Notebook/examples_index.html)


## 更新日誌
* 2018.04.19 10:42 Wed America/Boston
	* 初稿完成
* 2018.07.11 11:38 Wed America/Boston
    * 更新版本至 `5.2`
* 2018.11.29 10:45 Thu America/Boston
    * 更新版本至 `5.3.1`
* 2018.12.22 19:27 Sat America/Boston
    * Anaconda版本號格式由`major/minor`更改爲`year.month`，更新版本至`2018.12`，詳情見 [Anaconda Distribution 2018.12 Released](https://www.anaconda.com/blog/developer-blog/anaconda-distribution-2018-12-released/)。
* 2019.04.09 13:32 Tue America/Boston
    * 更新版本至`2019.03`


[anaconda]:https://www.anaconda.com "The Most Popular Python Data Science Platform"
[jupyter]:https://jupyter.org
[python]:https://www.python.org

<!-- End -->
