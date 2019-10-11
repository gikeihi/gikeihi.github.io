---
title: git常用命令
date: 2019-01-31 09:56:44
categories: git
tags:
    - git
---

git的一些常用命令
<!-- more -->
## 配置
- 查看
  ``` bash
  git config -l #查看所有配置
  git config --global -l #查看全局配置
  git config --local -l #查看本地配置
  ```
- 设置
  ``` bash
  git config --global user.name "John Doe" #设置用户名
  git config --global user.email johndoe@example.com #设置用户信箱
  git config --global credential.helper store #设置权限保存，cache保留15分钟
  git config --global core.autocrlf true # true：提交时CRLF->LF,签出时LF->CRLF；input:提交时CRLF->LF,签出时不转换；false：不转换
  ```
- 本地设置（.gitattributes）
  ```
  # Set the default behavior, in case people don't have core.autocrlf set.
  *.sh		text eol=lf
  ```
## 初始化
- 现有工程的初始化以及远程PUSH
  1. 进入项目目录，初始化本地仓库  
    ``` bash
    git init
    ```
  2. 把所有文件添加到stage
    ``` bash
    git add .
    ```
  3. 提交到head
    ``` bash
    git commit -m "init project"
    ```
  4. 在远程新建仓库  
    例：https://github.com/gikeihi/springboot.git
  5. 添加远程仓库地址
    ``` bash
    git remote add origin https://github.com/gikeihi/springboot.git
    ```
  6. PUSH到远程仓库
    ``` bash
    git push -u origin master​
    ```