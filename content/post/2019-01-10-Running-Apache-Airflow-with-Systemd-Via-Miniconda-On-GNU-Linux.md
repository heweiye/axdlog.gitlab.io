---
title: Running Apache Airflow with Systemd Via Miniconda On GNU/Linux
slug: Running Apache Airflow with Systemd Via Miniconda On GNU Linux
date: 2019-01-10T16:32:16-05:00
lastmod: 2019-04-25T21:58:16-04:00
draft: false
keywords: ["Apache Airflow", "Systemd", "Miniconda"]
description: "How to run Apache Airflow with systemd service via Miniconda on GNU/Linux"
categories:
- Python
tags:
- Python
- Miniconda

comment: true
toc: true

---

This article documents how to run [Apache Airflow][apacheairflow] with [systemd](https://en.wikipedia.org/wiki/Systemd) service on GNU/Linux. Airflow is installed using [Miniconda][miniconda] on AWS ec2 instances (*RHEL 7.6* / *Ubuntu 18.04* / *SLES 15* / *Amazon Linux 2*).

## Introduction
[Apache Airflow][apacheairflow] (or simply Airflow) is a platform to programmatically author, schedule, and monitor workflows.

The Latest release version is **1.10.3** (April 09, 2019), more details in [CHANGELOG](https://github.com/apache/airflow/blob/master/CHANGELOG.txt).

<!--more-->

```bash
# extract latest version info
# curl -fsL / wget -qO-
wget -qO- https://raw.githubusercontent.com/apache/airflow/master/CHANGELOG.txt | sed -n '/^airflow/I{1p}'

# Airflow 1.10.3, 2019-04-09
# Airflow 1.10.2, 2019-01-19
# AIRFLOW 1.10.1, 2018-11-13
```

>Use Airflow to author workflows as **directed acyclic graphs** (DAGs) of tasks. The Airflow scheduler executes your tasks on an array of workers while following the specified dependencies. Rich command line utilities make performing complex surgeries on DAGs a snap. The rich user interface makes it easy to visualize pipelines running in production, monitor progress, and troubleshoot issues when needed.

More details in official document <https://airflow.apache.org>.


## Definition
Configuration definition

Variable | Value | Explanation
---|---|---
umaks | 022 |
$SHELL | /bin/bash
User | airflow |
Group | airflow |
AIRFLOW_HOME | /etc/airflow | defaul is `~/airflow`
AIRFLOW_CONFIG | /etc/airflow/airflow.cfg |
AIRFLOW_RUN | /run/airflow |
AIRFLOW_LOG | ~~/var/log/airflow~~<br>/etc/airflow/logs | prompt fatal error, use default `/etc/airflow/logs` works properly
Miniconda dir | /opt/Miniconda3/ |


## PIP
`pip` is used to install [Apache Airflow][apacheairflow], we need to install it firstly. The following are two methods.

### Via Python3

```bash
umask 022

# For Debian/Ubuntu
sudo apt-get update
sudo apt-get install -y python3 python3-pip
sudo apt-get install python3-pip --reinstall -y

# For SUSE
sudo zypper in -yl python3 python3-pip

# For RHEL/AMZN2/CentOS
# sudo yum install -y python36 python36-pip
sudo yum install -y python3
```

Upgrading to latest version

```bash
# For Debian/Ubuntu
sudo apt-get update
sudo apt-get install python3 -y
sudo apt-get install python3-pip --reinstall -y

# pip3 --version
# pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)

# ModuleNotFoundError: No module named 'pip._internal'
# python3 -m pip --version

# create soft link to /usr/local/bin/
[[ -d /usr/local/bin ]] || sudo mkdir -pv /usr/local/bin
sudo ln -fs $(which pip3) /usr/local/bin/pip

# `umask 022` makes directories under /usr/local/lib/python3.x/dist-packages has o+rx permission
# Upgrade pip version -U/--upgrade
(umask 022; sudo pip install pip --upgrade)

# The directory '/home/ubuntu/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
# sudo -H pip install pip --upgrade

pip --version
# pip 19.1 from /usr/local/lib/python3.6/dist-packages/pip (python 3.6)
```

### Via Miniconda
Official documents

* [Download page](https://conda.io/miniconda.html)
* [MD5 checksums page](https://repo.continuum.io/miniconda/)
* [Cryptographic hash verification](https://conda.io/docs/user-guide/install/download.html#cryptographic-hash-verification)

```bash
# if umask is 027, it will result permission problem for normal user
umask 022

# Python 3
python_ver=${python_ver:-3}
miniconda_dir="/opt/Miniconda${python_ver}"
download_tool='wget -qO-'  # or 'curl -fsL'

download_link=$($download_tool https://conda.io/miniconda.html | sed -r -n '/Miniconda'"${python_ver}"'.*Linux-x86_64/{s@.*href="([^"]+)".*$@\1@g;p}')
# https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
[[ -n "${download_link}" ]] || download_link="https://repo.anaconda.com/miniconda/Miniconda${python_ver}-latest-Linux-x86_64.sh"
download_pack_name="${download_link##*/}"

# save package to directory ~
cd ~
# curl -fsL -O "${download_link}"
$download_tool "${download_link}" > "${download_pack_name}"

# md5 checksum   (md5 filename / shasum -a 256 filename)
# md5_checksum=$($download_tool https://repo.continuum.io/miniconda/ | sed -r -n '/Miniconda3-latest-Linux-x86_64.sh/,/<\/tr>/{/<\/tr>/{x;s@[[:space:]]*<[^>]+>[[:space:]]*@@g;;p};h}')

# defaults to $HOME/miniconda3 if not specify `-p`
sudo bash "${download_pack_name}" -b -f -p "${miniconda_dir}"

# '$SHELL' gives you the default shell. '$0' gives you the current shell.
# setting $PATH in ~/.bashrc or ~/.zshrc
shell_rc='.bashrc'
[[ "$0" == 'zsh' && -f ~/.zshrc ]] && shell_rc='.zshrc'
shell_rc="$HOME/${shell_rc}"

sed -r -i '/Miniconda.*start/,/Miniconda.*end/d' ${shell_rc}
echo -e "# Miniconda${python_ver} start\nexport PATH=\"${miniconda_dir}/bin:\$PATH\"\n# Miniconda${python_ver} end" >> ${shell_rc}

# sudo tee /etc/profile.d/miniconda.sh << EOF
# # Miniconda${python_ver} start
# export PATH=${miniconda_dir}/bin:\$PATH
# # Miniconda${python_ver} end
# EOF

# make new $PATH work (important)
. ${shell_rc}

# conda config --show
# permanently deiable error report send
conda config --set report_errors False

# update conda
conda_bin_path=${pip_bin_path:-'conda'}
conda_bin_path=$(which conda)

sudo "${conda_bin_path}" update conda -y
sudo "${conda_bin_path}" upgrade conda -y

# conda version info
conda --version
# conda 4.6.14
```

## Airflow
Official documents

* https://airflow.apache.org/installation.html
* https://airflow.apache.org/start.html

### Essential packages

```bash
# Debian/Ubuntu
sudo apt-get install -y gcc

# SLES/OpenSUSE
sudo zypper in -yl gcc

# RHEL/CentOS/Amzn2
sudo yum install -y gcc
```

### Installing

```bash
umask 022

user_name='airflow'
group_name='airflow'
# create group
sudo groupadd -r "${group_name}" 2> /dev/null
# create user without login privilege
sudo useradd -r -g "${group_name}" -s /bin/false -M "${user_name}"

# make sure login user can enter directory /etc/airflow
login_user="${USER:-}"
sudo usermod -a -G "${group_name}" "${login_user}"

unset airflow_home
airflow_home=${airflow_home:-"$HOME/airflow"}
# airflow needs a home, ~/airflow is the default, but you can lay foundation somewhere else if you prefer (optional)
export AIRFLOW_HOME="${airflow_home}"

# /opt/Miniconda3/bin/conda
# /opt/Miniconda3/bin/pip
pip_bin_path=${pip_bin_path:-'pip'}
pip_bin_path=$(which pip)
[[ -z "${pip_bin_path}" && -f "${miniconda_dir}/bin/pip" ]] && pip_bin_path="${miniconda_dir}/bin/pip"

# RuntimeError: By default one of Airflow's dependencies installs a GPL dependency (unidecode). To avoid this dependency set SLUGIFY_USES_TEXT_UNIDECODE=yes in your environment when you install or upgrade Airflow. To force installing the GPL version set AIRFLOW_GPL_UNIDECODE
export SLUGIFY_USES_TEXT_UNIDECODE=yes
# export AIRFLOW_GPL_UNIDECODE=yes

# install from pypi using pip
# pip install apache-airflow # $HOME/airflow
# sudo pip install apache-airflow # /usr/local/bin/airflow
# sudo ${pip_bin_path} install apache-airflow    # Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-7v18zo3i/apache-airflow/
sudo SLUGIFY_USES_TEXT_UNIDECODE=yes ${pip_bin_path} install apache-airflow

# path $HOME/miniconda3/bin/airflow
# which airflow --> /opt/Miniconda3/bin/airflow
airflow_bin_path=$(which airflow)
# Error /bin/sh: 1: airflow: not found
sudo ln -fs $(which airflow) /bin/airflow

# https://www.cnblogs.com/lwglinux/p/7100400.html
# run `airflow webserver` prompt errors
# FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn': 'gunicorn'
# which gunicorn --> /opt/Miniconda3/bin/gunicorn
sudo ln -fs $(which gunicorn) /bin/gunicorn

# Attention: /usr/local/bin not work for airflow,gunicorn

# https://github.com/jd/tenacity/issues/117  syntax error when importing tenacity at python 3.7
# apache-airflow 1.10.2 has requirement tenacity==4.8.0, but you'll have tenacity 5.0.2 which is incompatible.
# apache-airflow 1.10.3 has requirement tenacity==4.12.0, but you'll have tenacity 5.0.4 which is incompatible.

# from tenacity.async import AsyncRetrying
#                   ^
# SyntaxError: invalid syntax
sudo ${pip_bin_path} install tenacity --upgrade
```

>The PID file for the webserver will be stored in $AIRFLOW_HOME/airflow-webserver.pid or in /run/airflow/webserver.pid if started by systemd. -- https://airflow.apache.org/start.html

```bash
umask 022  # directory permission is 0755
# - creating dir /run/airflow
airflow_run_dir='/run/airflow'
[[ -d /run/airflow ]] || sudo mkdir -p "${airflow_run_dir}"
sudo chown -R "${user_name}":"${group_name}" "${airflow_run_dir}"

# - creating log directory /var/log/airflow
# Not work, if change base_log_folder, will prompt error   PermissionError: [Errno 13] Permission denied: '/var/log/airflow/scheduler/2019-01-23/../../../opt'
# airflow_log_dir='/var/log/airflow'
# [[ -d "${airflow_log_dir}" ]] || sudo mkdir -p "${airflow_log_dir}"
# sudo chown -R "${user_name}":"${group_name}" "${airflow_log_dir}"
# sudo chmod 0755 "${airflow_log_dir}"
```

Initializing, moving generated directory `~/airflow` to `/etc/`

```bash
umask 022

# initialize the database   ~/airflow
${airflow_bin_path} initdb

# copy configuration files in ~/airflow to directory /etc
sudo cp -R ~/airflow /etc/
airflow_home='/etc/airflow'
airflow_cfg="${airflow_home}/airflow.cfg"
default_airflow_home="$HOME/airflow"
sudo sed -r -i '/'"${default_airflow_home//\//\\/}"'/{s@'"${default_airflow_home}"'@'"${airflow_home}"'@g}' "${airflow_cfg}"

# for base_log_folder,,child_process_log_directory,dag_processor_manager_log_location
# default_airflow_log_dir="${airflow_home}/logs"
# sudo sed -r -i '/'"${default_airflow_log_dir//\//\\/}"'/{s@'"${default_airflow_log_dir}"'@'"${airflow_log_dir}"'@g}' "${airflow_cfg}"

[[ -d "${airflow_home}/dags" ]] || sudo mkdir -p "${airflow_home}/dags"
sudo chown -R "${user_name}":"${group_name}" "${airflow_home}"
# export AIRFLOW_HOME="${airflow_home}"

[[ -d "${default_airflow_home}" ]] && rm -rf "${default_airflow_home}"

funcAirflowConfigSetting(){
    l_key="${1:-}"
    l_val="${2:-}"
    l_cfg="${3:-${airflow_cfg}}"
    l_delimiter='|'

    [[ -f "${l_cfg}" && -n "${l_key}" && -n "${l_val}" ]] && sudo sed -r -i '/^'"${l_key}"'[[:space:]]*=/{s'${l_delimiter}'^([^=]+=).*$'${l_delimiter}'\1 '"${l_val}"''${l_delimiter}'g}' "${l_cfg}"
}

# Whether to load the examples that ship with Airflow. It's good to get started, but you probably want to set this to False in a production environment
funcAirflowConfigSetting 'load_examples' 'True'

# funcAirflowConfigSetting 'web_server_port' '8080'
# funcAirflowConfigSetting 'base_url' 'http://localhost:8080'


# funcAirflowConfigSetting 'base_log_folder' "${airflow_log_dir}"
# The executor class that airflow should use. Choices include SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
# funcAirflowConfigSetting 'executor' 'CeleryExecutor'
# funcAirflowConfigSetting 'sql_alchemy_conn' ''
# funcAirflowConfigSetting 'broker_url' ''
# funcAirflowConfigSetting 'celery_result_backend' ''
# funcAirflowConfigSetting 'web_server_host' ''
# funcAirflowConfigSetting 'flower_host' ''
```


### Testing
This is optional operation.

```bash
# - 1. start the web server, default port is 8080
# ${airflow_bin_path} webserver -p 8080
# ${airflow_bin_path} webserver --pid /run/airflow/webserver.pid

# This command prompt error on RHEL/Amzn2, works well on Ubuntu/SUSE
# Error: [Errno 13] Permission denied: '/home/ec2-user'
sudo -u airflow AIRFLOW_HOME="${airflow_home}" ${airflow_bin_path} webserver --pid /run/airflow/webserver.pid

# - 2. start the scheduler
# ${airflow_bin_path} scheduler
sudo -u airflow ${airflow_bin_path} scheduler

# visit localhost:8080 in the browser and enable the example dag in the home page
```

```bash
# - list dags
sudo -u airflow AIRFLOW_HOME=/etc/airflow/ /opt/Miniconda3/bin/airflow list_dags

# - manually run dag 'tutorial'
sudo -u airflow AIRFLOW_HOME=/etc/airflow/ /opt/Miniconda3/bin/airflow backfill -s $(date +'%Y-%m-%d') -e $(date +'%Y-%m-%d') tutorial
```

### Systemd service
Official document
* [Running Airflow with systemd](https://airflow.apache.org/howto/run-with-systemd.html)


```bash
scheduler_runs=5

# Ubuntu/Debian use /etc/default/airflow
# CentOS/Amzn2 use /etc/sysconfig/airflow  (Aman2 has both two dirs)
sysconfig_path='/etc/default/airflow'
[[ -d /etc/sysconfig ]] && sysconfig_path='/etc/sysconfig/airflow'

sudo tee "${sysconfig_path}" << EOF
# This file is the environment file for Airflow. Put this file in /etc/sysconfig/airflow per default
# configuration of the systemd unit files.
#
AIRFLOW_HOME=${airflow_home}
AIRFLOW_CONFIG=\$AIRFLOW_HOME/airflow.cfg
SCHEDULER_RUNS=${scheduler_runs}
# User=${user_name}
# Group=${group_name}
EOF

sudo chmod o+r "${sysconfig_path}"


# - airflow-webserver.service
sudo tee /etc/systemd/system/airflow-webserver.service << EOF
[Unit]
Description=Airflow webserver daemon
After=network.target postgresql.service mysql.service redis.service rabbitmq-server.service
Wants=postgresql.service mysql.service redis.service rabbitmq-server.service

[Service]
EnvironmentFile=${sysconfig_path}
User=airflow
Group=airflow
Type=simple
ExecStart=${airflow_bin_path} webserver --pid /run/airflow/webserver.pid
Restart=on-failure
RestartSec=5s
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF


# - airflow-scheduler.service
# Make sure to specify the SCHEDULER_RUNS variable in this file when you run the scheduler.
sudo tee /etc/systemd/system/airflow-scheduler.service << EOF
[Unit]
Description=Airflow scheduler daemon
After=network.target postgresql.service mysql.service redis.service rabbitmq-server.service
Wants=postgresql.service mysql.service redis.service rabbitmq-server.service

[Service]
EnvironmentFile=${sysconfig_path}
User=airflow
Group=airflow
Type=simple
ExecStart=${airflow_bin_path} scheduler
# -n \${SCHEDULER_RUNS}
# --stdout STDOUT       Redirect stdout to this file
# --stderr STDERR       Redirect stderr to this file
# -l LOG_FILE, --log-file LOG_FILE Location of the log file

Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF

# If specify `-n ${SCHEDULER_RUNS}`, the icons in left two cloumns in page `DAGs` will disapper. If not, it works well. Don't know the reason.
```

Systemd service operating command. More details in
* [Overview of systemd for RHEL 7](https://access.redhat.com/articles/754933).
* [MANAGING SERVICES WITH SYSTEMD](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)

```bash
sudo systemctl daemon-reload

sudo systemctl status airflow-webserver.service
sudo systemctl start airflow-webserver.service
# sudo systemctl enable airflow-webserver.service

sudo systemctl status airflow-scheduler.service
sudo systemctl start airflow-scheduler.service
# sudo systemctl enable airflow-scheduler.service

sudo systemctl stop airflow-webserver.service
sudo systemctl stop airflow-scheduler.service
sudo systemctl start airflow-webserver.service
sudo systemctl start airflow-scheduler.service
```

## Snapshot
DAG list

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2019-04-25_Apache_Airflow/2019-04-25_22-38-17.png)

DAG `tutorial`

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2019-04-25_Apache_Airflow/2019-04-25_22-38-44.png)

DAG task instances

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2019-04-25_Apache_Airflow/2019-04-25_22-39-54.png)


## Uninstalling operation
Removing all configurations from your system.

```bash
cd ~

# ~/.bashrc or ~/.zshrc
shell_rc='.bashrc'
[[ "$0" == 'zsh' && -f ~/.zshrc ]] && shell_rc='.zshrc'
shell_rc="$HOME/${shell_rc}"

. ${shell_rc}

airflow_bin_path=$(which airflow)

[[ -f /bin/gunicorn || -L /bin/gunicorn ]] && sudo rm -f /bin/gunicorn
[[ -f /bin/airflow || -L /bin/airflow ]] && sudo rm -f /bin/airflow

[[ -d "${airflow_bin_path%/bin*}" ]] && sudo rm -rf "${airflow_bin_path%/bin*}"

ls /etc/systemd/system/airflow-* | xargs -l | sed -r -n 's@.*/([^\/]+)$@\1@g;p' | while read -r line; do sudo systemctl stop $line; done

sudo rm -f /etc/systemd/system/airflow-*
sudo systemctl daemon-reload

[[ -f /etc/sysconfig/airflow ]] && sudo rm -f /etc/sysconfig/airflow
[[ -f /etc/default/airflow ]] && sudo rm -f /etc/default/airflow

[[ -d /etc/airflow ]] && sudo rm -rf /etc/airflow
[[ -d /run/airflow ]] && sudo rm -rf /run/airflow
[[ -d /var/log/airflow ]] && sudo rm -rf /var/log/airflow

[[ -n ${HOME:-} && -d "$HOME/airflow" ]] && sudo rm -rf "$HOME/airflow"
sed -r -i '/Miniconda.*start/,/Miniconda.*end/d' ~/.bashrc
```

## Bug Occuring
If I set custom log dir `/var/log/airflow` for derivate `base_log_folder`, `child_process_log_directory`, `dag_processor_manager_log_location` in airflow config file `/etc/airflow/airflow.cfg`. While start service `airflow-scheduler.service`, it will prompt error info like this

```bash
airflow[8030]:   [Previous line repeated 3 more times]
airflow[8030]:   File "/opt/Miniconda3/lib/python3.7/os.py", line 221, in makedirs
airflow[8030]:     mkdir(name, mode)
airflow[8030]: PermissionError: [Errno 13] Permission denied: '/var/log/airflow/schedule/2019-01-23/../../../opt'
```

But if I just use defaul log directory `/etc/airflow/logs`, it works properly.

```bash
ls -lha /etc/airflow/logs/*
/etc/airflow/logs/dag_processor_manager:
total 224K
drwxr-xr-x 2 airflow airflow 4.0K Jan 23 10:26 .
drwxr-xr-x 5 airflow airflow 4.0K Jan 23 10:27 ..
-rw-r--r-- 1 airflow airflow 216K Jan 23 10:51 dag_processor_manager.log

/etc/airflow/logs/scheduler:
total 12K
drwxr-xr-x 3 airflow airflow 4.0K Jan 23 10:25 .
drwxr-xr-x 5 airflow airflow 4.0K Jan 23 10:27 ..
drwxr-xr-x 2 airflow airflow 4.0K Jan 23 10:24 2019-01-23
lrwxrwxrwx 1 airflow airflow   38 Jan 23 10:25 latest -> /etc/airflow/logs/scheduler/2019-01-23

/etc/airflow/logs/tutorial:
total 20K
drwxrwxrwx 5 airflow airflow 4.0K Jan 23 10:28 .
drwxr-xr-x 5 airflow airflow 4.0K Jan 23 10:27 ..
drwxrwxrwx 5 airflow airflow 4.0K Jan 23 10:28 print_date
drwxrwxrwx 5 airflow airflow 4.0K Jan 23 10:29 sleep
drwxrwxrwx 5 airflow airflow 4.0K Jan 23 10:29 templated
```

## Change Log
* Jan 10, 2019 16:32 Thu -0500 EST
    * first draft
* Jan 23, 2019 10:42 Wed -0500 EST
    * optimization, upgrade vererion to `1.10.2`
* Apr 25, 2019 21:58 Thu -0400 EST
    * optimization, upgrade vererion to `1.10.3`


[apacheairflow]: https://github.com/apache/airflow "Airflow is a platform to programmatically author, schedule and monitor workflows."
[miniconda]: https://conda.io/miniconda.html "A free minimal installer for conda."


<!-- End -->
