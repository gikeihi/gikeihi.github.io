---
title: 配置VPC对等连接
date: 2018-05-31 14:55:29
categories: AWS
tags:
    - aws
    - vpc peering
---
配置VPC对等连接，让两个VPC之间不公开的路由流量。
<!-- more -->
## 创建对等连接
### 发出请求
在VPC的控制控制台，左侧菜单选择[对等连接]，在右侧页面点击[创建对等连接]，跳到如下页面进行填写。
- 根据通信要求，选择请求方VPC和接收方VPC
- 注意两个VPC的内部CIDR段不可以有重叠

{% asset_img CreateVpcPeering.png 创建对等连接图片 %}
### 接收请求
上面的设置只是发起了连接请求，接收方需要承认请求才可。
- 在以下页面上选中对等连接，在菜单中点击[接收请求]

{% asset_img AcceptVpcPeering.png 接受对等连接图片 %}
## 设置路由
在一个VPC中的EC2向另外一个VPC中EC2发起通信连接，需要把请求路由给对方，所以要配置路由表。
- 这个路由表双方都需要配置

{% asset_img ConfigRouteTableA.png 接受对等连接图片 %}
{% asset_img ConfigRouteTableB.png 接受对等连接图片 %}

## 设置安全组
安全组的设置跟在同一个VPC没什么不同，根据需求，向对方的开放权限即可。