---
title: "Docker api"
date: 2022-10-24T15:41:56+08:00
draft: false
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---

# docker api
> 是restful的
- docker remote api:用來操作映像或容器
- docker registry api:管理遠端映像倉庫
- docker cloud api:操作docker雲端服務
### Docker Remote API
```
取得docker運行宿主機環境介面
sudo curl --unix-socket /var/run/docker.sock http://localhost/info
遠端打的話可以用

api用法 前面網址是基底
sudo curl http://host:port/info
列出容器
GET /containers/json
列出inages
GET /images/json
```
### docker registry api
```
取得image資訊
Get /v2/[name]/manifests/[references]
pull image
GET /v2/[name]/blobs/[digest]
推送映像
先判斷registry已經準備好
POSt /v2/[name]/blobs/uploads

```