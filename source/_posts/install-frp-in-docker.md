---
title: install frp in docker
date: 2021-04-09 12:34:51
categories: network
tags: 
    - docker
    - frp
---
使用frp进行内网穿透
<!-- more -->
# 问题
经常会遇到需要https链接的情况，需要穿透到你内网，请求到本机进行调试。
# 方案
## 服务器端启动
- 配置文件Dockerfile
```
FROM amd64/alpine:3.10

LABEL maintainer="gikeihi <gikeihi@gmail.com>"

ENV FRP_VERSION 0.36.0

RUN cd /root \
    &&  wget --no-check-certificate -c https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_linux_amd64.tar.gz \
    &&  tar zxvf frp_${FRP_VERSION}_linux_amd64.tar.gz  \
    &&  cd frp_${FRP_VERSION}_linux_amd64/ \
    &&  cp frps /usr/bin/ \
    &&  mkdir -p /etc/frp \
    &&  cp frps.ini /etc/frp \
    &&  cd /root \
    &&  rm frp_${FRP_VERSION}_linux_amd64.tar.gz \
    &&  rm -rf frp_${FRP_VERSION}_linux_amd64/ 

ENTRYPOINT /usr/bin/frps -c /etc/frp/frps.ini
```
- 制作命令docker.sh
```bash
docker build --rm -f ./Dockerfile -t gikeihi/frp .
```
- 配置frps.ini
```ini
[common]
bind_port = 7000
vhost_http_port = 80
```
- 启动start.sh
```bash
docker run --rm --name frps --network wejoin-nw -p 7000:7000 -d -v "$PWD"/frps.ini:/etc/frp/frps.ini gikeihi/frp
```
- 配置caddy
```
https://[domain] {
    tls xxx@gmail.com
    proxy / frps:80 {
        websocket
        transparent
    }
}
```
## 客户端启动
- 配置frpc.ini
```ini
[common]
# 服务器端IP地址
server_addr = xxx.xxx.xxx.xxx
server_port = 7000

[web]
type = http
local_port = 8080
custom_domains = [domain]

[websocket]
type = tcp
local_port = 8080
custom_domains = [domain]
```
- 启动
```sh
./frpc.exe -c ./frpc.ini
```
