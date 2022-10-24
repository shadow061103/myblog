---
title: "docker安裝Web server"
date: 2022-10-24T15:37:57+08:00
draft: true
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker安裝Web server
### Apache
#### 下指令
```
 docker run -it --name apache ubuntu:16.04 /bin/bash
 #apt-get update && apt-get -y install apache2 && apt-get clean
 
 //開啟TLS安全加密模組
 #a2ensite default-ssl
 # service apache2 reload
 #a2enmod ssl
 
 docker commit -m 'apache server' apache kuan/apache
 docker run -d -p 1002:80 kuan/apache /usr/sbin/apache2ctl -D FOREGROUND
```
#### Dockerfile
```
FROM ubuntu:16.04
MAINTAINER kuan <shadow061103@gmail.com>
RUN apt-get update && apt-get -y install apache2 && apt-get clean
RUN /usr/sbin/a2ensite default-ssl
RUN /usr/sbin/a2enmod ssl
EXPOSE 80
EXPOSE 443
CMD  ["/usr/sbin/apache2ctl","-D","FOREGROUND"]
```
docker build -t kuan/apache ./apache
docker run -d -P --name apache kuan/apache
打開瀏覽器 localhost:port 應該就會看到Apache了
80port是http 443port是https

### Nginx
#### 下指令
```
docker run -it --name nginx debian:jessie /bin/bash
# apt-get update && apt-get install --no-install-recommends --no-install-suggests -y nginx
# apt-get install -y ca-certificates
```
#### dockerfile
```
FROM debian:jessie
MAINTAINER kuan
RUN apt-get update && apt-get install --no-install-recommends --no-install-suggests -y ca-certificates nginx
EXPOSE 80 443
CMD ["nginx","-g","dameon off;"]
```
docker build -t kuan/nginx ./nginx
docker run -d --name nginx -P kuan/nginx