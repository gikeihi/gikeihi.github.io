---
title: Hexo发布方案（Travis CI）
date: 2018-05-17 23:16:00
categories: Hexo
tags: 
    - hexo
    - travis
---
使用持续集成服务Travis CI来简化hexo的发布。
<!-- more -->
## 现在的问题
使用hexo来部署github pages。

### 安装配置
- master分支用来储放生成的静态文件
- 创建hexo分支，储放hexo运行环境

### 文章作成
每次的文章作成，都要执行一下步骤
- 新建blog
- 执行`hexo g`生成静态页面
- 执行`hexo d`发布
- 把hexo的执行环境push到服务器。

其中第2，3步的执行步骤非常固定，可以用持续集成服务自动完成。

## Travis CI
通过多方比较，发现Travis CI比较适合。  
首先它有以下特点：
- 可以用Github账户直接登陆
- 有Linux环境
- 对于开源项目免费

### 注册
直接用Github的用户登陆即可。

### 取得GIthub的Personal access tokens
在`Settings`->`Developer settings`->`Personal access tokens`里，点击`Generate new token`，创建新的`token`，权限只要打开一下即可。
{% asset_img TokenSetting.png 权限设置图片 %}

### 配置
打开需要持续集成的Repository，在Setting页面上添加一个环境变量`GITHUB_TOKE`，我上一步取得的`token`复制进去。
{% asset_img TravisSetting.png 集成设置图片 %}

### .travis.yml
分支的根目录下添加`.travis.yml`文件，内容如下
``` yaml
language: node_js

node_js:
   - "8.11.2"

branches:
  only:
    - hexo

cache:
  directories:
    - node_modules

before_install:
  - npm install -g hexo-cli

install:
  - npm install

script:
  - hexo generate

after_success:
  - git config user.name "gikeihi"
  - git config user.email "gikeihi@gmail.com"
  - sed -i'' "s~https://github.com/gikeihi/gikeihi.github.io.git~https://${GITHUB_TOKEN}@github.com/gikeihi/gikeihi.github.io.git~" _config.yml
  - hexo deploy

```