---
title: Migrating Personal Hugo Static Blog From GitHub Pages To GitLab Pages
slug: Migrating Personal Hugo Static Blog From GitHub Pages To GitLab Pages
date: 2019-05-24T19:04:07-04:00
lastmod: 2019-05-24T19:04:07-04:00
draft: false
keywords: ["Hugo", "GitHub Pages", "GitLab Pages", "Pipelines"]
description: "Using Hugo And GitLab Pages To Deploy Personal Blog Automatically"

categories:
- Blog
tags:
- Github
- GitLab
- Hugo
- Git

comment: true
toc: true

---

As [GitHub and Export Controls](https://help.github.com/en/articles/github-and-export-controls) said *GitHub.com, GitHub Enterprise Server, and the information you upload to either product may be subject to US export control laws, including U.S. Export Administration Regulations (the EAR).*, I decide to migrate my personal hugo static blog from [GitHub][github] to [GitLab][gitlab].

More importantly, [GitLab Pages][gitlab_pages] is simpler and faster than [GitHub Pages][github_pages].

<!--more-->

Old blog is [**Using Hugo and Travis CI To Deploy Blog To Github Pages Automatically**]({{< relref "2018-04-10-Using-Hugo-and-Travis-CI-To-Deploy-Blog-To-Github-Pages-Automatically.md" >}}).

If you wanna discuss more about contents in my blog, you may consider join my [**Slack**](https://join.slack.com/t/maxdsre/shared_invite/enQtNjM0NTY1MjA2MTc5LTUyN2Y1ZjM0OWZhODgzNjhlY2Q3Yjk2ODJjNzIzOTUzMjMwOWYwMjkwNDdkNzg2OGUwZjIzNWMwMTM0ODcxYjA) channel.


## Official Documentations
Official documentation page is [GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/index.html).

* [Exploring GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/introduction.html)
    * Custom error codes pages
* [Category Vision - GitLab Pages](https://about.gitlab.com/direction/release/pages/)

Setting procedures

1. [Static sites and GitLab Pages domains](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_one.html 'Understand what is a static website, and how GitLab Pages default domains work.')
    * [Limitations](https://docs.gitlab.com/ee/user/project/pages/introduction.html#limitations)
2. [Projects for GitLab Pages and URL structure](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_two.html 'Forking projects and creating new ones from scratch, understanding URLs structure and baseurls.')
3. [GitLab Pages custom domains and SSL/TLS Certificates](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_three.html 'How to add custom domains and subdomains to your website, configure DNS records and SSL/TLS certificates.')
4. [Creating and Tweaking GitLab CI/CD for GitLab Pages](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_four.html 'Understand how to create your own .gitlab-ci.yml for your site.')

GitLab CI/CD Pipeline

* [GitLab CI/CD Pipeline Configuration Reference](https://gitlab.com/help/ci/yaml/README)


## Operation Roadmap
1. [GitLab][gitlab] create group, import project from [GitHub][github];
2. Change default branch, unprotect protected branch;
    * Delete default `master` branch, rename branch `code` to `master`, change default branch to `master`
3. Create `.gitlab-ci.yml`;
4. CloudFlare
    * DNS setting;
    * SSL certificate generating;
5. Add custom domain in [GitHub Pages][github_pages];


## Migrating Project
[GitLab][gitlab] supports import project from [GitHub][github], but it needs access token generated from [GitHub][github]. So you need to generate a personal access token from [GitHub][github] (Settings --> Developer settings --> Personal access tokens --> Generate new token) first.


### GitLab Group
Creating new group `axdlog`, project name `axdlog.gitlab.io`.

[GitLab][gitlab] home page

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_13:58:55_GitLab_Homepage.png)

Sign in page
![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_13:59:34_Sign in.png)

Two Factor Authentication

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:00:12_Signin_2FA.png)

After sign in

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:01:53_Projects_Explore.png)

New group button

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:04:37_New_Group_Button.png)

Add new group

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:05:24_New_Group.png)

New group page

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:05:39_New_Group_Overview.png)


### New Project
[GitLab][gitlab] provides 4 methods to create project.

Method 1 - Blank project

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:05:59_NewProject_1.png)

Method 2 - Create from template

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:06:10_NewProject_2.png)

Method 3 - Import project

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:06:16_New_Project_3.png)

Method 4 - CI/CD for external repo

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:06:21_New_Project_4.png)


### Import project
Here choose method 3 *Import project*.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:06:57_GitHub_import.png)

Generating personal access token from [GitHub][github].

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:07:36_GitHub_Token_Page.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:08:17_GitHub_Token_Generating.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:08:32_GitHub_Token_Generated.png)

Pasting token

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:08:47_Input_GitHub_Token.png)

#### Import process
Change path to `axdlog/axdlog.gitlab.io`.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:10:36_GitHub_import_1.png)

Scheduled status

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:10:48_GitHub_import_2.png)

Success status

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:11:06_GitHub_import_3.png)


### Project Overview
Now you can browser you project in [GitLab][gitlab] web page.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:11:41_GitHub_Repo_Imported.png)

SSH url

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:44:24_Project_Clone_SSH.png)


## Branch Operation
[GitLab][gitlab] protect default branch defaultly, you need to change default branch, unprotect protected branch first. Or it will prompt errors

>remote: GitLab: You can only delete protected branches using the web interface.

Web Page: Settings --> Repository -->  Protected Branches --> Protected branch --> Unprotect

>remote: GitLab: The default branch of a project cannot be deleted.

Web Page: Settings --> Repository --> Default Branch


Delete default `master` branch, rename branch `code` to `master`, change default branch to `master`

Default Branch

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:48:35_Default_Branch.png)

Protected branch

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:48:58_Protected_Branch.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:49:14_Unprotect_Branch.png)

Change Default Branch

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_14:55:38_Default_Branch_Change.png)


### Cli Command
```bash
# list all branches local and remote
git branch -a

# remove existed master branch
git push origin --delete master

# rename local branch from 'code' to 'master'
git branch -m code master    

# push the new branch, set local branch to track the new remote branch
git push --set-upstream origin master

# change default branch (remotes/origin/HEAD) to branch master
git remote set-head origin master

# delete the old branch 'code'
git push origin --delete code
```

Operation records

```bash
$git clone git@gitlab.com:axdlog/axdlog.gitlab.io.git
Cloning into 'axdlog.gitlab.io'...
remote: Enumerating objects: 17349, done.
remote: Total 17349 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (17349/17349), 40.34 MiB | 10.82 MiB/s, done.
Resolving deltas: 100% (9151/9151), done.

$cd axdlog.gitlab.io/

$git branch -a
* code
  remotes/origin/HEAD -> origin/code
  remotes/origin/code
  remotes/origin/image
  remotes/origin/master

$git push origin --delete master
To gitlab.com:axdlog/axdlog.gitlab.io.git
 - [deleted]           master

$git branch -m code master

$git branch -a
* master
  remotes/origin/HEAD -> origin/code
  remotes/origin/code
  remotes/origin/image

$git push --set-upstream origin master
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: To create a merge request for master, visit:
remote:   https://gitlab.com/axdlog/axdlog.gitlab.io/merge_requests/new?merge_request%5Bsource_branch%5D=master
remote:
To gitlab.com:axdlog/axdlog.gitlab.io.git
 * [new branch]        master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.

$git remote set-head origin master

$git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/code
  remotes/origin/image
  remotes/origin/master

$git push origin --delete code
To gitlab.com:axdlog/axdlog.gitlab.io.git
 - [deleted]           code

$git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/image
  remotes/origin/master
```

## .gitlab-ci.yml
Creating file `.gitlab-ci.yml` in project root path, change variable `baseURL` to *https://axdlog.gitlab.io* in file `config.toml`.

```yml
# All available Hugo versions are listed here: https://gitlab.com/pages/hugo/container_registry
# https://gitlab.com/pages/hugo/blob/registry/Dockerfile
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

stages:
  - test
  - deploy

# before_script:

test:
  stage: test
  script:
  - hugo
  except:
  - master

pages:
  stage: deploy
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master
```

Commit info

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:02:57_gitlab-ci_yml_commit_.png)


## Pipelines
### Stages
status running

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:03:12_Pipelines_running.png)

status passed
![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:03:44_Pipelines_passed.png)

deploy stages

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:03:52_Pipeline_deploy_stages.png)

Job detail

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:04:09_pages_Jobs_detail.png)

### Artifacts
[Hugo][hugo] generate html file in directory `public`, [GitLab Pages][gitlab_pages] save them in to `Artifacts`. You can browse or download it.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:04:38_public_Artifacts.png)


## GitLab Pages
Switching to pages page (Settings --> Pages), you can see the default access pages url.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:05:12_Pages_page.png)

Browsering this url, it may appear 404 page, just wait minutes then reload page.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:08:55_pages_404.png)

It works

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:20:13_Pages_in_browser.png)


## CloudFlare
Using SSL certificate provided by [CloudFlare][cloudflare].

### Site Overview
[CloudFlare][cloudflare] homepage

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:22:54_Cloudflare_homepage.png)

Sign in page

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:23:08_Cloudflare_signin_page.png)

Dashboard

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:23:29_Cloudflare_dashboard.png)


### Add Site
![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:23:52_Cloudflare_addsite.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:24:39_Cloudflare_addsite_2.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:25:35_Cloudflare_addsite_3.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:25:47_Cloudflare_addsite_4.png)

You can set CNAME records latter.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:27:04_Cloudflare_addsite_5.png)

Change your nameservers in your domain register.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:27:45_Cloudflare_addsite_6.png)


### DNS Page
DNS CNAME records

type|key|value
---|---|---
CNAME|axdlog.com|axdlog.gitlab.io
CNAME|www|axdlog.com

<!-- TXT|_gitlab-pages-verification-code.axdlog.com|gitlab-pages-verification-code=0efdcc54eab467eafba72ae24fc85bea -->

Add CNAME records

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:29:56_Cloudflare_DNS_page.png)

### Origin Certificates
Creating SSL certificate, more details in [Issuing Certificates](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_three.html#issuing-certificates)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:30:37_Cloudflare_Origin_Cert.png)

Here choose **RSA**, [GitLab][gitlab] doesn't support **ECC**.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:31:58_Cloudflare_Origin_Cert_1.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:32:33_Cloudflare_Origin_Cert_2.png)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:32:49_Cloudflare_Origin_Cert_3.png)

## New Pages Domain
Pasting certificate and private key generated from [CloudFlare][cloudflare]. [GitLab Pages][gitlab_pages] needs to upload the Cloudflare Origin CA root certificate. Just paste [cloudflare_origin_rsa.pem](https://support.cloudflare.com/hc/en-us/article_attachments/206709108/cloudflare_origin_rsa.pem) below your certificate (PEM).

More details in

* [GitLab - Setting up GitLab Pages with CloudFlare Certificates](https://about.gitlab.com/2017/02/07/setting-up-gitlab-pages-with-cloudflare-certificates/)
* [Cloudflare - Managing Cloudflare Origin CA certificates](https://support.cloudflare.com/hc/en-us/articles/115000479507)

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:34:10_GitLab_New_Pages_Domain.png)

According to [DNS TXT record](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_three.html#dns-txt-record), you need to create a `TXT` record containing a verification code to prove that you own the domain. The code will be displayed after you [add your custom domain to GitLab Pages settings](https://docs.gitlab.com/ee/user/project/pages/getting_started_part_three.html#add-your-custom-domain-to-gitlab-pages-settings).

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:34:33_Pages_Domains_unverified.png)



### Custom Domain Verification
Adding verification code to DNS `TXT` record to verify ownership.

type|key|value
---|---|---
TXT|*_gitlab-pages-verification-code.axdlog.com* |gitlab-pages-verification-code=0efdcc54eab467eafba72ae24fc85bea

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:35:45_Cloudflare_DNS_Add_Verification_TXT.png)

verifing success

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:36:02_Pages_Domains_verified.png)

Pages overview, now it has custom domain `axdlog.com`.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:36:24_Pages_Overview.png)

## Testing
The web browser fails to load css file, you need to change `bashURL` to `axdlog.com` in file `config.toml`.

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:45:58_Pages_In_Browser_Without_CSS.png)

## Result
Visiting custom domain in web browser

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:50:05_Pages_In_Browser_Works.png)

SSL certificate

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_15:52:31_SSL_Cert.png)


## Slack Notifications
[GitLab][gitlab] supports [Slack][slack] notification, just following

* [GitLab Slack application](https://docs.gitlab.com/ee/user/project/integrations/gitlab_slack_application.html)
* [Slack Notifications Service](https://docs.gitlab.com/ee/user/project/integrations/slack.html)


Setting

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_16:01:03_Slack_notifications_setting.png)

Testing

![](https://gitlab.com/axdlog/axdlog.gitlab.io/raw/image/blog-image/2019-05-24-GitLab-Pages/2019-05-24_16:22:51_gitlab_slact_notification_testing.png)



## Chnage Log
* May 24, 2019 19:03 Fri -000 EST
    * first draft



[hugo]: https://gohugo.io "The world’s fastest framework for building websites"
[github]: https://github.com
[github_pages]: https://pages.github.com
[gitlab]: https://gitlab.com
[gitlab_pages]: https://about.gitlab.com/product/pages/
[cloudflare]: https://www.cloudflare.com
[slack]: https://slack.com "Where work happens"

<!-- End -->
