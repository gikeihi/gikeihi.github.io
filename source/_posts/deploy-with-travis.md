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
使用hexo来部署github pages，需要经过一下几个步骤
- 安装hexo
- 新建blog
- 执行`hexo g`生成静态页面
- 执行`hexo d`发布