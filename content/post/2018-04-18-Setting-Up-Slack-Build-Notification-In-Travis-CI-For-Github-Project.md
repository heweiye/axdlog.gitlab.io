---
title: Setting Up Slack Build Notification in Travis CI for Github Project
slug: Setting Up Slack Build Notification in Travis CI for Github Project
date: 2018-04-18T13:31:34-04:00
lastmod: 2018-04-18T13:31:34-04:00
draft: false
keywords: ["Slack", "Slack Notification", "Travis CI", "GitHub"]
description: "How to set up Slack build notification in Travis CI for Github project"
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

By default, [Travis CI][travisci] uses email to send build notification, it also support other method, e.g. [Slack][slack]. This article documents how to set up [Slack][slack] notification for [GitHub][github] project integrated [Travis CI][travisci].

<!--more-->

I take my personal blog [**AxdLog**](https://axdlog.com) as operation example.

item | details
---|---
repo | [MaxdSre/maxdsre.github.io](https://github.com/MaxdSre/maxdsre.github.io)
channel | github_axdlog

## Official Document
[Travis CI][travisci] official documents

* [Configuring Build Notifications][config_build_notification]
* [Encryption keys](https://docs.travis-ci.com/user/encryption-keys/)
* [Configuring Build Notifications](https://api.slack.com/docs/message-formatting)


## Slack
This step doesn't include [Slack][slack] account registration. Clicking link <https://slack.com/signin> to login in.

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-51-02.png)

Inputing user name, password

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-53-07.png)


### Create New Channel
Clicking `Channel` or `+` on the left of the page to create a new channel

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_13-57-07.png)

Here I set name as `github_axdlog` and choose `private`

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-13-38.png)

newly created channel

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-13-58.png)


### Token Generation
Clicking link <https://my.slack.com/services/new/travis> into Travis configuration page, choose target channel (`github_axdlog`) in the selection menu

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-14-38.png)

Newly generated token is `maxdsre:ne18Xpc5RbzM1hIdd34nN2aE`

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-15-34.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-15-52.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-16-04.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-16-16.png)

Encrypting your credentials

>Your integration token is semi-secret, but we recommend [encrypting your credentials](http://docs.travis-ci.com/user/encryption-keys/) using the [Travis command line client](https://github.com/travis-ci/travis#readme).

### Encrypting Credentials
After I failed to install `travis` according to official docs, I choose [Docker][docker] image to generate encrypted credential.

If you don't know how to install [Docker][docker], you may consider reference my blog [Installing And Configuring Docker Community Edition(CE) On GNU/Linux]({{< relref "2017-03-06-Installing-Configuring-Docker-Community-Edition-CE-On-GNU-Linux.md" >}}).


```bash
docker pull ruby
docker run -ti --rm ruby bash
```

Executing the following commands in container

```bash
# install travis
gem install travis

touch .travis.yml
travis encrypt "maxdsre:ne18Xpc5RbzM1hIdd34nN2aE" --add notifications.slack.rooms -r MaxdSre/maxdsre.github.io
cat .travis.yml
```

details

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

Error info

GitHub repo name

>Can't figure out GitHub repo name. Ensure you're in the repo directory, or specify the repo name via the -r option (e.g. travis <command> -r <owner>/<repo>)

.travis.yml

>no .travis.yml found


token `maxdsre:ne18Xpc5RbzM1hIdd34nN2aE` 經`travis`加密後生成的字符串爲

```txt
o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
```

## .travis.yml
Refering to official document [Configuring Build Notifications][config_build_notification], configuring directive `notifications` in file `.travis.yml` which is saved in the root dir of project `MaxdSre/maxdsre.github.io`.

```yml
notifications:
  email: false    # default notification method
  slack:
    rooms:
      - secure: o5TEEslShhI5AGKaVMbV5bPT7Hkh/lQkqd2T76fXfgXVGpxGWY+rQmijyh4hm7APKzOwxw64J/L7rTVKlW14QlCXe4ff8+diooG365g6j9h8esuXpJhRDtP72SrPHkAOwo8loX72DR7uazRUYSs0AOGwmYHyctLnhNOmWPVkp3ISAK9unEOIZ0UJ7381rZVWTiVReAVrbpe5jlX3nSbBdineKVar+511BHO+eKTU28AlCFgUSG3isvV1dUhZZfHHUaOx3ApzZAmx7GYDBIIqcfZ6r5x1PmZIDjnH3v4TvxRhUpAp6L5IGbpqhAps5dg73nZfzJz4+bM4ruMRBo9PDfgVbA+9zMpN8/yBL6E/OS2u045TJ491n8PE/+GoajMCzijuoKgHjTzWyVhIV9dEmVOpjOZL9rv6R1mqKoUwbV8rs8WjqciBlfoC6WwZOAWTcXok5RIMmSQ94oBZEtGSmTtALS5D0D7uCuHJt2Nz+GhrKbuwM/eSRYUwyWvNJdqSTbFnEJp27AhP5hxHrzg7uEv9cablPNf1NTFquZJH/ROUf0nKqsVZOJMQElRdcr37Es4tSg6WTKYs4XIeh1R9Yjws53KLOv2phTvFzhGPmycgPrADhHVnGoWtePyNCwASc+x7HQJQoUM6Y2i9AVruHN3XmsXq7dSIu3srqPy2lRw=
    on_success: always  # change: send a notification when the build status changes.
    on_failure: always  # always: always send a notification.
```

### Template
Default message template

```bash
# push
- "Build <%{build_url}|#%{build_number}> (<%{compare_url}|%{commit}>) of %{repository_slug}@%{branch} by %{author} %{result} in %{duration}"

# pull
- "Build <%{build_url}|#%{build_number}> (<%{compare_url}|%{commit}>) of %{repository_slug}@%{branch} in PR <%{pull_request_url}|#%{pull_request_number}> by %{author} %{result} in %{duration}"
```

custom template, the final format

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

## Testing
Committing codes to GitHub after file `.travis.yml` is modified.

Travis CI build info

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-54-57.png)

Slack notification

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_14-54-26.png)

Build fail notification

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_16-03-24.png)

Build success notification

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-18_slack_notification_in_travis_ci/2018-04-18_16-13-33.png)

## Change Log
* 2018.04.18 16:07 Tue America/Boston
    * first draft

[travisci]: https://travis-ci.org "Test and Deploy with Confidence"
[slack]: https://slack.com "Where work happens"
[github]: https://github.com
[docker]:https://www.docker.com "Docker"
[config_build_notification]:https://docs.travis-ci.com/user/notifications#Configuring-Slack-notifications

<!-- End -->
