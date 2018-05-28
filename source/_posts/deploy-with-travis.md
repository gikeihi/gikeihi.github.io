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
