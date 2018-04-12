---
title: "Auto Deploy Blog to Github Pages via Hugo and Travis CI"
date: 2018-04-11T00:20:07-04:00
lastmod: 2018-04-11T00:20:07-04:00
draft: false
keywords: []
description: "Using Hugo and Travis CI to deploy personal blog to Github pages automatically"

categories:
- Blog
tags:
- Github
- Hugo
- Travis CI
- Git

comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: true
hiddenFromHomePage: false
contentCopyright: ""
reward: false
mathjax: false
mathjaxEnableSingleDollar: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams:
  enable: false
  options: ""
---

個人Blog採用靜態Blog形式託管在[Github][github]上。此前使用的是[Hexo][hexo]，因其包依賴關係複雜，部署流程繁瑣，故將整個部署環境封裝到Docker鏡像中以實現快速部署，但仍較爲繁瑣。現轉用執行速度快、操作簡便的[Hugo][hugo]。本文記錄如何在GNU/Linux中通過[Travis CI][travisci]將[Hugo][hugo]生成的Blog內容自動同步到[Github][github]，實現持續集成、部署。

**注意**：本文旨在記錄關鍵操作，不涉及帳號註冊、Git安裝配置等過程。

<!--more-->

## Prerequisite
前提條件

1. 在[Github][github]註冊帳號；
2. 在[Travis CI][travisci]註冊帳號(可通過Github帳號登入)；
3. 本地系統安裝`git`，`hugo`；
4. 個人域名(可選，此處指定域名`axdlog.com`)；


## Official Docs
官方文檔

本文中的相關操作皆以官方文檔爲操作憑據

* Github
    * [GitHub Pages Basics](https://help.github.com/categories/github-pages-basics/)
    * [User, Organization, and Project Pages](https://help.github.com/articles/user-organization-and-project-pages/)
    * [Creating Project Pages using the command line](https://help.github.com/articles/creating-project-pages-using-the-command-line/)
    * [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)
* Hugo
    * [Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
* Travis CI
    * [GitHub Pages Deployment][travisci-github-page]
    * [Building a Python Project](https://docs.travis-ci.com/user/languages/python/)
    * [Best Practices in Securing Your Data](https://docs.travis-ci.com/user/best-practices-security/)


## Hugo Installation
[Hugo][hugo]的Github地址爲 <https://github.com/gohugoio/hugo>，下載地址頁面爲 <https://github.com/gohugoio/hugo/releases>。最新釋出版本信息可通過API <https://api.github.com/repos/gohugoio/hugo/releases/latest> 獲取。

[Hugo][hugo]官方的安裝文檔頁[Install Hugo](https://gohugo.io/getting-started/installing/)，如果使用的是GNU/Linux，可考慮使用本人寫的[Python script](https://github.com/MaxdSre/Python-learning-quiz/blob/master/LearningQuiz/Hugo-StaticSiteGeneratorInstallation.ipynb)安裝。

當前版本信息

```bash
# which hugo
/usr/local/bin/hugo

# hugo version
Hugo Static Site Generator v0.38.2 linux/amd64 BuildDate: 2018-04-09T08:17:17Z
```

### Create A New Site
如何通過Hugo創建一個新的站點，詳見官方文檔 [Get Started](https://gohugo.io/getting-started/)、[Quick Start](https://gohugo.io/getting-started/quick-start/)。

>If this is your first time using Hugo and you’ve [already installed Hugo on your machine](https://gohugo.io/getting-started/installing/), we recommend the [quick start](https://gohugo.io/getting-started/quick-start/).

此處且將站點名稱命名爲`quickstart`(其實就是目錄名，不一定是最終的網站名稱)。

Hugo生成的目錄結構如下

```bash
# tree -L 2
.
├── archetypes
│   └── default.md
├── config.toml
├── content
│   ├── about.md
│   └── post
├── data
├── layouts
├── static
└── themes
    └── even

8 directories, 3 files
```

**注意**：這些不是最終的Blog內容，是用於生成Blog內容的文件。

目錄`content`中存放自己需要發佈的Blog (Markdown文件)；

如果要預覽Blog最終效果，可通過命令`hugo server`在本地開啓一個Web服務器，端口號`1313`，在瀏覽器中打開即可。

```bash
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

在項目所在目錄中執行命令`hugo`，會生成名爲`public`的目錄，該目錄存放着最終的Blog內容文件。將`public`目錄中的所有內容上傳至Github的`master`分支中即可。

演示過程如下

```bash
# hugo

                   | EN   
+------------------+-----+
  Pages            | 426  
  Paginator pages  |  24  
  Non-page files   |   0  
  Static files     |  34  
  Processed images |   0  
  Aliases          |  95  
  Sitemaps         |   1  
  Cleaned          |   0  

Total in 403 ms

# tree -L 1
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── public
├── static
└── themes

7 directories, 1 file
```


## Github Repository Operation
**重要**：本部分的處理思路、操作非常重要，Git配置步驟省略。

使用Github Page託管Blog內容，需要創建名爲`<username>.github.io`的倉庫，並將Hugo生成的Blog內容提交到倉庫的`master`中。

因Hugo生成的內容分爲兩部分，源文件和`public`目錄中的Blog內容。故通過`git checkout --orphan`在同一倉庫中創建2個分支：
* 分支`code`用於存放源文件；
* 分支`master`用於存放`public`目錄中的Blog內容；

處理思路與官方文檔 [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)不同，需要注意。

### Create Empty Repository
本人GitHub帳號名爲`MaxdSre`，故創建倉庫`maxdsre.github.io`。

創建倉庫後，出現如下提示信息

>or create a new repository on the command line

```bash
echo "# maxdsre.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:MaxdSre/maxdsre.github.io.git
git push -u origin master
```

>or push an existing repository from the command line

```bash
git remote add origin git@github.com:MaxdSre/maxdsre.github.io.git
git push -u origin master
```

### Pushing Repository
操作分爲3步：

1. 創建`code`分支，提交代碼到倉庫；
2. 在目標目錄中執行`hugo`生成`public`目錄，將`public`目錄中內容轉移到臨時目錄中， 清空目標目錄，將臨時目錄中文件複製到該目錄中；
3. 通過命令`git checkout --orphan`創建`master`分支，將目錄中內容提交到倉庫。

進入目標目錄（此處爲`/PATH/quickstart/`）後，順序執行如下操作命令

### Branch code

```bash
# Initialized empty Git repository in /PATH/.git/
git init

# Switched to a new branch 'code', equal to 'git branch code && git checkout code'
git checkout -b code

# exclude dir public/
echo '/public/' >> .gitignore
sed -r -i '/^\/public\/$/{$!d}' .gitignore

# Add file contents to the index
git add .

# Show the working tree status
# git status

# Record changes to the repository
git commit -m "Hugo generator code"

# Manage set of tracked repositories
git remote add origin git@github.com:MaxdSre/maxdsre.github.io.git

# Update remote refs along with associated objects to branch 'code'
git push -u origin code
```

### Branch master
在目錄中執行`hugo`生成`public`目錄。通過命令`git branch -a`可查看倉庫的分支信息。

```bash
# execute command hugo
hugo

# create temporary dir
HUGO_TEMP_DIR=$(mktemp -d)
cp -R public/* "$HUGO_TEMP_DIR"

# create orphan branch 'master'
git checkout --orphan master

# empty current dir
rm .git/index
git clean -fdx

# copy back contents in dir public/
cp -R "$HUGO_TEMP_DIR"/. .

# add, commit, push
git add .
# git status
git commit -m 'initial blog content'
git push -u origin master

# remove temp dir
[[ -d "$HUGO_TEMP_DIR" ]] && rm -rf "$HUGO_TEMP_DIR"
```

### Branch image
此操作爲可選操作，本人創建第3個分支`image`用作圖牀，本文中的圖片即存放在該分支中。

```bash
git checkout --orphan image
rm .git/index
git clean -fdx

# add content to this branch

git add .
git commit -m 'initial blog images'
git push -u origin image
```


## Travis CI Configuration
[Travis CI][travisci]官方說明文檔 [GitHub Pages Deployment][travisci-github-page]。

### Github Access Token Generation
> You’ll need to generate a [personal access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) with the public_repo or repo scope (repo is required for private repositories). Since the token should be private, you’ll want to pass it to Travis securely in your [repository settings](https://docs.travis-ci.com/user/environment-variables#Defining-Variables-in-Repository-Settings) or via [encrypted variables](https://docs.travis-ci.com/user/environment-variables#Defining-encrypted-variables-in-.travis.yml) in .travis.yml.

在Github的[Developer settings](https://github.com/settings/tokens)頁面中生成`personal access tokens`，若是私有倉庫，勾選`repo`；若是公開倉庫，勾選`public_repo`。此處本人選擇了 `public_repo`, `repo:status`, `repo_deployment`三項。

生成的是長度爲`40`的隨機字符串，注意請勿泄漏。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-13-54-github-access-token.png)


### Environment Variables Setting
生成的`token`須在[Travis CI][travisci]的目標Repo中設置，頁面地址爲 <https://travis-ci.org/MaxdSre/maxdsre.github.io/settings>。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-11-05-travisci-system-var-setting.png)

確保`General`中的`Build only if .travis.yml is present`, `Build pushed branches`, `Build pushed pull requests`已啓用。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-10-39-travisci-general-setting.png)

官方文檔[GitHub Pages Deployment][travisci-github-page]定義的默認`token`名爲`GITHUB_TOKEN`，其它指令詳見文檔。

> Notice that the values are not escaped when your builds are executed. Special characters (for bash) should be escaped accordingly.

此處定義相關變量，稍後會寫入`.travis.yml`文件。

Environment Variables | Value
---|---
GITHUB_USERNAME | MaxdSre
GITHUB_EMAIL | maxdsre@gmail.com
GITHUB_TOKEN | 75e8b72618ebf48df0b235cp4affd79e167b2489 (假設值)


### .travis.yml directives
[Hugo][hugo]是用Golang構建的，故最開始在`.travis.yml`中用`language: go`，但[Travis CI][travisci]構建Golang環境耗時太長(數分鐘)，且出現`hugo`安裝失敗的清空。看到他人教程中有用Python的，恰巧最近在學習Python，便選擇Python做爲構建環境。因[Travis CI][travisci]的容器環境是基於Ubuntu的，故可以使用`dpkg`、`apt-get`等命令，但需要添加`sudo`。

經過反復測試，最終指令如下

```yml
# https://docs.travis-ci.com/user/deployment/pages/
# https://docs.travis-ci.com/user/languages/python/
language: python

python:
    - "3.6"

install:
    # install latest release version
    - wget $(wget -qO- https://api.github.com/repos/gohugoio/hugo/releases/latest | sed -r -n '/browser_download_url/{/Linux-64bit.deb/{s@[^:]*:[[:space:]]*"([^"]*)".*@\1@g;p}}')
    - sudo dpkg -i hugo*.deb
    - pip install Pygments
    - rm -rf public 2> /dev/null

script:
    - hugo
    - echo 'axdlog.com' > public/CNAME

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN  # Set in travis-ci.org dashboard, marked secure
  email: $GITHUB_EMAIL
  name: $GITHUB_USERNAME
  verbose: true
  keep-history: true
  local-dir: public
  target_branch: master  # branch contains blog content
  on:
    branch: code  # branch contains Hugo generator code
```

將指令寫入到文件`.travis.yml`中，並將該`.travis.yml`上傳至repo的`code`分支中。


## Continuous Integration
所有操作完成後，只需將需要發佈的Blog上傳到repo的`code`分支中，[Travis CI][travisci]會自動進行部署操作，將更新的Blog內容更新到repo的`master`分支中，實現Blog的持續集成，自動部署。

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-35-56-travisci-deploy-status.png)

部署過程日誌

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-36-29-travisci-deploy-log.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2018-04-10-go_travisci_github_page/2018-04-11_00-36-57-travisci-deploy-log.png)


## References
* [Continuous Integration of Hugo Website using Travis CI and Github](https://jellis18.github.io/post/2017-12-03-continuous-integration-hugo/)
* [Travis continuous delivery](https://github.com/ldez/travis-continuous-delivery-hugo-deploy)
* [How to create a website like freshswift.net using Hugo, Travis CI, and GitHub Pages](https://medium.com/zendesk-engineering/how-to-create-a-website-like-freshswift-net-using-hugo-travis-ci-and-github-pages-67be6f480298)

## Change logs
* 2018-04-11 00:20 Wed America/Boston
    * 初稿完成


[hexo]: https://hexo.io "A fast, simple & powerful blog framework"
[hugo]: https://gohugo.io "The world’s fastest framework for building websites"
[github]: https://github.com
[travisci]: https://travis-ci.org "Test and Deploy with Confidence"
[travisci-github-page]: https://docs.travis-ci.com/user/deployment/pages/ "Travis CI - GitHub Pages Deployment"

<!-- End -->
