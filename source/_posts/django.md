---
title: Django的一些基本使用
date: 2019-11-27 14:41:40
categories: python
tags: 
    - python
    - Django
---

讲述django的一些基本使用
<!-- more -->
# 安装
## Python的安装
## Django的安装
- 使用pip安装
```sh
$ pip install Django
```
- 产看版本
```sh
$ python -m django --version
```
# 创建项目
```sh
$ django-admin startproject mysite
```
# 创建app
```sh
$ python manage.py startapp polls
```
# 运行
```sh
$ python manage.py runserver
```
```sh
$ python manage.py runserver 8080
```
```sh
$ python manage.py runserver 0.0.0.0：8080
```
为了执行最后一个命令，需要在`settings.py`里添加以下内容
```python
ALLOWED_HOSTS = ['*']
```
# 配置log
## 打印所有的SQL  
配置`settings.py`
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```
# 额外的插件
## django-debug-toolbar
- 安装
```sh
$ pip install django-debug-toolbar
```
- 配置`settings.py`
```python
if DEBUG:
    INTERNAL_IPS = [
        '127.0.0.1',
        '192.168.11.20',
    ]
    # django debug toolbar
    INSTALLED_APPS.append('debug_toolbar')
    MIDDLEWARE.append('debug_toolbar.middleware.DebugToolbarMiddleware')
    DEBUG_TOOLBAR_CONFIG = {
        'JQUERY_URL': '//cdn.bootcss.com/jquery/2.1.4/jquery.min.js',
        'SHOW_COLLAPSED': True,
        'SHOW_TOOLBAR_CALLBACK': lambda x: True,
    }
```
## django-extensions
- 安装
```sh
$ pip install django-extensions
```
- 配置`settings.py`
```python
INSTALLED_APPS = (
    ...
    'django_extensions',
)
```
- 用例
```sh
$ python .\manage.py show_urls
```