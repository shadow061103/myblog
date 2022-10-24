---
title: "docker的網路"
date: 2022-10-24T15:36:11+08:00
draft: true
tags: ["docker"]
type: post
author: "Steve Lin"
description: ""
---
# docker的網路
### 介紹
- 透過Network Namespace虛擬出網路環境，提供獨立的網路設備、IP協定堆疊、IP路由表、防火牆
- docker啟動時會在宿主機架一個docker0的虛擬網路，連接宿主機跟容器(用Weth pair)
	- 這個是僅限於容器跟宿主機溝通，不對外 

#### 查網路設定
- inspect的NetworkSettings
```
docker inspect [name]
```
### port映射
- 容器可以透過預設的docker0連接到外網，但是外網不能連到容器，因為可能同時好幾個容器在使用docker0
- 建立容器的時候加上-P，會在宿主機隨機找可用的port，再綁定到容器的port上
- -p是指定特定的port 前面是宿主機的port後面是docker的port
`docker run -p <hport>:<cport> `
- -p可以設定多個
- 可以限制特定IP
`-p <ip>:<hport>:<cport>`
### 容器間通訊
- 建立容器時加上--link參數 指定container
`--link [container name]`
- 連接其他容器時不使用IP，而是直接指定容器名稱
`docker run -d -p 80:80 -p 443:443 --name web --link redis-lab nginx`
- 可以用環境變數檢查
```
docker exec -it web /bin/bash
root@e4e93bef390b:/# env
HOSTNAME=e4e93bef390b
PWD=/
REDIS_LAB_PORT_6379_TCP=tcp://172.17.0.3:6379
REDIS_LAB_PORT=tcp://172.17.0.3:6379
PKG_RELEASE=1~bullseye
HOME=/root
REDIS_LAB_ENV_REDIS_VERSION=6.2.6
REDIS_LAB_PORT_6379_TCP_PROTO=tcp
REDIS_LAB_NAME=/web/redis-lab
NJS_VERSION=0.7.0
TERM=xterm
REDIS_LAB_ENV_GOSU_VERSION=1.12
REDIS_LAB_PORT_6379_TCP_ADDR=172.17.0.3
SHLVL=1
REDIS_LAB_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-6.2.6.tar.gz
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
REDIS_LAB_PORT_6379_TCP_PORT=6379
NGINX_VERSION=1.21.4
REDIS_LAB_ENV_REDIS_DOWNLOAD_SHA=5b2b8b7a50111ef395bf1c1d5be11e6e167ac018125055daa8b5c2317ae131ab
_=/usr/bin/env
```