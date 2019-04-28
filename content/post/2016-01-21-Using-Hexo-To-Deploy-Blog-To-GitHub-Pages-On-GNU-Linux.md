---
title: Using Hexo To Deploy Blog To GitHub Pages On GNU/Linux
slug: Using Hexo To Deploy Blog To GitHub Pages On GNU Linux
date: 2016-01-21T19:02:32+08:00
lastmod: 2019-04-28T14:29:02-04:00
draft: false
keywords: ["Hexo", "Github", "Node.js", "Git"]
description: "Using Hexo To Deploy Blog To GitHub Pages On GNU/Linux"

categories:
- Blog
tags:
- Hexo
- GitHub

comment: true
toc: true

---

此爲本人在GitHub上搭建個人部落格的操作過程，過程很艱辛，結果很美好，一天時間總算沒有白費。Docker化操作見blog 『[Using Docker Container To Auto Deploy Blog To GitHub Pages]({{< relref "2016-06-16-Using-Docker-Container-To-Auto-Deploy-Blog-To-GitHub-Pages.md" >}})』

<!--more-->

## Preparartion

`hexo`需要使用[npm](https://www.npmjs.com/ 'npmjs')安裝，而[npm](https://www.npmjs.com/ 'npmjs')是[Node.js](https://nodejs.org/en/ 'Node.js')的包管理器，已經預置在`Node.js`。

故安裝[hexo](https://hexo.io/ 'hexo')之前，操作系統上須已經安裝有`Git`, `Node.js`環境。

### Related Software

| Software | Official Website |
| :--- | :--- |
| `CentOS7` | <https://www.centos.org/> |
| `Git` | <https://github.com/git/git> |
| `Node.js` | <https://nodejs.org/en/> |


系統相關信息
```sh
[flying@lemp ~]$ date -R
Thu, 21 Jan 2016 12:56:44 +0800
[flying@lemp ~]$ cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
[flying@lemp ~]$ uname -r
3.10.0-327.4.4.el7.x86_64
[flying@lemp ~]$ git --version
git version 2.7.0
[flying@lemp ~]$ node --version
v5.4.0
[flying@lemp ~]$ npm --version
3.3.12
[flying@lemp ~]$
```


---
## Install Hexo
執行命令
```
sudo npm install hexo-cli -g
```

注意：需要使用root權限運行`npm install`命令，否則會出現如下報錯
```
npm WARN install Couldn't install optional dependency: Unsupported
npm WARN checkPermissions Missing write access to /usr/lib/node_modules
...
npm ERR! Error: EACCES: permission denied, access '/usr/lib/node_modules'
...
npm ERR! Please try running this command again as root/Administrator.
```

操作過程
```
[flying@lemp ~]$ sudo npm install hexo-cli -g
[sudo] password for flying:
npm WARN install Couldn't install optional dependency: Unsupported
/usr/bin/hexo -> /usr/lib/node_modules/hexo-cli/bin/hexo
/usr/lib
└─┬ hexo-cli@0.2.0
  ├── abbrev@1.0.7
  ├── bluebird@3.1.1
  ├─┬ chalk@1.1.1
  │ ├── ansi-styles@2.1.0
  │ ├── escape-string-regexp@1.0.4
  │ ├─┬ has-ansi@2.0.0
  │ │ └── ansi-regex@2.0.0
  │ ├── strip-ansi@3.0.0
  │ └── supports-color@2.0.0
  ├─┬ hexo-fs@0.1.5
  │ ├─┬ chokidar@1.4.2
  │ │ ├─┬ anymatch@1.3.0
  │ │ │ ├── arrify@1.0.1
  │ │ │ └─┬ micromatch@2.3.7
  │ │ │   ├─┬ arr-diff@2.0.0
  │ │ │   │ └── arr-flatten@1.0.1
  │ │ │   ├── array-unique@0.2.1
  │ │ │   ├─┬ braces@1.8.3
  │ │ │   │ ├─┬ expand-range@1.8.1
  │ │ │   │ │ └─┬ fill-range@2.2.3
  │ │ │   │ │   ├── is-number@2.1.0
  │ │ │   │ │   ├── isobject@2.0.0
  │ │ │   │ │   ├── randomatic@1.1.5
  │ │ │   │ │   └── repeat-string@1.5.2
  │ │ │   │ ├── preserve@0.2.0
  │ │ │   │ └── repeat-element@1.1.2
  │ │ │   ├── expand-brackets@0.1.4
  │ │ │   ├── extglob@0.3.2
  │ │ │   ├── filename-regex@2.0.0
  │ │ │   ├─┬ kind-of@3.0.2
  │ │ │   │ └── is-buffer@1.1.1
  │ │ │   ├── normalize-path@2.0.1
  │ │ │   ├─┬ object.omit@2.0.0
  │ │ │   │ ├─┬ for-own@0.1.3
  │ │ │   │ │ └── for-in@0.1.4
  │ │ │   │ └── is-extendable@0.1.1
  │ │ │   ├─┬ parse-glob@3.0.4
  │ │ │   │ ├── glob-base@0.3.0
  │ │ │   │ └── is-dotfile@1.0.2
  │ │ │   └─┬ regex-cache@0.4.2
  │ │ │     ├── is-equal-shallow@0.1.3
  │ │ │     └── is-primitive@2.0.0
  │ │ ├── async-each@0.1.6
  │ │ ├── glob-parent@2.0.0
  │ │ ├── inherits@2.0.1
  │ │ ├─┬ is-binary-path@1.0.1
  │ │ │ └── binary-extensions@1.4.0
  │ │ ├─┬ is-glob@2.0.1
  │ │ │ └── is-extglob@1.0.0
  │ │ ├── path-is-absolute@1.0.0
  │ │ └─┬ readdirp@2.0.0
  │ │   ├─┬ minimatch@2.0.10
  │ │   │ └─┬ brace-expansion@1.1.2
  │ │   │   ├── balanced-match@0.3.0
  │ │   │   └── concat-map@0.0.1
  │ │   └─┬ readable-stream@2.0.5
  │ │     ├── core-util-is@1.0.2
  │ │     ├── isarray@0.0.1
  │ │     ├── process-nextick-args@1.0.6
  │ │     ├── string_decoder@0.10.31
  │ │     └── util-deprecate@1.0.2
  │ └── graceful-fs@4.1.2
  ├─┬ hexo-util@0.5.1
  │ ├─┬ camel-case@1.2.2
  │ │ ├─┬ sentence-case@1.1.3
  │ │ │ └── lower-case@1.1.3
  │ │ └── upper-case@1.1.3
  │ ├── highlight.js@9.1.0
  │ └── html-entities@1.2.0
  ├── minimist@1.2.0
  └─┬ tildify@1.1.2
    └── os-homedir@1.0.1

[flying@lemp ~]$
```

hexo相關信息
```
[flying@lemp ~]$ type hexo
hexo is hashed (/usr/bin/hexo)
[flying@lemp ~]$ which hexo
/usr/bin/hexo

#hexo版本信息
[flying@lemp ~]$ hexo version
hexo-cli: 0.2.0
os: Linux 3.10.0-327.4.4.el7.x86_64 linux x64
http_parser: 2.6.0
node: 5.4.0
v8: 4.6.85.31
uv: 1.8.0
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 56.1
modules: 47
openssl: 1.0.2e
[flying@lemp ~]$
```

---
## Setup Hexo
安裝目錄以`/home/flying/ndoe_hexo`爲例

### Setup Blog Dir
執行命令
```
mkdir /home/flying/node_hexo
cd /home/flying/node_hexo
hexo init LempStacker.github.io
cd LempStacker.github.io
npm install
```


操作過程
```
[flying@lemp node_hexo]$ hexo init LempStacker.github.io
INFO  Cloning hexo-starter to ~/node_hexo/LempStacker.github.io
Cloning into '/home/flying/node_hexo/LempStacker.github.io'...
remote: Counting objects: 40, done.
remote: Total 40 (delta 0), reused 0 (delta 0), pack-reused 40
Unpacking objects: 100% (40/40), done.
Checking connectivity... done.
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into 'themes/landscape'...
remote: Counting objects: 818, done.
remote: Total 818 (delta 0), reused 0 (delta 0), pack-reused 818
Receiving objects: 100% (818/818), 2.54 MiB | 16.00 KiB/s, done.
Resolving deltas: 100% (430/430), done.
Checking connectivity... done.
Submodule path 'themes/landscape': checked out 'cec7746c852676d311246c789a0f0a0f28da4c9f'
INFO  Install dependencies
npm WARN install Couldn't install optional dependency: Unsupported
npm WARN install Couldn't install optional dependency: Unsupported
npm WARN prefer global marked@0.3.5 should be installed with -g

> dtrace-provider@0.6.0 install /home/flying/node_hexo/LempStacker.github.io/node_modules/dtrace-provider
> node scripts/install.js

hexo-site@0.0.0 /home/flying/node_hexo/LempStacker.github.io
├─┬ hexo@3.1.1
│ ├── abbrev@1.0.7
│ ├── archy@1.0.0
│ ├── bluebird@2.10.2
│ ├─┬ bunyan@1.5.1
│ │ ├─┬ dtrace-provider@0.6.0
│ │ │ └── nan@2.2.0
│ │ ├─┬ mv@2.1.1
│ │ │ ├── ncp@2.0.0
│ │ │ └─┬ rimraf@2.4.5
│ │ │   └─┬ glob@6.0.4
│ │ │     ├─┬ inflight@1.0.4
│ │ │     │ └── wrappy@1.0.1
│ │ │     └── once@1.3.3
│ │ └── safe-json-stringify@1.0.3
│ ├─┬ chalk@1.1.1
│ │ ├── ansi-styles@2.1.0
│ │ ├── escape-string-regexp@1.0.4
│ │ ├─┬ has-ansi@2.0.0
│ │ │ └── ansi-regex@2.0.0
│ │ ├── strip-ansi@3.0.0
│ │ └── supports-color@2.0.0
│ ├─┬ cheerio@0.19.0
│ │ ├─┬ css-select@1.0.0
│ │ │ ├── boolbase@1.0.0
│ │ │ ├── css-what@1.0.0
│ │ │ ├── domutils@1.4.3
│ │ │ └── nth-check@1.0.1
│ │ ├─┬ dom-serializer@0.1.0
│ │ │ └── domelementtype@1.1.3
│ │ ├── entities@1.1.1
│ │ └─┬ htmlparser2@3.8.3
│ │   ├── domelementtype@1.3.0
│ │   ├── domhandler@2.3.0
│ │   ├── domutils@1.5.1
│ │   └── entities@1.0.0
│ ├─┬ hexo-cli@0.1.9
│ │ ├── bluebird@3.1.1
│ │ └── minimist@1.2.0
│ ├── hexo-front-matter@0.2.2
│ ├─┬ hexo-fs@0.1.5
│ │ ├── bluebird@3.1.1
│ │ ├─┬ chokidar@1.4.2
│ │ │ ├─┬ anymatch@1.3.0
│ │ │ │ ├── arrify@1.0.1
│ │ │ │ └─┬ micromatch@2.3.7
│ │ │ │   ├─┬ arr-diff@2.0.0
│ │ │ │   │ └── arr-flatten@1.0.1
│ │ │ │   ├── array-unique@0.2.1
│ │ │ │   ├─┬ braces@1.8.3
│ │ │ │   │ ├─┬ expand-range@1.8.1
│ │ │ │   │ │ └─┬ fill-range@2.2.3
│ │ │ │   │ │   ├── is-number@2.1.0
│ │ │ │   │ │   ├── isobject@2.0.0
│ │ │ │   │ │   ├── randomatic@1.1.5
│ │ │ │   │ │   └── repeat-string@1.5.2
│ │ │ │   │ ├── preserve@0.2.0
│ │ │ │   │ └── repeat-element@1.1.2
│ │ │ │   ├── expand-brackets@0.1.4
│ │ │ │   ├── extglob@0.3.2
│ │ │ │   ├── filename-regex@2.0.0
│ │ │ │   ├─┬ kind-of@3.0.2
│ │ │ │   │ └── is-buffer@1.1.1
│ │ │ │   ├── normalize-path@2.0.1
│ │ │ │   ├─┬ object.omit@2.0.0
│ │ │ │   │ ├─┬ for-own@0.1.3
│ │ │ │   │ │ └── for-in@0.1.4
│ │ │ │   │ └── is-extendable@0.1.1
│ │ │ │   ├─┬ parse-glob@3.0.4
│ │ │ │   │ ├── glob-base@0.3.0
│ │ │ │   │ └── is-dotfile@1.0.2
│ │ │ │   └─┬ regex-cache@0.4.2
│ │ │ │     ├── is-equal-shallow@0.1.3
│ │ │ │     └── is-primitive@2.0.0
│ │ │ ├── async-each@0.1.6
│ │ │ ├── glob-parent@2.0.0
│ │ │ ├── inherits@2.0.1
│ │ │ ├─┬ is-binary-path@1.0.1
│ │ │ │ └── binary-extensions@1.4.0
│ │ │ ├─┬ is-glob@2.0.1
│ │ │ │ └── is-extglob@1.0.0
│ │ │ ├── path-is-absolute@1.0.0
│ │ │ └─┬ readdirp@2.0.0
│ │ │   └─┬ readable-stream@2.0.5
│ │ │     ├── process-nextick-args@1.0.6
│ │ │     └── util-deprecate@1.0.2
│ │ └── graceful-fs@4.1.2
│ ├─┬ hexo-i18n@0.2.1
│ │ └── sprintf-js@1.0.3
│ ├─┬ hexo-util@0.1.7
│ │ ├── ent@2.2.0
│ │ └── highlight.js@8.9.1
│ ├─┬ js-yaml@3.5.2
│ │ ├─┬ argparse@1.0.4
│ │ │ └── lodash@4.0.0
│ │ └── esprima@2.7.1
│ ├── lodash@3.10.1
│ ├─┬ minimatch@2.0.10
│ │ └─┬ brace-expansion@1.1.2
│ │   ├── balanced-match@0.3.0
│ │   └── concat-map@0.0.1
│ ├── moment@2.10.6
│ ├── moment-timezone@0.3.1
│ ├─┬ nunjucks@1.3.4
│ │ ├─┬ chokidar@0.12.6
│ │ │ └─┬ readdirp@1.3.0
│ │ │   ├── graceful-fs@2.0.3
│ │ │   ├── minimatch@0.2.14
│ │ │   └── readable-stream@1.0.33
│ │ └─┬ optimist@0.6.1
│ │   ├── minimist@0.0.8
│ │   └── wordwrap@0.0.3
│ ├── pretty-hrtime@1.0.1
│ ├─┬ strip-indent@1.0.1
│ │ └── get-stdin@4.0.1
│ ├─┬ swig@1.4.2
│ │ └─┬ uglify-js@2.4.24
│ │   ├── async@0.2.10
│ │   ├── uglify-to-browserify@1.0.2
│ │   └─┬ yargs@3.5.4
│ │     ├── camelcase@1.2.1
│ │     ├── decamelize@1.1.2
│ │     ├── window-size@0.1.0
│ │     └── wordwrap@0.0.2
│ ├─┬ swig-extras@0.0.1
│ │ └─┬ markdown@0.5.0
│ │   └── nopt@2.1.2
│ ├── text-table@0.2.0
│ ├─┬ through2@1.1.1
│ │ ├─┬ readable-stream@1.1.13
│ │ │ ├── core-util-is@1.0.2
│ │ │ ├── isarray@0.0.1
│ │ │ └── string_decoder@0.10.31
│ │ └── xtend@4.0.1
│ ├─┬ tildify@1.1.2
│ │ └── os-homedir@1.0.1
│ ├── titlecase@1.0.2
│ └─┬ warehouse@1.0.3
│   ├── cuid@1.2.5
│   └─┬ JSONStream@1.0.7
│     ├── jsonparse@1.2.0
│     └── through@2.3.8
├─┬ hexo-generator-archive@0.1.4
│ ├─┬ hexo-pagination@0.0.2
│ │ └── utils-merge@1.0.0
│ └── object-assign@2.1.1
├── hexo-generator-category@0.1.3
├─┬ hexo-generator-index@0.2.0
│ └── object-assign@4.0.1
├── hexo-generator-tag@0.1.2
├─┬ hexo-renderer-ejs@0.1.1
│ ├── ejs@1.0.0
│ └── object-assign@4.0.1
├─┬ hexo-renderer-marked@0.2.8
│ ├─┬ hexo-util@0.5.1
│ │ ├── bluebird@3.1.1
│ │ ├─┬ camel-case@1.2.2
│ │ │ ├─┬ sentence-case@1.1.3
│ │ │ │ └── lower-case@1.1.3
│ │ │ └── upper-case@1.1.3
│ │ ├── highlight.js@9.1.0
│ │ └── html-entities@1.2.0
│ ├── marked@0.3.5
│ └── object-assign@4.0.1
├─┬ hexo-renderer-stylus@0.3.0
│ ├─┬ nib@1.1.0
│ │ └─┬ stylus@0.49.3
│ │   ├─┬ glob@3.2.11
│ │   │ └── minimatch@0.3.0
│ │   └── mkdirp@0.3.5
│ └─┬ stylus@0.52.4
│   ├── css-parse@1.7.0
│   ├─┬ debug@2.2.0
│   │ └── ms@0.7.1
│   ├─┬ glob@3.2.11
│   │ └─┬ minimatch@0.3.0
│   │   ├── lru-cache@2.7.3
│   │   └── sigmund@1.0.1
│   ├── mkdirp@0.5.1
│   ├── sax@0.5.8
│   └─┬ source-map@0.1.34
│     └── amdefine@1.0.0
└─┬ hexo-server@0.1.3
  ├── bluebird@3.1.1
  ├─┬ compression@1.6.1
  │ ├─┬ accepts@1.3.1
  │ │ ├── mime-types@2.1.9
  │ │ └── negotiator@0.6.0
  │ ├── bytes@2.2.0
  │ ├─┬ compressible@2.0.7
  │ │ └── mime-db@1.21.0
  │ ├── on-headers@1.0.1
  │ └── vary@1.1.0
  ├─┬ connect@3.4.0
  │ ├─┬ finalhandler@0.4.0
  │ │ ├── escape-html@1.0.2
  │ │ └── unpipe@1.0.0
  │ └── parseurl@1.3.1
  ├── mime@1.3.4
  ├─┬ morgan@1.6.1
  │ ├── basic-auth@1.0.3
  │ ├── depd@1.0.1
  │ └─┬ on-finished@2.3.0
  │   └── ee-first@1.1.1
  ├── object-assign@4.0.1
  ├─┬ opn@3.0.3
  │ └── object-assign@4.0.1
  └─┬ serve-static@1.10.2
    ├── escape-html@1.0.3
    └─┬ send@0.13.1
      ├── depd@1.1.0
      ├── destroy@1.0.4
      ├── escape-html@1.0.3
      ├── etag@1.7.0
      ├── fresh@0.3.0
      ├── http-errors@1.3.1
      ├── range-parser@1.0.3
      └── statuses@1.2.1

INFO  Start blogging with Hexo!

#切換到LempStacker.github.io/目錄下
[flying@lemp node_hexo]$ cd LempStacker.github.io/

#列出目錄下內容
[flying@lemp LempStacker.github.io]$ ls
_config.yml  node_modules  package.json  scaffolds  source  themes

#執行npm install
[flying@lemp LempStacker.github.io]$ npm install
npm WARN install Couldn't install optional dependency: Unsupported
npm WARN install Couldn't install optional dependency: Unsupported

#使用tree命令以樹狀形式列出目錄下文件
[flying@lemp LempStacker.github.io]$ tree -L 1
.
├── _config.yml
├── node_modules
├── package.json
├── scaffolds
├── source
└── themes

4 directories, 2 files
[flying@lemp LempStacker.github.io]$
```


#### File Explanation

| file | Explanation |
| :--- | :--- |
| `_config.yml` | Site [configuration](https://hexo.io/docs/configuration.html) file. You can configure most settings here. |
| `db_json` |  |
| `package.json` |  |
| `node_modules` |  |
| `scaffolds` | [Scaffold](https://hexo.io/docs/writing.html#Scaffolds) folder. When you create a new post, Hexo bases the new file on the scaffold. |
| `themes` | [Theme](https://hexo.io/docs/themes.html) folder. Hexo generates a static website by combining the site contents with the theme. |
| `source` | Source folder. This is where you put your site’s content. |


### Start the server
執行命令
```
hexo server
```
會提示`INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`

在瀏覽器中輸入URL `http://0.0.0.0:4000/`，可以查看本地博客頁面，按`Ctrl+C`停止。


操作過程
```sh
[flying@lemp LempStacker.github.io]$ hexo server
INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
^CINFO  See you again
[flying@lemp LempStacker.github.io]$
```

---
## Configuration Hexo
配置文件`_config.yml`，相關參數說明見 [Configuration](https://hexo.io/docs/configuration.html 'Hexo')。具體修改可參考 [Hexo+github搭建个人博客](http://tiandawu.com/2016/01/14/BuildMyBlog/) 中相關操作。

本人目的是將域名`lempstacker.com`解析到`LempStacker.github.io`上，故在`deploy`中需要添加`plugins: -hexo-generator-cname`。


配置詳情

```bash
# Site
title: LempStacker
subtitle: Love you, my lover
description: 小馬的技術部落格，專注於GNU Linux，目標Full Stack
author: lempstacker
language: zh
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://lempstacker.com
root: /
permalink: :title/
permalink_defaults:

...
...

# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:LempStacker/LempStacker.github.io.git
  branch: master
  plugins: -hexo-generator-cname
```

### Bulid sitemap
爲SEO，建立網站地圖，方便搜索引擎抓取

安裝相關插件
```
npm install hexo-generator-sitemap --save

#本人沒有安裝此選項
npm install hexo-generator-baidu-sitemap --save
```

在配置文件`_config.yml`中添加
```yml
# auto create sitemap
sitemap:
path: sitemap.xml
#baidusitemap:
#path: baidusitemap.xml
```

提交後，在瀏覽器中輸入<https://lempstacker.com/sitemap.xml>即可查看生成的內容


---
## Fetch Remote Git Repo
```
[flying@lemp LempStacker.github.io]$ git init
Initialized empty Git repository in /home/flying/node_hexo/LempStacker.github.io/.git/
[flying@lemp LempStacker.github.io]$ git remote add origin git@github.com:LempStacker/LempStacker.github.io.git
[flying@lemp LempStacker.github.io]$ git fetch origin
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
From github.com:LempStacker/LempStacker.github.io
 * [new branch]      master     -> origin/master
[flying@lemp LempStacker.github.io]$ git merge origin/master
[flying@lemp LempStacker.github.io]$
```

以下操作請不要使用`sudo`操作，會報錯
```
Permission denied (publickey).
fatal: Could not read from remote repository
```
GitHub官方說明 [Error: Permission denied (publickey)](https://help.github.com/articles/error-permission-denied-publickey/ 'GitHub')


---
## Deploy to GitHub
因有獨立域名`lempstacker.com`，故需要先執行如下命令
```
npm install hexo-generator-cname --save
```

操作過程
```sh
[flying@lemp LempStacker.github.io]$ npm install hexo-generator-cname --save
npm WARN install Couldn't install optional dependency: Unsupported
npm WARN install Couldn't install optional dependency: Unsupported
hexo-site@0.0.0 /home/flying/node_hexo/LempStacker.github.io
└── hexo-generator-cname@0.3.0

[flying@lemp LempStacker.github.io]$
```

執行如下命令
```
git add .
git commit -m "描述信息
git push origin master
hexo g
npm install hexo-deployer-git --save
hexo d
git fetch origin
git merge origin/master
```

在瀏覽器中輸入`http://lempstacker.com/`即能正常訪問剛生成的網站了。


## hexo `_config.yml`

此爲`_config.yml`文件，備份用

```YAML
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: LempStacker
#subtitle: Love you, my lover
subtitle: Master Technology, Change Destiny
description: The only thing I can control is myself
#description: 小馬的技術部落格，專注於GNU Linux，目標Full Stack
keywords: Linux,CentOS7,LEMP,MariaDB,Nginx,PHP,Python,Docker,Full Stack
author: lempstacker
language: en
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
#url: http://lempstacker.com
url: https://lempstacker.com   # with the https protocol
enforce_ssl: lempstacker.com   # without any protocol
root: /
permalink: :title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
#不渲染README.md，在source下生成README.md
skip_render: README.md

# Writing
#new_post_name: :title.md # File name of new posts
new_post_name: :year-:month-:day-:title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: http://hexo.io/plugins/
## Themes: http://hexo.io/themes/
#theme: landscape
theme: maupassant

# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:LempStacker/LempStacker.github.io.git
  branch: master
  plugins: -hexo-generator-cname

hightlight:
  enable: true
  auto_detect: true
  line_number: true
  tab_replace:


# 自动生成sitemap
#npm install hexo-generator-sitemap --save
#npm install hexo-generator-baidu-sitemap --save
sitemap:
path: sitemap.xml
#baidusitemap:
#path: baidusitemap.xml
```

在主題的head中

```bash
meta(name='description', content=config.description)
meta(name='keywords', content=config.keywords)
meta(name='canonical' href='#{config.url}/#{page.path}')
```

---
## Maupassant Edit
主題設置備份，項目[路徑](https://github.com/tufu9441/maupassant-hexo)

### https
在文件`layout/_partial/head.jade`中添加如下內容

```jade
script.
  var host = "lempstacker.com";
  if ((host == window.location.host) && (window.location.protocol != "https:")){
    window.location.protocol = "https";
  }
```

### css
在文件`source/css/style.css`中line *1083* 左右，將內容更改如下

```css
// All lines in gutter and code container
.line {
    height:    1.3em;
    // font-size: 13px;
    //change code size
    font-size: 1em;
}
```

### foot
文件`layout/_partial/footer.jade`，將其中更換爲如下內容

```
#footer= 'Copyright © ' + date(Date.now(), 'YYYY') + ' '
  a(href=url_for('.'), rel='nofollow')= config.title + ' '
  | is licensed under <img style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" />.
  |  Powered by
  a(rel='nofollow', target='_blank', href='https://hexo.io')  Hexo.
  a(rel='nofollow', target='_blank', href='https://github.com/tufu9441/maupassant-hexo')  Theme
  |  by
  a(rel='nofollow', target='_blank', href='https://github.com/pagecho')  Cho.
```

### search
文件`layout/_widget/search.jade`，對應更改`placeholder`中內容
* `Google Search`
* `Self Search`


### `_config.yml`

```yml
fancybox: true ## If you want to use fancybox please set the value to true.
duoshuo: ## Your duoshuo_shortname, e.g. username
disqus: ## Your disqus_shortname, e.g. username
uyan: ## Your uyan_id. e.g. 1234567
gentie: ## Your gentie_productKey, e.g. fc799538c7ad4cf5a5a0c2877a90cbd7
google_search: true ## Use Google search, true/false.
baidu_search: ## Use Baidu search, true/false.
swiftype: ## Your swiftype_key, e.g. m7b11ZrsT8Me7gzApciT
tinysou: ## Your tinysou_key, e.g. 4ac092ad8d749fdc6293
self_search: true ## Use a jQuery-based local search engine, true/false.
google_analytics:  ## Your Google Analytics tracking id, e.g. UA-42425684-2
baidu_analytics: ## Your Baidu Analytics tracking id, e.g. 8006843039519956000
show_category_count: false ## If you want to show the count of categories in the sidebar widget please set the value to true.
toc_number: true ## If you want to add list number to toc please set the value to true.
shareto: true ## If you want to use the share button please set the value to true.
busuanzi: true ## If you want to use Busuanzi page views please set the value to true.
widgets_on_small_screens: false ## Set to true to enable widgets on small screens.

menu:
  - page: home
    directory: .
    icon: fa-home
  # - page: reading
  #   directory: reading/
  #   icon: fa-book
  - page: learning
    directory: learning/
    icon: fa-graduation-cap
#  - page: archive
#    directory: archives/
#    icon: fa-archive
  - page: about
    directory: about/
    icon: fa-user
  # - page: rss
  #   directory: atom.xml
  #   icon: fa-rss


widgets: ## Six widgets in sidebar provided: search, category, tag, recent_posts, rencent_comments and links.
  - search
  - category
  - tag
  - recent_posts
  # - recent_comments
  - links

links:
  - title: Linux.com
    url: https://www.linux.com/
  - title: Opensource.com
    url: https://opensource.com/


# timeline:
#   - num: 1
#     word: 2014/06/12-Start
#   - num: 2
#     word: 2014/11/29-XXX
#   - num: 3
#     word: 2015/02/18-DDD
#   - num: 4
#     word: More

# Static files
js: js
css: css

# Theme version
version: 0.0.0
```

---
## References
* [Hexo](https://hexo.io/)
* [Hexo TW](https://hexo.io/zh-tw/)
* [Hexo Documentation](https://hexo.io/docs/)
* [hexo-npm](https://www.npmjs.com/package/hexo)
* [Using own domain with github and hexo](http://readorskip.com/2015/06/08/Using-own-domain-with-github-and-hexo/)
* [Hexo+github搭建个人博客](http://tiandawu.com/2016/01/14/BuildMyBlog/)
* [使用GitHub和Hexo搭建免费静态Blog](http://wsgzao.github.io/post/hexo-guide/)
* [How to use Hexo and deploy to GitHub Pages](https://gist.github.com/yt8yt/ff97d36f7a35948a8351)
* [Hexo生成sitemap站点地图的方法](http://blog.kenai.cc/article/hexo/hexo-skills/)
* [Hexo系列教程: (五)部署时保证README.md不被渲染](http://iread.io/2015/09/hexo-guide-5/)
* [starsky](http://starsky.gitcafe.io/categories/%E2%9B%BA%E7%94%B5%E8%84%91%E6%8A%80%E6%9C%AF/hexo/)

---
## Change Log
* 2016.01.21 18:50 Thu Asia/Beijing
    * 初稿完成
* 2016.01.26 19:22 Tue Asia/Beijing
    * 更新sitemap，添加README.md
* 2016.07.08 15:21 Fri Asia/Shanghai
    * 添加`Maupassant Edit`、更改`_config.yml`
* 2017.01.31 12:07 Tue America/Boston
    * 主題更新，配置修改
* 2017.05.04 09:21 Thu America/Boston
    * 主題更新，配置修改
* 2019.04.28 14:33 Sun America/Boston
    * 勘誤，遷移到新Blog


<!-- End -->
