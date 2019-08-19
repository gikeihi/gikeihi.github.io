---
title: docker基本应用
date: 2018-08-25 13:35:53
categories: Docker
tags:
    - docker
---
简单介绍docker的基本使用方法
<!-- more -->
## 为什么要用docker
### 出现问题
应用的开发部署遇到以下问题
- 软件环境安装设置  
环境的配置非常繁琐，不管是开发环境，测试环境，还是部署环境，需要的安装设置的软件有很多，各个软件又有版本的限制，以及对其他软件的依赖。  
这些东西手动安装起来非常麻烦，且易出错。  
- 环境共存  
由于各个软件所依赖的其他软件的版本可能会导致冲突，无法在同一环境下按照。
- 环境复制迁移  
环境的安装已经如此麻烦，从另外一个系统上安装同一环境将变的非常困难。
### 解决方案
- 虚拟机
  - 资源占用多
  - 冗余步骤多
  - 启动慢
- 容器
  - 启动快
  - 资源占用少
  - 体积小

## Docker的安装
参照官网文档，https://docs.docker.com/install/
## 基本使用
### 版本查看
``` bash
docker version
docker-compose version
docker-machine version
```
## 几种常见docker的启动
### 创建网络
```bash
docker network create -d bridge some-network
```
### Mysql
- 启动
```bash
docker run --name=some-mysql --network=some-network --restart=always -d -v "$PWD/mysql/conf":/etc/mysql/conf.d -v "$PWD/mysql/data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:8.0.17
```
- 链接
```bash
docker run --rm --network=some-network -it mysql mysql -hsome-mysql -uexample-user -p
```
- 备份
```bash
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
```
- 恢复
```bash
docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql
```
### Nginx
```bash
docker run --name=some-nginx --network=some-network --restart=always -d -p 80:80 -p 443:443 -v "$PWD/nginx/conf/default.conf":/etc/nginx/conf.d/default.conf nginx:alpine
```
### Mongo
- 启动
```bash
docker run --name=some-mongo --network=some-network --restart=always -d -p 27017:27017 -v /root/workspace/mongo/data:/data/db mongo
```
- 备份恢复
```bash
docker run --rm -v "$PWD/copy.sh":/copy.sh mongo bash copy.sh
```
copy.sh
```bash
mongodump --host 192.168.1.10 --archive=/var/dbname.gz --gzip --db dbname
mongorestore --host 192.168.1.15 --drop --archive=/var/dbname.gz --gzip
```
`192.168.1.10`要备份的mongo所在服务器IP，`192.168.1.15`要恢复的mongo所在服务器IP。
### Java
- 创建一个gradle cache的volume
```bash
docker volume create gradle-cache
```
- 执行一个project的gradle
```bash
docker run --rm \
-v gradle-cache:/home/gradle/.gradle \
-v "$PWD"/master:/home/gradle/master \
-v "$PWD"/somme-project:/home/gradle/somme-project \
-w /home/gradle/somme-project \
gradle:5.5.1-jdk8 \
gradle :somme-project:clean :somme-project:bootJar --console=plain
```
- 启动一个spring boot的project
```bash
docker build -f ./Dockerfile.dev -t category/some-project .
docker stop some-project || true
docker run -p 8082:8082 --rm --name some-project --network saas-nw -d saas/some-project
```
`Dockerfile.dev`
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","-Dspring.profiles.active=prd","/app.jar"]
EXPOSE 8082
```