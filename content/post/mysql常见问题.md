---
title: mysql常见问题
#slug: chinese-test
date: 2020-11-30
categories:
- mysql
tags:
- mysql
thumbnailImagePosition: left
thumbnailImage: /img/mysql-front.jpeg
coverImage: /img/mysql2.jpeg
---
mysql常见问题
<!--more-->



# problem

## 设置mysql密码

``` mysql
# 跳过密码登录后，用update语句添加密码
mysql> update mysql.user set authentication_string=password('123qwe') where user='root' and Host = 'localhost';
mysql> set password for root@localhost = password('123');
```

## 允许远程访问

``` mysql
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;
```

## 修改默认字符集

查看默认字符集

``` mysql
mysql> show variables like '%character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.05 sec)
```

修改配置文件:
``` conf
[client]
default-character-set=utf8

[mysqld]
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
```

重启mysql

``` mysql
mysql> show variables like '%character_set%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

```

## 大小写敏感

修改配置文件
```
[mysqld]
lower_case_table_names
```

重启mysql

``` mysql
mysql> show variables like '%lower%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_file_system | OFF   |
| lower_case_table_names | 1     |
+------------------------+-------+
2 rows in set (0.01 sec)


```

## java连接mysql配置示例:

``` yaml
spring:
  jpa:
    database: mysql
    generate-ddl: true
    show-sql: false
  datasource:
    username: root
    password: 123123123
    url: jdbc:mysql://localhost:3306/test?useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver

```


## 时区问题

在mysql中查询时间`select now();`为当前时间(北京时间),通过java代码插入数据则差14小时,为mysql数据库服务中时区问题

[参考](https://blog.csdn.net/yjgithub/article/details/80404002)

查看mysql服务时区:

此时区与东八区差14小时
``` mysql
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM|
+------------------+--------+
2 rows in set (0.01 sec)
```

修改为东八区:

``` mysql
mysql> set global time_zone = '+8:00';
Query OK, 0 rows affected (0.00 sec)

mysql> set time_zone = '+8:00';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

查询mysql当前时间: 

``` mysql 
mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2019-05-11 11:44:45 |
+---------------------+
1 row in set (0.00 sec)

```
查看版本
``` mysql
mysql> select version();
+-------------------------+
| version()               |
+-------------------------+
| 5.7.26-0ubuntu0.18.04.1 |
+-------------------------+
1 row in set (0.00 sec)


```

再从java代码插入数据库,时间即为正常时间

| 类 | 格式 |
| --- | --- |
| java.sql.Date | 只包含日期 |
| java.sql.Time | 只包含时间 |
| java.sql.Timestamp | 包含日期和时间 |

new java.sql.Timestamp(new java.util.Date().getTime());

go中出现
unsupported Scan, storing driver.Value type []uint8 into type *time.Time

解决办法:
链接地址后面加上parseTime=true
root:123123123@tcp(127.0.0.1:3306)/graphql?charset=utf8&parseTime=true

mysqld.cnf加上
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

