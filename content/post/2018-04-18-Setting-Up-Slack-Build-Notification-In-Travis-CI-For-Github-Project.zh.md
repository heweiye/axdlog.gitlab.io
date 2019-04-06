---
title: 爲集成了Travis CI的GitHub項目配置Slack信息推送
slug: Setting Up Slack Build Notification in Travis CI for Github Project
date: 2018-04-18T13:31:34-04:00
lastmod: 2018-04-18T13:31:34-04:00
draft: false
keywords: ["AxdLog", "Slack", "Slack Notification", "Travis CI", "GitHub"]
description: "如何爲集成了Travis CI的GitHub項目配置Slack信息推送"
categories:
- Blog
- DevOps
tags:
- Github
- Travis CI
- Slack

comment: true
toc: true

---

[Travis CI][travisci]默認通過郵件發送構建信息，[Travis CI][travisci]也支持通過其它方式發送信息，如[Slack][slack]。本文記錄如何已集成[Travis CI][travisci]的[GitHub][github]項目配置[Slack][slack]信息推送。

<!--more-->

本文以個人Blog [**AxdLog**](https://axdlog.com) 爲操作示例

item | details
---|---
repo | [MaxdSre/maxdsre.github.io](https://github.com/MaxdSre/maxdsre.github.io)
channel | github_axdlog

## 官方文檔
以下爲[Travis CI][travisci]官方文檔

* [Configuring Build Notifications][config_build_notification]
* [Encryption keys](https://docs.travis-ci.com/user/encryption-keys/)
* [Configuring Build Notifications](https://api.slack.com/docs/message-formatting)


## Slack
此處不涉及[Slack][slack]的帳號註冊，點擊鏈接 <https://slack.com/signin> 進入登錄頁面。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-51-02.png)

輸入用戶名、密碼

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-53-07.png)


### 創建頻道
點擊窗口左側的`Channel`或其後的`+`創建新的channel(頻道)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-57-07.png)

此處命名爲`github_axdlog`並設置爲私有

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-13-38.png)

新創建的頻道

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-13-58.png)


### 生成 Token
點擊鏈接 <https://my.slack.com/services/new/travis> 進入設置[Travis CI][travisci]的頁面，在下拉列表框中選擇目標頻道，此處爲`github_axdlog`。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-14-38.png)

生成的token爲`maxdsre:ne18Xpc5RbzM1hIdd34nN2aE`

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-15-34.png)

頁面其它信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-15-52.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-16-04.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-16-16.png)

頁面`Encrypting your credentials`部分有一段話

>Your integration token is semi-secret, but we recommend [encrypting your credentials](http://docs.travis-ci.com/user/encryption-keys/) using the [Travis command line client](https://github.com/travis-ci/travis#readme).

### 憑證加密
按照說明文檔並未能成功安裝`travis`，後通過[Docker][docker]容器實現token的加密操作。[Docker][docker]的介紹、安裝和配置，可參考本人Blog [在GNU/Linux中安裝Docker CE（社區版）全記錄]({{< relref "2017-03-06-Installing-Configuring-Docker-Community-Edition-CE-On-GNU-Linux.zh.md" >}})。

```bash
docker pull ruby
docker run -ti --rm ruby bash
```

容器啓動後，執行如下命令

```bash
# 安裝 travis
gem install travis

touch .travis.yml
travis encrypt "maxdsre:ne18Xpc5RbzM1hIdd34nN2aE" --add notifications.slack.rooms -r MaxdSre/maxdsre.github.io
cat .travis.yml
```

操作過程

```bash
root@a24abf051547:/# travis --version
Shell completion not installed. Would you like to install it now? |y| y
1.8.8
root@a24abf051547:/# travis encrypt "maxdsre:ne18Xpc5RbzM1hIdd34nN2aE" --add notifications.slack.rooms
Can't figure out GitHub repo name. Ensure you're in the repo directory, or specify the repo name via the -r option (e.g. travis <command> -r <owner>/<repo>)
root@a24abf051547:/# travis encrypt "maxdsre:ne18Xpc5RbzM1hIdd34nN2aE" --add notifications.slack.rooms -r MaxdSre/maxdsre.github.io
no .travis.yml found
root@a24abf051547:/# touch .travis.yml
root@a24abf051547:/# travis encrypt "maxdsre:ne18Xpc5RbzM1hIdd34nN2aE" --add notifications.slack.rooms -r MaxdSre/maxdsre.github.io
root@a24abf051547:/# cat .travis.yml
notifications:
  slack:
    rooms:
      secure: o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
root@a24abf051547:/#
```

注意報錯信息

GitHub repo name

>Can't figure out GitHub repo name. Ensure you're in the repo directory, or specify the repo name via the -r option (e.g. travis <command> -r <owner>/<repo>)

.travis.yml

>no .travis.yml found


token `maxdsre:ne18Xpc5RbzM1hIdd34nN2aE` 經`travis`加密後生成的字符串爲

```txt
o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
```

## .travis.yml
根據官方文檔 [Configuring Build Notifications][config_build_notification] 在文件`.travis.yml`中配置`notifications`指令（文件存放在項目`MaxdSre/maxdsre.github.io`的根目錄下）。

```yml
notifications:
  email: false    # default notification method
  slack:
    rooms:
      - secure: o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
    on_success: always  # change: send a notification when the build status changes.
    on_failure: always  # always: always send a notification.
```

### 消息模板
默認消息模板爲
```bash
# push
- "Build <%{build_url}|#%{build_number}> (<%{compare_url}|%{commit}>) of %{repository_slug}@%{branch} by %{author} %{result} in %{duration}"

# pull
- "Build <%{build_url}|#%{build_number}> (<%{compare_url}|%{commit}>) of %{repository_slug}@%{branch} in PR <%{pull_request_url}|#%{pull_request_number}> by %{author} %{result} in %{duration}"
```

此處進行自定義，最終的格式如下

```
notifications:
  email: false    # default notification method
  slack:
    rooms:
      - secure: o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
    on_success: always  # change: send a notification when the build status changes.
    on_failure: always  # always: always send a notification.
    template:
      - "Repo `%{repository_slug}` *%{result}* build (<%{build_url}|#%{build_number}>) for commit (<%{compare_url}|%{commit}>) on branch `%{branch}`."
      - "Execution time: *%{duration}*"
      - "Message: %{message}"
```

## 測試
修改`.travis.yml`後，將修改後代碼提交到GitHub，測試配置是否成果。

Travis CI 構建信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-54-57.png)

推送信息測試

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-54-26.png)

構建失敗信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_16-03-24.png)

構建成功信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_16-13-33.png)


## 更新日誌
* 2018.04.18 16:07 Tue America/Boston
    * 初稿完成

[travisci]: https://travis-ci.org "Test and Deploy with Confidence"
[slack]: https://slack.com "Where work happens"
[github]: https://github.com
[docker]:https://www.docker.com "Docker"
[config_build_notification]:https://docs.travis-ci.com/user/notifications#Configuring-Slack-notifications

<!-- End -->
