---
title: Deploying Project Codes To Multiple Remote Hosts Via SSH
date: 2017-10-24T16:46:16+08:00
lastmod: 2018-04-11T10:11:16-04:00
draft: false
keywords: ["Tomcat", "SSH", "Java", "Maven", "Shell Script", "Deployment"]
description: "Using shell script to deploy project codes to multiple remote hosts via ssh"
categories:
- Production Case
tags:
- Shell Script
- SSH
- Tomcat
- Java
comment: true
toc: true
---

截至目前(2017-10)，公司測試、驗證、生產環境的代碼仍使用人工手動部署，其中緣由，歷史與現實原因皆有。數月前，研發部門對系統架構進行了一次更改，其中加入了負載均衡(load balance)功能。架構更改後，項目由單節點(主機)運行變成多節點(主機)運行，服務器數量增加了近一倍，加之公司有30多個自有項目在運行。若仍使用人工部署方式，一則運維同事壓力巨大，二則嚴重影響生產效率。在IT運維部經理的要求下，本人着手進行代碼部署的自動化工作。

本文記錄如何使用`SSH`將項目代碼傳輸到遠程內網主機中，進行代碼部署操作，並給出Shell Script代碼(簡易版)。

<!--more-->

## Existing Conditions & Requirements
現有前提條件

1. 公司服務器託管於多個IDC機房，每個機房配置一臺跳板主機，配置有MFA多重認證；同一機房中的其它主機爲內網主機，只能通過跳板主機登錄；
2. 跳板主機的`/etc/hosts`中有各內網主機的別名，跳板主機通過`SSH Keys`連接各內網主機；
3. 跳板主機通過`SSH Keys`連接各內網主機；
4. 項目運行在各內網主機中；
5. 代碼打包後，測試人員提供ftp下載地址，運維人員下載後進行部署(需登入每一臺相關的內網主機)；
6. 項目名、代碼包名、部署目錄名不一致，驗證和生產環境的項目部署路徑不一致；
7. 開發語言Java，項目通過`Tomcat`和`Maven`運行；
8. 代碼包格式有`.wav`、`.zip`、`.tar.gz`等；
9. Java的安裝路徑不一致；

關於需求，部門經理的要求是能夠通過ftp下載地址自動識別、部署。


## Analysis
根據現有情況，進行分析

>一機房中的其它主機爲內網主機，只能通過跳板主機登錄；

逐個登入項目所在的內網主機進行部署，不太好操作。直接在跳板主機中操作，通過SSH傳輸執行腳本到特定內網主機。

>項目名、代碼包名、部署目錄名不一致，驗證和生產環境的項目部署路徑不一致

要求研發部門、IT運維部門對項目名稱、部署路徑進行一致性處理的難度非常大，此爲歷史遺留問題。考慮通過整理一份關係對應表解決該問題，但工作量巨大，本人需花長時間整理。

>開發語言Java，項目通過`Tomcat`和`Maven`運行

將項目部署方式一併加入對應關係表中。


## Challenge
遇到的問題

>如何在跳板主機中將相關代碼傳輸到特定的內網主機中並執行?

通過`SSH`和管道符`|`將代碼傳輸到內網主機，代碼示例

```bash
operation_path=${operation_path:-'/PATH/operation.sh'}

cat "${operation_path}" | ssh REMOTE_HOST "dd of=/tmp/kjw_operation.sh &> /dev/null; bash /tmp/kjw_operation.sh; rm -f /tmp/kjw_operation.sh 1>/dev/null"
```

考慮到謹慎部署，採用`while`循環進行遍歷操作，而非使用`parallel`進行並行操作。


## Code Snippet
整個腳本分爲3部分

* 核心配置文件
* 入口文件(在跳板主機中執行)
* 執行文件(在內網主機中執行)

### Config File
配置文件格式爲

>主機 - 部署類型 - 部署路徑名 - 項目路徑名 - 安裝包名 -是否已啓動

示例如下

```
APP1|tomcat|tomcat-flycar-business-web|flycar-business-web|flycar-business-web
APP1|tomcat|tomcat-flycar-chargestation|flycar-chargestation|flycar-chargestation
APP1|tomcat|tomcat-flycar-driver|flycar-driver|flycar-car-driver
APP1|tomcat|tomcat-flycar-finance|flycar-finance|flycar-finance
APP1|tomcat|tomcat-flycar-lgwx|flycar-lgwx|flybuscar-weixin
APP1|tomcat|tomcat-flycar-pc-dispatch|flycar-pc-dispatch|flycar-pc-dispatch
APP1|tomcat|tomcat-flycar-user-mobile|flycar-user-mobile|flycar-user-mobile
APP1|tomcat|tomcat-flycar-user-web|flycar-user-web|flycar-user-web
APP1|tomcat|tomcat-flycar-weixin|flycar-weixin|flycar-weixin
APP1|maven||tomcat-sms-auth|sms-auth
APP1|maven||tomcat-sms-bi|sms-bi
APP1|maven||tomcat-sms-bpm|sms-bpm
APP1|maven||tomcat-sms-bs|sms-bs
APP1|maven||tomcat-sms-qywx|sms-qywx
```

### Entry File
入口文件，將驗證和生產環境的部署腳本整合在一起。

```bash
#!/usr/bin/env bash
set -u
set -o pipefail

funcExitStatement(){
    local item="${1:-}"
    if [[ -n "${item}" ]]; then
        echo "${item}"
        exit
    fi
}

if [[ "$#" -eq 0 ]]; then
    echo -e "本脚本用法如下:"
    echo -e "bash /PATH/deploy.sh ftp_link deploy_environment\n"
    echo -e "ftp_link: 安装包ftp下载链接"
    echo -e "deploy_environment: 部署环境类型(生产或验证环境)，可用字母p和v指定，若未指定则默认为生产环境\n"
    echo -e "注意 JAVA_HOME 变量值"
    exit
fi

# 脚本所在路径
script_saved_path=${script_saved_path:-}
script_saved_path=$(dirname $(readlink -f "$0"))

deploy_environment_type=${deploy_environment_type:-"${2:-}"}
# 主机 - 部署类型 - 部署路径名 - 项目路径名 - 安装包名 -是否已启动
config_path=${config_path:-}
curl_command=${curl_command:-}

case "${deploy_environment_type,,}" in
    verify|v )
        # 验证环境
        config_path="${script_saved_path}/config_verify.txt"
        curl_command="curl -u XXXXX:XXXXXX"
        ;;
    product|p|* )
        # 生产环境
        config_path="${script_saved_path}/config_product.txt"
        curl_command="curl -u YYYYY:YYYYYY"
        ;;
esac

[[ -s "${config_path}" ]] || funcExitStatement '未找到代码部署配置文件，脚本即将退出!'
operation_path=${operation_path:-"${script_saved_path}/operation.sh"}
[[ -s "${operation_path}" ]] || funcExitStatement '未找到代码部署脚本，脚本即将退出!'

# 1.获取下载链接、下载安装包
if [[ "${1:-}" == '' ]]; then
    funcExitStatement '未指定下载链接，脚本即将退出!'
elif [[ ! "${1}" =~ ^ftp:// ]]; then
    funcExitStatement '下载链接必须以ftp://开头!'
else
    $curl_command -I "${1}" &>/dev/null
    if [[ $? -eq 0 ]]; then
        echo "经测试下载链接有效"
    else
        funcExitStatement '经测试: 下载链接无效，请确定部署环境，脚本即将退出!'
    fi
fi

download_link=${download_link:-"${1}"}
package_full_name=${package_full_name:-"${download_link##*/}"}
package_name=${package_name:-}
package_name=$(echo "${package_full_name}" | sed -r 's/^([^[:digit:]]+)-.*/\1/g')
[[ -z "${package_name}" ]] && funcExitStatement '无法提取安装包名，脚本即将退出!'

# 提取安装包对应的主机、服务等信息
project_info_list=${project_info_list:-}
project_info_list=$(awk -F\| 'match($5,/'"${package_name}"'/){print}' "${config_path}")
[[ -z "${project_info_list}" ]] && funcExitStatement "未查询到 ${package_name} 对应的服务，脚本即将退出!"

echo "${project_info_list}" | while IFS="|" read -r host_name deploy_type deploy_path_name project_path_name package_name is_running; do

    config_info="${host_name}|${deploy_type}|${deploy_path_name}|${project_path_name}|${package_name}"
    # $1  download_link   /  $2  config_info   / $3  curl_command
    cat "${operation_path}" | ssh "${host_name}" "dd of=/tmp/kjw_operation.sh &> /dev/null; bash /tmp/kjw_operation.sh \"${download_link}\" \"${config_info}\" \"${curl_command}\"; rm -f /tmp/kjw_operation.sh 1>/dev/null"

done

# Script End
```


### Execution File
執行文件

```bash
#!/usr/bin/env bash
set -u
set -o pipefail

download_link=${download_link:-"${1}"}
config_info=${config_info:-"${2}"}
curl_command=${curl_command:-"${3}"}

deploy_default_dir=${deploy_default_dir:-'/usr/local/webmiddleware'}
project_default_dir=${project_default_dir:-'/project'}
package_save_dir=${package_save_dir:-'/tmp'}

package_full_name=${package_full_name:-"${download_link##*/}"}
package_name=${package_name:-}
package_name=$(echo "${package_full_name}" | sed -r 's/^([^[:digit:]]+)-.*/\1/g')

suffix_type=${suffix_type:-}
if [[ "${package_full_name}" =~ tar.gz$ ]]; then
    suffix_type='tar.gz'
elif [[ "${package_full_name}" =~ .zip$ ]]; then
    suffix_type='zip'
elif [[ "${package_full_name}" =~ .war$ ]]; then
    suffix_type='war'
fi


funcExitStatement(){
    local item="${1:-}"
    if [[ -n "${item}" ]]; then
        echo "${item}"
        exit
    fi
}

funcStartProjectProcess(){
    local l_deploy_type="${1:-}"
    local l_deploy_dir="${2:-}"
    local l_project_dir="${2:-}"

    # 设置JAVA环境变量
    export JAVA_HOME=/usr/local/java
    export PATH=/usr/local/java:$PATH

    case "${l_deploy_type}" in
        'tomcat' )
            if [[ -d "${l_deploy_dir}" && -s "${l_deploy_dir}/bin/startup.sh" ]]; then
                /bin/bash "${l_deploy_dir}/bin/startup.sh"
            fi
            ;;
        'maven' )
            if [[ -d "${l_project_dir}" &&  -s "${l_project_dir}/bin/start.sh" ]]; then
                /bin/bash "${l_project_dir}/bin/start.sh"
            fi
            ;;
    esac
}

readonly c_normal='\e[0m'
readonly c_red='\e[31;1m'
readonly c_blue='\e[34m'

echo "${config_info}" | while IFS="|" read -r host_name deploy_type deploy_path_name project_path_name package_name is_running; do
    deploy_dir="${deploy_default_dir}/${deploy_path_name}"
    project_dir="${project_default_dir}/${project_path_name}"

    echo -e "\n${c_red}主机名:${c_normal} ${host_name}\n${c_red}安装包:${c_normal} ${package_name}\n${c_red}部署路径:${c_normal} ${deploy_dir}\n${c_red}项目路径:${c_normal} ${project_dir}\n"

    if [[ ! -d "${deploy_dir}" ]]; then
        echo "部署目录 ${project_dir} 不存在，请核实，自动跳过"
        continue
    fi

    # 下载安装包
    package_save_path=${package_save_path:-"${package_save_dir}/${package_full_name}"}
    echo "准备下载安装包:"
    $curl_command -fL# -o "${package_save_path}" "${download_link}"

    if [[ -s "${package_save_path}" ]]; then
        echo "安装包 ${package_full_name} 下载成功"
    else
        /bin/rm -f "${package_save_path}"
        echo "链接 ${download_link} 下载失败，自动跳过"
        continue
    fi

    # 查询已存在的备份
    existed_backup_list=${existed_backup_list:-}
    existed_backup_list=$(find "${project_default_dir}/" -maxdepth 1 -type d -name "${project_path_name}-*.bak" -print)

    # 移除旧备份目录
    if [[ -n "${existed_backup_list}" ]]; then
        echo -e "\n准备移除已存在的备份目录"

        echo "${existed_backup_list}" | while IFS="" read -r line; do
            if [[ -d "${line}" ]]; then
                /bin/rm -rf "${line}"
                [[ -d "${line}" ]] || echo "成功移除备份目录: ${line}"
            fi
        done
    fi

    # 创建新备份目录
    current_date=${current_date:-"$(date +"%Y%m%d-%H%M")"}
    project_dir_backup=${project_dir_backup:-"${project_dir}-${current_date}.bak"}

    if [[ ! -d "${project_dir}" ]]; then
        echo -e "\n项目目录 ${project_dir} 检测不存在，准备创建"
    else
        echo -e "\n项目目录 ${project_dir} 检测存在，准备进行备份操作"
        cp -R "${project_dir}" "${project_dir_backup}"
        [[ -d "${project_dir_backup}" ]] && echo "成功创建备份目录 ${project_dir_backup}"

        # Kill进程ID
        pid_old=${pid_old:-''}
        pid_old=$(ps -ef | awk '$0~/java.*'"${deploy_dir##*/}"'/&&$0!~/awk.*print/{print $2;exit}')
        if [[ "${pid_old}" == '' ]]; then
            echo -e "\n未侦测到${package_name}进程\n"
        elif [[ "${pid_old}" -gt 0 ]]; then
            kill -9 "${pid_old}"
            echo -e "\n侦测到${package_name}进程(PID ${pid_old})，进程停止成功\n"
        fi

        echo "准备移除项目目录 ${project_dir}"
        /bin/rm -rf "${project_dir}"
        [[ -d "${project_dir}" ]] || echo "成功移除项目目录 ${project_dir}"
        echo "准备重新创建项目目录 ${project_dir}"

    fi

    # 创建新项目目录
    mkdir -p "${project_dir}"
    [[ -d "${project_dir}" ]] && echo "成功创建项目目录 ${project_dir}"

    # 解压
    echo -e "\n准备解压安装包 ${package_full_name}"
    case "${suffix_type}" in
        'tar.gz' )
            tar xf "${package_save_path}" -C "${project_dir}"
            ;;
        zip|war )
            unzip "${package_save_path}" -d "${project_dir}" 1>/dev/null
            ;;
    esac

    if [[ $(ls -A "${project_dir}" | wc -l) -ne 0 ]]; then
        echo "安装包解压成功"
    else
        echo "安装包解压失败，自动跳过"
        continue
    fi

    [[ -s "${package_save_path}" ]] && /bin/rm -f "${package_save_path}"


    # 启动进程
    echo -e "\n准备启动${package_name}服务"
    funcStartProjectProcess "${deploy_type}" "${deploy_dir}" "${project_dir}"

    pid_new=${pid_new:-}
    pid_new=$(ps -ef | awk '$0~/java.*'"${deploy_dir##*/}"'/&&$0!~/awk.*print/{print $2;exit}')

    if [[ "${pid_new}" -gt 0 ]]; then
        echo -e "\n${package_name}进程启动成功，新进程ID: ${pid_new}"
        echo -e "\n项目${package_name}部署完毕，部署路径: ${project_dir}"
        echo -e "\n进程详情"
        ps -ef | awk '$0~/java.*'"${deploy_dir##*/}"'/&&$0!~/awk.*print/{print;exit}'

    else
        [[ -d "${project_dir}" ]] && /bin/rm -rf "${project_dir}"

        if [[ ! -d "${project_dir_backup}" ]]; then
            echo "项目${package_name}启动失败，自动跳过!"
            continue
        else
            echo "项目${package_name}启动失败，准备执行回滚操作"
            mv "${project_dir_backup}" "${project_dir}"
            funcStartProjectProcess "${deploy_type}" "${deploy_dir}" "${project_dir}"

            if [[ $(ps -ef | awk '$0~/java.*'"${deploy_dir##*/}"'/&&$0!~/awk.*print/{print $2;exit}') -gt 0 ]]; then
                echo "项目${package_name}已回滚"
            else
                echo "项目${package_name}回滚失败，请核查，自动跳过"
                continue
            fi

        fi

    fi

    echo -e "\n${c_red}---------------  Split Line  ---------------${c_normal}\n\n"

done

# Script End
```

## Example
實例演示

執行
```bash
bash deploy.sh ftp://XXX.XXX.XXX.XXX/PATH/flybus-provider-XXX.war
```

輸出如下

```bash

经测试下载链接有效

主机名: CORE1
安装包: flybus-provider
部署路径: /usr/local/webmiddleware/tomcat-flybus-provider1
项目路径: /project/flybus-provider1

准备下载安装包:
######################################################################## 100.0%
安装包 flybus-provider-3.9.0-release-20171129093531.war 下载成功

准备移除已存在的备份目录
成功移除备份目录: /project/flybus-provider1-20171128-1558.bak

项目目录 /project/flybus-provider1 检测存在，准备进行备份操作
成功创建备份目录 /project/flybus-provider1-20171130-0932.bak

侦测到flybus-provider进程(PID 16238)，进程停止成功

准备移除项目目录 /project/flybus-provider1
成功移除项目目录 /project/flybus-provider1
准备重新创建项目目录 /project/flybus-provider1
成功创建项目目录 /project/flybus-provider1

准备解压安装包 flybus-provider-3.9.0-release-20171129093531.war
安装包解压成功

准备启动flybus-provider服务
Tomcat started.

flybus-provider进程启动成功，新进程ID: 33089

项目flybus-provider部署完毕，部署路径: /project/flybus-provider1

进程详情
root      33089      1  3 09:32 ?        00:00:00 /usr/local/java/bin/java -Djava.util.logging.config.file=/usr/local/webmiddleware/tomcat-flybus-provider1/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms2048m -Xmx2048m -XX:PermSize=128M -XX:MaxPermSize=256m -Djava.awt.headless=true -Djmagick.systemclassloader=false -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:../logs/bus_provider_gc.log -XX:+PrintHeapAtGC -XX:+PrintGC -Duser.timezone=GMT+8 -Djava.endorsed.dirs=/usr/local/webmiddleware/tomcat-flybus-provider1/endorsed -classpath /usr/local/webmiddleware/tomcat-flybus-provider1/bin/bootstrap.jar:/usr/local/webmiddleware/tomcat-flybus-provider1/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/webmiddleware/tomcat-flybus-provider1 -Dcatalina.home=/usr/local/webmiddleware/tomcat-flybus-provider1 -Djava.io.tmpdir=/usr/local/webmiddleware/tomcat-flybus-provider1/temp org.apache.catalina.startup.Bootstrap start

---------------  Split Line  ---------------



主机名: CORE1
安装包: flybus-provider
部署路径: /usr/local/webmiddleware/tomcat-flybus-provider2
项目路径: /project/flybus-provider2

准备下载安装包:
######################################################################## 100.0%
安装包 flybus-provider-3.9.0-release-20171129093531.war 下载成功

准备移除已存在的备份目录
成功移除备份目录: /project/flybus-provider2-20171128-1558.bak

项目目录 /project/flybus-provider2 检测存在，准备进行备份操作
成功创建备份目录 /project/flybus-provider2-20171130-0932.bak

侦测到flybus-provider进程(PID 16335)，进程停止成功

准备移除项目目录 /project/flybus-provider2
成功移除项目目录 /project/flybus-provider2
准备重新创建项目目录 /project/flybus-provider2
成功创建项目目录 /project/flybus-provider2

准备解压安装包 flybus-provider-3.9.0-release-20171129093531.war
安装包解压成功

准备启动flybus-provider服务
Tomcat started.

flybus-provider进程启动成功，新进程ID: 33186

项目flybus-provider部署完毕，部署路径: /project/flybus-provider2

进程详情
root      33186      1  0 09:32 ?        00:00:00 /usr/local/java/bin/java -Djava.util.logging.config.file=/usr/local/webmiddleware/tomcat-flybus-provider2/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms2048m -Xmx2048m -XX:PermSize=128M -XX:MaxPermSize=256m -Djava.awt.headless=true -Djmagick.systemclassloader=false -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:../logs/bus_provider_gc.log -XX:+PrintHeapAtGC -XX:+PrintGC -Duser.timezone=GMT+8 -Djava.endorsed.dirs=/usr/local/webmiddleware/tomcat-flybus-provider2/endorsed -classpath /usr/local/webmiddleware/tomcat-flybus-provider2/bin/bootstrap.jar:/usr/local/webmiddleware/tomcat-flybus-provider2/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/webmiddleware/tomcat-flybus-provider2 -Dcatalina.home=/usr/local/webmiddleware/tomcat-flybus-provider2 -Djava.io.tmpdir=/usr/local/webmiddleware/tomcat-flybus-provider2/temp org.apache.catalina.startup.Bootstrap start

---------------  Split Line  ---------------



主机名: CORE2
安装包: flybus-provider
部署路径: /usr/local/webmiddleware/tomcat-flybus-provider1
项目路径: /project/flybus-provider1

准备下载安装包:
######################################################################## 100.0%
安装包 flybus-provider-3.9.0-release-20171129093531.war 下载成功

准备移除已存在的备份目录
成功移除备份目录: /project/flybus-provider1-20171128-1558.bak

项目目录 /project/flybus-provider1 检测存在，准备进行备份操作
成功创建备份目录 /project/flybus-provider1-20171130-0932.bak

侦测到flybus-provider进程(PID 22663)，进程停止成功

准备移除项目目录 /project/flybus-provider1
成功移除项目目录 /project/flybus-provider1
准备重新创建项目目录 /project/flybus-provider1
成功创建项目目录 /project/flybus-provider1

准备解压安装包 flybus-provider-3.9.0-release-20171129093531.war
安装包解压成功

准备启动flybus-provider服务
Tomcat started.

flybus-provider进程启动成功，新进程ID: 5716

项目flybus-provider部署完毕，部署路径: /project/flybus-provider1

进程详情
root      5716     1  0 09:32 ?        00:00:00 /usr/local/java/bin/java -Djava.util.logging.config.file=/usr/local/webmiddleware/tomcat-flybus-provider1/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms2048m -Xmx2048m -XX:PermSize=128M -XX:MaxPermSize=256m -Djava.awt.headless=true -Djmagick.systemclassloader=false -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:../logs/bus_provider_gc.log -XX:+PrintHeapAtGC -XX:+PrintGC -Duser.timezone=GMT+8 -Djava.endorsed.dirs=/usr/local/webmiddleware/tomcat-flybus-provider1/endorsed -classpath /usr/local/webmiddleware/tomcat-flybus-provider1/bin/bootstrap.jar:/usr/local/webmiddleware/tomcat-flybus-provider1/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/webmiddleware/tomcat-flybus-provider1 -Dcatalina.home=/usr/local/webmiddleware/tomcat-flybus-provider1 -Djava.io.tmpdir=/usr/local/webmiddleware/tomcat-flybus-provider1/temp org.apache.catalina.startup.Bootstrap start

---------------  Split Line  ---------------



主机名: CORE2
安装包: flybus-provider
部署路径: /usr/local/webmiddleware/tomcat-flybus-provider2
项目路径: /project/flybus-provider2

准备下载安装包:
######################################################################## 100.0%
安装包 flybus-provider-3.9.0-release-20171129093531.war 下载成功

准备移除已存在的备份目录
成功移除备份目录: /project/flybus-provider2-20171128-1558.bak

项目目录 /project/flybus-provider2 检测存在，准备进行备份操作
成功创建备份目录 /project/flybus-provider2-20171130-0932.bak

侦测到flybus-provider进程(PID 22746)，进程停止成功

准备移除项目目录 /project/flybus-provider2
成功移除项目目录 /project/flybus-provider2
准备重新创建项目目录 /project/flybus-provider2
成功创建项目目录 /project/flybus-provider2

准备解压安装包 flybus-provider-3.9.0-release-20171129093531.war
安装包解压成功

准备启动flybus-provider服务
Tomcat started.

flybus-provider进程启动成功，新进程ID: 5802

项目flybus-provider部署完毕，部署路径: /project/flybus-provider2

进程详情
root      5802     1  0 09:32 ?        00:00:00 /usr/local/java/bin/java -Djava.util.logging.config.file=/usr/local/webmiddleware/tomcat-flybus-provider2/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms2048m -Xmx2048m -XX:PermSize=128M -XX:MaxPermSize=256m -Djava.awt.headless=true -Djmagick.systemclassloader=false -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:../logs/bus_provider_gc.log -XX:+PrintHeapAtGC -XX:+PrintGC -Duser.timezone=GMT+8 -Djava.endorsed.dirs=/usr/local/webmiddleware/tomcat-flybus-provider2/endorsed -classpath /usr/local/webmiddleware/tomcat-flybus-provider2/bin/bootstrap.jar:/usr/local/webmiddleware/tomcat-flybus-provider2/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/webmiddleware/tomcat-flybus-provider2 -Dcatalina.home=/usr/local/webmiddleware/tomcat-flybus-provider2 -Djava.io.tmpdir=/usr/local/webmiddleware/tomcat-flybus-provider2/temp org.apache.catalina.startup.Bootstrap start

---------------  Split Line  ---------------
```


## After The Words
在多主機部署代碼是剛需，自動部署可極大減輕運維人員的工作量，有效提高生產效率。

如果有需要，可將其更改爲併發部署，整個項目部署速度更快。

腳本經過測試可用，已經上傳至IT運維部的Gitlab代碼庫中。

然而，今天的上線，運維同事仍舊採用逐個登錄內網主機部署的方式。本人去年用PHP的`Symfony`框架開發的 **資產管理平臺** 也是這樣的境況，心中有些苦澀。


## Change Logs
* 2017.10.24 18:40 Tue Asia/Shanghai
    * 初稿完成
* 2017.11.30 09:42 Thu Asia/Shanghai
    * 添加操作實例
* 2018.04.11 10:11 Wed America/Boston
    * 勘誤，遷移到新Blog

<!-- End -->
