---
title: Full Recording Of Symfony Installation On CentOS 7
date: 2016-08-23T21:03:16+08:00
lastmod: 2018-04-12T11:20:02-04:00
draft: true
keywords: ["Symfony", "PHP"]
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


[Symfony][symfony]是一款流行度很高的[PHP][php]開發框架，本文記錄如何在`CentOS 7`中安裝、配置該框架。

<!--more-->

## Preparation

### Development Environment
開發環境信息

item|content
---|---
OS|`CentOS Linux release 7.2.1511 (Core)`
Kernel|`3.10.0-327.28.3.el7.x86_64`
Composer|`1.2.0`
PHP|`5.6.24`
Nginx|`1.10.1`
MariaDB|`10.1.16`

`LEMP`開發環境的安裝、配置可參考本人之前Blog [LEMP Installation and Nginx Optimization](https://lempstacker.github.io/tw/lemp-installation-and-nginx-optimization/)

Symfony對PHP環境的要求參見官方文檔 [Requirements for Running Symfony](http://symfony.com/doc/master/reference/requirements.html)。

如何在Web Server中配置Symfony項目，參見官方文檔 [Configuring a Web Server](http://symfony.com/doc/current/setup/web_server_configuration.html)。

Symfony 4.0 requires PHP 7.1.3 or higher to run, in addition to other minor requirements.

### Symfony Installer Installation

>`The Symfony Installer` is a small PHP application that must be installed once in your computer. It greatly simplifies the creation of new projects based on the Symfony framework.

執行如下命令進行`Symfony Installer`的安裝

```sh
sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony

sudo chmod a+x /usr/local/bin/symfony
```

如果無法進行以上操作，可參考[Symfony installation based on Composer](https://symfony.com/doc/master/setup.html#creating-symfony-applications-without-the-installer)安裝`composer`，建議安裝。

#### Composer Installation
以下操作過程參考自Composer[官方文檔](https://getcomposer.org/download/)。

```sh
#Download the installer to the current directory

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

#Verify the installer SHA-384
php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

#Run the installer
php composer-setup.php

#Remove the installer
php -r "unlink('composer-setup.php');"

#move composer.phar to executed path
mv composer.phar /usr/local/bin/composer
```

## Project Creation
在目錄`～/symfony`中執行如下命令，創建名爲`arsenal`的project

```sh
symfony new arsenal
```

安裝完成後出現提示如下信息

```
Preparing project...

Symfony 3.1.3 was successfully installed. Now you can:

* Change your current directory to `/home/flying/symfony/arsenal`

* Configure your application in `app/config/parameters.yml` file.

* Run your application:
   1. Execute the `php bin/console server:run` command.
   2. Browse to the <http://localhost:8000> URL.

* Read the documentation at http://symfony.com/doc
```

在項目所在目錄中執行命令
```php
php bin/console server:run
```
如果提示

```txt
[OK] Server running on http://127.0.0.1:8000

// Quit the server with CONTROL-C.
```
即說明symfony啓動成功，可在瀏覽器URL中輸入
```http
http://127.0.0.1:8000
```
查看。

**注意**： Symfony依賴PHP函數`proc_open`、`proc_get_status`，需確保此二者 **未添加** 進php配置文件`/etc/php.ini`的`disable_functions`參數中。否則會報錯，具體見 *Error 1* 部分。

在瀏覽器URL中輸入
```http
http://localhost:8000/config.php
```
可檢測Symfony的應用程序配置情況。

也可根據[Requirements for Running Symfony](http://symfony.com/doc/master/reference/requirements.html)的說明執行如下命令

```sh
php bin/symfony_requirements
```
查看當前系統的PHP環境是否滿足Symfony的要求，

### Installing an Existing Symfony Application
如果是安裝已經存在的Symfony項目，可參考 [Installing an Existing Symfony Application](http://symfony.com/doc/current/setup.html#installing-an-existing-symfony-application) 進行相關操作。


## Create First Page in Symfony
### Route and Controller
分2步驟
* Create a route
* Create a controller

在目錄`src/AppBundle/Controller`創建測試文件`LuckyController.php`，寫入如下信息

```php
<?php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Response;

// 注意控制器的名字與class後的名字必須一致
class LuckyController
{
    //  此處的“/lucky/numbers”即route
    /**
     * @Route("/lucky/numbers")
     */

    // 此處的numberAction即controller
    public function numAction()
    {
        $number = mt_rand(0, 100);

        // 使用的是引用的Symfony\Component\HttpFoundation\Response
        return new Response(
            '<html><body>Lucky number: '.$number.'</body></html>'
        );
    }
}
```

在瀏覽器URL中輸入
```http
http://localhost:8000/app_dev.php/lucky/numbers
```
或
```http
http://localhost:8000/lucky/numbers
```
即可看到頁面信息。

代碼中`@Route("/lucky/numbers")`的作用是指定route，即URL後的`/lucky/numbers`


### Rendering a Template

>make sure that `LuckyController` extends Symfony's base `Controller` class:

```php
// src/AppBundle/Controller/LuckyController.php

// ...
// --> add this new use statement
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class LuckyController extends Controller
{
    // ...
}
```

使用函數`render()`提供template(模板)

```php
// src/AppBundle/Controller/LuckyController.php

// ...
class LuckyController extends Controller
{
    /**
     * @Route("/lucky/numbers")
     */
    public function numberAction()
    {
        $number = mt_rand(0, 100);

        return $this->render('lucky/number.html.twig', array(
            'number' => $number
        ));
    }
}
```

在目錄`app/Resources/view`下創建目錄`lucky`，同時創建文件`number.html.twig`。

Symfony中如何使用template參見官方文檔 [Creating and Using Templates](http://symfony.com/doc/master/templating.html)。


## Project Structure

* **`app/`**: 包含配置文件(configuration)和模板(templates)，此處的都是非PHP代碼
* **`src/`**: PHP文件和代碼，日常操作最頻繁的
* `bin/`: 可執行文件，如`bin/console`
* `tests/`: 自動測試？The automated tests (e.g. Unit tests) for your application live here.
* `var/`: 自動生成文件如緩存(var/cache/)和日誌(var/logs/)
* `vendor/`: 第三方庫，通過composer包管理器下載的第三方庫
* `web/`: project的文檔根目錄，存放公開的可訪問的文件(publicly accessible file)，如CSS,js,images等。


## Error Occurred
### Error 1
在項目所在目錄中運行`php bin/console server:run`時報錯

```php
[Symfony\Component\Process\Exception\RuntimeException]

The Process class relies on proc_open, which is not available on your PHP installation.
```

查看PHP配置文件`/etc/php.ini`，參數`disable_functions`中禁用了`proc_open`，將其移除，執行如下命令重啓PHP-FPM服務

```sh
sudo systemctl restart php-fpm
```

再次執行，報錯

```php
[Symfony\Component\Debug\Exception\ContextErrorException]

Warning: proc_get_status() has been disabled for security reasons
```

將`proc_get_status`從`disable_functions`中移除，重啓php-fpm服務後再次執行正常

```
[OK] Server running on http://127.0.0.1:8000
```

### Error 2
在瀏覽器URL中輸入`http://127.0.0.1:8000/config.php`，出現如下提示

```
This script analyzes your system to check whether is ready to run Symfony applications.

To enhance your Symfony experience, it’s recommended that you fix the following:

1. posix_isatty() should be available
Install and enable the `php_posix` extension (used to colorize the CLI output).

2. intl extension should be available
Install and enable the `intl` extension (used for validators).
```

执行命令`php bin/symfony_requirements`可得到相关提示信息。

對於`php_posix`，參考[how to install posix in php](http://stackoverflow.com/questions/2197366/how-to-install-posix-in-php#answer-2197780)，需安裝`php-process`；對於`intl`，安裝`php-intl`。

執行如下命令安裝
```sh
sudo yum install -y --enablerepo=remi-php56 php-process php-intl

php56-php-intl php56-php-process environment-modules php56-php-common php56-php-pecl-jsonc php56-php-pecl-zip php56-runtime tcl
```
無效，仍有提示 *RECOMMENDATIONS*


## References
* [how to install posix in php](http://stackoverflow.com/questions/2197366/how-to-install-posix-in-php#answer-2197780)
* [Routing](http://symfony.com/doc/master/routing.html)
* [Controller](http://symfony.com/doc/master/controller.html)
* [Creating and Using Templates](http://symfony.com/doc/master/templating.html)
* [Service Container](http://symfony.com/doc/master/service_container.html)


## Change Logs
* 2016.08.23 21:06 Tue Asia/Shanghai
    * 初稿完成


[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."
[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"

---
* Note Time: 2016.08.23 21:06 Tue
* Note Location: Asia/Shanghai
* Writer: lempstacker
