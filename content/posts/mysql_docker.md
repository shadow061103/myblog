---
title: "Docker建立Mysql資料庫"
date: 2022-10-27T17:09:09+08:00
draft: true
tags: ["DB","docker"]
type: post
author: "Steve Lin"
description: ""
---
---

tags: DB
---
# Docker建立Mysql資料庫
### docker建立container
1.docker pull mysql:8
2.docker run --name mysql_test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8
3.可以用mysql workbench登入 127.0.0.1:3306 
### 下指令進入mysql client
1.docker exec -it <container id> bash 進入互動模式
2.`mysql -h localhost -u root -p` 進入client 就可以下指令
    
https://hub.docker.com/_/mysql
    
### 練習
[[面試][資料庫]關聯式資料庫要如何設計避免超賣？](https://ithelp.ithome.com.tw/articles/10271229)