---
title: Installing And Setting Up The Symfony Framework On CentOS 7
date: 2016-08-23T21:03:16+08:00
lastmod: 2018-04-12T22:57:16-04:00
draft: false
keywords: ["Symfony", "PHP", 'LEMP', 'Twig']
description: "Full recording of Symfony installation on CentOS 7"
categories:
- PHP
tags:
- Symfony
- PHP
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
contentCopyright: ""
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
---


[Symfony][symfony]是一款[PHP][php]開發框架，目前最新版本是`4.0`，長期支持版是`3.4`。本文記錄如何在`CentOS 7`中安裝、配置Symfony 4。

[Symfony][symfony]官方文檔頁 <https://symfony.com/doc/current/index.html>

<!--more-->

操作系統信息

Info | Details
:--- | :---
OS Version | `CentOS Linux release 7.4.1708 (Core)`
Kernel Version | `3.10.0-693.21.1.el7.x86_64`

已啓用`SELinux`和`Firewalld`防火牆

```bash
[maxdsre@CentOS ~]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28

[maxdsre@CentOS ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources:
  services: dhcpv6-client
  ports: 22/tcp 80/tcp 443/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks: echo-request echo-reply timestamp-reply timestamp-request
  rich rules:

[maxdsre@CentOS ~]$
```


## Requirements
Symfony 4.0

>Symfony 4.0 requires PHP 7.1.3 or higher to run, in addition to other minor requirements. -- [Requirements for Running Symfony](https://symfony.com/doc/master/reference/requirements.html)

Symfony application

>To create your new Symfony application, first make sure you're using PHP 7.1 or higher and have [Composer][composer] installed. -- [Installing & Setting up the Symfony Framework](https://symfony.com/doc/current/setup.html)

Composer

>Composer requires PHP 5.3.2+ to run. A few sensitive php settings and compile flags are also required, but when using the installer you will be warned about any incompatibilities. -- [System Requirements](https://getcomposer.org/doc/00-intro.md#system-requirements)


## PHP Installation
`LEMP`開發環境的安裝、配置可參考本人Blog [LEMP Stack Installation And Configuration On CentOS 7
](https://axdlog.com/2016/lemp-stack-installation-and-configuration-on-centos-7/)

版本信息

Info | Details
:--- | :---
PHP | `7.2.4`
Nginx | `1.13.12`
MySQL | `5.7.21`


```bash
[maxdsre@CentOS ~]$ nginx -v
nginx version: nginx/1.13.12

[maxdsre@CentOS ~]$ mysql --version
mysql  Ver 14.14 Distrib 5.7.21, for Linux (x86_64) using  EditLine wrapper

[maxdsre@CentOS ~]$ php --version
PHP 7.2.4 (cli) (built: Mar 27 2018 17:23:35) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.4, Copyright (c) 1999-2018, by Zend Technologies
[maxdsre@CentOS ~]$
```


## Composer Installation
安裝方案來自[Composer][composer]官方文檔 [Download Composer](https://getcomposer.org/download/)。

```bash
# Download the installer to the current directory
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Verify the installer SHA-384
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Run the installer
php composer-setup.php

# Remove the installer
php -r "unlink('composer-setup.php');"

# move composer.phar to executed path
sudo mv composer.phar /usr/local/bin/composer
sudo chown root:root /usr/local/bin/composer
```

版本信息

Info | Details
:--- | :---
Composer| `1.6.3`


```bash
[maxdsre@CentOS ~]$ composer -V
Composer version 1.6.3 2018-01-31 16:28:17
[maxdsre@CentOS ~]$ composer --version
Composer version 1.6.3 2018-01-31 16:28:17
[maxdsre@CentOS ~]$
```

## Symfony
[Symfony][symfony]官方文檔 [Installing & Setting up the Symfony Framework](https://symfony.com/doc/current/setup.html)。

Symfony 4不需要單獨安裝Symfony，直接通過[Composer][composer]創建項目。

### New Symfony Project
此處假設項目名爲`axdlog`

```bash
# Create new project
composer create-project symfony/website-skeleton axdlog

# Checking for Security Vulnerabilities
cd axdlog
composer require sec-checker --dev
```

**注意**：[Composer][composer]需要啓用函數`proc_open`、`proc_get_status`，相關設置在文件`/etc/php.ini`的`disable_functions`中，執行如下命令即可啓用：

```bash
sudo sed -r -i '/disable_functions[[:space:]]*=/{s@(proc_open|proc_get_status),?@@g;}' /etc/php.ini
# restart php-fpm service
sudo systemctl restart php-fpm
```

否則會出現報錯

proc_open

>Failed to download symfony/website-skeleton from dist: The Process class relies on proc_open, which is not available on your PHP installation.

proc_get_status

>Warning: proc_get_status() has been disabled for security reasons in phar:///usr/local/bin/composer/vendor/symfony/process/Process.php on line 1279
>
PHP Warning:  proc_get_status() has been disabled for security reasons in phar:///usr/local/bin/composer/vendor/symfony/process/Process.php on line 1279


安裝完成後，出現提示信息

```
Some files may have been created or updated to configure your new packages.
Please review, edit and commit them: these files are yours.


 What's next?


  * Run your application:
    1. Change to the project directory
    2. Execute the php -S 127.0.0.1:8000 -t public command;
    3. Browse to the http://localhost:8000/ URL.

       Quit the server with CTRL-C.
       Run composer require server --dev for a better web server.

  * Read the documentation at https://symfony.com/doc


 Database Configuration


  * Modify your DATABASE_URL config in .env

  * Configure the driver (mysql) and
    server_version (5.7) in config/packages/doctrine.yaml


 How to test?


  * Write test cases in the tests/ folder
  * Run php bin/phpunit

```

當前目錄創建了一個名爲`axdlog`的目錄

```bash
[maxdsre@CentOS axdlog]$ echo $PWD
/home/maxdsre/axdlog
[maxdsre@CentOS axdlog]$ tree -L 2
.
├── assets
├── bin
│   ├── console
│   └── phpunit
├── composer.json
├── composer.lock
├── config
│   ├── bundles.php
│   ├── packages
│   ├── routes
│   ├── routes.yaml
│   ├── services_test.yaml
│   └── services.yaml
├── package.json
├── phpunit.xml.dist
├── public
│   └── index.php
├── src
│   ├── Controller
│   ├── Entity
│   ├── Kernel.php
│   ├── Migrations
│   └── Repository
├── symfony.lock
├── templates
│   └── base.html.twig
├── tests
├── translations
├── var
│   ├── cache
│   └── log
├── vendor
│   ├── autoload.php
│   ├── bin
│   ├── composer
│   ├── doctrine
│   ├── easycorp
│   ├── egulias
│   ├── fig
│   ├── jdorn
│   ├── monolog
│   ├── nikic
│   ├── ocramius
│   ├── phpdocumentor
│   ├── psr
│   ├── sensio
│   ├── swiftmailer
│   ├── symfony
│   ├── twig
│   ├── webmozart
│   └── zendframework
└── webpack.config.js

36 directories, 16 files
[maxdsre@CentOS axdlog]$
```

### Existing Symfony Project
對於已存在的Symfony項目，參考 [Installing an Existing Symfony Application](https://symfony.com/doc/current/setup.html#installing-an-existing-symfony-application) 進行相關操作。

### Running
```bash
# move into your new project and install the server
cd axdlog
composer require server --dev

# start the server
php bin/console server:run
```

操作過程如下

```bash
[maxdsre@CentOS axdlog]$ composer require server --dev
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Generating autoload files
ocramius/package-versions:  Generating version class...
ocramius/package-versions: ...done generating version class
Executing script cache:clear [OK]
Executing script assets:install --symlink --relative public [OK]

[maxdsre@CentOS axdlog]$ php bin/console server:run

 [OK] Server listening on http://127.0.0.1:8000                                                         

 // Quit the server with CONTROL-C.                                                                     

PHP 7.2.4 Development Server started at Thu Apr 12 21:55:37 2018
Listening on http://127.0.0.1:8000
Document root is /home/maxdsre/axdlog/public
Press Ctrl-C to quit.

```

本地監聽了`8000`端口，通過SSH端口轉發，將遠程主機的`8000`端口轉發到本地`8000`端口。

瀏覽器中輸入`127.0.0.1:8000/config.php` 可檢測Symfony的應用程序配置情況。


打開`127.0.0.1:8000`，顯示如下

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_21-59-00_welcome_info.png)

Symfony Profiler

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-00-10_profiler.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-00-29_profiler.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-00-57_profiler.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-01-18_profiler.png)


## Create First Page in Symfony
[Symfony][symfony]官方文檔 [Create your First Page in Symfony
](https://symfony.com/doc/current/page_creation.html)。

### Route and Controller
分2步驟

* Create a route
* Create a controller

進入目錄`axdlog`

在目錄`src/AppBundle/Controller`創建測試文件`LuckyController.php`，寫入如下信息

```
<?php
// src/Controller/LuckyController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;

class LuckyController
{
    public function number()
    {
        $number = mt_rand(0, 100);

        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }
}
```

在文件`config/routes.yaml`中配置URL路由地址

```yaml
# config/routes.yaml

# the "app_lucky_number" route name is not important yet
app_lucky_number:
    path: /lucky/number
    controller: App\Controller\LuckyController::number
```


瀏覽器中輸入 `127.0.0.1:8000/lucky/number`，出現如下信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-20-50_lucky_num.png)

###　Annotation Routes
URL路由地址可通過`config/routes.yaml`配置，也可通過`annotation`實現

```bash
# install annotations package
composer require annotations
```

將文件`LuckyController.php`內容修改如下

```
<?php
// src/Controller/LuckyController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    /**
    * @Route("/lucky_number")
    */
    public function number()
    {
        $number = mt_rand(0, 100);

        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }
}
```

註釋`config/routes.yaml`中的配置，瀏覽器中輸入 `127.0.0.1:8000/lucky_number`，出現如下信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-32-42_lucky_num.png)

### Rendering a Template
>If you're returning HTML from your controller, you'll probably want to render a template. Fortunately, Symfony comes with [Twig](http://twig.sensiolabs.org): a templating language that's easy, powerful and actually quite fun.

```bash
cd axdlog

# install Twig
composer require twig
```

使用函數`render()`渲染template(模板)，將文件`LuckyController.php`內容修改如下

```
<?php
// src/Controller/LuckyController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

// class LuckyController
class LuckyController extends Controller
{
    /**
    * @Route("/twig/lucky_number")
    */
    public function number()
    {
        $number = mt_rand(0, 100);

        return $this->render('lucky/number.html.twig', array(
            'number' => $number,
        ));
    }
}
```

模板文件存放在目錄`templates/`中，創建文件`templates/lucky/number.html.twig`，寫入如下內容

```twig
{# templates/lucky/number.html.twig #}

<h1>Your lucky number is {{ number }}</h1>
```

瀏覽器中輸入 `127.0.0.1:8000/twig/lucky_number`，出現如下信息

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-08-23_symfony_install_setup_centos7/2018-04-12_22-44-37_twig.png)

有關template的詳細說明參見官方文檔 [Creating and Using Templates](https://symfony.com/doc/master/templating.html)。


## Project Structure

directory | explanation
:--- | :---
`config/` | 配置文件存放路徑
`src/` | PHP代碼存放路徑
`templates/` | Twig模板文件
`bin/` | 可執行文件，如`bin/console`
`var/` | 緩存(`var/cache/`)和日誌(`var/logs/`)
`vendor/` | 第三方庫存放路徑
`public/` | 項目文檔路徑


## Web Server Configuration
[Symfony][symfony]官方文檔 [Configuring a Web Server
](https://symfony.com/doc/current/setup/web_server_configuration.html)。

此處暫不討論。


## Change Logs
* 2016.08.23 21:06 Tue Asia/Shanghai
    * 初稿完成
* 2018.04.12 22:57 Thu America/Boston
    * 內容更新，勘誤，遷移到新Blog


[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"
[nginx]: https://www.nginx.com "High Performance Load Balancer, Web Server, & Reverse Proxy"
[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."
[composer]: https://getcomposer.org "Dependency Manager for PHP"


<!-- End -->
