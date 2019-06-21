---
title: 解决eclipse下mapstrcuct和lombok冲突问题
date: 2019-06-21 12:55:36
categories: eclipse
tags:
    - eclipse
    - java
    - mapstruct
    - lombok
---
通过合并`lombok-1.18.6.jar`和`mapstruct-processor-1.3.0.Final.jar`解决eclipse下，mapstruct的interface不生成implement的问题。
<!-- more -->
## 问题
lombok和mapstruct都是java下非常好用的辅助类库。但是他们两个在eclipse并不能友好共存。在启用lombok的情况下，mapstrcuct的interface并不能自动生成impletemnt。  
比如在`build.gradle`中配置
``` groovy
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'net.ltgt.apt-eclipse'
dependencies {
    annotationProcessor("org.mapstruct:mapstruct-processor:${mapstructVersion}")
    annotationProcessor("org.projectlombok:lombok:${lombokVersion}")
}
```
执行`gradle eclipse`会自动添加`Java Compiler->Annotation Processing->Facotry Path`
{% asset_img FactoryPath.png factorypath配置图片 %}
但是这时候，你在mapstruct的interface类里按`ctrl+t`发现它并没有实现类。  
## 原因
原因比较复杂，查看这个issue
https://github.com/mapstruct/mapstruct/issues/1159
## 解决
根据以上issue中 https://github.com/mapstruct/mapstruct/issues/1159#issuecomment-486045870 所提供的方法，用以下步骤合并两个类库可以解决这个问题

### 环境
- eclipse 2018-12 (4.10.0)
- lombok 1.18.6
- mapstruct 1.3.0.Final
### 步骤
- 解压缩lombok-1.18.6.jar
- 解压缩mapstruct-processor-1.3.0.Final.jar
- 复制`lombok-1.18.6/org`目录到`mapstruct-processor-1.3.0.Final`目录
- 复制`lombok-1.18.6/META-INF/services/org.mapstruct.ap.internal.processor.ModelElementProcessor`目录到`mapstruct-processor-1.3.0.Final/META-INF/services`目录
- 用文本编辑器打开`lombok-1.18.6/META-INF/services/javax.annotation.processing.Processor`,在最后追加`org.mapstruct.ap.MappingProcessor`
- 把`lombok-1.18.6`目录里的内容打包成`lombok-1.18.6.jar`
- 把`lombok-1.18.6.jar`放到`eclipse`目录下