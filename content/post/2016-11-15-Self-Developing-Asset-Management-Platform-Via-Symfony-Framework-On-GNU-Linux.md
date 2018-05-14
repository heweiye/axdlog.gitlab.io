---
title: Self-Developing Asset Management Platform Based On Symfony Framework
slug: Self-Developing Asset Management Platform Based On Symfony Framework
date: 2016-11-15T17:54:13+08:00
lastmod: 2018-04-16T20:55:08-04:00
draft: false
keywords: ["AxdLog", "Symfony", "PHP", 'Nginx']
description: "Self-Developing Asset Management Platform Based On Symfony Framework"
categories:
- Production Case
- PHP
tags:
- PHP
- Nginx
- Symfony
comment: true
toc: true

---

根據公司直屬領導指示，進行 **資產管理系統平臺** 的開發，整個項目由本人獨立完成，項目週期2個月左右，現已部署上線，可通過公司內網訪問。

因本人此前從事[PHP][php]開發，故選擇[PHP][php]做爲開發語言，開發框架選擇[Symfony][symfony]。如何部署Symfony項目，詳見本人 [blog]({{< relref "2016-11-15-Configuring-Nginx-Web-Server-For-Symfony-Framework-On-GNU-Linux.md" >}})。

<!--more-->

**說明**：此項目爲2016年爲公司運維部門做的內部項目，使用[Symfony 3][symfony]開發。因是公司項目，故沒有單獨留存項目代碼。該項目上線後並未得到公司的重視，幾乎沒有使用。運行數據庫、項目代碼的服務器後被騰出給公司其它項目用。之後公司內部的Gitlab奔潰過一次，導致所有數據丟失，包括項目代碼，不得不說是一個遺憾。此次遷移只做勘誤（2018年04月16日）。


## SQL Design
數據表設計

創建數據庫

```sql
drop database if exists arsenal;

-- 創建數據庫
create database if not exists arsenal
    default character set=utf8
    default collate=utf8_general_ci;

-- 進入數據庫
use arsenal;
```

創建數據表

```sql
-- 存儲用戶session信息
-- http://symfony.com/doc/current/doctrine/pdo_session_storage.html#preparing-the-database-to-store-sessions
create table if not exists sessions (
    sess_id varbinary(128) not null primary key comment 'session id',
    sess_data blob not null comment 'session data (dynamic clomumn)json 格式',
    sess_time int unsigned not null comment 'session create or update time (timestamp)',
    sess_lifetime mediumint unsigned not null comment 'session life time，單位s，默認1440s即24小時'
) engine=innodb default charset=utf8 collate=utf8_bin comment '存儲用戶session信息';

-- 系統模塊表
create table if not exists sys_module (
    id smallint unsigned not null auto_increment primary key comment '系統模塊表自增id',
    name char(60) not null comment '模塊名稱',
    icon varchar(30) default null comment 'Font Awesome Icons圖標名稱，如fa-archive',
    route_name varchar(60) default null comment '該模塊的route名稱',
    action_name varchar(60) default null comment '該模塊的action名稱',
    level enum('top','second','third','fourth','fifth','sixth','seventh','eighth') not null default 'top' comment '模塊等級,默認為top(一級)模塊，即模組,後一級為前一級的子模塊',
    parent_module smallint unsigned default 0 comment '模塊所屬的上一級模塊，對應表sys_module，默認為0表示一級模塊，即模組，不設置外鍵約束',
    is_show enum('yes','no') not null default 'yes' comment '是否顯示該模塊,默認yes顯示',
    remarks varchar(180) default null comment '該模塊備註信息',
    adder mediumint unsigned not null comment '該item的添加者,對應表sys_user中的id，不設置外鍵約束',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，對應表sys_user中id，不設置外鍵約束',
    unique key `module_nameLevel` (`level`,`name`) comment 'index-模塊名稱,使用唯一索引',
    key `module_partent_module` (`parent_module`) comment 'index-上一級模塊索引'
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '系統模塊表';

-- 創建系統用戶角色表
create table if not exists sys_user_role (
    id smallint unsigned not null auto_increment primary key comment '用戶角色自增id',
    name char(60) not null comment '用戶角色名稱',
    is_allow enum('yes','no') not null default 'yes' comment '是否允許登入系統，默認為yes允許',
    sys_module_list text default null comment '用戶角色所屬的各模塊id，對應表sys_module中id，以json格式存儲',
    remarks varchar(180) default null comment '該模塊備註信息',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，對應表sys_user中id，不設置外鍵約束',
    unique key `sys_user_name` (`name`) comment 'index-用戶權限名稱,使用唯一索引'
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '系統用戶角色表(用戶組)';

-- 創建系統用戶表
create table if not exists sys_user (
    id mediumint unsigned not null auto_increment primary key comment '系統用戶自增id',
    username char(40) not null comment '用戶登陸名稱',
    sys_user_role smallint unsigned not null comment '用戶角色,表sys_user_role中id為其外鍵',
    password varchar(48) not null comment '用戶登陸密碼,md5加密後長度32位,sha1加密後長度40位',
    initial_password varchar(48) not null comment '用戶初始密碼',
    email varchar(60) null comment '用戶郵箱地址',
    gender enum ('male','female','unknown') not null default 'unknown' comment '用戶性別,默認unknown',
    is_allow enum ('yes','no') not null default 'yes' comment '用戶是否允許登陸,默認允許',
    remarks varchar(300) not null comment '備註信息',
    employee mediumint unsigned default null comment '該帳號所屬的公司職員，對應表employee中id，默認為空，不設置外鍵約束',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，對應表sys_user中id，不設置外鍵約束',
    key `sys_user_employee` (`employee`) comment 'index-僱員名稱，使用普通索引',
    unique key `sys_user_username` (`username`) comment 'index-用戶名稱,使用唯一索引',
    constraint `fk_user_sys_userRole` foreign key (`sys_user_role`) references `sys_user_role` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '系統用戶表';

-- 系統用戶登陸記錄表
create table if not exists sys_user_login_log (
    id int unsigned not null auto_increment primary key comment '系統用戶登陸記錄自增id',
    sys_user mediumint unsigned not null comment '登陸用戶的id,表sys_user中id為其外鍵',
    login_time timestamp not null default current_timestamp comment '登入系統時間,格式 YYYY-MM-DD HH:MM:SS',
    logout_time timestamp null default null comment '登出系統時間,格式 YYYY-MM-DD HH:MM:SS,默認為NULL',
    login_details varchar(300) default null comment '用戶登陸信息,如瀏覽器信息等',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    constraint `fk_sys_user_login_log_sysUser` foreign key (`sys_user`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '系統用戶登陸記錄表';

-- 創建子公司表
create table if not exists company (
    id smallint unsigned not null auto_increment primary key comment '公司自增id',
    parent_company smallint unsigned not null default 0 comment '上一級公司id，對應表company中id，默認為0代表總公司，不設置外鍵約束',
    is_exists enum('yes','no') not null default 'yes' comment '公司是否存在',
    name char(90) not null comment '公司名稱',
    short_name varchar(60) default null comment '公司簡稱',
    internal_name varchar(60) default null comment '公司內部命名，如xx事業部',
    type varchar(150) not null comment '公司類型',
    address_registed varchar(210) not null comment '公司註冊地址',
    address varchar(210) not null comment '公司目前地址',
    legal_representative varchar(60) not null comment '公司法定代表人',
    founding_date  date not null comment '公司成立時間,格式 YYYY-MM-DD',
    registration_institution varchar(150) not null comment '登記機關',
    business_license_number varchar(50) not null comment '營業執照證照編號(20位)',
    registration_no varchar(50) default null comment '營業執照註冊號(15位)',
    unified_social_credit_code varchar(50) null comment '統一社會信用代碼(18位)',
    business_term_begin date not null comment '營業期限起始時間,格式 YYYY-MM-DD',
    business_term_end date default null comment '營業期限截止時間,格式 YYYY-MM-DD',
    website varchar(180) default null comment '公司網站',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    unique key `company_name` (`name`) comment 'index-子公司名稱,使用唯一索引',
    key `company_parent_company` (`parent_company`) comment 'index-母公司id索引',
    constraint `fk_company_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '子公司表';

-- 公司部門表
create table if not exists department (
    id smallint unsigned not null auto_increment primary key comment '公司部門表自增id',
    name char(60) not null comment '部門名稱',
    name_old varchar(60) null comment '部門之前所用的名稱',
    company smallint unsigned not null comment '部門所屬公司,表company中id為其外鍵',
    level enum('top','second','third','fourth','fifth','sixth','seventh','eighth') not null default 'top' comment '部門等級,默認為top(一級)部門,後一級為前一級的子部門',
    parent_department smallint unsigned default 0 comment '部門所屬上級部門，對應表department中id，默認為0表示一級部門，不設置外鍵約束',
    instruction text default null comment '部門情況簡介',
    founding_date date default null comment '部門成立時間,格式 YYYY-MM-DD',
    is_exist enum('yes','no') not null default 'yes' comment '部門是否仍存在,未被撤銷,默認存在',
    closed_date date default null comment '部門撤銷時間,格式 YYYY-MM-DD',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    unique key `department_name` (`name`,`company`) comment 'index-公司部門名稱與所屬公司二者確定一條item',
    key `department_parent_department` (`parent_department`) comment 'index-上一級部門id索引',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    constraint `fk_department_company` foreign key (`company`) references `company` (`id`) on update cascade,
    constraint `fk_department_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_department_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment 'raxtone公司部門表';

-- 公司部門負責人表
create table if not exists department_director (
    id mediumint unsigned not null auto_increment primary key comment '部門負責人表自增id',
    department smallint unsigned not null comment '部門id，表department中id為其外鍵',
    director mediumint unsigned null comment '負責人，對應表employee中id，不設置外鍵約束',
    remarks text default null comment '部門負責人信息備註',
    is_part_time enum('no','yes') not null default 'no' comment '是否是兼任，默認為no，不是兼任',
    start_date date default null comment '任職起始時間,格式 YYYY-MM-DD',
    end_date date default null comment '任職結束時間,格式 YYYY-MM-DD',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    key `department_director_director` (`director`) comment 'index-部門負責人id',
    constraint `fk_department_director_department` foreign key (`department`) references `department` (`id`) on update cascade,
    constraint `fk_department_director_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_department_director_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment 'raxtone公司部門負責人表';

-- 公司僱員表
create table if not exists employee (
    id mediumint unsigned not null auto_increment primary key comment '公司僱員自增id',
    employee_no char(20) not null comment '員工編號',
    name char(60) not null comment '僱員姓名',
    job_title varchar(90) not null comment '僱員職位名稱',
    gender enum ('male','female') not null comment '僱員性別',
    birth date default null comment '僱員出生時間,格式 YYYY-MM-DD',
    department smallint unsigned not null comment '僱員所屬部門,表department中id為其外鍵',
    identity_no char(30) default null comment '僱員身份證號碼',
    personal_email varchar(80) default null comment '僱員個人郵箱地址',
    mobile_phone_no varchar(20) default null comment '僱員個人移動電話號碼',
    hiredate date not null comment '僱員入職時間,格式 YYYY-MM-DD',
    leavedate date default null comment '僱員離職時間,格式 YYYY-MM-DD',
    is_leave enum('no','yes') not null default 'no' comment '是否離職，默認為no未離職',
    remarks varchar(300) default null comment '僱員信息備註',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    unique key `employee_name` (`name`,`employee_no`) comment 'index-公司僱員姓名、員工編號,二者同時確定一條item',
    constraint `fk_employee_department` foreign key (`department`) references `department` (`id`) on update cascade,
    constraint `fk_employee_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_employee_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '公司僱員表';

-- 服務器參數信息表
-- http://stackoverflow.com/questions/23087105/mariadb-dynamic-columns-json
-- 品牌 server_brand
-- 具體型號 server_brand_version
-- 產品類別 機架式 server_product_catagory
-- 產品結構 2U server_product_mix
-- CPU型號 Xeon E5-2630 v2 server_cpu_model
-- CPU頻率 server_cpu_frequency
-- 標配CPU數量 2顆 standart_cpu_no
-- 最大CPU數量 max_cpu_no
-- 主板芯片組  motherboard_chipset_version
-- 主板擴展槽 motherboard_expansion_slot
-- 內存類型 memory_tpye
-- 內存品牌 memory_brand
-- 內存容量 memory_capacity
-- 內存插槽數量 memory_slot_no
-- 硬盤接口類型 hard_disk_interface_type
-- 標配硬盤容量 hard_disk_capacity
-- 硬盤架數 hard_disk_tray_no
-- 硬盤槽位 hard_disk_slot
-- 網絡控制器個數 network_controller_no
-- 網絡控制器速度 network_controller_speed
-- 購買日期 buy_date
-- 保修期限 warranty_period

-- server_room根據實際情況設置
create table if not exists ops_server (
    id mediumint unsigned not null auto_increment primary key comment '服務器自增id',
    details blob comment '服務器參數配置信息,如品牌信號、主板芯片組、CPU、硬盤、內存、網卡等相關信息',
    server_room enum ('地點1','地點2','地點3') not null default '地點1' comment '服務器所在機房,默認為空',
    server_cabinet varchar(30) not null comment '所屬機櫃的名稱',
    server_cabinet_order tinyint unsigned not null comment '服務器在機櫃中的位置序號',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    key `server_serverRoom` (`server_room`) comment 'index-服務器所在機房',
    constraint `fk_ops_server_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_ops_server_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '服務器表';

-- 服務器變更記錄表
create table if not exists ops_server_change_record (
    id int unsigned not null auto_increment primary key comment '服務器變更記錄自增id',
    server mediumint unsigned not null comment '服務器id,表server中id為其外鍵',
    operator mediumint unsigned not null comment '對服務器進行操作，操作者id,表employee中id為其外鍵',
    operate_time_begin timestamp not null comment '操作起始時間',
    operate_time_end timestamp null comment '操作結束時間',
    description text not null comment '操作細節簡述',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    constraint `fk_ops_server_change_record_server` foreign key (`server`) references `ops_server` (`id`) on update cascade,
    constraint `fk_ops_server_change_record_operator` foreign key (`operator`) references `employee` (`id`) on update cascade,
    constraint `fk_ops_server_change_record_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '服務器信息變更記錄表';

-- 域名信息表
create table if not exists ops_domain (
    id smallint unsigned not null auto_increment primary key comment '域名錶自增id',
    website char(90) not null comment '域名網址',
    name varchar(120) not null comment '域名名稱',
    company smallint unsigned not null comment '域名所屬的公司,表company中id為其外鍵',
    responsible_person mediumint unsigned not null comment '域名備案負責人,表employee中id為其外鍵',
    domain_name_registrar blob not null comment '域名註冊商,含名稱、地址、網址、聯繫電話等信息',
    internet_service_provider varchar(90) not null comment '網絡接入服務商',
    ip_addr varchar(130) not null comment 'IP地址,IPV4或IPV6',
    record_num varchar(90) not null comment '備案號',
    audit_time date not null comment '備案審核時間',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    key `domain_website` (`website`) comment 'index-域名網站索引',
    constraint `fk_domain_company` foreign key (`company`) references `company` (`id`) on update cascade,
    constraint `fk_domain_responsiblePerson` foreign key (`responsible_person`) references `employee` (`id`) on update cascade,
    constraint `fk_domain_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_domain_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '公司域名錶';

-- 公司資產分類
create table if not exists asset_category (
    id smallint unsigned not null auto_increment primary key comment '資產分類自增id',
    name char(60) not null comment '資產分類名稱',
    level enum('top','second','third','fourth','fifth','sixth','seventh','eighth') not null default 'top' comment '資產分類等級,默認為top頂級(一級)分類,後一級為前一級的子分類',
    parent_category smallint unsigned default 0 comment '該分類的父級分類,頂級分類為0,本表的id，不設置外鍵約束',
    is_show enum('yes','no') default 'yes' comment '是否顯示,默認為yes顯示',
    remarks text default null comment '該分類備註信息',
    adder mediumint unsigned not null comment '分類的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    unique key `asset_category_name` (`level`,`name`) comment 'index-資產分類名稱，使用唯一索引，name和level共同確定一條item',
    key `asset_parent_category` (`parent_category`) comment 'index-上一級分類id索引',
    constraint `fk_asset_category_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_asset_category_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '公司資產分類表,含多個等級的分類';

-- 公司資產清單
create table if not exists asset_inventory (
    id mediumint unsigned not null auto_increment primary key comment '資產清單自增id',
    asset_category smallint unsigned not null comment '資產所屬分類,表asset_category中id為其外鍵',
    asset_no char(50) default null comment '資產編號(唯一)',
    name char(150) not null comment '資產名稱',
    purchase_date date default null comment '資產購置時間',
    maintenance_state enum('well','broken') default 'well' comment '資產保存狀態',
    maintenance_department smallint unsigned not null comment '資產負責維護的部門',
    remarks text default null comment '資產備註說明',
    property blob default null comment '資產屬性，json格式存儲',
    adder mediumint unsigned not null comment '分類的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    unique key `asset_inventory_name` (`name`,`asset_no`) comment 'index-資產名稱，name和asset_no二者確定一條item',
    constraint `fk_asswt_inventory_mDepartment` foreign key (`maintenance_department`) references `department` (`id`) on update cascade,
    constraint `fk_asset_inventory_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_asset_inventory_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '公司資產清單表';

-- select id,name,asset_no,asset_category,column_list(property) from asset_inventory;
-- select id,name,asset_no,asset_category,column_json(property) from asset_inventory;
-- select id,name,asset_no,asset_category,column_get(property,'內存' as char) as 'CPU' from asset_inventory;
-- update asset_inventory set property=column_add(property, '顯卡型號','column_add') where id=3;
-- update asset_inventory set property=column_delete(property, '顯卡型號') where id=3;

-- 資產使用記錄
create table if not exists asset_usage_log (
    id mediumint unsigned not null auto_increment primary key comment '資產使用記錄自增id',
    asset mediumint unsigned not null comment '使用的資產，表asset_inventory中的id為其外鍵',
    scope enum('個人','集體') default '個人' comment '該資產使用範圍，分配個人使用還是集體使用',
    user mediumint unsigned not null comment '資產使用者，表employee中id為其外鍵',
    assigner mediumint unsigned not null comment '資產分配者，表employee中id為其外鍵',
    remarks text default null comment '資產使用備註說明',
    start_time timestamp not null default current_timestamp comment '資產開始使用時間',
    end_time timestamp null default null comment '資產使用結束或歸還時間，默認為null，表示該資產被分配使用，如果有具體時間，表示該資產閒置',
    adder mediumint unsigned not null comment '該item的添加者,表sys_user中的id為其外鍵',
    create_time timestamp not null default current_timestamp comment 'item入庫時間,格式 YYYY-MM-DD HH:MM:SS',
    update_time timestamp null default null on update current_timestamp comment '數據更新時間,格式 YYYY-MM-DD HH:MM:SS',
    updater mediumint unsigned default null comment 'item更新者，表sys_user中id為其外鍵',
    constraint `fk_asset_usage_log_asset` foreign key (`asset`) references `asset_inventory` (`id`) on update cascade,
    constraint `fk_asset_usage_log_employee` foreign key (`user`) references `employee` (`id`) on update cascade,
    constraint `fk_asset_usage_log_assigner` foreign key (`assigner`) references `employee` (`id`) on update cascade,
    constraint `fk_asset_usage_log_adder` foreign key (`adder`) references `sys_user` (`id`) on update cascade,
    constraint `fk_asset_usage_log_updater` foreign key (`updater`) references `sys_user` (`id`) on update cascade
)engine=innodb default charset=utf8 collate=utf8_general_ci comment '公司資產使用記錄表';
```

## Snapshots
項目快照

### GitLab
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-21-53_gitlab.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-22-13_arsenalPHP.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-22-44_README.png)

### Login Page
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-22-52_login.png)

### Dashboard
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-23-06_dashboard.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-23-15_menu.png)

### Module

#### Sys User
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-23-32_sysuser.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-23-50_sysuser_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-24-08_sysuser_create.png)

#### Module
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-24-18_module.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-24-35_module_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-24-47_module_create.png)

#### User Role
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-25-05_userrole.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-25-22_userrole_detail.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-25-30_userrole_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-25-40_userrole_create.png)

#### Employee
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-26-19_employee_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-26-30_employee_create.png)

#### Department
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-26-48_department.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-27-21_department_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-27-32_department_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-27-46_department_create.png)

#### Asset Category
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-27-52_category_edit.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-28-01_category_create.png)

#### Inventory
![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-28-09_inventory.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-28-29_inventor_edit_1.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-28-32_inventory_edit_2.png)

![](https://raw.githubusercontent.com/MaxdSre/maxdsre.github.io/image/blog-image/2016-11-15_Symfony_Arsenal_Asset_Platform/arsenal2016-11-15_15-28-45_inventory_create.png)


## Change Logs
* 2016.11.15 17:51 Tue Asia/Shanghai
    * 初稿完成
* 2018.04.16 21:33 Mon America/Boston
    * 勘誤，遷移到新Blog


[symfony]: https://symfony.com "High Performance PHP Framework for Web Development"
[php]: https://secure.php.net "PHP is a popular general-purpose scripting language that is especially suited to web development."

<!-- End -->
