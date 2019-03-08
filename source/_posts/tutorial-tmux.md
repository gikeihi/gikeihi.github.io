---
title: tmux的简单使用
date: 2019-03-08 10:58:20
categories: Linux
tags:
  - linux
  - shell
  - tmux
---
tmux的简单命令记录。
<!-- more -->
## 会话操作
### shell下操作tmux
- 新建名称为foo的会话，并进入会话
``` bash
tmux new -s foo
```
- 列出所有会话
``` bash
tmux ls
```
- 恢复上一次会话
``` bash
tmux a
```
- 恢复名称为 foo 的会话，会话默认名称为数字
``` bash
tmux a -t foo
```
- 删除名称为 foo 的会话
``` bash
tmux kill-session -t foo
```
- 删除所有的会话
``` bash
tmux kill-server
```
### tmux会话中的操作
按下ctrl+b
- $ 重命名当前会话
- s 选择会话列表
- d detach 当前会话，返回至 shell 主进程  

退出tmux
- ctrl+d
- exit

## 使用例
### 后台运行程序
- 先创建一个会话
``` bash
tmux new -s foo
```
- 运行一个程序
``` bash
java -jar xxx.jar
```
- detach会话  
点击ctrl+b，d
