---
title: 编译JDK源代码
date: 2018-06-27 11:06:52
categories: java
tags:
    - java
    - jdk
---
重新编译JDK源代码，让其包含debug信息。
<!-- more -->
## 问题
当我们对程序进行debug时，有时候会追踪到jdk的源代码里面去。但是这时候会发现在IDE的debug界面上是无法查看类变量和本地变量信息（仅仅能按顺序查看方法参数的信息）。这是由于jdk自带的包`rt.jar`没有包含debug信息，所以需要重新编译源代码，并覆盖原有包`rt.jar`。
## 编译
下面以jdk8为例来编译一下源代码
### 准备
- 解压`${JAVA_HOME}\src.zip`到目录`${workdir}\jdk8_src`，删除不需要编译的package，基本上只留下`java`,`javax`,`org`即可。
- 在`${workdir}`目录下执行一下命令，生产编译对象列表   
``` bat
dir /B /S /X jdk8_src\*.java > filelist.txt
```
- 复制`${JAVA_HOME}\\jre\lib\rt.jar`到目录`${workdir}\rt.jar`
- 新建目录`${workdir}\jdk8_class`
### 编译
在`${wowrkdir}`目录下，执行以下命令编译源代码，生成class
``` bat
javac -J-Xms16m -J-Xmx1024m -sourcepath jdk8_src -cp rt.jar -d jdk8_class -g @filelist.txt >> log.txt 2>&1
```
查看log.txt文件，应该会发现有很多警告，但是没有错误。
### 打包
在目录`${workdir}\jdk8_class`目录下执行以下命令打包。
``` bat
jar cf0 rt_debug.jar *
```
## 导入包
把生产的rt_debug.jar文件复制到`${JAVA_HOME}\jre\lib\endorsed\`目录下，如果没有这个目录新建一下。
## 测试
在IDE（eclipse）里新建一个项目，测试一下jdk源代码是否可以debug。
第一次使用，会发生找不到源代码的情况，只要把`rt_debug.jar`和`${JAVA_HOME}\src.zip`绑定一下即可。