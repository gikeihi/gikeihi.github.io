---
title: 创建wordpress服务
date: 2018-05-11 21:26:58
categories: AWS
tags: 
    - aws
    - apache
    - wordpress
---
Apache，php，wordpress的安装与配置。
<!-- more -->
## 登陆服务器
`13.115.11.35`为Web服务器的IP地址
``` bash
$ ssh -i common.pem ec2-user@13.115.11.35
```
## 更新系统
``` bash
$ sudo yum update
```
## 安装设定Apache
### 安装Aapche
``` bash
$ sudo yum install httpd
```
### 启动Apache
``` bash
$ sudo systemctl start httpd.service
```
### 设置开机启动
``` bash
$ sudo systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```
### 添加用户ec2-user到组apache
由于apache默认是用用户`apache`和组`apache`来运行的。
此处是为了让用户ec2-user能够编辑属于组apache里的文件。
``` bash
$ sudo usermod -a -G apache ec2-user
```
重新登陆服务器
## 安装测试PHP
### 安装
``` bash
$ sudo yum install php php-mysql
```
### 重启apache服务器
本步骤是为了反映apache对php的支持
``` bash
$ sudo systemctl restart httpd.service
```
### 测试
编辑文件
``` bash
sudo vi /var/www/html/info.php
```
输入
``` php
<?php phpinfo(); ?>
```
打开页面，如果显示PHP信息则表示安装成功。
```
http://13.115.11.35/info.php
```
删除测试文件
``` bash
$ sudo rm /var/www/html/info.php
```
## 安装wordpress
### 下载wordpress程序
``` bash
$ wget https://wordpress.org/latest.tar.gz
```
此处若没有`wget`命令的话，使用以下命令安装
``` bash
$ sudo yum install wget
```
### 配置wordpress
解压缩
``` bash
$ tar -xzf latest.tar.gz
```
复制配置文件
``` bash
$ cd wordpress
$ cp wp-config-sample.php wp-config.php
$ vi wp-config.php
```
修改DB信息
``` php
define('DB_NAME', 'wordpress-db');
define('DB_USER', 'wordpress-user');
define('DB_PASSWORD', 'wordpress-password');
define('DB_HOST', '10.0.2.16');
```
在以下网站获取密钥，并覆盖掉`wp-config.php`中的相应位置。
```
https://api.wordpress.org/secret-key/1.1/salt/ 
```
### 复制wordpress
复制wordpress到/var/www/html目录下，修改文件归属，以及访问权限
``` bash
$ sudo mv * /var/www/html
$ sudo chown -R root:apache /var/www
$ find /var/www -type d -exec sudo chmod 2775 {} +
$ find /var/www -type f -exec sudo chmod 0664 {} +
```
### 打开网站并配置wordpress
{% asset_img Installation.png This is an example image %}