---
title: 日文键盘下，中文输入法的键值错位问题
date: 2019-06-18 10:12:46
categories: util
tags:
    - keyboard
    - ime
    - chinese input method
    - japanese keyboard
---
通过更改注册表，解决日文键盘下，中文输入法的键值错位问题
<!-- more -->
## 问题
键盘是日文布局时，当切换到中文输入法，发现输入的值跟键盘上的表示值并不对应，仔细确认用的英文键盘的布局。
## 解决思路
既然这样，只要切换输入法后，调用日文键盘的布局就好了。
## 方法
- 打开输入注册表
- 找到相应中文输入法，比如windows自带的中文输入法应该是
```
\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layouts\00000804
```
- 把其中的Layout File，从‘KBDUS.DLL’改为‘KBDJPN.DLL’
- 重启电脑
## 补足
好像这个修改方法在windows 10的1809更新以后失效了。
期待这个问题的解决。
## 临时解决方案
修改的方法还是以上，但不是改成`KBDJPN.DLL`，而是改成`KBD106.DLL’`