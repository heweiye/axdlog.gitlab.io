---
title: Using Doctrine DBAL To Work With Database In Symfony 4
slug: Using Doctrine DBAL To Work With Database In Symfony 4
date: 2016-08-24T04:38:08+08:00
lastmod: 2018-04-14T17:13:16-04:00
draft: false
keywords: ["Symfony", "PHP", 'Doctrine', 'DBAL']
description: ""
categories:
- PHP
tags:
- Symfony
- PHP
comment: true
toc: true

---

Web應用通常使用數據庫存儲數據，[**LEMP**](https://lemp.io/)中的 **M** 即指數據庫（默認爲 [MySQL][mysql]）。PHP框架[Symfony][symfony]是通過集成的第三方庫 [Doctrine][doctrine] 與數據庫交互（通過[Composer][composer]安裝）。[Doctrine][doctrine]支持2中形式 [Doctrine ORM](https://symfony.com/doc/master/doctrine.html) 和 [Doctrine DBAL][doctrinedbal]。`DBAL`是`Database Abstraction Layer`（數據庫抽象層）的縮寫，[Doctrine DBAL][doctrinedbal]支持原生的SQL語句。

本文記錄如何在[Symfony 4][symfony]中通過[Doctrine DBAL][doctrinedbal]進行數據庫的[CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete "wikipedia")操作。（與Symfony 3相比，變動很大）

<!--more-->

## Prerequisite
本人Blog [Installing And Setting Up The Symfony Framework On CentOS 7]({{< relref "2016-08-23-Installing-And-Setting-Up-The-Symfony-Framework-On-CentOS-7.md" >}}) 詳細記錄了[Symfony][symfony]、[Composer][composer]的安裝、配置過程。


LEMP環境信息

Info | Details
:--- | :---
OS Version | `CentOS Linux release 7.4.1708 (Core)`
Kernel Version | `3.10.0-693.21.1.el7.x86_64`
Nginx | `1.13.12`
MySQL | `5.7.21`
PHP | `7.2.4`
Composer| `1.6.4`


通過`annotation`配置URL路由規則

```bash
# install annotations package
composer require annotations

composer require symfony/var-dumper
# composer require debug
```


### Doctrine Installation
操作依據 [How to Use Doctrine DBAL](https://symfony.com/doc/master/doctrine/dbal.html)

通過如下命令安裝 [Doctrine][doctrine]

```bash
composer require doctrine/doctrine-bundle
```

操作過程

```bash
[maxdsre@CentOS axdlog]$ composer require doctrine/doctrine-bundle
Using version ^1.8 for doctrine/doctrine-bundle
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Generating autoload files
ocramius/package-versions:  Generating version class...
ocramius/package-versions: ...done generating version class
Executing script cache:clear [OK]
Executing script assets:install --symlink --relative public [OK]
Executing script security-checker security:check [OK]

[maxdsre@CentOS axdlog]$
```


## DBAL configuration

### Testing Database Info
創建測試帳號、數據庫，具體信息如下

Item | Detial
:--- | :---
Host | `127.0.0.1`
Port | `3306`
Username | `symfony`
Password | `Symfony@2018_dbal`
Database | `dbal`

SQL語句如下

```sql
-- create database
create database if not exists dbal;
-- create mysql user
create user 'symfony'@'%' identified by 'Symfony@2018_Dbal';
-- grant privileges
grant all on dbal.* to 'symfony'@'%';
-- refresh privileges
flush privileges;
```

操作過程

```sql
MySQL [(none)]> create database if not exists dbal;
Query OK, 1 row affected (0.00 sec)

MySQL [(none)]> create user 'symfony'@'%' identified by 'Symfony@2018_Dbal';
Query OK, 0 rows affected (0.01 sec)

MySQL [(none)]> grant all on dbal.* to 'symfony'@'%';
Query OK, 0 rows affected (0.00 sec)

MySQL [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MySQL [(none)]> show grants for 'symfony'@'%';
+---------------------------------------------------+
| Grants for symfony@%                              |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO 'symfony'@'%'               |
| GRANT ALL PRIVILEGES ON `dbal`.* TO 'symfony'@'%' |
+---------------------------------------------------+
2 rows in set (0.00 sec)

MySQL [(none)]>
```

### DBAL configuration
配置數據庫有2種方式：

* 項目目錄下的文件`.env`
* 項目目錄下的文件`config/packages/doctrine.yaml`

按照文檔 [Installing & Setting up the Symfony Framework](https://symfony.com/doc/current/setup.html) 創建項目後(此處項目名`axdlog`)，目錄中生成一個名爲`.env`的隱藏文件，配置指令是`DATABASE_URL`。

```bash
###> doctrine/doctrine-bundle ###
# Format described at http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html#connecting-using-a-url
# For an SQLite database, use: "sqlite:///%kernel.project_dir%/var/data.db"
# Configure your db driver and server_version in config/packages/doctrine.yaml
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
###< doctrine/doctrine-bundle ###
```

此處應爲

```bash
DATABASE_URL=mysql://symfony:Symfony@2018_Dbal@127.0.0.1:3306/dbal
```

但這種配置方式有一個問題，如果用戶密碼含有`:`、`@`等特殊字符，解析時可能會出錯。

通過文件`doctrine.yaml`配置更爲妥當，具體配置參照官方文檔 [DoctrineBundle Configuration ("doctrine")](https://symfony.com/doc/master/reference/configuration/doctrine.html#doctrine-dbal-configuration)。


配置信息如下

```yml
parameters:
    # Adds a fallback DATABASE_URL if the env var is not set.
    # This allows you to run cache:warmup even if your
    # environment variables are not available yet.
    # You should not need to change this value.
    env(DATABASE_URL): ''

doctrine:
    dbal:
        dbname: dbal
        host: 127.0.0.1
        port: 3306
        user: 'symfony'
        password: 'Symfony@2018_Dbal'
        # configure these for your database server
        driver: 'pdo_mysql'
        server_version: '5.7'
        charset: utf8mb4
        default_table_options:
            charset: utf8mb4
            collate: utf8mb4_unicode_ci

        # With Symfony 3.3, remove the `resolve:` prefix
        url: '%env(resolve:DATABASE_URL)%'
    orm:
        auto_generate_proxy_classes: '%kernel.debug%'
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true
        mappings:
            App:
                is_bundle: false
                type: annotation
                dir: '%kernel.project_dir%/src/Entity'
                prefix: 'App\Entity'
                alias: App
```

```php
// <?php
// src/Controller/DatabaseController.php
namespace App\Controller;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
// 數據庫連接驅動
use Doctrine\DBAL\Driver\Connection;

// class name is same to file name
class DatabaseController
{
    /**
    * @Route("/dbal")
    */
    public function indexAction(Connection $connection)
    {
        // $users = $connection->fetchAll('SELECT * FROM users');

        $result = $connection->fetchAll('SELECT id,name,region FROM province');
        var_dump($result);
        // return new Response(dump($result));
        // Error: The Response content must be a string or object implementing __toString(), "array" given.

    }
}
```

在項目所在目錄中執行
```php
php bin/console server:run
```
在瀏覽器URL中輸入對應的URL地址即可看到相關輸出信息。


## CRUD
以下操作命令參考自Doctrine官方文檔 [4. Data Retrieval And Manipulation](https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/data-retrieval-and-manipulation.html)。

**說明**:`return new Response()`只能返回字符串，無法返回數組，故而使用`var_dump()`打印。

數據庫增刪改查，以測試數據庫`dbal`中的`province`數據表為例:

插入測試數據


```sql
DROP TABLE IF EXISTS `province`;

CREATE TABLE `province` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '省級列表自增id',
  `name` char(24) NOT NULL COMMENT '省份名稱',
  `region` enum('華北','東北','華東','中南','西南','西北') NOT NULL COMMENT '省所屬區域:1華北、2東北、3華東、4中南(華中,華南)、5西南、6西北',
  PRIMARY KEY (`id`),
  KEY `ind_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='中國大陸省份列表';

LOCK TABLES `province` WRITE;
INSERT INTO `province` VALUES (1,'北京市','華北'),(2,'天津市','華北'),(3,'河北省','華北'),(4,'山西省','華北'),(5,'内蒙古自治区','華北'),(6,'辽宁省','東北'),(7,'吉林省','東北'),(8,'黑龙江省','東北'),(9,'上海市','華東'),(10,'江苏省','華東'),(11,'浙江省','華東'),(12,'安徽省','華東'),(13,'福建省','華東'),(14,'江西省','華東'),(15,'山东省','華東'),(16,'河南省','中南'),(17,'湖北省','中南'),(18,'湖南省','中南'),(19,'广东省','中南'),(20,'广西壮族自治区','中南'),(21,'海南省','中南'),(22,'重庆市','西南'),(23,'四川省','西南'),(24,'贵州省','西南'),(25,'云南省','西南'),(26,'西藏自治区','西南'),(27,'陕西省','西北'),(28,'甘肃省','西北'),(29,'青海省','西北'),(30,'宁夏回族自治区','西北'),(31,'新疆维吾尔自治区','西北');
UNLOCK TABLES;
```

```sql
MySQL [dbal]> select * from province;
+----+--------------------------+--------+
| id | name                     | region |
+----+--------------------------+--------+
|  1 | 北京市                   | 華北   |
|  2 | 天津市                   | 華北   |
|  3 | 河北省                   | 華北   |
|  4 | 山西省                   | 華北   |
|  5 | 内蒙古自治区             | 華北   |
|  6 | 辽宁省                   | 東北   |
|  7 | 吉林省                   | 東北   |
|  8 | 黑龙江省                 | 東北   |
|  9 | 上海市                   | 華東   |
| 10 | 江苏省                   | 華東   |
| 11 | 浙江省                   | 華東   |
| 12 | 安徽省                   | 華東   |
| 13 | 福建省                   | 華東   |
| 14 | 江西省                   | 華東   |
| 15 | 山东省                   | 華東   |
| 16 | 河南省                   | 中南   |
| 17 | 湖北省                   | 中南   |
| 18 | 湖南省                   | 中南   |
| 19 | 广东省                   | 中南   |
| 20 | 广西壮族自治区           | 中南   |
| 21 | 海南省                   | 中南   |
| 22 | 重庆市                   | 西南   |
| 23 | 四川省                   | 西南   |
| 24 | 贵州省                   | 西南   |
| 25 | 云南省                   | 西南   |
| 26 | 西藏自治区               | 西南   |
| 27 | 陕西省                   | 西北   |
| 28 | 甘肃省                   | 西北   |
| 29 | 青海省                   | 西北   |
| 30 | 宁夏回族自治区           | 西北   |
| 31 | 新疆维吾尔自治区         | 西北   |
+----+--------------------------+--------+
31 rows in set (0.00 sec)

MySQL [dbal]>
```

### Retrieve Data
#### fetchColumn
>`fetchColumn($column)` - Retrieves only one column of the next row specified by column index. Moves the pointer forward one row, so that consecutive calls will always return the next row.

```php
public function indexAction(Connection $connection)
{
    $result = $connection->fetchColumn('SELECT name FROM dbal.province order by id');
    return new Response(dump($result));
    #var_dump($result);
}
```

#### fetchArray
>`fetchArray()`: Numeric index retrieval of first result row of the given query:

```php
public function indexAction(Connection $connection)
{
    $result = $connection->fetchArray('SELECT id,name,region FROM dbal.province where region=1 order by id');
    var_dump($result);
}
```

```
array(3) { [0]=> string(1) "1" [1]=> string(9) "北京市" [2]=> string(6) "華北" }
```

#### fetchAssoc
>`fetchAssoc()`: Retrieve assoc row of the first result row.

```php
public function indexAction(Connection $connection)
{
    $result = $connection->fetchAssoc('SELECT id,name,region FROM dbal.province where region=1 order by id');
    var_dump($result);
}
```

```
array(3) { ["id"]=> string(1) "1" ["name"]=> string(9) "北京市" ["region"]=> string(6) "華北" }
```

#### fetchAll
>`fetchAll($fetchStyle)` - Retrieves all rows from the statement.

```php
public function indexAction(Connection $connection)
{
   $result = $connection->fetchAll('SELECT id,name,region FROM dbal.province order by id where region=1');
   var_dump($result);
}
```

```
array(5) { [0]=> array(3) { ["id"]=> string(1) "1" ["name"]=> string(9) "北京市" ["region"]=> string(6) "華北" } [1]=> array(3) { ["id"]=> string(1) "2" ["name"]=> string(9) "天津市" ["region"]=> string(6) "華北" } [2]=> array(3) { ["id"]=> string(1) "3" ["name"]=> string(9) "河北省" ["region"]=> string(6) "華北" } [3]=> array(3) { ["id"]=> string(1) "4" ["name"]=> string(9) "山西省" ["region"]=> string(6) "華北" } [4]=> array(3) { ["id"]=> string(1) "5" ["name"]=> string(18) "内蒙古自治区" ["region"]=> string(6) "華北" } }
```

### Update Data
>`executeUpdate($sql, $params, $types)` - Create a prepared statement for the passed SQL query, bind the given params with their binding types and execute the query. This method returns the number of affected rows by the executed query and is useful for `UPDATE`, `DELETE` and `INSERT` statements.

Executes a prepared statement with the given SQL and parameters and returns the affected rows count

```php
public function indexAction(Connection $connection)
{
    $sql = "update dbal.province set name='Testing' where id=1";
    $update = $connection->executeUpdate($sql);
    $result = $connection->fetchAssoc('SELECT id,name,region FROM dbal.province where id=1');
    var_dump($result);
}
```

```
array(3) { ["id"]=> string(1) "1" ["name"]=> string(7) "Testing" ["region"]=> string(6) "華北" }
```

### Insert Data
>insert(): Insert a row into the given table name using the key value pairs of data.

```php
public function indexAction(Connection $connection)
{
    $insert = $connection->insert('dbal.province', array('name' => 'DBAL', 'region' => 1));
    // $conn->insert('user', array('username' => 'jwage'));
    // INSERT INTO user (username) VALUES (?) (jwage)

    $result = $connection->fetchAssoc('SELECT id,name,region FROM dbal.province order by id desc limit 1');
   var_dump($result);
}
```

```
array(3) { ["id"]=> string(2) "32" ["name"]=> string(4) "DBAL" ["region"]=> string(6) "華北" }
```

### Delete
>`delete()`: Delete all rows of a table matching the given identifier, where keys are column names.

```php
public function indexAction(Connection $connection)
{
    $delete = $connection->delete('dbal.province', array('id' => 1));
    // $conn->delete('user', array('id' => 1));
    // DELETE FROM user WHERE id = ? (1)

    $result = $connection->fetchAll('SELECT id,name FROM dbal.province where region=1 order by id');
    var_dump($result);
}
```

```
array(5) { [0]=> array(2) { ["id"]=> string(1) "2" ["name"]=> string(9) "天津市" } [1]=> array(2) { ["id"]=> string(1) "3" ["name"]=> string(9) "河北省" } [2]=> array(2) { ["id"]=> string(1) "4" ["name"]=> string(9) "山西省" } [3]=> array(2) { ["id"]=> string(1) "5" ["name"]=> string(18) "内蒙古自治区" } [4]=> array(2) { ["id"]=> string(2) "32" ["name"]=> string(4) "DBAL" } }
```


## References
* [Databases and the Doctrine ORM](https://symfony.com/doc/master/doctrine.html)
* [How to Use Doctrine DBAL](https://symfony.com/doc/master/doctrine/dbal.html)
* [The VarDumper Component](https://symfony.com/doc/current/components/var_dumper.html)
* [Data Retrieval And Manipulation](https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/data-retrieval-and-manipulation.html)


## Change Logs
* 2016.08.24 12:39 Wed Aisa/Shanghai
    * 初稿完成
* 2018.04.14 19:13 Sat America/Boston
    * 更新、勘誤，遷移到新Blog


[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"
[doctrine]: https://www.doctrine-project.org "The Open-Source PHP ORM and Persistence Tools Project"
[doctrinedbal]: https://www.doctrine-project.org/projects/dbal.html "Powerful PHP database abstraction layer (DBAL) with many features for database schema introspection, schema management and PDO abstraction."
[mysql]: https://www.mysql.com "MySQL is the world's most popular open source database."
[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."
[composer]: https://getcomposer.org "Dependency Manager for PHP"


<!-- End -->
