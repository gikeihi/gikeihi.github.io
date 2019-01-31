---
title: git常用命令
date: 2019-01-31 09:56:44
categories: git
tags:
    - git
---

git的一些常用命令
<!-- more -->
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