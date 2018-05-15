---
title: 安装MariaDB服务器
date: 2018-05-10 22:38:11
categories: AWS
tags: 
    - aws
    - mariadb
---
在Amazon Linux上安装MariaDB服务器，并创建用户。
<!-- more -->
## 登陆服务器
通过登陆Nat服务器，再登录到DB服务器。
``` bash
$ ssh -i common.pem ec2-user@54.92.57.144
$ ssh -i common.pem ec2-user@10.0.2.16
$ sudo yum update
```
## MariaDB的安装设置
### 安装
``` sbash
$ sudo yum install mariadb mariadb-server
```
### 启动
``` bash
$ sudo systemctl start mariadb
```
### 开机自动运行
``` bash
$ sudo systemctl enable mariadb
```
### 设置root密码
``` bash
$ mysql_secure_installation
Enter current password for root (enter for none):
Set root password? [Y/n] y
New password:xxxxxxxx            #设置密码
Re-enter new password:xxxxxxxx   #再次输入
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y

Thanks for using MariaDB!
```
### 测试登陆
``` bash
$ mysql -uroot -p mysql
Enter password:      #输入上面设置的密码
MariaDB [mysql]>
```
### 创建DB
``` sql
MariaDB [mysql]> CREATE DATABASE `wordpress-db`;
Query OK, 1 row affected (0.00 sec)
```
### 查看DB
``` sql
MariaDB [mysql]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress-db       |
+--------------------+
4 rows in set (0.00 sec)
```
### 创建用户
``` sql
MariaDB [mysql]> CREATE USER 'wordpress-user' IDENTIFIED BY 'wordpress-password';
Query OK, 0 rows affected (0.00 sec)
```
### 查看用户
``` sql
MariaDB [mysql]> SELECT User, Host, Password FROM mysql.user;
+----------------+-----------+-------------------------------------------+
| User           | Host      | Password                                  |
+----------------+-----------+-------------------------------------------+
| root           | localhost | *174E6B8594D3674ED6C9F4EF64E06917012B298C |
| root           | 127.0.0.1 | *174E6B8594D3674ED6C9F4EF64E06917012B298C |
| root           | ::1       | *174E6B8594D3674ED6C9F4EF64E06917012B298C |
| wordpress-user | %         | *9683A3C2776728286D4308EC6BCA661BBEB2DF3E |
+----------------+-----------+-------------------------------------------+
4 rows in set (0.00 sec)
```
### 赋予用户权限
``` sql
MariaDB [mysql]> GRANT ALL PRIVILEGES ON `wordpress-db`.* TO 'wordpress-user';
Query OK, 0 rows affected (0.00 sec)
```
### 查看权限
``` sql
MariaDB [mysql]> show grants for 'wordpress-user';
+----------------------------------------------------------------+
| Grants for wordpress-user@%                                    |+----------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wordpress-user'@'%' IDENTIFIED BY PASSWORD '*9683A3C2776728286D4308EC6BCA661BBEB2DF3E' |
| GRANT ALL PRIVILEGES ON `wordpress-db`.* TO 'wordpress-user'@'%'|
+-----------------------------------------------------------------+
```
### 立刻生效
``` sql
MariaDB [mysql]> FLUSH PRIVILEGES;
```