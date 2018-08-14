---
title: Ansible Brief Introduction and Installation On GNU/Linux
slug: Ansible Brief Introduction and Installation On GNU Linux
date: 2016-03-27T12:33:43+08:00
lastmod: 2018-08-05T10:05:34+08:00
draft: false
keywords: ["AxdLog", "Ansible", "Python"]
description: "Ansible Brief Introduction and Installation On GNU/Linux"
categories:
- DevOps
tags:
- Ansible

comment: true
toc: true

---


[Ansible][ansible]是一款IT自動化工具，可用於配置系統、部署軟件、安排更爲高級的IT任務，如連續部署或滾動更新。

>Ansible is an IT automation tool. It can configure systems, deploy software, and orchestrate more advanced IT tasks such as continuous deployments or zero downtime rolling updates. --- [About Ansible](http://docs.ansible.com/ansible/)

本文簡要介紹[Ansible][ansible]，並通過Shell腳本實現在GNU/Linux中的安裝。

<!--more-->

## Origins
[Ansible][ansible]的起源

### Author
[Michael DeHaan](https://www.linkedin.com/in/michaeldehaan/)是[Ansible][ansible]的作者，他先後在[Red Hat][redhat]、[Puppet][puppet]任職。

[Cobbler](https://en.wikipedia.org/wiki/Cobbler_(software))是其在[Red Hat][redhat]時的項目。

離開[Puppet][puppet]後，他於`February 23, 2012`開始了[Ansible][ansible]項目。更多細節內容見：

* [THE ORIGINS OF ANSIBLE](https://www.ansible.com/blog/2013/12/08/the-origins-of-ansible)
* [THANKS MICHAEL!](https://www.ansible.com/blog/thanks-michael)


### Ansible
關於 [Ansible][ansible] 這個名字的由來，[Learn Ansible - Russ McKendrick](https://www.packtpub.com/virtualization-and-cloud/learn-ansible)的[Ansible's story](https://www.packtpub.com/mapt/book/virtualization_and_cloud/9781788998758/1/ch01lvl1sec10/ansible%27s-story)章節有簡短的介紹。

[Ansible][ansible]這個詞最早出現在科幻小說作家[Ursula K. Le Guin](https://en.wikipedia.org/wiki/Ursula_K._Le_Guin)於1966年出版的小說[Rocannon's World](https://en.wikipedia.org/wiki/Rocannon%27s_World)中，Ansible是一種虛構的設備，可以比光更快地發送、接收信息。

1974年，[Ursula K. Le Guin](https://en.wikipedia.org/wiki/Ursula_K._Le_Guin)在其另一部小說[The Dispossessed: An Ambiguous Utopia](https://en.wikipedia.org/wiki/The_Dispossessed)中通過探索數學理論使得Ansible這種技術成爲可能。

>In 1974, Ursula K. Le Guin's novel The Dispossessed: An Ambiguous Utopia, was published; this book features the development of the Ansible technology by exploring the (fictional) details of the mathematical theory that would make such a device possible.


## Acquisition
[Red Hat](http://www.redhat.com/en)於 **2015.10.16** 宣佈收購[Ansible][ansible]，詳見 [FAQ: Red Hat Acquires Ansible](http://www.redhat.com/en/about/blog/faq-red-hat-acquires-ansible)。


## Official Site
Site | Link |
:--- | :--- |
Official Site | https://www.ansible.com
GitHub | https://github.com/ansible/ansible
Documentation | https://docs.ansible.com/ansible/
Blog | https://www.ansible.com/blog
Twitter | https://twitter.com/ansible

可通過如下命令查看最新釋出版本

```bash
# wget -qO-
curl -fsL https://releases.ansible.com/ansible/ansible-latest.tar.gz.sha | sed -r -n 's@.*ansible-([[:digit:].]+)\..*$@\1@g;p'
```

## Features
[Ansible][ansible]使用[Python][python]語言開發，控制臺(Control Machine)通過`SSH`管理遠程主機(Managed Node)，可同時對多臺主機進行並行(parallel)操作。

[Ansible][ansible]的版本釋出週期爲4個月。

>Ansible’s release cycles are usually about four months long. Due to this short release cycle, minor bugs will generally be fixed in the next release versus maintaining backports on the stable branch. Major bugs will still have maintenance releases when needed, though these are infrequent. -- [What Version To Pick?](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#what-version-to-pick)

### Idempotence
[Ansible][ansible]具有 **冪等性**(idempotence)，即相同命令運行多次，結果均相同。在配置工具(Configuration Tool)領域中有2種思想：

type|explanation
---|---
idempotence|冪等，每一次的運行結果都相同
convergence|收斂、趨同，運行多次，逐步接近預期效果

關於[Ansible][ansible]的 **idempotence**，其開發者[Michael DeHaan](http://michaeldehaan.net/)寫過一篇 [Idempotence, convergence, and other silly fancy words we use too often](https://groups.google.com/forum/#!msg/ansible-project/WpRblldA2PQ/lYDpFjBXDlsJ)，有興趣者可自行查看。

### Via SSH
[Ansible][ansible]無需安裝數據庫、守護進程(daemon)，所有操作通過SSH進行。

>Ansible by default manages machines over the SSH protocol. -- [Basics / What Will Be Installed](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#basics-what-will-be-installed)


### Push-based
[Ansible][ansible]將主機分爲2類：

Type | Explanation
:--- | :---
Control Machine | 控制主機，用於管理 Managed Node
Managed Node | 被管理的主機

與[Puppet][puppet]等工具相比，[Ansible][ansible]更爲輕量。因爲[Ansible][ansible]無需在各node中添加數據庫、安裝`agent`或運行`daemon`(守護進程等)。

站在`Managed Node`的角度，二者的顯著差異是：

* [Ansible][ansible]屬於 **push-based**
    * 由`Control Machine`推送
* [Puppet][puppet]屬於 **pull-based**
    * 由`Managed Node`通過安裝的agent進行週期性檢查，從`Control Machine`拉取配置信息到本地

## Requirements
[Python][python]版本要求

Host Type | Python version
---|---
Control Machine | Python 2 (2.6 or 2.7) or Python 3 ( >= 3.5)
Managed Node | Python 2 (2.6 or 2.7) or Python 3 ( >= 3.5)

>If you have SELinux enabled on remote nodes, you will also want to install `libselinux-python` on them before using any copy/file/template related functions in Ansible. -- [Managed Node Requirements](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#managed-node-requirements)


### Control Machine
> Please note that some modules and plugins have additional requirements. For modules these need to be satisfied on the ‘target’ machine and should be listed in the module specific docs.

### Managed Node

* 如果遠程主機中的`SELinux`已啓用，則須安裝`libselinux-python`以支持copy、file、template等功能；
* [Ansible][ansible]默認使用的解釋器(interpreter)位於`/usr/bin/python`，但某些新的Linux發行版默認只有Python3 (`/usr/bin/python3`)，可通過變量`ansible_python_interpreter`設置路徑或安裝Python2；


## Installation
[Ansible][ansible]官方文檔 [Installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)，介紹了[Ansible][ansible]的安裝、配置方式。

[Ansible][ansible]的代碼公佈在GitHub上

Type | Url
---|---
GitHub | https://github.com/ansible/ansible
Tarball | http://releases.ansible.com/ansible/


[Ansible][ansible]官方提供包管理器安裝、pip安裝、源碼編譯安裝、Tar包安裝等安裝方式。詳見 [Installing the Control Machine](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-machine)

爲了安裝最新釋出的版本，此處選擇通過Tar包安裝，以下爲Shell腳本實現。

### Shell Script
```bash
#!/usr/bin/env bash
set -u  #Detect undefined variable
set -o pipefail #Return return code in pipeline fails

# Write: MaxdSre
# Date: Sun 2018-08-05 09:22:32 +0800 CST
# Target: Installing Ansible On GNU/Linux Via Tarball Package.

[[ "$UID" -ne 0 ]] && echo 'this script requires superuser privileges (eg. root, su).' && exit

installation_dir=${installation_dir:-'/opt/Ansible'}
config_dir=${config_dir:-'/etc/ansible'}
readonly temp_save_dir='/tmp'
readonly bak_suffix='_bak'

python_interpreter=${python_interpreter:-'python'}
[[ -n $(which python3 2> /dev/null || command -v python3 2> /dev/null) ]] && python_interpreter='python3'

download_tool=${download_tool:-'wget -qO-'}
[[ -n $(which curl 2> /dev/null || command -v curl 2> /dev/null) ]] && download_tool='curl -fsL'

latest_release_pack_url='https://releases.ansible.com/ansible/ansible-latest.tar.gz'
# https://releases.ansible.com/ansible/ansible-latest.tar.gz
# https://releases.ansible.com/ansible/ansible-latest.tar.gz.sha
latest_sha256_info=$($download_tool "${latest_release_pack_url}.sha")
# 747e4cca09c10833ffe3a7c53af310d2d387bd4f499ec6e1bde60662606aaff8  ansible-2.6.2.tar.gz
sha256_dgst="${latest_sha256_info%% *}"
online_release_version=$(echo "${latest_sha256_info}" | sed -r -n 's@.*ansible-([[:digit:].]+)\..*$@\1@g;p')

is_need_download=${is_need_download:-0}
is_existed=${is_existed:-0}
setup_python_path=${setup_python_path:-"${installation_dir}/setup.py"}

if [[ -s "${setup_python_path}" ]]; then
    cd "${installation_dir}"
    [[ $($python_interpreter "${setup_python_path}" --version) != "${online_release_version}" ]] && is_need_download=1
    is_existed=1
else
    is_need_download=1
fi

echo -e "online release version is ${online_release_version}\nis existed ${is_existed}, is need download ${is_need_download}."

# remote existed files about ansible
# sudo rm -rf /etc/ansible/ /opt/Ansible /usr/local/lib/python*/dist-packages/ansible* /usr/local/bin/ansible*

if [[ "${is_need_download}" -eq 1 ]]; then
    temp_pack_save_path="${temp_save_dir}/${latest_release_pack_url##*/}"
    $download_tool "${latest_release_pack_url}" > "${temp_pack_save_path}"

    if [[ $(shasum -a 256 "${temp_pack_save_path}" 2>/dev/null | sed -r 's@^[[:space:]]*([^[:space:]]+).*$@\1@g') != "${sha256_dgst}" ]]; then
        echo 'fail to sha256 dgst verifin'
        exit
    fi

    if [[ "${is_existed}" -eq 1 && ! -d "${installation_dir}${bak_suffix}" ]]; then
        mv "${installation_dir}" "${installation_dir}${bak_suffix}"
    fi

    [[ -d "${installation_dir}" ]] || mkdir -p "${installation_dir}"

    # decompress
    tar xf "${temp_pack_save_path}" -C "${installation_dir}" --strip-components=1

    # cp config file to /etc
    if [[ "${is_existed}" -ne 1 && -d "${installation_dir}/examples/" ]]; then
        [[ -d "${config_dir}" ]] || mkdir -p "${config_dir}"
        cp -R "${installation_dir}"/examples/* "${config_dir}"
    fi

    # install
    if [[ -s "${setup_python_path}" ]]; then
        cd "${installation_dir}"
        ${python_interpreter} "${setup_python_path}" build &> /dev/null
        ${python_interpreter} "${setup_python_path}" install &> /dev/null
    fi

    # /usr/local/lib/python2.x/dist-packages/ansible
    # /usr/local/lib/python3.x/dist-packages/ansible*
    [[ -f "${temp_pack_save_path}" ]] && rm -f "${temp_pack_save_path}"

    if [[ -s '/usr/local/bin/ansible' ]]; then
        [[ -d "${installation_dir}${bak_suffix}" ]] && rm -rf "${installation_dir}${bak_suffix}"

        #$USER exist && $SUDO_USER not exist, then use $USER
        [[ -n "${USER:-}" && -z "${SUDO_USER:-}" ]] && login_user="$USER" || login_user="$SUDO_USER"

        echo -e 'version info\n'
        if [[ "${login_user}" == 'root' ]]; then
            ansible --version
        else
            sudo -u "${login_user}" ansible --version
        fi

    else
        [[ -d "${installation_dir}" ]] && rm -rf "${installation_dir}"
        [[ -d "${installation_dir}${bak_suffix}" ]] && mv "${installation_dir}${bak_suffix}" "${installation_dir}"
        echo 'fail to install/upgrade ansible'
    fi

fi

# Script End
```

[Ansible][ansible]的可執行程序默認安裝在目錄`/usr/local/bin/`

```bash
# Debian GNU/Linux 9.5 (stretch)

# which ansible
/usr/local/bin/ansible

# ansible --version
ansible 2.6.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/maxdsre/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.5/dist-packages/ansible-2.6.2-py3.5.egg/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.5.3 (default, Jan 19 2017, 14:11:04) [GCC 6.3.0 20170118]
```

## Configuration Files
[Ansible][ansible]官方文檔 [Ansible Configuration Settings
](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)

變量 `ANSIBLE_CONFIG`　用於覆蓋默認配置文件

配置文件路徑

file | explanation
---|---
/etc/ansible/ansible.cfg | 配置文件，可選
~/.ansible.cfg | 覆蓋默認配置，可選


## Command Line Tools
[Ansible][ansible]提供如下命令

command | explanation
---|---
[ansible](https://docs.ansible.com/ansible/latest/cli/ansible.html)|Define and run a single task 'playbook' against a set of hosts
[ansible-config](https://docs.ansible.com/ansible/latest/cli/ansible-config.html)|View, edit, and manage ansible configuration.
ansible-connection|
[ansible-console](https://docs.ansible.com/ansible/latest/cli/ansible-console.html)|REPL console for executing Ansible tasks.
[ansible-doc](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html)|plugin documentation tool
[ansible-galaxy](https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html)|
[ansible-inventory](https://docs.ansible.com/ansible/latest/cli/ansible-inventory.html)|
[ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html)|Runs Ansible playbooks, executing the defined tasks on the targeted hosts.
[ansible-pull](https://docs.ansible.com/ansible/latest/cli/ansible-pull.html)|pulls playbooks from a VCS repo and executes them for the local host
[ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)|encryption/decryption utility for Ansible data files


## Components
[Ansible][ansible]通過[playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)進行配置、部署工作，具體功能由各[module](https://docs.ansible.com/ansible/latest/user_guide/modules.html)實現。爲提升代碼的複用(reuse)性，可使用[role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_roles.html)組織`playbook`，而`role`的創建可通過[Ansible Galaxy](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html)實現。

關於`playbook`，中文被翻譯爲`劇本`，但個人認爲 **`藍圖`** 或 **`任務清單`** 更爲貼切、更易於理解。

### Modules
`module`在[Ansible][ansible]中佔據重要地位，目前有如下模塊：

* [Cloud modules](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html)
* [Clustering modules](https://docs.ansible.com/ansible/latest/modules/list_of_clustering_modules.html)
* [Commands modules](https://docs.ansible.com/ansible/latest/modules/list_of_commands_modules.html)
* [Crypto modules](https://docs.ansible.com/ansible/latest/modules/list_of_crypto_modules.html)
* [Database modules](https://docs.ansible.com/ansible/latest/modules/list_of_database_modules.html)
* [Files modules](https://docs.ansible.com/ansible/latest/modules/list_of_files_modules.html)
* [Identity modules](https://docs.ansible.com/ansible/latest/modules/list_of_identity_modules.html)
* [Inventory modules](https://docs.ansible.com/ansible/latest/modules/list_of_inventory_modules.html)
* [Messaging modules](https://docs.ansible.com/ansible/latest/modules/list_of_messaging_modules.html)
* [Monitoring modules](https://docs.ansible.com/ansible/latest/modules/list_of_monitoring_modules.html)
* [Net Tools modules](https://docs.ansible.com/ansible/latest/modules/list_of_net_tools_modules.html)
* [Network modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html)
* [Notification modules](https://docs.ansible.com/ansible/latest/modules/list_of_notification_modules.html)
* [Packaging modules](https://docs.ansible.com/ansible/latest/modules/list_of_packaging_modules.html)
* [Remote Management modules](https://docs.ansible.com/ansible/latest/modules/list_of_remote_management_modules.html)
* [Source Control modules](https://docs.ansible.com/ansible/latest/modules/list_of_source_control_modules.html)
* [Storage modules](https://docs.ansible.com/ansible/latest/modules/list_of_storage_modules.html)
* [System modules](https://docs.ansible.com/ansible/latest/modules/list_of_system_modules.html)
* [Utilities modules](https://docs.ansible.com/ansible/latest/modules/list_of_utilities_modules.html)
* [Web Infrastructure modules](https://docs.ansible.com/ansible/latest/modules/list_of_web_infrastructure_modules.html)
* [Windows modules](https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html)


所有模塊 [All Modules](https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html)列表。

因[Ansible][ansible]一直在開發、維護中，每當一個版本釋出，某些模塊中的指令可能會被廢除，或添加新的指令，建議經常翻閱[官方文檔](https://docs.ansible.com/ansible)。


## Getting Started
如何使用[Ansible][ansible]，詳見 [Getting Started](https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html)

## Bibliography
* [raymii.org](https://raymii.org/s/tags/ansible.html)


## References
* [Ansible Documentation](https://docs.ansible.com/ansible)


## Change Logs
* 2016.03.27 12:31 Sun Asia/Beijing
    * 初稿修改完成
* 2016.03.28 10:58 Mon Asia/Beijing
    * 添加`Bibliography`
* 2018-08-05 10:05 Sun Asia/Shanghai
    * 勘誤，更新，遷移到新Blog


[redhat]:https://www.redhat.com/en
[ansible]:https://www.ansible.com
[puppet]:https://puppetlabs.com
[python]:https://www.python.org

<!-- End -->
